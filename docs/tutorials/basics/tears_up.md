---
article: Changing the tears stat
authors: catinsurance
blurb: Learn the nuances behind changing the tears stat.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - Lua
---

Creating an item that gives a simple tears up or down is more complex than it may appear.

???+ note ":modding-repentogon: Tears Up with REPENTOGON"
	REPENTOGON makes modifying the tears stat much easier! See [this article](../repentogon/adding_stats.md) for more information.

## Explanation
Making a simple item like Sad Onion is actually a bit complicated, because the tears stat as displayed in the FoundHud in Repentance(+) is not the same as what we change internally.

The game uses something called [MaxFireDelay](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#maxfiredelay), or tear delay, which is the minimum amount of game updates (30 game updates per second) before another tear can be fired. You can find more information about it [here, on the wiki](https://bindingofisaacrebirth.wiki.gg/wiki/Tears#Tear_Delay_Calculation).

The in-game display of the tears stat is your **tears per second**, or your fire rate. To get the tears per second from the MaxFireDelay, use the following:
```lua
local function toTearsPerSecond(maxFireDelay)
  return 30 / (maxFireDelay + 1)
end
```

Conversely, you can get the MaxFireDelay from the tears per second using the following:
```lua
local function toMaxFireDelay(tearsPerSecond)
  return (30 / tearsPerSecond) - 1
end
```

## Example
Now we can put it all together. **The following is technically a "fire rate" up, since a "tears up" follows the soft cap of 5 (depending on the character you're playing).**
```lua
local mod = RegisterMod("My Mod", 1)
local HAPPY_ONION = Isaac.GetItemIdByName("Happy Onion")
local HAPPY_ONION_TEARS = 1.1

local function toTearsPerSecond(maxFireDelay)
  return 30 / (maxFireDelay + 1)
end

local function toMaxFireDelay(tearsPerSecond)
  return (30 / tearsPerSecond) - 1
end

function mod:SadOnionStats(player, cacheFlag)
  local count = player:GetCollectibleNum(HAPPY_ONION)
  local tearsPerSecond = toTearsPerSecond(player.MaxFireDelay)
  tearsPerSecond = tearsPerSecond + (count * HAPPY_ONION_TEARS)
  player.MaxFireDelay = toMaxFireDelay(tearsPerSecond)
end

mod:AddCallback(ModCallbacks.MC_EVALUATE_CACHE, mod.SadOnionStats, CacheFlag.CACHE_FIREDELAY)
```