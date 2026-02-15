---
article: Saving data
authors: catinsurance
blurb: Learn how to save and load save data.
comments: true
tags:
    - Tutorial
    - Intermediate
    - Repentance
    - Repentance+
    - Lua
---

Storing and loading data is surprisingly simple, but how you structure your mod's save data leaves a lot to consider.

## Saving and loading
Let's start with the most basic concept, which is saving and loading.

Save data for mods are stored in the `data` folder in your game's directory. This is so that mod save data is not deleted when your mod is uninstalled. The save data for your mod is located within a folder in `data` that is named after your mod's folder name.

<p align="center">
  <img src="../../assets/saving_data/data_folder.png" alt="Data folder with Sheriff as the example mod" />
</p>

Notice that there are 3 files in here. **Each file in here corresponds to a save file in game.** So `save1.dat` corresponds to the left-most save file, `save2.dat` is the middle save file, and `save3.dat` is the right-most one. Save data is stored in plain text, so you can open these up in a text editor if you'd like.

**Note that saving and loading data is relative to a save file.** As you'll see later on, the methods that you use to save and load data can only be called within a run, and will save and load from the file corresponding to the save file you're playing on. This is for security reasons, as mods being able to create arbitrary files on the user's computer would be very dangerous.

There are 4 functions that are a part of your [Mod Reference](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html) that you need to know about:

1. [`HasData()`](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html#hasdata) - Returns a `boolean` that signifies if your mod has any save data.
2. [`LoadData()`](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html#loaddata) - Returns the `string` stored in the `saveX.dat` file.
3. [`SaveData(string data)`](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html#savedata) - Saves an arbitrary `string` to the `saveX.dat` file. Keep in mind that the file this saves to is decided by the file the run is being played on.
4. [`RemoveData()`](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html#removedata) - Deletes the `saveX.dat` file if it exists.

The actual act of saving a string to a file is very straightforward.

```lua
local mod = RegisterMod("My mod", 1)

mod:AddCallback(ModCallbacks.MC_POST_GAME_STARTED, function()
    if mod:HasData() then
        local loadedData = mod:LoadData()
        print(loadedData) -- "Test!"
    else
        mod:SaveData("Test!")
    end
end)
```

In the above example, we check to see if our mod has save data when a run is started. If they do, we load that save data (stored as a `string`) and print it to the debug console. Otherwise, we create new save data with the value `"Test!"`.

## Structuring save data with JSON
We know how to save data, but how do we structure it? **Isaac has a built-in JSON library that we can use to convert our Lua tables into JSON that we can save.** We can store our mod's data in a table, then store it using the JSON library. There are caveats to this that will be covered later on, but observe the following example to see the most basic implementation of this:

```lua
local mod = RegisterMod("My mod", 1)
local json = require("json") -- This requires the built-in JSON library.

mod:AddCallback(ModCallbacks.MC_POST_GAME_STARTED, function()
    if mod:HasData() then
        local loadedData = mod:LoadData()
        local saveData = json.decode(loadedData)

        print(saveData.Bar) -- Hello, world!
    else
        local dataToSave = {
            Foo = 1,
            Bar = "Hello, world!"
        }
        local encoded = json.encode(dataToSave)
        mod:SaveData(encoded)
    end
end)
```

In the above example, we check to see if our mod has save data when a run has started. If they do, we **deserialize** the JSON string that is our save data. To "deserialize" is to convert data from a "serialized" format (such as a `string`) back into a data structure we can use (in this case, a table). `json.decode` takes a string and returns the decoded data, which in our case is a table with a few values in it.

If we don't have save data for our mod, we'll create a table and **serialize** it. To **serialize** is to convert a data structure like our table here into a "serialized" format, such as a `string`. This table can have anything in it, ***but there are rules we must follow as to not break our save data.***

## Important caveats with the JSON library
There are many things to look out for when it comes to the built-in JSON library that Isaac provides. Below is a comprehensive list of all issues that you may run into with encoding and decoding data with the JSON library.

### Sparse arrays
For starters, let's look at how it handles **sparse arrays**. A sparse array is an array (an ordered table where all the keys are integers, starting from 1) with **gaps** in it. The JSON encoder will fill in the gaps with `"null"` until there are no gaps left, even if this requires inserting millions of values.
```lua
local normalArray = {"Apple", "Orange", "Banana", "Pear"}

local alsoNormalArray = {
    [1] = "Apple",
    [2] = "Orange",
    [3] = "Banana",
    [4] = "Pear"
}

-- This will create 997 "null" entries in your array when saved to file.
-- This can bloat file sizes, and is generally unexpected behavior!
local sparseArray = {
    [1] = "Apple",
    [2] = "Orange",
    [1000] = "Banana",
    [1001] = "Pear"
}
```

### Invalid table key type
Tables can only have either strings or integers as keys. You should not attempt to encode tables with functions, other tables, decimals (floats), or `userdata`s (Isaac objects such as `Vector`s) as keys to values. Additionally, the JSON encoder will attempt to convert any decimal (float) keys in a given table to strings, which will lead to unexpected behavior when trying to index the later decoded data with that same decimal.
```lua
-- This won't work!
local badData = {
    [Vector(0, 1)] = "Apple"
}

-- No decimals!
-- Typically decimals are fine, but they convert all number keys to strings in the table.
-- So if you encode this table then decode it, indexing with the number 0.1 will return nil, while indexing with the string "0.1" will return "Apple".
-- This is typically unexpected behavior.
local alsoBadData = {
    [0.1] = "Apple",
    [1.1] = "Orange",
}

-- You get the idea!
local superBadData = {
    [badData] = "Apple",
}
```

### Mixed table key types
Tables can only have either strings or integers as keys. **Mixing the two would cause all integers to be converted to strings, which is unexpected as it's quite literally not what you saved.**
```lua
-- There's string and integer keys here!
local badTable = {
    Apple = "Banana",
    [2] = "Orange"
}

-- Same as before, just a different way of doing it.
local alsoBadTable = {
    ["Apple"] = "Banana",
    [2] = "Orange"
}

-- No decimals!
-- Typically decimals are fine, but they convert all number keys to strings in the table.
-- So if you encode this table then decode it, indexing with the number 0.1 will return nil, while indexing with the string "0.1" will return "Apple".
-- This is typically unexpected behavior.
local superBadTable = {
    [1] = "Apple",
    [1.1] = "Banana"
}
```

### Invalid numbers
Infinite numbers or NaN numbers are unable to be read by the JSON decoder, causing data to become **broken**.
```lua
local badTable = {
    Apple = 1/0 -- inf
    Banana = -1/0 -- -inf
    Orange = 0/0 -- NaN
}
```

### Improper value types
Tables containing anything other than strings, numbers, booleans, or other tables cannot be encoded by the provided JSON library.
```lua
local badTable = {
    Apple = 1,
    Banana = function() -- Prevents this table from being encoded.
        return 2
    end
}
```

### Circular references
Tables containing a reference to itself or a table higher up in its hierarchy will cause stack overflows.
```lua
local badTable = {
    Apple = badTable
}
```

## Further resources
- [IsaacSaveManager](https://github.com/catinsurance/IsaacSaveManager) is a library that will handle the complicated aspect of saving data for you, as well as give numerous extra utilities for saving data pertaining to the room, run, and floor.