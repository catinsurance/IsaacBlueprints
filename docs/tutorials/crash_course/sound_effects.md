---
article: Sounds
authors: catinsurance
blurb: Learn how to add sound effects.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - XML
    - Repentance
    - Repentance+
---

{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

While the game has a massive catalog of sound effects to use, custom sound effects can be vital for adding the necessary feedback to actions and events.

## Video tutorial
(This tutorial covers both sounds and music)
[![Sound and Music | Youtube Tutorial](https://img.youtube.com/vi/qJB74iaRCf8/0.jpg)](https://youtu.be/qJB74iaRCf8 "Video tutorial")

## Vanilla sounds
Isaac has a massive catalog of over 1000 sound effects as of Repentance+. All sounds have a numerical id, with the ids for vanilla sounds being found under the [`SoundEffect`](https://wofsauge.github.io/IsaacDocs/rep/enums/SoundEffect.html) enumerator. The docs page for this enumerator also allows you to quickly preview any sound. Sounds tend to be the largest contributor to file size in a mod, so make sure to look through the list of vanilla sounds before implementing your own.

## Encoding custom sounds
Before you add your sound to the game using XML, it's very important to ensure it is encoded correctly. The game expects all sounds to be in `.wav` format and encoded in 16-bit. **Any sound that is incorrectly encoded will instead be a very loud, high-pitched static noise!** There is no restriction on bitrate or audio channel options.

There are many ways to encode a sound effect with the proper configuration:

- [Audacity](https://www.audacityteam.org/download/) is a free, open-source program with a suite of audio editing tools. It includes an option to export sound files in `.wav` format. **You can choose the "Signed 16-bit PCM" encoding format during the export process for use in Isaac.**
- For an online alternative, [convertio.co](https://convertio.co/mp3-wav/) lets you convert `.mp3` files to `.wav` with a multitude of codecs. **You can choose "PCM_S16LE (Uncompressed)" from this list to encode sounds for use in Isaac.**

## Adding custom sounds
All custom sound effects are defined in a [`sounds.xml`](https://wofsauge.github.io/IsaacDocs/rep/xml/sounds.html) file, located in the `content` folder at the root of your mod folder. In this folder, there must be a root `sounds` tag. This tag has a `root` property, which should point to the root directory of where your sound effects are stored starting from your mod's `resources` folder, usually just `sfx`.

```xml
<sounds root="sfx/">

</sounds>
```

When defining a new sound, you use a `sound` tag with a `name` property declaring the name for your sound. You'll use the sound's name to grab its id later.

Within a `sound` tag must be at least one `sample` tag, although there can be as many as desired. Samples are individual audio files to play when your sound is played. If multiple samples are provided, then the game will choose a random one from the list to play. Samples have a `weight` attribute that **is required but never used by the game.** Every sound in vanilla has this set to "1".

```xml
<sounds root="sfx/">
    <sound name="New Sound">
        <sample weight="1" path="my_sound_effect.wav" />
    </sound>

    <!-- Chooses one sample at random to play when sound is played -->
    <sound name="New Sound 2">
        <sample weight="1" path="my_other_effect1.wav" />
        <sample weight="1" path="my_other_effect2.wav" />
        <sample weight="1" path="my_other_effect3.wav" />
    </sound>
</sounds>
```

## Playing sounds with Lua
To play sounds, you'll need to use the [`SFXManager`](https://wofsauge.github.io/IsaacDocs/rep/SFXManager.html) object, which you can instantiate with `SFXManager()`. Then use the [`Play`](https://wofsauge.github.io/IsaacDocs/rep/SFXManager.html#play) function.

```lua
local sfxManager = SFXManager()

sfxManager:Play(SoundEffect.SOUND_1UP) -- Id of 1
```

To play a custom sound, first get its id with [`Isaac.GetSoundIdByName`](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getsoundidbyname).

```lua
local sfxManager = SFXManager()
local soundId = Isaac.GetSoundIdByName("New Sound")

sfxManager:Play(soundId)
```

Lastly, you can use the optional arguments of `Play` for more control over how the sound is played. The arguments are in the following order:

1. `integer` **Sound id:** The only required argument, this is id number of the sound.
2. `float` **Volume:** A multiplier determining how loud the sound is. Default is 1.
3. `int` **FrameDelay:** The global "cooldown" before the sound can be played again, in game ticks (30 ticks per second). Default is 2, meaning the sound cannot be played again for 2 game ticks.
4. `boolean` **Loop:** Determines if the sound is looped after its finished playing. Default is `false`.
5. `float` **Pitch:** A multiplier determining the pitch of the sound, or how high or low it is. Default is 1.
6. `float` **Pan:** A multiplier determining the pan of the sound, or how far to the left or right the sound plays in the player's speakers or headphones, with -1 being fully to the left and 1 being fully to the right. This is only respected when the `.wav` file played only has a single audio track (no non-mono audio files). Default is 0, which makes it play in the middle. 

```lua
local sfxManager = SFXManager()
local soundId = Isaac.GetSoundIdByName("New Sound")

-- All of these are the default values besides the first argument.
sfxManager:Play(
    soundId, -- Sound id (required)
    1,       -- Volume
    2,       -- Frame delay
    false,   -- Loop
    1,       -- Pitch
    0        -- Pan
)
```