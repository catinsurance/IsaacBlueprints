---
article: Costumes
authors: benevolusgoat
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - XML
    - Repentance
    - Repentance+
    - REPENTOGON
---

Costumes are the extra layers of sprites that are added onto Isaac to change his looks. These can be seen from passive collctibles, on a character such as their hair or other accessories, and other sources. This tutorial will cover how to create and add your own custom costumes

## Creating a costume
All costumes require an `.anm2` file. You can learn how to create one here (TODO: ANM2s definitely need their own dedicated crash course page).

When creating your costume, it is best to reference, copy, and/or modify an existing costume from the game as it will already have all the animations and frames laid out for you. You can find them after [extracting the game's resources](creating_a_mod.md/#extracting-the-games-resources) inside `resources/gfx/characters/`.

## Costume layers
All of the game's costumes work by layers. A costume can occupy one or more layers, but each layer can only hold one costume. There are a total of **15 layers**. Below is the list of costumes, their displayed priority in order of top to bottom:

| Layer-Name | Description |
|:--|:--|
|ghost|Reversed for displaying Isaac's ghost upon death|
|extra|Exclusively used by Mega Mush for the transformation animations, displaying Isaac's regular sprite before growing/after shrinking|
|top0|Used for wing costumes when walking upwards. Attached to the body's direction, but renders above the head|
|head(5,4,3,2,1)|Item costumes for the head|
|head0|Character-specific costumes (e.g. Magdelene's hair)|
|head|Directly replaces Isaac's head
|body(0,1)|Item costumes for the body|
|body|Directly replaces Isaac's body|
|back|Attached to the head. Exclusively used by the Venus costume for the back view of the hair. Is best seen with a costume that removes the body|
|glow|Attached to the head. Used for glowing auras behind Isaac's head|

A costume can have any combinations of these layers. Any layers not a part of this list will not appear on Isaac.

If two costumes that occupy the same layer are added onto Isaac, the one with the highest priority will show. Otherwise, if equal priority, the one that was added most recently will show. Priorities are defined in the `costumes.xml` file.

If a costume with more than one layer has one of their layers conflicting with another costume, and the other costumes is prioritized, the other layers will still remain on Isaac, allowing a unique mismatch of different costumes on multiple layers.

## Adding a costume
All costumes entries are defined in a [costumes2.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/costumes2.html) file, located in the `content` folder at the root of your mod folder. A tutorial on adding your own costumes are covered [here](https://wofsauge.github.io/IsaacDocs/rep/tutorials/AddingCostumesWithoutLUA.html).

???+ warning
    In regards to adding Null costumes via Lua in the tutorial linked above, is it VERY easy to accidentally crash the game. `Isaac.GetCostumeIdByPath` will return `-1` if the costume's path isn't defined in `costumes.xml`. If you attempt to add a costume with an ID of -1 or any ID that doesn't correlate to an existing costume, the game will always crash.

## Adding character-specific costume replacements
Some characters have unique head shapes or faces, and as such require special variants of a costume in order for it to look more natural on that character. These are added through costume suffix folders. Inside the `resources/gfx/characters/` path, you can have folders named `costumes_` follwed by a defined suffix. Either define `costumeSuffix` in your character's [players.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/players.html?) entry (e.g. `costumeSuffix="mycharacter"`) or use an existing suffix from the vanilla characters. The complete folder name should look something like `costumes_mycharacter`. For reference, the existing vanilla suffixes are:

- shadow (Dark Judas, Tainted Judas)
- lilith
- keeper
- apollyon
- forgotten
- forgottensoul
- lilithb

Within your costume suffix folder, simply place your costume .png file with the exact same name as the costume you wish to replace. The game will automatically handle using this costume sprite instead of the default one for the desired character.

## Adding a character-specific costume
There are 3 ways to add a costume such that it will only appear on your own custom character.

### Lua
This method technically works, but it is not advised, as it can easily be removed by means such as the Mother's Shadow chase/Knife Piece 2 sequence and items that reroll all your items, such as D4. You can add your costume under the `MC_POST_PLAYER_INIT` callback for when your player initializes, and for extra safety, on `MC_POST_NEW_ROOM` in case it was accidentally removed. You may see older mods use this method.

```lua
local mod = RegisterMod("My Mod", 1)
local MY_PLAYER = Isaac.GetPlayerTypeByName("My Character", false)
local MY_COSTUME = Isaac.GetCostumeIdByPath("gfx/characters/my_costume.anm2")

function mod:AddPlayerCostume(player)
	--Check by PlayerType, not name, to ensure it's our character
	if player:GetPlayerType() == MY_PLAYER then
		--Add the costume
		player:AddNullCostume(MY_COSTUME)
	end
end

function mod:AddCostumeOnNewRoom()
	--Most reliable way to search through players in the room
	for _, ent in ipairs(Isaac.FindByType(EntityType.ENTITY_PLAYER)) do
		--Isaac.FindByType passes an Entity. Turn it into an EntityPlayer to use player-specific functions
		local player = ent:ToPlayer()
		--Call the function above that already has the code we want
		mod:AddPlayerCostume(player)
	end
end

--The 0 here is for actual players, so not co-op babies, but still including co-op ghosts
mod:AddCallback(ModCallbacks.MC_POST_PLAYER_INIT, mod.AddPlayerCostume, 0)
mod:AddCallback(ModCallbacks.MC_POST_NEW_ROOM, mod.AddCostumeOnNewRoom)
```

### XML (Non-REPENTOGON)
Defining and adding a costume to a character with this method will ensure that the costume is never removed under any normal circumstances. As there is no method to define our own custom costumes in the [players.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/players.html?) file, we can make do with adding an existing costume and replacing its sprite only if its on our character.

Inside the `players.xml` file where you have your custom character defined, add the `costume` variable and set it to an existing vanilla null costume. You can get a good idea of the IDs of each null costume through the [NullItemID](https://wofsauge.github.io/IsaacDocs/rep/enums/NullItemID.html) enumeration, or looking in the `costumes2.xml` folder inside the game's extracted resources. In this case, we'll be using Dead Tainted Lazarus' costume.

```xml
<players root="gfx/characters/costumes/"
         portraitroot="gfx/ui/stage/"
         nameimageroot="gfx/ui/boss/">
	<player name="My Character" skin="character_mycharacter.png" hp="6"
			nameimage="playername_mycharacter.png"
			portrait="playerportrait_mycharacter.png" costume="34" costumeSuffix="mycharacter"
			birthright="???" />
</players>
```

From here, you'll want a costumeSuffix folder for your character. Follow the tutorial above for [character-specific costume replacements](costumes.md#adding-character-specific-costume-replacements) for reference, and your costume sprite should be an edit of `character_009b_lazarus2hair.png`.

### XML (REPENTOGON)
REPENTOGON adds a method to add your own costume through the `modcostume` variable you can put in inside `players.xml`.

### :fontawesome-solid-code: costumes2.xml {: .subHeader .example_code}
Define a null costume in `costumes2.xml` by setting the type to "none". Unlike normal circumstances, you'll need to set an ID! This will not overlap with IDs from costumes of other types (passive, familar, active, trinket) or from other mods.

```xml
<costumes anm2root="gfx/">
    <costume id="0" anm2path="characters/some_null_costume.anm2" type="none" />
</costumes>
```

### :fontawesome-solid-code: players.xml {: .subHeader .example_code}
Add `modcostume` to your character's `players.xml` entry and set it to the ID you defined for it.

```xml
<players root="gfx/characters/costumes/"
         portraitroot="gfx/ui/stage/"
         nameimageroot="gfx/ui/boss/">
	<player name="My Character" skin="character_mycharacter.png" hp="6"
			nameimage="playername_mycharacter.png"
			portrait="playerportrait_mycharacter.png" modcostume="0"
			birthright="???" />
</players>
```
