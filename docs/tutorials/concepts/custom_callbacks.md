---
article: Custom callbacks
authors: benevolusgoat
blurb: Learn how to create, trigger, and manage custom callbacks.
comments: true
tags:
    - Tutorial
    - Intermediate
    - Repentance
    - Repentance+
    - Lua
---

## Introduction

Isaac's modding API has a [callback](https://en.wikipedia.org/wiki/Callback_(computer_programming)) system meant for mods. A function can be a assigned to a particular callback ID, which will then called upon later when that callback triggers. For example, the very first callback ID is 1, which under the ModCallbacks enumeration is assigned to [MC_NPC_UPDATE](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_npc_update). Any functions assigned to it will be run for every "NPC" entity in a room, every game update, so 30 frames a second.

This tutorial covers how to create your own custom callbacks using Isaac's own callback system.

## Basic callback setup

Instead of numeric IDs for callbacks, any mod can assign a string instead. This will serve as a unique identifier for your callback and will work right out the gate.

Make sure the string is something unique so that it doesn't conflict with other mods. A good way of preventing conflicts is prefixing it with a name unique to your mod, such as `"EPIPHANY_TEST_CALLBACK"`.

Use [ModRef:AddCallback](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html#addcallback) (or `ModRef:AddPriorityCallback`) to add a function to your callback, then [Isaac.RunCallback](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#runcallback) to run your callback, passing as many arguments as desired.

```Lua
local mod = RegisterMod("TestMod", 1)
local MY_CALLBACK = "TESTMOD_MY_CALLBACK"

function mod:TestCallback(arg1, arg2, arg3)
	print(arg1, arg2, arg3)
end
mod:AddCallback(MY_CALLBACK, mod.TestCallback)

Isaac.RunCallback(MY_CALLBACK, "hello", "world", 1) -- Result will be the console printing "hello world 1"
```

## Returned values

You can return any value when your callback is ran and have it be returned through `Isaac.RunCallback`. Returning a value will stop any remaining callbacks from running.

```Lua
local mod = RegisterMod("TestMod", 1)
local MY_CALLBACK = "TESTMOD_MY_CALLBACK"

mod:AddCallback(MY_CALLBACK, function()
	return "foo"
end)

mod:AddCallback(MY_CALLBACK, function()
	return "bar"
end)

local returnValue = Isaac.RunCallback(MY_CALLBACK)
print(returnValue) --Will always print "foo", as it was added earlier than the one that returns "bar".
```

## Optional argument

[Isaac.RunCallbackWithParam](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#runcallbackwithparam) will only run callbacks with a specified argument. This can be a value of any type.

```Lua
local mod = RegisterMod("TestMod", 1)
local MY_CALLBACK = "TESTMOD_MY_CALLBACK"

function mod:Test1()
	print("test1")
end

mod:AddCallback(MY_CALLBACK, mod.Test1)   --No optional argument specified. Will always run.

function mod:Test2()
	print("test2")
end

mod:AddCallback(MY_CALLBACK, mod.Test2, true)   --Will run if the optional arg is equal to true.

function mod:Test3()
	print("test3")
end

mod:AddCallback(MY_CALLBACK, mod.Test3, EntityType.ENTITY_TEAR)   --Will run if the optional arg is equal to 2, as EntityType.ENTITY_TEAR is also equal to 2.

Isaac.RunCallbackWithParam(MY_CALLBACK) --prints "test1"
Isaac.RunCallbackWithParam(MY_CALLBACK, true) --prints "test1" and "test2"
Isaac.RunCallbackWithParam(MY_CALLBACK, EntityType.ENTITY_TEAR) --prints "test1" and "test3"
```

## Mod compatibility

Custom callbacks are especially advantageous for allowing other mods to interact with your code. The following example has Mod 1 return a custom value if Mod 2 passes Mod 1's custom character.

Mod 1's main.lua:

```Lua
local mod = RegisterMod("TestMod1", 1)
local MY_CHAR = Isaac.GetPlayerTypeByName("My Custom Char")

function mod:ReturnANumber(player)
	return 0
end

--Optional argument set to MY_CHAR, so it will only run if the param passed in Isaac.RunCallbackWithParam matches it.
mod:AddCallback("TESTMOD2_CUSTOM_CALLBACK", mod.ReturnANumber, MY_CHAR)
```

Mod 2's main.lua:

```Lua
local mod = RegisterMod("TestMod2", 1)
local MY_CALLBACK = "TESTMOD2_CUSTOM_CALLBACK"

--Will print a number (if any mod returned as such) every time a player is initialized.
function mod:GetAFunnyNumber()
	--The callback to run, player:GetPlayerType() as the optional argument, and player as the first argument.
	local result = Isaac.RunCallbackWithParam(MY_CALLBACK, player:GetPlayerType(), player)
	if result ~= nil then
		print(result) --Prints 0, which Mod 1 returned if the player was Mod 1's custom character.
	end
end

mod:AddCallback(ModCallbacks.MC_POST_PLAYER_INIT, mod.GetAFunnyNumber)
```

## Custom run behaviour

The default behaviour for all callbacks are breaking after the first value returned, meaning the remaining functions for that callback run will be skipped. To setup different behaviour, you need to handle running the callbacks manually via [Isaac.GetCallbacks](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getcallbacks).

This will return the array the game holds with the information for every callback added with the provided identifier. The returned array contains further tables with the following variables:

```
{
    Mod = <mod reference>,
    Function = function(mod, callback args),
    Priority = integer (default 0),
    Param = entity id / other param (default -1),
}
```

The following example shows a value being passed from one callback to the next:

```Lua
local mod = RegisterMod("TestMod", 1)
local MY_CALLBACK = "TESTMOD2_CUSTOM_CALLBACK"

mod:AddCallback(MY_CALLBACK, function(_, num)
	return num + 2
end)

mod:AddCallback(MY_CALLBACK, function(_, num)
	return num * 2
end)

local number = 1

local callbacks = Isaac.GetCallbacks(MY_CALLBACK)
--Callback order is automatically sorted by priority, so its recommended to use ipairs instead of pairs to loop through the table.
for _, callback in ipairs(callbacks) do
	--Mimic default behaviour by passing the mod reference, then any arguments.
	local result = callback.Function(callback.Mod, number)
	if result ~= nil then
		number = result
	end
end

print(number) --prints "6", as the callbacks did 1 + 2 = 3, then 3 * 2 = 6.
```

### Advanced parameters

Manually handling return behaviour also means manually handling the optional arguments. The following result will print "real gaming" from the second callback due to its optional arguments matching:

```Lua
local mod = RegisterMod("TestMod", 1)
local MY_CALLBACK = "TESTMOD2_CUSTOM_CALLBACK"

mod:AddCallback(MY_CALLBACK, function(_, num)
	print("hello!")
end, {1, 2})

mod:AddCallback(MY_CALLBACK, function(_, num)
	print("real gaming")
end, {2, 3})

local callbacks = Isaac.GetCallbacks(MY_CALLBACK)

for _, callback in ipairs(callbacks) do
	local param = callback.Param
	if type(param) == "table" then
		if param[1] == 2 and param[2] == 3 then
			callback.Function(callback.Mod, number)
		end
	elseif not param then
		callback.Function(callback.Mod, number)
	end
end
```
