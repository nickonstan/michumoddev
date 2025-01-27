---
layout: post
title: "Terraforming Mars Board Generator"
date:   2025-01-26 13:10:00 -0300
---

I believe that a new board, with all the variety it introduces, breathes new life into Terraforming Mars. That's why *Hellas & Elysium* is one of my favorite expansions to the original board game and why I was super excited about *Utopia & Cimmeria* and *Amazonis & Vastitas* when they were announced. I thought it would be cool to be able to enjoy these boards on the digital version of the game since I'm spending disproportionately more hours playing *TM* online, saving the longer, 4-player offline sessions for every other weekend. Even though there is an official DLC for *Hellas & Elysium*, it's unlikely that we'll be seeing Utopia, Cimmeria, Amazonis, and Vastitas any time soon. So I set out on a journey to create my own mod to make it possible, learning a whole lot in the process.

The mod is still in development. I'm going to share more about the process in future posts.

For the rest of this article, when referring to *the game*, I'm referring to the digital version of *Terraforming Mars*, published by Asmodee Digital.
## JS-based Board Generator
Today I want to showcase an external, web-based tool I made to aid in the creation of custom boards. The idea was to be able to create fully customized boards, and export them in a format that the game understands. The tool uses *Honeycomb* and *SVG.js* to calculate and render the hex grid. Everything else is vanilla *HTML*, *CSS*, and *JS*. To understand the inner workings of the game, I used the *dnSpy* .NET debugger and assembly editor.
![TM Board Generator](/assets/images/tmboardgen_overview.jpg)
### Features
This is a list of already implemented features:
- Create hexagonal grids/boards of **various sizes**.
- Set tiles as reserved for **oceans**.
- Assign and remove **bonus resources** by drag-and-dropping.
- Set tile **costs** (M€ the player needs to pay to be able to place a tile on that slot).
- Mark tiles as **volcanic areas**.
- Place resources (**Microbe**, **Animal**, **Floater**, **Science**, **Asteroid**) and bonuses (**Oxygen**) not used (yet) in official boards.
- Save boards locally or load previously saved boards.
- Export boards as a *JSON* file that the game can read.

I'll probably implement new features as more ideas surface.
### Hex Grid Size
![Hex Grid Section](/assets/images/tmboardgen_grid_size_section.jpg)
**Size**, both in-game and in my tool, refers to the **number of hexes** on the equator row. All official boards have a size of 9, except *Amazonis* that has a size of 11. Changing the size constructs and renders a new grid.
### Show Coordinates & Show TileIDs
![Coordinates & TileIDs](/assets/images/tmboardgen_coordinates_tileids.jpg)
This is functionality I added to help me troubleshoot the export-import process. *Show Coordinates* renders the **cube coordinates** of each tile. These coordinates are stored in the `TilePosition` attribute of each tile in-game. There is a [fantastic article by Red Blob Games](https://www.redblobgames.com/grids/hexagons/) on hexagonal grids that helped my understanding a lot; check it if you want to learn more. `TileID` is a number identifying a tile in game. The tile with `TileID: 1` is the leftmost one on the top row. A size-9 board has **61** tiles in total.
### Ocean or not?
![Ocean Placement](/assets/images/tmboardgen_ocean_placement.jpg)
To mark a tile as reserved for oceans **right-click** on it. Clicking it again will revert it to a normal tile. In game this is stored in the `EGridTileType` *enum*, with value `0` meaning "Normal", and value `1` meaning "Ocean".
### Resources/Bonuses
![Resource/Bonus Assignment](/assets/images/tmboardgen_bonus_assignment.jpg)
*Note: I consider Steel, Titanium, Heat, etc. to be resources, while "raise Temperature", "place Ocean", etc. to be bonuses. The game doesn't make a clear distinction between them so I'll refer to all of them as simply bonuses*

Adding bonuses to the board is straightforward: just **drag** and **drop** them on the tile you want. Each tile has a capacity of 3 bonuses at most (the *tile cost* "anti-bonus" counts as one). To remove a bonus **left-click on it**. **Refreshing the page removes all resources**.

In-game bonuses are stored in the *enum* `EBonusType`, so each bonus corresponds to a numeric value:
```csharp
Card = 1,
Steel = 2,
Titanium = 3,
Plant = 4
```
Each tile has a `Bonuses` *List* that holds these numeric values. If, for example, I want a tile to have 2 *Steel* and 1 *Plant*, I should set `Bonuses` to `[2, 2, 4]`.
### Location Names
![Location Name Assignment](/assets/images/tmboardgen_location_name_assignment.jpg)
You can set a tile as a **volcanic area** by right-clicking it and selecting the desired name from the dropdown menu. Choosing "None" reverts the tile back to a normal one.

Like tile type and bonuses, location names are stored in an *enum* (`EGridTileLocationName`). The game makes heavy use of *enums* in general. "Extending" them at runtime has proven to be a real pain in the ass but more on that in the post(s) regarding the mod.
### Randomizer
![Randomizer Section](/assets/images/tmboardgen_randomizer_section.jpg){: .no-wrap-image}
The tool features a **randomizer** for spreading oceans and bonus resources. 
#### Randomizing Oceans
You can choose the **minimum** and **maximum** amount of tiles reserved for oceans to be set by the randomizer. Official boards (besides *Amazonis*) have 12 ocean slots (9 for ocean tiles and 3 for special, ocean-only projects, such as *Mohole Area*). This is something to keep in mind when designing a board. Nevertheless, I thought it would be fun to experiment with boards that have more (or less) ocean slots, as long as they can be fully terraformed.
#### Randomizing Resources
You can tell the randomizer that you don't want a resource included on the board by clicking on its respective icon. The icon will turn grey to reflect that. The randomizer doesn't place *Ocean*, *Oxygen*, and *Temperature* bonuses, neither assigns *cost* to tiles. There is a logic **based on weights** that affects two things:
1. The proportion of tiles with **0** (50%), **1** (30%), **2** (17%), or **3** (3%) resources.
2. The probability of appearance of each bonus (some, such as *Plants*, are more common than others).

I chose the weights after studying the the bonus distribution of the official boards. The idea is for the randomizer to churn out a semi-balanced board to act as a starting point for further manual editing. There is no way to change the weights from the UI at this moment, but I may introduce one in the future.

### More ideas
- There are still some expansion-dependent bonuses to add, such as the *Colony* (*Colonies*) and the *Delegate* (*Turmoil*). Implementing the mechanics of the expansions needed for these bonuses to make sense would be an immensely difficult undertaking, so I'm going to skip them for now.
- Find a way to change the Mars texture that acts as a background for the grid based on the board used. This is something that the game does already for *Tharsis*, *Hellas* and *Elysium*.

Using the board generator in conjunction with the mod (in its current state) I was able to create fully playable custom boards, which is a promising start.