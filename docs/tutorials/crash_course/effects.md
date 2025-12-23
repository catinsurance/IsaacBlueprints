---
article: Effects
authors: benevolusgoat
blurb: Learn how to create custom effects.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Video
    - Lua
    - XML
    - Repentance
    - Repentance+
    - REPENTOGON
---

{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

Effects are miscellaneous special effect entities that can be used for arbitrary purposes. The poof of an enemy spawning, the devil and angel statues, the splatter of a tear, and many more are all effect entities. This tutorial will cover how to create one of your own.

## Video tutorial

[![Entities and Effects | Youtube Tutorial](https://img.youtube.com/vi/sxgf2SH6ZGs/0.jpg)](https://youtu.be/sxgf2SH6ZGs "Video tutorial")

## entities2.xml entry

Following the [Entity Basics](entity_basics.md) tutorial, this is already the majority of the work needed to create an effect entity.

The code below is an entities2.xml entry for an effect named "Sad Rain Cloud". Entity effects must have `id` set to `1000`. For `variant`, it must follow the rules mentioned previously under "[What to set for `id`/`variant`/`subtype`](entity_basics.md#what-to-set-for-idvariantsubtype)". Here, it's been given an arbitrary id of `2096`.

```xml
<entities anm2root="gfx/" version="5">
    <entity anm2path="effect_rain_cloud.anm2" id="1000" variant="2096" name="Sad Rain Cloud" />
</entities>
```

Once in-game, you can spawn your effect using the debug console. Type `spawn rain cloud` and, so long as its the first option available, press ENTER and it will spawn in the middle of the room. Alternatively, typing `1000.2096` to spawn it using its ID and variant will also do.

![Debug console](../assets/effects/raincloud_console.png)

![Rain Cloud spawned into the room](../assets/effects/raincloud_spawn.png)

## Lua code

The entity now exists in the game, but it does nothing on its own. This is where the Lua half of the tutorial comes in, where we will have the rain cloud hover above and follow the player.

Start by setting up your `main.lua` with the essentials. Create a function and attach it to the [MC_POST_EFFECT_INIT](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_post_effect_init) callback, which passes the [EntityEffect](https://wofsauge.github.io/IsaacDocs/rep/EntityEffect.html) object being initialized.

```Lua
--Define our mod reference as per usual
local mod = RegisterMod("My Mod", 1)

--The variant of your effect may change if it conflicts with another effect of the same variant.
--Use Isaac.GetEntityVariantByName in order to fetch the variant of the effect.
--There is also Isaac.GetEntityTypeByName, but the type of an effect is constant and will always be 1000.
local RAIN_CLOUD_VARIANT = Isaac.GetEntityVariantByName("Sad Rain Cloud")

--The callback passes one argument: The EntityEffect being initialized.
function mod:OnRainCloudInit(cloud)

end

--This callback will run when the effect is first spawned/initialized.
--MC_POST_EFFECT_INIT also accepts an optional argument to only run for our effect variant, hence inserting RAIN_CLOUD_VARIANT at the end.
mod:AddCallback(ModCallbacks.MC_POST_EFFECT_INIT, mod.OnRainCloudInit, RAIN_CLOUD_VARIANT)
```

The remaining code for our desired action is straightforward:

1. To follow the position of any desired entity, the `EntityEffect` class has a simple function that allows it to automatically: [EntityEffect:FollowParent](https://wofsauge.github.io/IsaacDocs/rep/EntityEffect.html#followparent).
2. The cloud will visually be on the same level as the player instead of above them, so it needs a [SpriteOffset](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#spriteoffset) to be positioned above the player while still following their exact position.

```Lua
local mod = RegisterMod("My Mod", 1)

local RAIN_CLOUD_VARIANT = Isaac.GetEntityVariantByName("Sad Rain Cloud")

function mod:OnRainCloudInit(cloud)
	--The player can be acquired by more efficient means, but for the purposes of this tutorial it will always look to the first player when spawned in.
	local player = Isaac.GetPlayer(0)

	--Cloud is visually positioned 20 pixels higher than normal
	cloud.SpriteOffset = Vector(0, -20)

	--When spawned in, it will immediately go to the player's positioned
	cloud.Position = player.Position
	--Have the effect track the player's position
	cloud:FollowParent(player)
end

mod:AddCallback(ModCallbacks.MC_POST_EFFECT_INIT, mod.OnRainCloudInit, RAIN_CLOUD_VARIANT)
```

The cloud now follows the player. Effects have no collision, so it will not interact with the player. It is also a temporary entity, so it will disappear once you exit the room.
