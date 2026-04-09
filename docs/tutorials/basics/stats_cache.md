---
article: Stat system basics
authors: Monwil
blurb: Learn how to understand and manipulate player stats.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - Lua
---

Many stats in Isaac are handled using a cache system. Knowing how it works at a basic level is essential when implementing custom stat modifiers.

## Introduction to stat cache

When loading into a run, the game calculates the value of each player stat by evaluating every possible modifier (such as passive items, character's starting stats or pill effects), then saves the results in a cache. From then on, the game reuses those cached values whenever possible to avoid additional computation.

When the game detects that a specific stat might've gotten changed (e.g. picking up Sad Onion might change tears stat), it reruns the full calculation for this stat, and updates the cache with the new value.

???- ""Cache" Definition"
    A **cache** in computer science is a temporary storage of data that is intended to be used later to avoid recomputing things that are costly to calculate. This can be something like a complicated math equation that takes a lot of time to complete. In this case, it's to avoid having to check and recalculate every stat given by items that the player has in their inventory.

## Adding custom stat modifiers

The [`MC_EVALUATE_CACHE`](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_evaluate_cache) callback runs right after a stat evaluation happens, providing the relevant player and a [`CacheFlag`](https://wofsauge.github.io/IsaacDocs/rep/enums/CacheFlag.html) variable indicating which stat has updated. Modders can use this to calculate their own stat modifiers after the vanilla ones.

Since previously applied stat changes are reset at the start of each stat evaluation, you should reapply them again each time.

It's important to ensure that a stat is updated only if the `CacheFlag` provided matches the stat in question. Otherwise, it might end up being unintentionally duplicated.

`MC_EVALUATE_CACHE` supports an optional parameter for the `CacheFlag`, meaning that passing a value from the `CacheFlag` enum as the third argument to [`AddCallback`](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html#addcallback) will automatically make the provided function only run for the given cache flag. An alternative to this is to check if the provided `CacheFlag` value is equal to the `CacheFlag` of the stat that's meant to be changed.

???- info "CacheFlag comparison"
    In some mods' code, you might run across `CacheFlag` checks that use a bitwise `&` operator. They look something like this:

    `if cacheFlags & CacheFlag.CACHE_SPEED == CacheFlag.CACHE_SPEED then`

    This is because cache flags are a [bit field](https://en.wikipedia.org/wiki/Bit_field) in order to allow storing multiple cache flags in one variable. Using bitwise operations on them checks if a given bit is contained in the bit field, rather than checking if **only** the given bit is enabled. This allows the condition to pass even if other cache flags were also enabled in the bit field.

    However, the game always invokes the `MC_EVALUATE_CACHE` callback for each flag individually, even if they're evaluated at the same time, which makes direct comparison also work perfectly fine in context of this particular callback.

    The following if-statements are both equally valid ways of checking if the provided `CacheFlag` variable is for `CACHE_SPEED`.

    ```lua
    if cacheFlags & CacheFlag.CACHE_SPEED == CacheFlag.CACHE_SPEED then

    if cacheFlags == CacheFlag.CACHE_SPEED then
    ```

## Triggering cache evaluation
When defining an item in your [items.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/items.html) file, one of the properties that can be set is the item's `cache` value. This property tells the game which caches to re-evaluate, with different cache names separated by a space.

Caches will be re-evaluated whenever an item is obtained or lost (passive items, or active items with `passivecache` set to `true`), or when the item is used (active items).

Attaching a simple stat modifier to a passive item only requires setting the correct caches in items.xml and a Lua function that actually applies the modifiers.

However, this isn't enough for every situation. Imagine an item working similarly to ["Money = Power"](https://bindingofisaacrebirth.wiki.gg/wiki/Money_%3D_Power), which applies a stat modifier dependent on external conditions (the amount of coins the player has). This would require re-evaluating stats not only when the item is picked up, but also whenever the coin count changes.

For such situations, the API allows triggering stat evaluations at any moment using the [AddCacheFlags](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html) and [EvaluateItems](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html) functions of [EntityPlayer](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html). The former specifies stats to be recalculated during the next evaluation, while the latter actually triggers said evaluation.

???- info ":modding-repentogon: Repentogon AddCacheFlags QoL"
    Repentogon adds an optional argument to [AddCacheFlags](https://repentogon.com/EntityPlayer.html#addcacheflags) which will automatically trigger `EvaluateItems` for you.

## Applying stat modifiers outside of the `MC_EVALUATE_CACHE` callback
As mentioned before, stats are reset completely when evaluated. This means that stat changes done outside of the `MC_CACHE_EVALUATE` callback may be overwritten at any time. Any conditional stat change should be done by storing the state of that change (e.g. using [`CollectibleEffects`](https://wofsauge.github.io/IsaacDocs/rep/TemporaryEffects.html) or [`GetData`](../concepts/entity_data.md)) and then triggering a stat evaluation to apply it, with the stat modification itself being hooked into an `MC_EVALUATE_CACHE` callback.

## Advanced stat calculations
Both [Tears](https://bindingofisaacrebirth.wiki.gg/wiki/Tears#Tear_Delay_Calculation) and [Damage](https://bindingofisaacrebirth.wiki.gg/wiki/Damage#Effective_Damage) use a more elaborate, multi-step formula to calculate their values.

It might be desirable to make modded stat-ups be affected by those formulas (such as the 5.00 soft Tears cap), or to apply them before specific vanilla multipliers (such as the ["Soy Milk"](https://bindingofisaacrebirth.wiki.gg/wiki/Soy_Milk) downwards damage multiplier).

Unfortunately, the vanilla API only allows interaction with stats after all vanilla calculations have been applied. This makes full consistency with vanilla impossible without completely recreating the stat system. :modding-repentogon: However, REPENTOGON does make this fully possible! See [this article](../repentogon/adding_stats.md) for more information.

## CacheFlag to Player variable

Each cache flag is typically associated with a specific variable in the `EntityPlayer` class that's expected to be changed within the `MC_EVALUATE_CACHE` callback. Below is a list of cache flags and their associated player variables:

???+ info "CacheFlag to Player variable"
	|:--|:--|
	|CACHE_DAMAGE|[EntityPlayer.Damage](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#damage)|
	|CACHE_FIREDELAY| [EntityPlayer.MaxFireDelay](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#maxfiredelay)|
	|CACHE_SHOTSPEED|[EntityPlayer.ShotSpeed](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#shotspeed)|
	|CACHE_RANGE|[EntityPlayer.TearRange](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#tearrange). Can also adjust [EntityPlayer.TearHeight](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#tearfallingheight) and [EntityPlayer.TearAcceleration](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#tearfallingacceleration) for handling arc shots.|
	|CACHE_SPEED|[EntityPlayer.MoveSpeed](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#movespeed)|
	|CACHE_TEARFLAG|[EntityPlayer.TearFlags](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#tearflags)|
	|CACHE_TEARCOLOR|[EntityPlayer.TearColor](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#tearcolor) and [EntityPlayer.LaserColor](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#lasercolor)|
	|CACHE_FLYING|[EntityPlayer.CanFly](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#canfly)|
	|CACHE_WEAPON|N/A|
	|CACHE_FAMILIARS|[EntityPlayer:CheckFamiliar](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#checkfamiliar)|
	|CACHE_LUCK|[EntityPlayer.Luck](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#luck)|
	|CACHE_SIZE|[Entity.SpriteScale](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#spritescale)|
	|CACHE_COLOR|[Entity.Color](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#color)|

## Example code
The following examples showcase how to work with stat caches in practice. The mod adds a custom passive item, which gives +1 Luck for each bone heart owned, at the cost of losing a bit of speed.

`items.xml`:
```xml
<items gfxroot="gfx/items/" deathanm2="gfx/death_items.anm2" version="1">
    <passive id="1" name="Bone Charm" description="Skeletal Vigor" gfx="placeholder.png" quality="0" tags="summonable nolostbr"
        cache="speed luck" 
	/> <!-- Specifying stats that should be updated on item pickup or drop. -->
</items>
```

`main.lua`:
```lua
--Create a mod table and get the auto-generated ID of the custom item.
local mod = RegisterMod("StatCacheExample", 1)
local BONE_CHARM_ID = Isaac.GetItemIdByName("Bone Charm")

--Putting numbers like these into variables is technically not mandatory,
--but for more complex files it can help with reading and modifying code.
local BONE_CHARM_SPEED_PENALTY = 0.10
local BONE_CHARM_LUCK_PER_BONE_HEART = 1.0

--Function where stat changes are applied.
---@param player EntityPlayer
---@param cacheFlag CacheFlag
function mod:BoneCharmEvaluateCache(player, cacheFlag)
    local itemCount = player:GetCollectibleNum(BONE_CHARM_ID)
    --Since we only want to change stats of players with the item,
    --return from the function early if they don't have it.
    if itemCount == 0 then
        return
    end

    --Ensuring cacheFlag matches the correct CacheFlag before updating a stat.
    if cacheFlag == CacheFlag.CACHE_SPEED then
        local speedToRemove = BONE_CHARM_SPEED_PENALTY * itemCount
        player.MoveSpeed = player.MoveSpeed - speedToRemove
    elseif cacheFlag == CacheFlag.CACHE_LUCK then
        local luckToAdd = BONE_CHARM_LUCK_PER_BONE_HEART * player:GetBoneHearts() * itemCount
        player.Luck = player.Luck + luckToAdd
    end
end

--Attaching the function to a callback.
mod:AddCallback(ModCallbacks.MC_EVALUATE_CACHE, mod.BoneCharmEvaluateCache)

--The only consistent way to detect heart count changes in base API is checking them every single frame,
--due to there not being callbacks for heart count changes.
---@param player EntityPlayer
function mod:PostPeffectUpdate(player)
    --Once again, don't want to run this code without the item owned.
    if not player:HasCollectible(BONE_CHARM_ID) then
        return
    end

    --GetData is used for storing previous bone heart count due to its simplicity.
    local playerData = player:GetData()
    local currentBoneHearts = player:GetBoneHearts()

    --Compare the current bone heart count of the player with one from previous frame.
    --If they are different, trigger cache evaluation, so the luck is updated to match new value.
    --Then save current value for use on the next update.

    --Do note that playerData.BoneCharmPreviousBoneHearts will be nil here at some point due to not being initialised before.
    --In this situation, it won't cause issues because of ~= (not equal to operator) working fine with nil.
    --However, it's a good idea to check if the value isn't nil when performing other operations.
    if playerData.BoneCharmPreviousBoneHearts ~= currentBoneHearts then
        --Only luck scales with bone heart count, so skip refreshing speed here.
        player:AddCacheFlags(CacheFlag.CACHE_LUCK)
        player:EvaluateItems()
    end
    playerData.BoneCharmPreviousBoneHearts = currentBoneHearts
end

--Attaching the function to a callback.
mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.PostPeffectUpdate)
```