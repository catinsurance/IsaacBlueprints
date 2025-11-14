---
article: Entity basics
authors: benevolusgoat, catinsurance
blurb: Learn how to create custom entities.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Video
    - No Lua
    - XML
    - Repentance
    - Repentance+
    - REPENTOGON
---

{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

Entities make up the majority of anything that moves inside The Binding of Isaac, from tears to enemies to Isaac himself. This tutorial will cover the basics on creating a custom entity.

## Video tutorial
[![Entities and Effects | Youtube Tutorial](https://img.youtube.com/vi/sxgf2SH6ZGs/0.jpg)](https://youtu.be/sxgf2SH6ZGs "Video tutorial")

## Creating an entity
To create an entity, you will only need an [entities2.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/entities2.html) file located within your mod's content folder. There is a lot of content involved with each of the XML's available tags, so this tutorial will cover over them in individual sections.

### `entities` tag
This is the root tag. All `entities2.xml` files must start and end with this tag.

???- info "root `entities` tag variables"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	| anm2root | str | Root path of the anm2 files used for each entity, set to `gfx/` by default.|
	| version | string | The version of the `items.xml` format. **This must always stay equal to 5**.|
	| deathanm2 | string | Optional. Root path of the anm2 file used for showing death portraits of enemies.|

### `entity` tag
This is the tag used to define an entity.

???- info "`entity` tag variables"
	This list excludes enemy-specific variables. You can find those in the [Enemies article](./enemies.md).

	???+ note
		The majority of variables, for most intensive purposes, are optional. The vanilla `entities2.xml` file contains more than necessary for every entry.

		- `name`, `id`, `variant`, `anm2path`, `friction`, and `shadowSize` are baseline variables for any entity.
		- `collisionRadius`, `collisionMass`, and `numGridCollisionPoints` should be included for entities with collision.

	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	| name | str | Name of the entity. |
	| id | int | Type of the entity. Max Value: `4095` |
	| variant | int | Variant of the entity. The maximum value is `4095`. If you leave this blank, then the game will automatically chose the next available number. However, the first available number is commonly `0`, so this is not recommended as it may cause issues. |
	| subtype | int | SubType of the entity. The maximum value is `255`. (The reason for this is that the hash map generator of the .stb format expects a specific bit-depth.) |
	| anm2path | string | Path to the `.anm2` file, relative to the given anm2root. Example: `001.000_Player.anm2` |
	| baseHP | int | Base number of hit points the entity starts with. Usually only relevant for enemies, but can be used on other entities. |
	| collisionDamage | float | Amount of damage the player will take when colliding with this entity. 1 = a half heart. |
	| collisionMass | float | The weight of the entity to determine how far it is pushed when another entity collides against it. The higher the number, the heavier they are. |
	| collisionRadius | float | Radius of the collision circle. This value is used for both entity <--> entity and entity <--> grid collisions. This changes the `Entity.Size` field. |
	| collisionRadiusXMulti | float | Multiplier for the X direction of the collision circle. This can be used to grant an entity an elliptical hitbox. |
	| collisionRadiusYMulti | float | Multiplier for the Y direction of the collision circle. This can be used to grant an entity an elliptical hitbox. |
	| collisionInterval | int | Number of game ticks between checks for collision. Default = `1`. |
	| numGridCollisionPoints | int | Number of points along the edge of the collision circle, which are used to detect collisions with grid entities. |
	| friction | float | Slipperiness of the entity. Default = `1`. Higher values make them slide more, similar as they would standing on ice. Slower values make them slide less. A value of `0` makes them unable to move. |
	| shadowSize | float | The radius of the shadow underneath the entity. |
	| tags | string | A space-separated list of tags which determine certain properties and behaviors. See more [here](#tags-explanation). |
	| gridCollision | string | Determines how the entity collides with grid objects such as rocks and pots. Note that Effect entities (id >999) never have grid collision. Possible values: ['none', 'nopits', 'ground', 'walls', 'floor']. |
	| gibAmount | int | The amount of gibs the entity will drop upon death. Used for dip familiars. The main method of applying gibs and its amount is used with the [gibs tag](entity_basics.md#gibs-tag)  |
	| gibFlags | string | Used values: ['poop']. Used for dip familiars. |

???- info ":modding-repentogon: REPENTOGON-only `entity` tag variables"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	| coinvalue | int | How much this coin pickup is worth when using [GetCoinValue](https://wofsauge.github.io/IsaacDocs/rep/EntityPickup.html#getcoinvalue) (either by the game or a lua call). |
	| customtags | string | Space-separated list of tags added by REPENTOGON which determine certain properties and behaviors. See [CustomTags](https://repentogon.com/xml/entities.html#customtags) section on the REPENTOGON Docs. |
	| nosplit | boolean | Allows preventing this NPC from being split by Meat Cleaver. |

Below is the very first entry inside the vanilla `entities2.xml` file, defining the player. You can use this as an example, but do not actually define new player entities yourself.
```XML
<entities anm2root="gfx/" version="5">
	<entity anm2path="001.000_Player.anm2" baseHP="10" boss="0" champion="0" collisionDamage="0" collisionMass="5" collisionRadius="10" friction="1" id="1" name="Player" numGridCollisionPoints="40" shadowSize="16" stageHP="0" variant="0">
        <gibs amount="0" blood="0" bone="0" eye="0" gut="0" large="0"/>
    </entity>
</entities>
```

### What to set for `id`/`variant`/`subtype`

Firstly, the `id`, or "type" of the enemy, should be set anywhere from 1 to 1000. Depending on the type of entitiy you're creating, you will want to refer to the [EntityType](https://wofsauge.github.io/IsaacDocs/rep/enums/EntityType.html) enum.

- IDs 1-9 are for specific, unique entity types, such as the player and projectiles.
- IDs 10-999 are for enemies.
- ID 1000 is custom effects.
- IDs above 1000 will be defined as effects, but may cause issues for not being the standard ID of 1000. More information can be found later below.

`variant` and `subtype` are for defining different types of the same entity, which have differing rules dependent on what vanilla entries already exist.

???+ bug "Variant auto-filling"
	Due to a bug, allowing the variant to auto-fill for Effect entities can have it use the variant for the "star flash" effect. This will cause unexpected behavior for your effect, and can make it not show up or be removed after a short period of time.

If an entity already has a defined `variant` and you use that `variant`, your entity may take on some of the properties of that entity `variant`. For custom variants, simply define a number not taken up by existing vanilla entries.

`subtype` should always be a number not taken up by existing vanilla entries.

???+ bug "Type overflowing"
	There is a maximum value you can set for `id`, `variant`, and `subtype`:

	- `id` is capped at `4095`.
	- `variant` is capped at `4095`.
	- `subtype` is capped at `255`. 
	
	The values can technically be set higher than these caps, but the game internally condenses all 3 values into a 32-bit integer with a limited amount of space per variable. Amounts higher than the intended cap can cause an overflow to occur, which can cause your entity to start behaving as a different entity if the condensed number lines up with something else. If the condensed value is over the 32-bit integer limit, anything outside those bounds will be lost.

	Generally speaking, `subtype` is safer to allow to overflow, and the vanilla game itself does this for certain enemies. For example, the Ball and Chain enemy uses its subtype as a bitfield for configuration in Basement Renovator. It can still cause issues though, and neither `id` or `variant` should ever go over 4095.

This entity entry defines a new tear variant by having its `id` set to `2`. It starts from variant `39`, as the last defined vanilla tear variant is `38`.
```xml
<entity anm2path="my_new_tear_variant.anm2" collisionMass="8" collisionRadius="7" friction="1" id="2" name="My New Tear" numGridCollisionPoints="8" shadowSize="8" variant="39" />
```

### Fetching the ID/variant/subtype of your entity with Lua
Your defined `id`/`variant` combination may overlap with other mods that happen to define the exact same combination. The game may assign new variants or new subtypes to compensate for the overlap. As such, it is good practice to fetch them with the following functions instead of directly using the numbers you provided:

- [Isaac.GetEntityTypeByName](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getentitytypebyname)
- [Isaac.GetEntityVariantByName](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getentityvariantbyname)
- :modding-repentogon: [Isaac.GetEntitySubTypeByName](https://repentogon.com/Isaac.html#getentitysubtypebyname)

For `subtype`, you must either note down the subtype manually, or use REPENTOGON.

## Tags explanation

The `tag` variable is used to define specific behavior for the entity that varies depending on the tag. You can define one or more tags from the list below:

???- info "`tag` list"
	| Tag-Name | Suffix |
	|:--|:--|
	| cansacrifice | Marks familiars that [Sacrificial Altar](https://bindingofisaacrebirth.wiki.gg/wiki/Sacrificial_Altar) can be used on.|
	| nodelirium | Blacklists a boss from being used by Delirium.|
	| fly | Indicates enemies which should be neutralized by Skatole. Does not affect Beelzebub.|
	| spider | Indicates enemies which should be neutralized by Bursting Sack.|
	| ghost | Indicates enemies which Vade Retro can kill at <50% HP.|
	| noreroll | Immunity to Ace cards that turn enemies into pickups. If a `devolve` tag is defined on the enemy, will prevent rerolls from D10 wisps.|
	| brimstone_soul | Friendly Ball wisps created by this enemy will fire Brimstone lasers.|
	| explosive_soul | Friendly Ball wisps created by this enemy will fire explosive tears.|
	| homing_soul | Friendly Ball wisps created by this enemy will fire homing tears.|
	| devilsacrifice | Repentance+ exclusive tag that allows your item to be paid for with damage instead of heart containers.|

## `entity` child tags

The `entity` tag has numerous child tags to help define additional information about the entity. Only a few are mentioned here, while the others are explained in [Creating enemies](enemies.md#devolve-child-tag).

### `gibs` tag

The `gibs` tag is used to define the gibs that are spawned when an entity is killed or destroyed. The entity will spawn an amount of gibs based on the `amount` variable and randomly select between one of the other gib variables that are set to `1`. The default for the other gib variables is 0 (disabled).

???- info "`gibs` list"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	| amount | int | How many gibs should be spawned.|
	| blood | int | Red meat/muscle particles. |
	| bone | int | Bone particles. |
	| chain | int | Chain gibs. Only used by [The Visage](https://bindingofisaacrebirth.wiki.gg/wiki/The_Visage) in vanilla. |
	| colorblood | int | Colors blood particles to be colored by the entity's [SplatColor](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#splatcolor). |
	| dust | int | Dust particles. |
	| eye | int | Small eyeball particles. |
	| gut | int | Gut particles (e.g. intestines). |
	| huge | int | Used by the Ultra Horsemen. Causes the screen to shake when the entity is killed and plays a meatier death sound. |
	| large | int | Plays a meatier death sound. |
	| poop | int | Poop particles. |
	| rock | int | Small rock particles. |

### `preload` and `preload-snd` tags

Both of these tags are old remnants of Isaac's code for preloading other entities or sounds when the entity is spawned for smoother loading times when it is spawned/played later by said entity. `preload-snd` serves no purpose for the PC version of Isaac as all sounds in the game are preloaded when the game launches. It is not known if `preload` has any useful applications.

???- info "`preload` tag variables"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	| name | str | Name of the entity. |
	| id | int | Id/type of the entity. |
	| variant | int | Variant of the entity. |
	| subtype | int | SubType of the entity. |

???- info "`preload-snd` tag variables"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	| id | int | ID of the sound effect. |