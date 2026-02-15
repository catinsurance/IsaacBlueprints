---
article: Light layers
authors: catinsurance
blurb: Learn how to create custom light sources.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - No Lua
---

Certain entities make light in the darkness. This is most visible in Mines dark rooms, and with Curse of Darkness. The way this is done is with a **null layer** in the `.anm2` of your Entity.

## Creating a light layer
To create a light layer in an entity's `.anm2`, create a new layer. Make sure you **tick the "Null" box,** and make sure that **the layer name starts with an asterisk (`*`).**

<p align="center">
  <img src="../../assets/light_layer/properties.png" alt="Creating a null layer" />
</p>

- Edit the `Scale X` and `Scale Y` properties to change the size of the light.
- Edit the `R`, `G`, and `B` properties under `Tint RGB` to edit the color the light.
- Edit the `Tint Alpha` property to edit the brightness of the light.

![A diagram showing the different properties of the light in the anm2 editor](../assets/light_layer/light_layer.png)

## Sprites only visible in the dark
You can make a normal sprite layer (orange in the editor) only visible in the darkness by adding an asterisk (`*`) at the start of the layer's name. This is used by enemies such as [One Tooths](https://bindingofisaacrebirth.wiki.gg/wiki/One_Tooth) to have a layer over their eyes that is visible in the dark.

Note that this does not affect surrounding lighting and only makes the layer visible in the dark.

<p style="vertical-align: middle;">
	<img align="left" src="../../assets/light_layer/one_tooth_anm2.jpg" alt="The anm2 file of a One Tooth" />
	<img align="right" src="../../assets/light_layer/one_tooth.jpg" alt="The eyes of a One Tooth are visible in the dark" />
</p>