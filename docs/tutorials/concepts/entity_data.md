---
article: Entity data
authors: Lizzie
comments: true
tags:
    - Tutorial
    - Intermediate
    - Repentance
    - Repentance+
    - Lua
---

You may have read the guide on [saving and storing data](./saving_data.md), which covers how to export data to a save file and have it persist between runs. This tutorial covers a similar concept that can be combined with the knowledge of saving data: creating data associated with objects.

## ``GetData`` and its drawbacks

Isaac provides a simple way of storing data within entities: a method known as [GetData](https://wofsauge.github.io/IsaacDocs/rep/Entity.html?h=getdata#getdata). Utilizing ``GetData`` is quite simple, as it can be referenced from any kind of ``Entity``. ``GetData`` simply returns a table, for the convenience of the user, which can be used to store any amount of arbitrary data and access it later.

Although ``GetData`` sounds convenient, it has several drawbacks which make it bad practice to depend on for modders. [The docs](https://wofsauge.github.io/IsaacDocs/rep) [provide some reasoning for this](https://wofsauge.github.io/IsaacDocs/rep/Entity.html?h=getdata#getdata), citing that the data table is shared, volatile, and will require its own saving and storing regardless of it being a feature of the API, but these can seem like upsides depending on the use case. Here are a few additional reasons why ``GetData`` is not ideal, and should most likely be avoided:

- ``GetData`` creates a *small* amount of overhead when called. Because it is calling the C++ side, it disrupts the flow of code. Storing the tables on the Lua side would allow for better flow and management of code. Additionally, this overhead can stack up if the method is called consistently. If you still wish to use ``GetData``, avoid calling it multiple times within the same function, and structure your code to be able to pass it to lower level functions instead of calling the method again.
- ``GetData`` does some amount of memory allocation on the C++ side, meaning its impact on performance can range depending on the speed and state of the user's memory.

If you are unbothered by these reasons, or performance is not a concern for you, here are a few more reasons why ``GetData`` might not be ideal:

- ``GetData`` does not account for items such as Glowing Hourglass. It may be hard to store a backup of data, as it must either be done in a sub-table or already stored locally.
- ``GetData`` is erased from an entity on ``MC_POST_ENTITY_REMOVE``, which runs before ``MC_POST_NPC_DEATH ``. 

Depending on the use case, ``GetData`` may still fit the needs of the modder, providing the convenience of being able to store smaller and simpler data that doesn't necessarily need to be scalable or need a complicated system. In cases where data needs to be scalable, or there are issues of performance, it may be worth writing a simple system to store data locally on the Lua side.

## Entity data holders

Conversely, writing a entity data holder has many upsides:

- Because the system is in the user's control, the user can control when the data is cleared.
- It can handle specific and custom behavior, such as behavior for instantiation, or whenever the data is cleared.
- Any issues or edge cases can be handled by the system through custom behavior.

While writing a entity data holder is fairly straightforward, there are still a few considerations:

- The user must find a way to store the data in relation to specific entities or objects. ``GetPtrHash`` is a good starting point, but has several issues: Because ``GetPtrHash`` returns a hash based on a pointer, when that memory is eventually cleared, the pointer will be reused. This means that if data is not cleared ahead of time, a new entity or object may reference the data of an old object by accident.
- The data must be cleared manually, otherwise it will continue to stack up and cause a memory leak. This is also important for avoiding the previous issue. 

In cases where writing a data holder might not be worth the time and effort, or seems undesirable, it may be worth sticking to ``GetData``. ``GetData`` provides undeniable convenience and value to simple and smaller mods, which may not need to be able to store their own data.

## Writing a data holder

To get started, simply create a new Lua file:
```lua
local dataHolder = {}

-- We store the data within the data holder as opposed to in it so it's easier to access
dataHolder.Data = {}

---Returns an entity's data
---@return table entityData
function dataHolder:GetEntityData(entity)
    -- We obtain the entity's pointer hash to easily reference it in a table
    local ptrHash = GetPtrHash(entity)
    -- Here we instantiate the data if it doesn't exist
    if not dataHolder.Data[ptrHash] then
        dataHolder.Data[ptrHash] = {}
        local entityData = dataHolder.Data[ptrHash]

        --[[
            We provide a default value of the entity's pointer to the data.
            The pointer helps us later be able to check if the entity exists

            For looping purposes, it may be inconvenient to store it here,
            if this is the case, making another table to store the pointer
            may be preferable. 
        --]] 
        entityData.Pointer = EntityPtr(entity)

        -- You may also add additional initialization steps here
        -- This may include defining default variables for your data:

        -- entityData.customValue = false
    end

    -- Finally, we return the data itself
    return dataHolder.Data[ptrHash]
end
return dataHolder
```

As it is just a holder, you may ``require`` this Lua file so that the data is persistent across the project. Using ``include`` would make the data local to the file you are including it in.

## Clearing data in your holder

Lastly, we will cover removing and clearing custom data. Additional callbacks should be registered to clear the information after specified events. Depending on the use case, this can range from clearing data when entities are removed directly in ``MC_POST_ENTITY_REMOVE``, clearing data when the entity is fully dead in ``MC_POST_NPC_DEATH``, or clearing entity data at the end of the room with ``MC_POST_NEW_ROOM``.

Below is an example of how to clear custom data on ``MC_POST_NEW_ROOM``:

```lua
-- Provide your data holder to be able to clear it
local dataHolder = require("data_holder")

-- For clearing custom data on MC_POST_NEW_ROOM
function MOD_NAME:ClearCustomData()
    -- Loop through the table of stored data
    for ptrHash, entityData in pairs(dataHolder.Data) do
        -- Check if the entity's pointer exists
        local entityPointer = (entityData and entityData.Pointer)
        -- If the pointer doesn't exist, or the entity's reference doesn't exist
        if not (entityPointer and entityPointer.Ref) then
            -- We know the entity doesn't exist anymore, therefore remove its data
            entityData.Data[ptrHash] = nil
        end
    end
end
MOD_NAME:AddCallback(ModCallbacks.MC_POST_NEW_ROOM, MOD_NAME.ClearCustomData)
```

Similarly, the following code clears entity data on removal or death:

```lua
-- Provide your data holder to be able to clear it
local dataHolder = require("data_holder")

function MOD_NAME:ClearDataOnRemoveOrDeath(entity)
    -- Get the pointer hash of the current entity
    local ptrHash = GetPtrHash(entity)
    -- Set the data to nil. This works even if it had no data prior
    dataHolder.Data[ptrHash] = nil
end
MOD_NAME:AddCallback(ModCallbacks.MC_POST_ENTITY_REMOVE, MOD_NAME.ClearDataOnRemoveOrDeath)
-- Alternatively clear it when the NPC_DEATH callback is run
-- MOD_NAME:AddCallback(ModCallbacks.MC_POST_NPC_DEATH, MOD_NAME.ClearDataOnRemoveOrDeath)
```

## Further resources
- [IsaacSaveManager](https://github.com/catinsurance/IsaacSaveManager) provides many utilities for creating associated data without the use of ``GetData``. Data Storage is also much more involved, covering many edge cases like Glowing Hourglass backup saves and reroll dependent and independent save information.