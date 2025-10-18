---
article: Entity basics
authors: benevolusgoat
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

{% include-markdown "hidden/unfinished_notice.md" start="<!-- start -->" end="<!-- end -->" %}
{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

Entities make up the majority of anything that moves inside The Binding of Isaac, from tears to enemies to Isaac himself. This tutorial will cover the basics on creating a custom entity.

## Video tutorial
[![Entities and Effects | Youtube Tutorial](https://img.youtube.com/vi/sxgf2SH6ZGs/0.jpg)](https://youtu.be/sxgf2SH6ZGs "Video tutorial")

## Creating an entity
To create an entity, you will only need an [entities2.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/entities2.html) file located within your mod's content folder. There is a lot of content involved with each of the XML's available tags, so this tutorial will cover over them in individual sections.

### entities tag
This is the root tag. All `entities2.xml` files must start and end with this tag.

???- info "root `entities` tag variables"
	| Variable-Name | Possible Values | Description |
	|:--|:--|:--|
	| anm2root | str | Root path of the anm2 files used for each entity, set to `gfx/` by default.|
	| version | string | The version of the `items.xml` format. **This must always stay equal to 5**.|
	| deathanm2 | string | Optional. Root path of the anm2 file used for showing death portraits of enemies.|

### entity tag
This is the tag used to define an entity.

???- info "`entity` tag variables"
	???+ note
		The majority of variables, for most intensive purposes, are optional. The vanilla `entities2.xml` file contains more than necessary for every entry.

		- `name`, `id`, `variant`, `anm2path`, `friction`, and `shadowSize` are baseline variables for any entity.
		- `collisionRadius`, `collisionMass`, and `numGridCollisionPoints` should be included for entities with collision.
	| Variable-Name | Possible Values | Description |
	|:--|:--|:--|
	| name | str | Name of the entity. |
	| id | int | Type of the entity. Max Value: `4095` |
	| variant | int | Variant of the entity. The maximum value is `4095`. If you leave this blank, then the game will automatically chose the next available number. However, the first available number is commonly `0`, so this is not recommended as it may cause issues. |
	| subtype | int | SubType of the entity. The maximum value is `255`. (The reason for this is that the hash map generator of the .stb format expects a specific bit-depth.) |
	| anm2path | string | Path to the `.anm2` file, relative to the given anm2root. Example: `001.000_Player.anm2` |
	| baseHP | int | Base number of hit points the entity starts with. Usually only relevant for enemies, but can be used on other entities. |
	| collisionDamage | float | Amount of damage an entity will take when colliding with this entity. |
	| collisionMass | float | The weight of the entity to determine how far it is pushed when another entity collides against it. The higher the number, the heavier they are. |
	| collisionRadius | float | Radius of the collision circle. This value is used for both entity <--> entity and entity <--> grid collisions. This changes the `Entity.Size` field. |
	| collisionRadiusXMulti | float | Multiplier for the X direction of the collision circle. This can be used to grant an entity an elliptical hitbox. |
	| collisionRadiusYMulti | float | Multiplier for the Y direction of the collision circle. This can be used to grant an entity an elliptical hitbox. |
	| collisionInterval | int | Number of game ticks till the next collision should be evaluated. Default = `1`. |
	| numGridCollisionPoints | int | Number of points along the edge of the collision circle, which are used to detect collisions with grid entities. |
	| friction | float | "Slippyness" of the entity. Default = `1`. Lower values make them slide more, similar as they would standing on ice. Higher values make them slide less. A value of `0` makes them unable to move. |
	| shadowSize | float | The size of the shadow underneath the entity. |
	| tags | string | Possible values: ['nodelirium', 'spider', 'explosive_soul', 'cansacrifice', 'ghost', 'brimstone_soul', 'homing_soul', 'fly', 'noreroll'].<br>See Chapter below for in depth explanations of the tags. |
	| gridCollision | string | Possible values: ['nopits', 'ground', 'none', 'walls', 'floor']. |
	| gibAmount | int | The amount of gibs the entity will drop upon death. Used for dip familiars. The main method of applying gibs and its amount is used with the [gibs tag](entity_basics.md#gibs-tag)  |
	| gibFlags | string | Used values: ['poop']. Used for dip familiars. |

The very first entry inside the vanilla `entities2.xml` file, defining the player. **Do not actually define new player entities yourself.**
```XML
<entities anm2root="gfx/" version="5">
	<entity anm2path="001.000_Player.anm2" baseHP="10" boss="0" champion="0" collisionDamage="0" collisionMass="5" collisionRadius="10" friction="1" id="1" name="Player" numGridCollisionPoints="40" shadowSize="16" stageHP="0" variant="0">
        <gibs amount="0" blood="0" bone="0" eye="0" gut="0" large="0"/>
    </entity>
</entities>
```

:modding-repentogon: REPENTOGON also adds the following variables:

| Variable-Name | Possible Values | Description |
|:--|:--|:--|
| coinvalue | int | How much this coin pickup is worth when using [GetCoinValue](wofsauge.github.io/IsaacDocs/rep/EntityPickup.html#getcoinvalue) (either by the game or a lua call). |
| customtags | string | Space-separated list of strings. See [CustomTags](https://repentogon.com/xml/entities.html#customtags) section on the REPENTOGON Docs. |
| nosplit | boolean | Allows preventing this NPC from being split by Meat Cleaver. |

### What to set for `id`/`variant`/`subtype`

Firstly, the `id`, or "type" of the enemy, should be set anywhere from 1 to 1000. Depending on the type of entitiy you're creating, you will want to refer to the [EntityType](https://wofsauge.github.io/IsaacDocs/rep/enums/EntityType.html) enum.
- IDs 1-9 are preset entity types.
- IDs 10-999 are for enemies.
- ID 1000 is custom effects.
- IDs beyond 1000 will be defined as effects, but may cause issues for not being the standard ID of 1000.

`variant` and `subtype` are for defining different types of the same entity, which have differing rules dependent on what vanilla entries already exist.
- If an entity already has a defined `variant` and you use that `variant`, your entity may take on some of the properties of that entity `variant`.
- For custom variants, simply define a number not taken up by existing vanilla entries.
- `subtype` should always be a number not taken up by existing vanilla entries.

This entity entry defines a new tear variant by having its `id` set to `2`. It starts from variant `39` as the last defined vanilla tear variant is `38`.
```xml
	<entity anm2path="my_new_tear_variant.anm2" collisionMass="8" collisionRadius="7" friction="1" id="2" name="My New Tear" numGridCollisionPoints="8" shadowSize="8" variant="39">
    </entity>
```

The maximum number you can set `id` and `variant` to is `4095`, while `subtype` is only `255`. The number can technically be set higher than these caps, but the game internally condences all 3 values into a 32-bit integer with a limited amount of space for each variable. Amounts higher than the intended cap can cause an overflow to occur, which leads to unintended issues, such as game crashes.

### Fetching the ID/variant/subtype of your entity with Lua
Your defined `id`/`variant` combination may overlap with other mods that happen to define the exact same combination. The game may assign new variants or new subtypes to compensate for the overlap. As such, it is good practice to fetch them with the following functions:
- [Isaac.GetEntityTypeByName](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getentitytypebyname)
- [Isaac.GetEntityVariantByName](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getentityvariantbyname)

For `subtype`, you must either note down the subtype manually, or use REPENTOGON, which introduces [Isaac.GetEntitySubTypeByName](https://repentogon.com/Isaac.html#int-getentitysubtypebyname-string-name)
- :modding-repentogon: Isaac.GetEntitySubTypeByName("Entity Name Here")

## Tags explanation

The `tag` variable is used to define specific behavior for the entity that varies depending on the tag. You can define one or more tags from the list below:

| Tag-Name | Suffix |
|:--|:--|
| cansacrifice | Marks familiars on which sacrificial altar can be used on.|
| nodelirium | Blacklists a boss from being used by Delirium.|
| fly | Indicates enemies which should be neutralized by Skatole (does NOT affect Beelzebub).|
| spider | Indicates enemies which should be neutralized by Bursting Sack.|
| ghost | Indicates enemies which Vade Retro can kill at <50% HP as a special interaction.|
| noreroll | Immunity to Ace cards that turn enemies into pickups. If a `devolve` tag is defined on the enemy, will prevent rerolls from D10 wisps.|
| brimstone_soul | Friendly Ball wisps created by this enemy will fire Brimstone lasers.|
| explosive_soul | Friendly Ball wisps created by this enemy will fire explosive tears.|
| homing_soul | Friendly Ball wisps created by this enemy will fire homing tears.|

## entity child tags

The `entity` tag has numerous child tags to help define additional information about the entity that's used for different situations depending on tag. Only a few are mentioned here while the others are explained in [Creating enemies](enemies.md#devolve-child-tag).

### `gibs` tag

The `gibs` tag is used to define the gibs that are spawned when an entity is killed or destroyed. The entity will spawn an amount of gibs based on the `amount` variable and randomly select between one of the other gib variables that are set to `1`.

| Variable-Name | Possible Values | Description |
|:--|:--|:--|
| amount | int | How many gibs should be spawned.|
| blood | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| bone | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| chain | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| colorblood | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| dust | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| eye | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| gut | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| huge | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| large | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| poop | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|
| rock | int | Possible values: [`0`,`1`] where `0` is off and `1` is on.|

### `preload` and `preload-snd` tags

Both of these tags are old remnants of Isaac's code for preloading other entities or sounds when the entity is spawned for smoother loading times when it is spawned/played later by said entity. `preload-snd` serves no purpose for the PC version of Isaac as all sounds in the game are preloaded when the game launches. It is not known if `preload` has any useful applications.

???- info "`preload` tag variables"
	| Variable-Name | Possible Values | Description |
	|:--|:--|:--|
	| name | str | Name of the entity. |
	| id | int | Type of the entity. Max Value: `4095` |
	| variant | int | Variant of the entity. The maximum value is `4095`. If you leave this blank, then the game will automatically chose the next available number. However, the first available number is commonly `0`, so this is not recommended as it may cause issues. |
	| subtype | int | SubType of the entity. The maximum value is `255`. (The reason for this is that the hash map generator of the .stb format expects a specific bit-depth.) |

???- info "`preload-snd` tag variables"
	| Variable-Name | Possible Values | Description |
	|:--|:--|:--|
	| id | int | ID of the sound effect. |