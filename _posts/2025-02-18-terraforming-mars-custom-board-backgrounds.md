---
layout: post
title: "MichuMod: Custom Board Backgrounds"
date: 2025-02-18 16:32 -03:00
categories: ["Terraforming Mars"]
---

Each board in *Terraforming Mars* has a distinct background. The planet's *northern ice cap* can be seen on the lower part of *Hellas*, upper part of *Utopia*, and center of *Vastitas*. Up until now, all custom boards were using the background texture of *Tharsis*. I wanted to change that.

There is a `Texture` field in the `TM_BoardDataSet` *ScriptableObject* called `BoardGroundTexture`. I found a free 3D model of Mars and rendered views for *Utopia*, *Cimmeria*, and *Vastitas*, trying to match the graphics used for the original boards. This is how the custom textures look after some post-production in *Photoshop*:
![Custom Ground Textures](/assets/images/michumod_custom_ground_textures.jpg)
*Utopia, Cimmeria, and Vastitas custom ground textures*

I then had to pass these textures to the constructor method of the custom `TM_BoardDataSet` objects. Here is how they look in the game:
![Custom Utopia Board](/assets/images/terraforming-mars-custom-utopia-board.jpg)
*Utopia Board in-game*
![Custom Cimmeria Board](/assets/images/terraforming-mars-custom-cimmeria-board.jpg)
*Cimmeria Board in-game*
![Custom Vastitas Board](/assets/images/terraforming-mars-custom-vastitas-board.jpg)
*Vastitas Board in-game*

The color for the **glow effect** that surrounds the planet can be set by creating and passing a *Unity* `Color` to the `BoardColor` field of the `TM_BoardDataSet` *ScriptableObject*. For example, for *Utopia*:
```csharp
boardDataSet.BoardColor = new Color(0.937f, 0.486f, 0.0f, 1.0f);
```
