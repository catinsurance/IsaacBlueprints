---
article: Trails
authors: catinsurance
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - Lua
    - REPENTOGON
---

Repentance introduced a new effect named `SPRITE_TRAIL (166)`. This effect is used by the eyeballs shout out by [Maze Roamers](https://bindingofisaacrebirth.wiki.gg/wiki/Bony#Maze_Roamer). This trail can be applied to any entity.

## Applying the trail

To apply the trail to an entity, you must spawn the trail then set its parent. You can also set the `MinRadius` property to change how long the base of the trail is, and the `SpriteScale` to change thickness. The following code example creates a long, green trail.

```lua
local entityParent = ... -- Set this to the parent of the trail 
local trail = Isaac.Spawn(EntityType.ENTITY_EFFECT, EffectVariant.SPRITE_TRAIL, 0, entityParent.Position, Vector.Zero, entityParent):ToEffect()
trail:FollowParent(entityParent)
trail.Color = Color(0, 1, 0, 1)
trail.MinRadius = 0.25
trail.SpriteScale = Vector.One
```

???+ warning "Warning"
    Setting the trail's color to black will make it invisible. This is because the trail's sprite is set to blend additively. :modding-repentogon: The blending mode can only be changed with [REPENTOGON](https://repentogon.com/).

    ```lua
    -- REPENTOGON example to change the blending mode.
    local entityParent = ... -- Set this to the parent of the trail
    local trail = Isaac.Spawn(EntityType.ENTITY_EFFECT, EffectVariant.SPRITE_TRAIL, 0, entityParent.Position, Vector.Zero, entityParent):ToEffect()
    trail:FollowParent(entityParent)
    trail.Color = Color(0, 1, 0, 1)
    trail.MinRadius = 0.25
    trail.SpriteScale = Vector.One

    local sprite = trail:GetSprite()
    local blendMode = sprite:GetLayer(0):GetBlendMode()
    blendMode:SetMode(BlendType.NORMAL)
    ```