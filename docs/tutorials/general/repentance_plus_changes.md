---
article: Modding changes in Repentance+
authors: benevolus
blurb: Learn all the new additions and changes to the modding api in Repentance+
comments: true
tags:
    - Intermediate
    - Repentance+
    - Lua
---

With the addition of the Repentance+ DLC, there have been a few updates to the modding API compared to Repentance. This article will list all of the additions and changes that have been made in the process.

## [Entity](https://wofsauge.github.io/IsaacDocs/rep/Entity.html)

The following functions have had an additional argument `IgnoreBosses` added at the end of the arguments, which will ignore the boss status effect cooldown that normally prevents bosses from gaining more status effects:

- [Entity:AddBurn](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#addburn)
- [Entity:AddCharmed](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#addcharmed)
- [Entity:AddFear](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#addfear)
- [Entity:AddFreeze](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#addfreeze)
- [Entity:AddMidasFreeze](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#addmidasfreeze)
- [Entity:AddPoison](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#addpoison)
- [Entity:AddShrink](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#addshrink)
- [Entity:AddSlowing](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#addslowing)

There is one new addition to the Entity class:

- [Entity:KillWithSource](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#killwithsource)

## [EntityPlayer](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html)

- [EntityPlayer:AddCollectible](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#addcollectible) has an additional argument: `ItemPoolType`. This allows you to manually define what item pool the item came from

## [Font](https://wofsauge.github.io/IsaacDocs/rep/Font.html)

- A new override to the [Font:DrawString](https://wofsauge.github.io/IsaacDocs/rep/Font.html#drawstring) function has been added that allows you to pass different sizes similarly to [Font:DrawStringScaled](https://wofsauge.github.io/IsaacDocs/rep/Font.html#drawstringscaled) as well as a brand new game object named [FontRenderSettings](https://wofsauge.github.io/IsaacDocs/rep/FontRenderSettings.html), allowing more precise control over how the font is rendered.

(TODO: Maybe insert some cool looking thing as an example of the new capabilities?? I am simply too lazy at the moment!!!)

## [Game](https://wofsauge.github.io/IsaacDocs/rep/Game.html)

- [Game:Fadein](https://wofsauge.github.io/IsaacDocs/rep/Game.html#fadein) has two new arguments: `ShowIcon` and `KColor`. `ShowIcon` appears to be non-functional. `KColor` will change the color of the screen as it fades back into the view of the game.
- [Game:Fadeout](https://wofsauge.github.io/IsaacDocs/rep/Game.html#fadeout) has one new argument: `KColor`. It will change the color of the screen that it will fade out into.

## [GridEntity](https://wofsauge.github.io/IsaacDocs/rep/GridEntity.html)

- [GridEntity:DestroyWithSource](https://wofsauge.github.io/IsaacDocs/rep/GridEntity.html#destroywithsource) is identical to [GridEntity:Destroy](https://wofsauge.github.io/IsaacDocs/rep/GridEntity.html#destroy), but you can define an [EntityRef](https://wofsauge.github.io/IsaacDocs/rep/EntityRef.html) as a source.
- [GridEntity:HurtWithSource](https://wofsauge.github.io/IsaacDocs/rep/GridEntity.html#hurtwithsource) is identical to [GridEntity:Hurt](https://wofsauge.github.io/IsaacDocs/rep/GridEntity.html#hurt), but you can define an [EntityRef](https://wofsauge.github.io/IsaacDocs/rep/EntityRef.html) as a source.

## HUD

A new argument has been added to both overrides of [HUD:ShowItemText](https://wofsauge.github.io/IsaacDocs/rep/HUD.html?h=hud#showitemtext): `ClearStack`. In Repentance+, HUD item text now stacks, showing one HUD text below the last one if its still on screen. The argument is `true` by default, which will resort to the behaviour of Repentance HUD text of removing all existing HUD text on screen before displaying the new one.

Example:

Repeating the following function 3 times will only show it once:

```Lua
Game():GetHUD():ShowItemText("foo", "bar", false)
```

![ShowItemText ClearStack left empty or true](../assets/repentance_plus_changes/hudtext_clearstack_true.png)

Doing the same thing again, but passing `ClearStack` as `false`, will show all three messages:

```Lua
Game():GetHUD():ShowItemText("foo", "bar", false, false)
```

![ShowItemText ClearStack set to false](../assets/repentance_plus_changes/hudtext_clearstack_false.png)

## [ItemPool](https://wofsauge.github.io/IsaacDocs/rep/ItemPool.html)

[ItemPool:GetCollectible](https://wofsauge.github.io/IsaacDocs/rep/ItemPool.html#getcollectible) has an additional argument: `BackupPoolType`. Accepts an [ItemPoolType] such that if the regular pool in `PoolType` is empty and `DefaultItem` is set to `CollectibleType.COLLECTIBLE_NULL`, it will draw from `BackupPoolType` instead of `ItemPoolType.POOL_TREASURE`.

Example:

After draining the library pool, this would normally return a treasure room item:

```Lua
Game():GetItemPool():GetCollectible(ItemPoolType.POOL_LIBRARY, true)
```

However, defining `BackupPoolType` with `ItemPoolType.POOL_DEVIL` will return an item from the devil item pool instead:

```Lua
Game():GetItemPool():GetCollectible(ItemPoolType.POOL_LIBRARY, true, nil, nil, ItemPoolType.POOL_DEVIL)
```

(NOTE: Doesn't appear to work in REPENTOGON? Wonder if its a latest vanilla version addition.)

## [Room](https://wofsauge.github.io/IsaacDocs/rep/Room.html)

- [Room:DamageGridWithSource](https://wofsauge.github.io/IsaacDocs/rep/Room.html#damagegridwithsource) is identical to [Room:DamageGrid](https://wofsauge.github.io/IsaacDocs/rep/Room.html#damagegrid), but you can define an [EntityRef](https://wofsauge.github.io/IsaacDocs/rep/EntityRef.html) as a source.
- [Room:DestroyGridWithSource](https://wofsauge.github.io/IsaacDocs/rep/Room.html#destroygridwithsource) is identical to [Room:DestroyGrid](https://wofsauge.github.io/IsaacDocs/rep/Room.html#destroygrid), but you can define an [EntityRef](https://wofsauge.github.io/IsaacDocs/rep/EntityRef.html) as a source.
- [Room:MamaMegaExplosion](https://wofsauge.github.io/IsaacDocs/rep/Room.html#mamamegaexplosion)'s `Position` argument is now optional, placed at `Vector.Zero` by default. It also has an additional optional argument to pass an [EntityPlayer](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html) as the source of the explosion.

## [Sprite](https://wofsauge.github.io/IsaacDocs/rep/Sprite.html)

- [Sprite:ReplaceSpritesheet](https://wofsauge.github.io/IsaacDocs/rep/Sprite.html#replacespritesheet) now returns a `boolean` instead of `nil`. It will return `true` if the spritesheet at the given layer id was successfully replaced *and* if the new spritesheet is not the same as the old one, otherwise returns `false`.

## [Options](https://wofsauge.github.io/IsaacDocs/rep/Options.html)

- One new variable: [Options.JacobEsauControls](https://wofsauge.github.io/IsaacDocs/rep/Options.html#jacobesaucontrols). `0` for "Classic" controls, `1` for "Better" controls.

## Enums

The following new enums have been added:

- [DrawStringAlignment](https://wofsauge.github.io/IsaacDocs/rep/enums/DrawStringAlignment.html): For use in `FontRenderSettings`

The following enums have been updated with new entries:

- [BackdropType](https://wofsauge.github.io/IsaacDocs/rep/enums/BackdropType.html): New additions are `DEATHMATCH` and `LIL_PORTAL`. `NUM_BACKDROPS` has been updated to reflect this.
- [ButtonAction](https://wofsauge.github.io/IsaacDocs/rep/enums/ButtonAction.html): Many pre-existing enumerations have been switched around with new values. New additions are `ACTION_JOINMULTIPLAYER`, `ACTION_MENUX`, and `ACTION_EMOTES`.
- [EffectVariant](https://wofsauge.github.io/IsaacDocs/rep/enums/EffectVariant.html): `BULLET_POOF_STATIC` and `UMBILICAL_CORD_HELPER` were added, but were existing Repentance entities previously without enumerations. New additions are `MEGA_BEAN_EXPLOSION`, `SPAWN_PENTAGRAM`, and `PLAYER_CREEP_YELLOW`.
- [GameStateFlag](https://wofsauge.github.io/IsaacDocs/rep/enums/GameStateFlag.html): New additions are `STATE_MEGA_SATAN_DOOR_OPENED`, `STATE_URIEL_KILLED`, `STATE_GABRIEL_KILLED`, and `STATE_MOTHER_HEART_DOOR_OPENED`. `NUM_STATE_FLAGS` has been updated to reflect this.
- [GridRooms](https://wofsauge.github.io/IsaacDocs/rep/enums/GridRooms.html): New additions are `ROOM_DEATHMATCH_IDX` and `ROOM_LIL_PORTAL_IDX`. `NUM_OFF_GRID_ROOMS` has been updated to reflect this.
- [Music](https://wofsauge.github.io/IsaacDocs/rep/enums/Music.html): The one new addition is `MUSIC_DEATHMATCH`. `NUM_MUSIC` has been updated to reflect this.
- [RoomType](https://wofsauge.github.io/IsaacDocs/rep/enums/RoomType.html): The one new addition is `ROOM_DEATHMATCH`. `NUM_ROOMTYPES` has been updated to reflect this.
- [SoundEffect](https://wofsauge.github.io/IsaacDocs/rep/enums/SoundEffect.html): Has a large quantity of new sound effects.
