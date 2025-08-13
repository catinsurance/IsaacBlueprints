---
article: Resources
authors: benevolusgoat
blurb: Learn how to create custom entities.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - No Lua
    - XML
    - Repentance
    - Repentance+
    - REPENTOGON
---

{% include-markdown "hidden/unfinished_notice.md" start="<!-- start -->" end="<!-- end -->" %}
{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

Entities make up the majority of anything that moves inside the Binding of Isaac, from tears to enemies to Isaac himself. This tutorial will cover the basics on creating a custom entity.

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

		- `name` and `id` are fully required.
		- `variant` `anm2path`, `friction`, and `shadowSize` are recommended to be treated as required for all entities.
		- `collisionRadius`, `collisionMass`, and `numGridCollisionPoints` should be included for entities with collision.
	| Variable-Name | Possible Values | Description |
	|:--|:--|:--|
	| name | str | Name of the entity. |
	| id | int | Type of the entity. Max Value: `4095` |
	| variant | int | Variant of the entity. The maximum value is `4095`. If you leave this blank, then the game will automatically chose the next available number. However, the first available number is commonly `0`, so this is not recommended. |
	| subtype | int | SubType of the entity. The maximum value is `255`. (The reason for this is that the hash map generator of the .stb format expects a specific bit-depth.) |
	| anm2path | string | Path to the `.anm2` file, relative to the given anm2root. Example: `001.000_Player.anm2` |
	| baseHP | int | Base number of hit points the entity starts with. Usually only relevant for enemies, but can be used on other entities. |
	| boss | int | Entity is a boss. Possible values: [`0`, `1`]. |
	| bossID | int | The unique boss ID associated with the entity, used for other things such as entries inside `bosspools.xml`. Custom entities cannot take advantage of this. |
	| champion | int | Allow champion variants of this entity. Possible values: [`0`, `1`]. |
	| collisionDamage | float | Amount of damage an entity will take when colliding with this entity. |
	| collisionMass | float | The weight of the entity to determine how far it is pushed when another entity collides against it. The higher the number, the heavier they are. |
	| collisionRadius | float | Radius of the collision circle. This value is used for both entity <--> entity and entity <--> grid collisions. This changes the `Entity.Size` field. |
	| collisionRadiusXMulti | float | Multiplier for the X direction of the collision circle. This can be used to grant an entity an elliptical hitbox. |
	| collisionRadiusYMulti | float | Multiplier for the Y direction of the collision circle. This can be used to grant an entity an elliptical hitbox. |
	| collisionInterval | int | Number of game ticks till the next collision should be evaluated. Default = `1`. |
	| numGridCollisionPoints | int | Number of points along the edge of the collision circle, which are used to detect collisions with grid entities. |
	| friction | float | "Slippyness" of the entity. Default = `1`. Lower values make them slide more, similar as they would standing on ice. Higher values make them slide less. A value of `0` makes them unable to move. |
	| shadowSize | float | The size of the shadow underneath the entity. |
	| stageHP | int | A multiplier for how much additional health the enemy gains with each stage. Read [here](https://bindingofisaacrebirth.wiki.gg/wiki/Stage_HP) for more information. |
	| tags | string | Possible values: ['nodelirium', 'spider', 'explosive_soul', 'cansacrifice', 'ghost', 'brimstone_soul', 'homing_soul', 'fly', 'noreroll'].<br>See Chapter below for in depth explanations of the tags. |
	| gridCollision | string | Possible values: ['nopits', 'ground', 'none', 'walls', 'floor']. |
	| portrait | int | Used for enemies in conjunction with the entities `deathanm2` tag to determine what portrait to show on [Isaac's Last Will](https://bindingofisaacrebirth.wiki.gg/wiki/Isaac%27s_Last_Will). The number correlates to the frame in the anm2 file, starting from `0`. |
	| hasFloorAlts | bool | If set to `true`, floor specific sprites should be used for this entity if they exist. See the chapter below for more informations |
	| reroll | bool | Sets whether or not this entity can be rerolled. Used for enemies. |
	| shutdoors | bool | Determines whether this entity will prevent doors from staying closed. Only used for enemies. Default = `true`. |
	| shieldStrength | int | Entity takes less damage relative to Isaac's DPS, known primarily as "armor". The higher the number, the less damage is taken. Default = `0`.<br>See [this page](https://bindingofisaacrebirth.wiki.gg/wiki/Damage_Scaling) for more information on the mechanic. |
	| gibAmount | int | The amount of gibs the entity will drop upon death. Suggestion default is `5`. |
	| gibFlags | string | Used values: ['poop']. |
	| bestiaryAnim | string | The animation to play when viewing this entity in the [Bestiary](https://bindingofisaacrebirth.wiki.gg/wiki/Bestiary_(Repentance)). |
	| bestiaryOverlay | string | The animation to play when viewing this entity in the Bestiary. |

The very first entry inside the vanilla `entities2.xml` file, defining the player. **Do not actually define player entities using this method.**
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

## Tags explanation

The `tag` variable is used to define specific behavior for the entity that varies depending on the tag. You can define one or more tags from the list below:

| Tag-Name | Suffix |
|:--|:--|
|cansacrifice| Marks familiars on which sacrificial altar can be used on|
|nodelirium| Blacklists a boss from being used by Delirium|
|fly|Indicates enemies which should be neutralized by Skatole (does NOT affect Beelzebub)|
|spider|Indicates enemies which should be neutralized by Bursting Sack|
|ghost|Indicates enemies which Vade Retro can kill at <50% HP as a special interaction|
|noreroll| Immunity from D10 rerolls and the Ace cards|
|brimstone_soul| Friendly Ball wisps created by this enemy will fire Brimstone lasers|
|explosive_soul| Friendly Ball wisps created by this enemy will fire explosive tears|
|homing_soul| Friendly Ball wisps created by this enemy will fire homing tears|
