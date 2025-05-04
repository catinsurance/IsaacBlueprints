---
article: Stat system basics
authors: Monwil
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - Lua
---

{% include-markdown "hidden/unfinished_notice.md" start="<!-- start -->" end="<!-- end -->" %}

Many stats in Isaac are handled using a cache system. Knowing how it works at a basic level is essential when implementing custom stat modifiers.

## Introduction to stat cache

When loading into a run, game calculates the value of each cached player stat by evaluating every possible modifier (such as passive items, character's starting stats or pill effects), then saves the results in a cache. From then on, game reuses those cached values whenever possible to avoid additional computation.

When game detects that a specific stat might've gotten changed (e.g. picking up Sad Onion might change tears stat), it reruns the full calculation for this stat, and updates cache with the new value.

## Adding custom stat modifiers

[MC_EVALUATE_CACHE](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_evaluate_cache) callback runs right after a stat evaluation happens, and it provides the relevant player as well as a [CacheFlag](https://wofsauge.github.io/IsaacDocs/rep/enums/CacheFlag.html) variable indicating which stat is updated. Modders can use this to inject their own stat modifiers on top of the vanilla ones.

Since previously applied stat changes are reset at the start of each stat evaluation, it's intended to reapply them again each time.

It's important to ensure that a stat is updated only if the CacheFlag provided matches the stat in question. Otherwise, it might easily end up being unintentionally duplicated. 

MC_EVALUATE_CACHE callback supports a CacheFlag optional param, which means that passing a value from CacheFlag enum as third argument to AddCallback will automatically make the provided function only run for the given cache flag. An alternative is checking if provided CacheFlag is equal to CacheFlag of stat that's meant to be changed.

??? note "CacheFlag comparison"
    In some mods' code you might run across CacheFlag checks that use a bitwise & operator. They look something like this:

    `if cacheFlags & CacheFlage.CACHE_SPEED == CacheFlage.CACHE_SPEED then`

    This is because cache flags are a [bit field](https://en.wikipedia.org/wiki/Bit_field) in order to allow storing multiple cache flags in one variable. Using bitwise operation as such effectively checks if a given bit is contained in the bit field, rather than checking if **only** the given bit is enabled. Thus allowing the condition to pass, even if other cache flags were also enabled in the bit field.

    However, game always invokes the MC_EVALUATE_CACHE callback for each flag indiviudally, even if they're evaluated at the same time, which makes direct comparison also work perfectly fine in context of this particular callback.

## Triggering cache evaluation
When defining an item in [items.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/items.html) file, one of the values that can be set is the item's `cache` value. Multiple cache values can also be set, separated by a space.

Those are the stats the item is supposed to modify, and will be reevaluated whenever an item is obtained or lost (if it's a passive item or `passivecache` is set to true), or when the item is used (if it's an active item).

Therefore, attaching a simple stat modifier to a passive item requires only setting the correct caches in items.xml and attaching a function to cache evaluation in order to apply the modifiers.

This however, isn't enough for every situation. Imagine an item working similarly to Money=Power, which applies a stat modifier dependent on external conditions (in this example, amount of coins the player has). This would require reevaluating stats not only when the item is picked up, but also whenever the coin count changes.

For such situations, API allows triggering stat evaluations at any moment using [AddCacheFlags](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html) and [EvaluateItems](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html) functions of [EntityPlayer](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html). As names might suggest, first of those functions specifies stats to be recalculated during next evaluation, while the second function actually triggers said evaluation.

??? note ":modding-repentogon: Repentogon AddCacheFlags QoL"
    Repentogon adds an optional argument to [AddCacheFlags](https://repentogon.com/EntityPlayer.html#addcacheflags), which can automatically trigger EvaluateItems right afterwards.

## Applying stat modifiers outside of cache callback
As mentioned before, stats are reset completely when evaluated. This combined with stat evaluation frequency being very inconsistent
(that is, depending on situation they might either not happen for minutes or trigger multiple times every second) makes any changes not synced with them very inconsistent as well. Any conditional stat change should be done by storing state of that change one way or another (e.g. using CollectibleEffects or GetData) and then triggering a stat evaluation to apply it, with the stat modification itself being hooked to MC_EVALUATE_CACHE callback.

## Advanced stat calculations
Both [Tears](https://bindingofisaacrebirth.wiki.gg/wiki/Tears#Tear_Delay_Calculation) and [Damage](https://bindingofisaacrebirth.wiki.gg/wiki/Damage#Effective_Damage) use a more elaborate multi-step formula to calculate their values.

It might be desirable to make modded stat ups be affected by those formulas (such as the soft Tears cap) or be apllied before specific vanilla multipliers (such as Soy Milk's damage decrease).

Unfortunately, API only allows interaction with the final stat values, thus consistency with vanilla is not possible without recreating the stat system.


## Example code
The following showcases how to work with stat cache in practice. The mod adds a custom passive item, which gives +1 luck for each bone heart owned at the cost of losing a bit of speed.

items.xml:
```xml
<items gfxroot="gfx/items/" deathanm2="gfx/death_items.anm2" version="1">
    <passive id="1" name="Bone Charm" description="Skeletal Vigor" gfx="placeholder.png" quality="0" tags="summonable nolostbr"
        cache="speed luck" 
	/> <!-- Specifying stats we want updated on item pickup or drop. -->
</items>
```

main.lua:
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
    --Once again, we don't want to run this code without the item owned.
    if not player:HasCollectible(BONE_CHARM_ID) then
        return
    end

    --We'll be using GetData for storing previous bone heart count due to its simplicity.
    local playerData = player:GetData()
    local currentBoneHearts = player:GetBoneHearts()

    --We compare current bone heart count of the player with one from previous frame.
    --If they are different, trigger cache evaluation, so the luck is updated to match new value.
    --Then save current value for use on the next update.

    --Do note that playerData.BoneCharmPreviousBoneHearts will certainly be nil here at some point due to not being initialised before.
    --In this situation, it won't cause issues because of ~= working fine with nil.
    --However, it's a good idea to check if the value isn't nil when performing other operations.
    if playerData.BoneCharmPreviousBoneHearts ~= currentBoneHearts then
        --Only luck scales with bone heart count, so we can skip refreshing speed here.
        player:AddCacheFlags(CacheFlag.CACHE_LUCK)
        player:EvaluateItems()
    end
    playerData.BoneCharmPreviousBoneHearts = currentBoneHearts
end

--Attaching the function to a callback.
mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.PostPeffectUpdate)
```