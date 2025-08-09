---
article: Creating Item Wisps
authors: Aeronaut
comments: true
tags:
    - Tutorial
    - Intermediate
    - Repentance
    - Repentance+
    - XML
    - Lua
---

**Wisps** are a new type of familiar introduced in the Repentance DLC which can be spawned by the [Book of Virtues](https://bindingofisaacrebirth.wiki.gg/wiki/Book_of_Virtues) item, and nearly every active item has an associated wisp with custom effects reminiscent of the active item. Isaac's modding API has a robust system for adding custom wisps which can be used to create one for your own modded active items.

## Registering the wisp
The first step to creating a custom wisp is to have it registered in a `wisps.xml` file present in your mod's `content` folder. The basic structure of a `wisps.xml` file looks like this:

```xml
<wisps gfxroot="gfx/familiar/wisps/">
    <color name="flame_blue" r="152" g="330" b="458"/>
	<color name="core_blue" r="255" g="356" b="510"/>
	<color name="tear_blue" or="64" og="89" ob="128"/>

    <wisp id="1" hp="2" damage="3" layer="1" flameColor="flame_blue" coreColor="core_blue" tearColor="tear_blue" />
</wisps>
```

There are two main child elements in this file to keep note of, the **color** element and **wisp** element.

### The color element

The color element stores a color for use inside your `wisps.xml` file, with a given **name** and RGB values. The RGB attributes that can be entered are **r**, **g**, **b**, **or**, **og**, **ob**, **cr**, **cg**, **cb**, and **ca**, and each of these attributes accept an integer value. Similar to the [`Color`](https://moddingofisaac.com/docs/rep/Color.html) class in the API, each of these attributes correspond to a member of the `Color` object - **r**, **g**, and **b** control the wisp's tint, **or**, **og**, and **ob** control the wisp's color offset, and **cr**, **cg**, **cb**, and **ca** control the wisp's colorize. However, the color defined in xml, when attached to a wisp, will not affect the wisp's `Color` value in game - in this example, the wisp would have a blue color in-game while having its `Color` be equal to `Color.Default`. It is also worth noting that the values in these two color systems correspond to each other - a wisp color value of 255 corresponds to a `Color` value of 1.

???+ warning "Warning"
    If a wisp attempts to reference a color element that has not been defined in the same file, it will instead default to a teal color. To avoid this from occurring on accident, and for ease of use, I would highly recommend copy-pasting the color elements from the base game's `wisps.xml` file, present in the resources folder (resources-dlc3 for Repentance users and extracted_resources for Repentance+ users).
    ![A wisp with its color undefined](../assets/item_wisps/invalid_wisp_color.png)

### The wisp element

The wisp element is where the appearance and behavior of a wisp is defined, and is how it is linked to your active item. The element has several attributes to control these factors. The most important of these attributes will be listed below, but a complete list of them can be found on the `wisps.xml` page of the [community docs](https://moddingofisaac.com/docs/rep/xml/wisps.html).

| Attribute | Value Type | Description | Example |
| ----------- | ----------- | ----------- | ----------- |
| id | integer | Links the defined wisp to your item. The id value should be the same one the item has defined in its `items.xml` entry. | `id="1"` |
| flameColor | string | The color of the wisp's flame. Uses the name of a color element defined earlier in the file. | `flameColor="flame_blue"` |
| coreColor | string | The color of the wisp's core. Uses the name of a color element defined earlier in the file. | `coreColor="core_blue"` |
| tearColor | string | The color of the tears shot by the wisp. Uses the name of a color element defined earlier in the file. | `tearColor="tear_blue"` |
| coreGfx | string | Changes the appearance of the wisp's core. Uses a .png file that is placed in resources/gfx/familiar/wisps. | `coreGfx="cross.png"` |
| hp | float | Defines the wisp's maximum hitpoints. Each projectile the wisp blocks causes it to lose 1 hp, and taking contact damage causes it to lose 2 hp. Default wisps have an hp value of 2. When deciding on an hp value for your wisp, keep in mind that active items with lower charge tend to have lower amounts of hp, and higher charge or single use actives have higher amounts. | `hp="3"` |
| damage | float | Defines the damage the wisp's tears deal. Default wisps have a damage value of 3. This value does not affect the amount of damage wisps deal on contact with enemies. | `damage="5.5"` |
| layer | integer (-1, 0, 1, or 2) | Defines how close the wisp orbits the player. Default wisps have a layer value of 1. A layer value of 2 will cause it to orbit at a further distance, while a value of 0 will cause it to orbit closer. A layer value of -1 will instead cause the wisp to not follow or orbit the player. **Entering any other positive value will cause the game to crash when the wisp attempts to be spawned.** | `layer="0"` |
| count | integer | Defines the number of wisps that should be spawned when the associated active item is used. If a count attribute isn't present, it will default to 1. | `count="2"` |
| tearFlags | integer | Defines what tear effect(s) the wisp's tears will have. A full list of what number corresponds to what tear effect can be found in the base game's `wisps.xml` file, or referenced off of the [`TearFlags`](https://moddingofisaac.com/docs/rep/enums/TearFlags.html) page on the docs. Multiple numbers can be inputted in this field, with a space separating each effect. | `tearFlags="2 34"` (This wisp will shoot homing, shielded tears.) |
| tearFlags2 | integer | Operates identically to the tearFlags attribute, but the defined tear effect will only have a chance to be shot by the wisp, controlled by the procChance attribute below. | `tearFlags2="4 22"` (This wisp will have a chance to shoot tears that apply poison and burn.) |
| procChance | float | Controls the chance for tear effects defined by the tearFlags2 attribute to be fired. | `procChance="0.25"` (The wisp will have a 25% chance to shoot a tear with tearFlags2's effects.) |

## Advanced wisp effects
While `wisps.xml` allows for many possible wisps that can be created, not everything can be made possible with the file, such as wisps that have an effect when they disappear or ones with special movement behavior. For effects like these, lua code will be required to create these wisps. This section will cover some methods that can be used as a jumping off point for creating wisps with more complex effects.

### The basics
Wisps are defined as familiar entities in the game's code, so any callbacks in the modding API that affect familiars would be useful for running code on wisps. These callbacks also have the optional argument `FamiliarVariant`, allowing the function to only run on familiars that match the provided `FamiliarVariant`. We can set this argument to the enum for the wisp (`FamiliarVariant.WISP`) so the function only affects them. Additionally, in the code, the wisp of a given item has its subtype value equal to the associated item's `CollectibleType`, so as long as your custom item has been defined and stored in a variable, said variable can be used to check if your wisp is the one that spawned from using the right item.

This example code snippet below will add behavior to a custom item's wisp, causing it to spawn a puddle of water every 3 seconds (assuming our custom item is called Isaac's Blueprint):

```lua
local mod = RegisterMod("WispExampleTutorial", 1)
local ISAACS_BLUEPRINT = Isaac.GetItemIdByName("Isaac's Blueprint")

function mod:IsaacsBlueprintWispFamiliarUpdate(familiar)
    -- Early return out of the function if the wisp is not from our item.
    if familiar.SubType ~= ISAACS_BLUEPRINT then return end

    -- This code will activate every 3 seconds the wisp has existed.
    if familiar.FrameCount % 90 == 0 then
        Isaac.Spawn(EntityType.ENTITY_EFFECT, EffectVariant.PLAYER_CREEP_HOLYWATER_TRAIL, 0, familiar.Position, Vector.Zero, nil)
    end
end
mod:AddCallback(ModCallbacks.MC_FAMILIAR_UPDATE, mod.IsaacsBlueprintWispFamiliarUpdate, FamiliarVariant.WISP) -- Note the 3rd parameter here, which causes the function to only affect wisp familiars.
```

Another common effect design for wisps is to have an effect activate when it dies, so here's an extension of the code snippet to give the wisp a death effect:

```lua
local mod = RegisterMod("WispExampleTutorial", 1)
local ISAACS_BLUEPRINT = Isaac.GetItemIdByName("Isaac's Blueprint")

function mod:IsaacsBlueprintWispFamiliarUpdate(familiar)
    -- Early return out of the function if the wisp is not from our item.
    if familiar.SubType ~= ISAACS_BLUEPRINT then return end

    -- This code will activate every 3 seconds the wisp has existed.
    if familiar.FrameCount % 90 == 0 then
        Isaac.Spawn(EntityType.ENTITY_EFFECT, EffectVariant.PLAYER_CREEP_HOLYWATER_TRAIL, 0, familiar.Position, Vector.Zero, nil)
    end

    -- This code will activate when the wisp has died.
    if familiar:HasMortalDamage() then
        Isaac.Spawn(EntityType.ENTITY_PICKUP, PickupVariant.PICKUP_HEART, HeartSubType.HEART_HALF_SOUL, familiar.Position, Vector.Zero, nil)
    end
end
mod:AddCallback(ModCallbacks.MC_FAMILIAR_UPDATE, mod.IsaacsBlueprintWispFamiliarUpdate, FamiliarVariant.WISP)
```

### Items that spawn zero wisps?
Looking through the base game's `wisps.xml`, you may notice some items that have their `count` attribute defined as 0, but are able to spawn wisps in game. What gives? Usually wisps have their `count` defined as 0 if they have special behavior with how they are spawned, like an item spawning a random amount of them on use (ex. Guppy's Head), or an item that doesn't have a wisp of its own, instead spawning wisps from other items (ex. Book of Sin). This behavior can be emulated in lua via a callback registered when an item is used, checking if the player has the Book of Virtues item, and adding the wisp using either [`player:AddWisp()`](https://moddingofisaac.com/docs/rep/EntityPlayer.html?h=addwisp#addwisp) or [`Isaac.Spawn()`](https://moddingofisaac.com/docs/rep/Isaac.html#spawn).

### The purpose of layer = -1
In a similar vein, it may seem odd that some wisps have a `layer` defined as -1, when these wisps have no movement behavior. This is to allow for custom movement behavior for these wisps. Changing the wisp's `Position` and/or `Velocity` in a familiar update callback can be used to code your own movement behavior for the wisp.