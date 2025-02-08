---
layout: post
title: "MichuMod: Programming Custom Milestones | Part 2"
date: 2025-02-08 14:05 -03:00
categories: ["Terraforming Mars"]
---

This is a continuation of [Part 1]({% post_url 2025-01-31-programming-custom-milestones %}).

Today I'm going to explain how the game handles the **progress of each player towards a *Milestone* or *Award***.

## Milestone and Award data basics
There is a "base" *ScriptableObject* called `TM_GameDataSet` that is essentially a collection of all the data that needed for a TM game, such as data related to the boards (`BoardDataSets`), the cards (`CardDataSet`), the initial *Terraforming Rating* (`InitialTerraformingRating`), and more. *Milestones* and *Awards* **are part of the board data**. When a new `TM_BoardDataSet` (which is itself a *ScriptableObject*) is created at the beginning of the game, there are two related objects passed to it:
1. `TM_AchievementDataInfo` and
2. `TM_AchievementDataSet_UI`

`TM_AchievementDataInfo` stores the **cost of *Milestones* (8, 8, 8) and *Awards* (8, 14, 20)**, the **points won each time a player buys one**, the `MilestoneType` of each *Milestone* and its `RequirementValue`. The `MilestoneType` is an *int* and refers to the value each *Milestone* has in the `EMilestoneType` *enum*. The `RequirementValue` defines **how much of something you have to have to be able to buy the *Milestone***. Yes, it's that abstract.

`TM_AchievementDataSet_UI` stores the connection between *Milestone/Award* values **and their visuals**, that is the icon/sprite that appear for its one in the game.

Since both of these objects, as well as their parent object, are *serializable*, it's straightforward to load the kind of data needed **from JSON files**. This is what the mod does: the **board/grid data**, ***Milestone* and *Award* data**, and even **the type and cost of *Standard Projects*** are loaded from external JSON files, easily editable in a text editor. When it comes to board/grid data, I've made an external tool that helps with the creation of these JSON files. I've already talked about it in [another post]({% post_url 2025-01-26-terraforming-mars-board-generator %}).

## And the actual logic?
As we can see, the logic behind *Milestones* and *Awards* is not included in the board data. Sure, we understand that the *Mayor Milestone* requires **3 of something** by looking at its `RequirementValue`, but **3 of what?**.

The logic is stored in the `AchievementHandler` class and is handled by the `GetPlayerMilestoneProgressValue` (*Milestones*) and `GetPlayerAwardProgressValue` (*Awards*) methods. These are triggered for every *Milestone* and *Award* in the current game and **return an *int* that indicates the player's progress**. For example, the *Researcher Milestone* requires the player to have 4 *Science Tags* in play. `GetPlayerMilestoneProgressValue` will go through each player's tags, count them, and return the value. That number is then compared to the `RequirementValue` that we saw earlier (which is 4 in this case). If the player meets the requirements, the *Button* for *Researcher* will activate allowing him/her to buy the achievement.

Having the original *Milestone* and *Award* logic as reference, I wrote a `HarmonyPatch` that hooks onto these methods and calculates the progress on the *Milestones* of *Utopia*, *Cimmeria*, *Amazonis*, and *Vastitas*. These are some examples:
### Milestones
#### Trader
```csharp
case "Trader":
    HashSet<EResourceType> uniqueResources = new HashSet<EResourceType>();

    foreach (KeyValuePair<int, TupleLH<EResourceType, int>> keyValuePair in playerBoardData.ResourcesOnCards)
    {
        uniqueResources.Add(keyValuePair.Value.Item1);
    }

    __result = uniqueResources.Count;
    return;
```
#### Planetologist

```csharp
case "Planetologist":
    int earthTags = playerBoardData.TagBank.GetTagQuantity(ETagType.Earth, false);
    int venusTags = playerBoardData.TagBank.GetTagQuantity(ETagType.Venus, false);
    int jovianTags = playerBoardData.TagBank.GetTagQuantity(ETagType.Jovian, false);
    int wildTags = playerBoardData.TagBank.GetTagQuantity(ETagType.Wild, false);

    __result = Mathf.Min(earthTags, venusTags, jovianTags, wildTags);
    return;
```

#### Geologist
```csharp
case "Geologist":
    HashSet<int> nearOrOnVolcanicTiles = new HashSet<int>();
    List<EGridTileType> allTileTypesGeologist = ((EGridTileType[])Enum.GetValues(typeof(EGridTileType))).ToList<EGridTileType>();

    foreach (TM_GridSlotTileData volcanicAreaSlot in boardData.GetNamedSlots(aPlayerLocalId, EGridTileLocationName.VolcanicArea))
    {
        int volcanicTileID = volcanicAreaSlot.Infos.TileID;

        List<TM_GridSlotTileData> playerPlacedTiles = boardData.GetPlacedTiles(aPlayerLocalId);
        if (playerPlacedTiles.Any(tile => tile.Infos.TileID == volcanicTileID))
        {
            nearOrOnVolcanicTiles.Add(volcanicTileID);
        }

        foreach (TM_GridSlotTileData adjacentTile in boardData.GetOwnedTilesInRange(volcanicTileID, allTileTypesGeologist, aPlayerLocalId, 1, true))
        {
            nearOrOnVolcanicTiles.Add(adjacentTile.Infos.TileID);
        }
    }
    __result = nearOrOnVolcanicTiles.Count;
    return;
```
### Awards
#### Zoologist
```csharp
case "Zoologist":
    __result = playerBoardData.CountResourceOnAllCard(EResourceType.Microbe) +
               playerBoardData.CountResourceOnAllCard(EResourceType.Animal);
    return;
```

#### Manufacturer
```csharp
case "Manufacturer":
    __result = playerBoardData.ResourceBank[EResourceType.Steel].ProductionQuantity +
               playerBoardData.ResourceBank[EResourceType.Heat].ProductionQuantity;
    return;
```

#### Landscaper
This was one of the trickiest ones.
```csharp
case "Landscaper":
    List<TM_GridSlotTileData> playerOwnedTiles = boardData.GetPlacedTiles(aPlayerLocalId);
    HashSet<int> visitedTiles = new HashSet<int>();
    int largestPatchSize = 0;

    foreach (TM_GridSlotTileData tile in playerOwnedTiles)
    {
        int tileID = tile.Infos.TileID;

        if (visitedTiles.Contains(tileID))
            continue;

        // Perform Breadth-First Search to find the size of this connected component
        int patchSize = MichuBoardUtils.GetConnectedTileGroupSize(tileID, playerOwnedTiles, visitedTiles, boardData);
        largestPatchSize = Mathf.Max(largestPatchSize, patchSize);
    }

    __result = largestPatchSize;
    return;
```

At this point I had fully functioning *Milestones* and *Awards* within the game. After implementing the "register/unregister" system that I described in [Part 1]({% post_url 2025-01-31-programming-custom-milestones %}), there was no need to patch any of the methods that deal with buying and limiting the total amount of achievements that can be bought.

## Moving on
What was missing now was purely visual: I had to design some icons and fix how the custom *Milestones* appear in the *player's log*. More on this in a future post.











