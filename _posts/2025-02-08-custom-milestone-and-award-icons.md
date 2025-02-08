---
layout: post
title: "MichuMod: Custom Milestone & Award Icons"
date: 2025-02-08 18:15 -03:00
categories: ["Terraforming Mars"]
---

This is a continuation of: [Part 2]({% post_url 2025-01-31-programming-custom-milestones-part-2 %}).

Getting the custom *Milestones* and *Awards* to work in the game was the tricky part, now comes some fun. I designed custom icons for *Utopia* and *Cimmeria* referencing the artistic style of the icons that are already in the game. After some hours of tracing and getting ideas down in *CorelDRAW*, I came up with these:
![Utopia & Cimmeria Milestone Icons](/assets/images/michumod_milestone_icons_utopia_cimmeria.jpg)
*Utopia & Cimmeria Milestones*

![Utopia & Cimmeria Award Icons](/assets/images/michumod_award_icons_utopia_cimmeria.jpg)
*Utopia & Cimmeria Awards*

I exported the icons in *png* format (with transparency) and wrote a little helper method to **generate Unity *Sprites* from external files**:
```csharp
public static Sprite LoadSpriteFromFile(string filePath)
{
    if (!File.Exists(filePath))
    {
        MelonLogger.Error($"File not found at path: {filePath}");
        return null;
    }

    byte[] fileData = File.ReadAllBytes(filePath);

    Texture2D tex = new Texture2D(2, 2, TextureFormat.RGBA32, true);
    if (tex.LoadImage(fileData))
    {
        tex.filterMode = FilterMode.Bilinear;
        tex.mipMapBias = -0.1f;
        tex.wrapMode = TextureWrapMode.Clamp;
        tex.anisoLevel = 0;
        return Sprite.Create(tex, new Rect(0, 0, tex.width, tex.height), new Vector2(0.5f, 0.5f));
    }
    else
    {
        MelonLogger.Error($"Failed to load texture from file: {filePath}");
        return null;
    }
}
```

I had to play around with `.filterMode`, `mipMapBias`, and `anisoLevel` a bit until the custom icons were as sharp/blurry as the original ones.

Each returned *Sprite* was assigned to a `TM_MilestoneDataInfo_UI` or `TM_AwardDataInfo_UI` object and passed to the new `TM_BoardDataSet` *ScriptableObject* at runtime.

To make sure that everything displays correctly (*Milestone* **title**, **description**, and **icon**), I had to do some extra patching of UI related methods, such as `UpdateRequirements`, `SetMilestoneTooltipInfo`, and `SetAwardTooltipInfo`.

This is how the custom *Milestones* and *Awards* appear in the game:
![Utopia & Cimmeria Milestones/Awards in game](/assets/images/michumod_milestones_awards_in_game.png)
*Custom Milestones & Awards in-game*

There is still some UI related work to do:
1. Design and add icons for the rest of *Milestones* and *Awards* (*Amazonis* & *Vastitas*).
2. Design and add icons for the custom boards (like *Hellas* & *Elysium* when selecting them in the game creation menu).
3. Understand how *tooltips* work for *named locations* and make them work for the newly added **volcanic areas**.

*Utopia* & *Cimmeria* are **100% playable** now.

In the next post I will probably explain **how I added Energy as an available *Bonus resource* on *Utopia*** and **how I added new *volcanic areas* on *Cimmeria*, *Amazonis*, and *Vastitas***.

I will also record and share some gameplay to showcase the mod.