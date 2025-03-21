---
layout: post
title: "Implementing Amazonis: Part 1"
date: 2025-03-21 17:20 -03:00
categories: ["Terraforming Mars"]
---

*Amazonis* is the first *Terraforming Mars* board that features a **91-tile hexagonal grid** instead of the typical 61-tile one. After my experience with *Utopia*, *Cimmeria*, and *Vastitas*, I thought that adding this board to the mod would be a simple matter of passing an extended JSON file with the data to a new `BoardDataSet`, set the `GridSize` field correctly, and that the game would handle the rest. **I was wrong**.

## Not enough GameObjects
The game wouldn't complain about the surplus of tile data passed to it, but the initialized board had space for only 61 tiles, at least visually. I noticed that the resources/bonuses of the tiles with IDs 62 to 67 would **spread over to the slots used for *Phobos Space Station*, *Ganymede Colony*, etc.**. After deeper inspection of the source code, I found out that the game **doesn't create new *GameObjects* to hold tile data dynamically**. Rather, the main game scene is set up only for *size-9* board in mind.

(*Size* refers to the number of hexes on the middle row/equator. Watch my [custom board design video](https://youtu.be/oRsXJ-rWG3s) for more info.)

## Inner workings of a board
The two classes responsible for the construction of the visual aspect of the board are `HexagonalBoard` and `HexagonalBoardTile`. `HexagonalBoardTileContainer` is a child *GameObject* of `HexagonalBoard` that holds `HexagonalBoardTile` entities. My initial idea was to create **clones of the tile on the center of the board** (TileID = 31), use custom logic to move the new objects around to form an extended, 91-tile board, and then delete all original *GameObjects*. I had to make sure to keep *Phobos, Ganymede, Dawn City*, etc. slots intact.

I wrote a `HarmonyPatch` to handle the initialization of `HexagonalBoard` only when *Amazonis* was chosen as the board and...

**It worked!** Now I had 91 tile slots that were properly populated with the tile data from my JSON file. There was an issue where the tiles were **unclickable** but I soon found out it was because all tiles were inheriting the `MeshCollider` of the original tile I cloned, together with its (wrong) `Position` value. It was solved easily by **rebuilding the `MeshCollider`** after initialization:
```csharp
// Rebuild the MeshCollider
MeshCollider meshCollider = newTileObj.GetComponent<MeshCollider>();
if (meshCollider != null )
{
    Mesh originalMesh = meshCollider.sharedMesh;
    meshCollider.sharedMesh = null;
    meshCollider.sharedMesh = originalMesh;
}
else
{
    MelonLogger.Warning($"TileID {tileID} has no MeshCollider!");
}
```

## Visual depth
There was still something that was bothering me: since all new tiles were clones of a single original tiles, **they shared the same `position.z` value**, which means they appeared to be positioned on the same plane. The game does a good job of introducing **depth** by arranging the tiles on the surface of a hemisphere that represents *Mars*. Tiles close to the center have greater `position.z` values and appear closer to the screen, while tiles on the border have smaller values and appear farther. There is also some kind of **rotation and scaling** going on that creates the impression that the face direction of each tile aligns with the *normals* of the hemispherical *mesh*. I wanted to mimic that. The `rotation`/`localRotation` and `scale`/`localScale` values of the original tiles were `0` and `1` accordingly, so I assumed that the *transformations* were already applied and I couldn't just copy them.

## A better solution
My solution was to **create a mapping between original and new tiles** and use that **to instantiate *GameObjects* based on their position on the board**. The new central tile would be a clone of the original central tile, and a new border tile would be a clone of the original border tile closest to it, inheriting the transformations needed to create that elusive sense of depth. An 91-tile board has an **extra outer ring** so I decided to clone each original border tile twice (or thrice where needed for symmetry):

![Amazonis HexTile Cloning Logic](/assets/images/amazonis_board_tile_cloning.png)
*Hex tile cloning logic for Amazonis*

Here you can see how the board looks in game right now:
![MichuMod: Amazonis Board](/assets/images/michumod_amazonis_board_wip.jpg)
*Amazonis Planitia in TM digital (WIP)*

The depth of the tiles on the outer ring still needs some adjustment but, visually, I'm almost there.

## Moving on
Even though the board is playable, there are still a lot of features that I need to implement to make it true to the original:
1. Update the *Global Parameter* requirements to finish the game (more *oceans*, extended *oxygen* and *temperature* meters).
2. Add the new bonuses: *Plant* production on the *temperature* track, *Card* on the *oxygen* track.
3. Come up and implement a replacement for the *Lobbyist* *Milestone* (that requires the *Turmoil* expansion).

## You thought it was over?
In the next post I will touch on **vertex colors** and explain how the game handles *visual effects* when placing tiles, and more specifically when placing *oceans* and *greeneries*. There is a direct connection between these visual effects and **the underlying Mars mesh/3D model** that makes creating smaller of bigger boards a much greater challenge.