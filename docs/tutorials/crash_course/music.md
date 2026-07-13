---
article: Music
authors: benevolusgoat
blurb: Learn how to add music.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - XML
    - Repentance
    - Repentance+
---

{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

Isaac has a robust OST, having music tracks for every stage, special rooms, and cutscenes. Adding custom music for your own mod can make it stand out when adding something of similar nature.

## Video tutorial
(This tutorial covers both sounds and music)
[![Sound and Music | Youtube Tutorial](https://img.youtube.com/vi/qJB74iaRCf8/0.jpg)](https://youtu.be/qJB74iaRCf8 "Video tutorial")

## Adding custom music
The game's music files are all in the `.ogg` file format, unlike sounds which use the `.wav` format.

All custom music is defined in a [`music.xml`](https://wofsauge.github.io/IsaacDocs/rep/xml/music.html) file, located in the `content` folder at the root of your mod folder. In this folder, there must be a root `music` tag. This tag has a `root` property, which should point to the root directory of where your music are stored starting from your mod's `resources` folder, usually just `music`.

```xml
<music root="music/">

</music>
```

When defining a new music track, you use a `track` tag with a `name` property declaring the name for your music track. You'll use the track's name to grab its id later.

Music tracks can be customized with intros, layers, and more. Many tracks in the vanilla game are composed of several `.ogg` files that come together to form the full thing.

???+ info "`track` tag variables"
	???+ note
		`name` and `path` are the only required variables. `intro` is only needed if you plan to loop the music at a certain point in the track. A few layer-related variables are not listed here; see the [Layer](music.md#layers) section below.
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	|name|string|The name of the track|
	|intro|string|The filepath to the intro of this track. Plays exactly once at the beginning of the song.|
	|path|string|The filepath to the main music file. Plays after the intro.|
	|loop|bool|Default: `false`. Whether or not the main track should loop indefinitely.|
	|mul|float|Default: `1`. Volume percentage of the music track, represented as a float value (e.g. 0.5 is 50% volume).|
	|layermode|int|Default: `1`. 1: Layer overlaps with the base track. 2: Layer plays instead of the base track.|
	|layerfadespeed|float|Default: `0.08`. The speed at which the music transitions between the main track and its layers, acting as the percentage amount of its progress every render frame (60 frames per second).|

Below is an example XML named "Chocobo":

```xml
<music root="music/">
    <track name="Chocobo" path="chocobo.ogg" loop="true" />
</music>
```

### Layers
Layers are additional music tracks that can be defined under the main music track. Layers can play on top of the main track or replace it, depending on what `layermode` is set to in the xml. They are not played by default and must be manually triggered with Lua. There are two ways to add layers:

1. The `track` tag contains the `layer`, `layerintro`, and `layermul` variables, equivalent to `path`, `intro`, and `mul` respectively. This is restricted to only being able to define information for one layer.

```xml
<music root="music/">
    <track name="Chocobo" path="chocobo.ogg" loop="true" layer="chobobo_angry.ogg" />
</music>
```

2. Adding the `layer` child node. It's added under the track node, having `intro`, `path`, and `mul` as possible variables. You can add more than one layer to a track, allowing for expanded customizability. Remember to open up the track node similarly to the root music note so that it can accept `layer` nodes.

```xml
<music root="music/">
    <track name="Chocobo" path="chocobo.ogg" loop="true">
		<layer path="chocobo_angry.ogg" />
	</track>
</music>
```

## Playing music with Lua
To play music, you'll need to use the [`MusicManager`](https://wofsauge.github.io/IsaacDocs/rep/MusicManager.html) object, which you can instantiate with `MusicManager()`. Then use the [`Play`](https://wofsauge.github.io/IsaacDocs/rep/MusicManager.html#play) function. However, the track will play at full volume by default, ignoring your music volume setting. This can be fixed by either using [`Options.MusicVolume`](https://wofsauge.github.io/IsaacDocs/rep/Options.html#musicvolume) for the Volume parameter or using [`MusicManager:UpdateVolume`](https://wofsauge.github.io/IsaacDocs/rep/MusicManager.html#updatevolume) after playing the music. Note that the latter option will cancel any fade actions and instantly update the music to play at its expected volume.

As a simple method of testing the music, we'll be using [`Input.IsButtonTriggered`](https://wofsauge.github.io/IsaacDocs/rep/Input.html#isbuttontriggered) to detect a specific keyboard input.

```lua
local mod = RegisterMod("My Mod", 1)
--You only need to call MusicManager() once
local music = MusicManager()

function mod:PlayMusicOnHotKey()
	--Detects if the T key is pressed, and will only trigger upon initially pressing the button. `0` is where the controller index is inserted, where the keyboard is always `0`.
	if Input.IsButtonTriggered(Keyboard.KEY_T, 0) then
		--Play the music track
		music:Play(Music.MUSIC_BASEMENT) -- Id of 1
		music:UpdateVolume() --Update to music option setting
	end
end

--Inputs are best detected at 60 FPS, so MC_POST_RENDER is used.
mod:AddCallback(ModCallbacks.MC_POST_RENDER, mod.PlayMusicOnHotKey)
```

To play a custom music track, get its id with [`Isaac.GetMusicIdByName`](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getmusicidbyname).

```lua
local mod = RegisterMod("My Mod", 1)
local music = MusicManager()
local musicId = Isaac.GetMusicIdByName("New Sound")

function mod:PlayMusicOnHotKey()
	if Input.IsButtonTriggered(Keyboard.KEY_T, 0) then
		music:Play(musicId)
	end
end

mod:AddCallback(ModCallbacks.MC_POST_RENDER, mod.PlayMusicOnHotKey)
```

For layers, they must be enabled through [`MusicManager:EnableLayer`](https://wofsauge.github.io/IsaacDocs/rep/MusicManager.html#enablelayer). The layer Id corresponds to the order the tracks were created in inside the `music.xml`, starting from `0`. When enabled, layers will fade in unless the `Instant` argument is set to true. Layers are also synced with the main music track, so it will play the layer track at whatever point the main track is at in its duration. Use [`MusicManager:DisableLayer`](https://wofsauge.github.io/IsaacDocs/rep/MusicManager.html#disablelayer) to disable a layer. As layers are connected to a main track, any modifications made to the main track such as fading in/out, pitch, etc, will also affect its layers.

```lua
local mod = RegisterMod("My Mod", 1)
local music = MusicManager()
local musicId = Isaac.GetMusicIdByName("New Sound")

function mod:PlayMusicOnHotKey()
	if Input.IsButtonTriggered(Keyboard.KEY_T, 0) then
		music:Play(musicId)
	end
	if Input.IsButtonTriggered(Keyboard.KEY_Y, 0) then
		--Enables layer 0 by default
		music:EnableLayer()
	end
end

mod:AddCallback(ModCallbacks.MC_POST_RENDER, mod.PlayMusicOnHotKey)
```

## :modding-repentogon: Jingles
Jingles are very brief music tracks, typically a few seconds long, for indicating specific events. Common examples of this are entering a treasure room or finding a secret room. Jingles cause the the main track to fade out most of the way, have the jingle play, then fade the main track back in. Utilizing Jingles yourself is exclusive to :modding-repentogon: REPENTOGON, where it's as simple as using [`MusicManager:PlayJingle`](https://repentogon.com/MusicManager.html#playjingle).