---
layout: post
title: "MichuMod: Programming Custom Milestones | Part 1"
date: 2025-02-05 10:40 -03:00
categories: ["Terraforming Mars"]
---

Programming and "injecting" custom *Milestones* into the game has been one of the most difficult tasks up until now. There are different mechanisms that handle each aspect of a *Milestone* (requirements, association to a board, progress calculation, UI representation) scattered throughout the source code. Fortunately, *Awards* are handled in a very similar manner, so getting one to work meant that the other would work as well; I just had to repeat the process.

Like boards, *Milestones* are stored in an *enum* (`EMilestoneType`), which is impossible to extend at runtime. I could have just edited the original *enum*, recompile, and export the assembly using *dnSpy*. This method would have an advantage: each method that handles the `EMilestoneType` type would "be aware of" and be able to handle my custom *Milestones* as well, reducing the need for extensive patching. However, every future update would overwrite the edited assembly, erasing all custom edits done to it, and rendering the mod unusable. Since *Terraforming Mars* is a game played mainly online, updates are more frequent than a single player title. I had to find a way to extend *enums* on the fly.

## Enum Extension System
After some research, I came across this method of faking *enum* functionality using a custom *class* and *Dictionaries*:
```csharp
public static class MichuEMilestoneType
{
    private static readonly Dictionary<string, EMilestoneType> _values = new Dictionary<string, EMilestoneType>();
    private static readonly Dictionary<EMilestoneType, string> _names = new Dictionary<EMilestoneType, string>();
    private static readonly Dictionary<long, EBoardType> _milestoneBoards = new Dictionary<long, EBoardType>();
    private static long _nextValue;
}
```
The two first *Dictionaries* store the relation between a *Milestone* name and its value, much like what an *enum* does. The original *enum* uses values that are **powers of 2**. This is because various methods that handle the logic behind *Milestones* use **bitwise operations**. Initially, I tried following that path but adding 20 *Milestones* (5 for each new board) made the value grow too fast and **overflow**. An *int* based *enum* can store values only up to `Int32.MaxValue`, which is `2,147,483,647`. This allows **up to 31 Milestones**, with 15 already existing in the game. So I decided to use consecutive values and write custom logic where needed. I quickly came into a standstill: counting the progress of each player and showing the *Milestones* in the UI was working, but purchasing was a mess. The game is using a *bitmask* to store and count purchased milestones and awards, so using values that are not powers of 2 was breaking it. Patching the 10+ methods using the `PurchasedAchievements` value to work with my not-bitwise-friendly *Milestones* was too much work and had many possibilities of introducing new errors. In the end, I came up with an alternative that uses powers of 2 but respects the *int* limitation of the original *enum*. Instead of "registering" all custom *Milestones* at the beginning of the game, my system registers only 5 at a time **depending on the selected board**. If another board is selected, the previous custom *Milestones* are "unregistered" (that is, deleted from the *Dictionary* shown above), and the 5 ones that are associated with the new board are registered in their place. This is possible **only because I'm introducing new data at runtime** (versus it being predefined from the start, like the original *Milestones*). This is an example of the *Milestones* (and their values) used for *Utopia Planitia*:

|  Milestone   |   Value | Board  |
| :----------: | ------: | ------ |
|   Manager    |   65536 | Utopia |
|   Pioneer    |  131072 | Utopia |
|    Trader    |  262144 | Utopia |
| Metallurgist |  524288 | Utopia |
|  Researcher  | 1048576 | Utopia |

When the user selects *Cimmeria* from the *Game Creation Menu*, the *Milestones* above are unregistered, and these take their place:

|   Milestone   |   Value | Board    |
| :-----------: | ------: | -------- |
| Planetologist |   65536 | Cimmeria |
|   Architect   |  131072 | Cimmeria |
|  Coastguard   |  262144 | Cimmeria |
|   Forester    |  524288 | Cimmeria |
|  Fundraiser   | 1048576 | Cimmeria |

## Registering new Milestones
The most important method of the custom class is the one used to **register new Milestones**:
```csharp
public static EMilestoneType RegisterMilestone(string milestoneName, EBoardType board, string descriptionText)
{
    if (_values.ContainsKey(milestoneName))
        MelonLogger.Error($"EMilestoneType {milestoneName} already exists");

    var value = (EMilestoneType)_nextValue;
    _nextValue <<= 1;

    _values[milestoneName] = value;
    _names[value] = milestoneName;

    _milestoneBoards[(long)value] = board;
    _milestoneDescriptions[value] = descriptionText;
    return value;
}
```
`_nextValue <<= 1;` makes sure that each *Milestone* has a **value that is double that of the previous one**, to maintain bitwise compatibility.

For example, to register the *Manager* milestone, I have to do this:
```csharp
MichuEMilestoneType.RegisterMilestone(
    "Manager",
    MichuEBoardTypes.Utopia,
    "Have 3 special tiles in play."
    );
```
The first argument here is the name of the *Milestone*, the second its **associated board** and the third the description that appears in the UI (more on this in another post).

The `MichuEMilestoneType` class comes with a number of smaller utility methods for checks (`IsValid()`, `HasFlag()`) and data retrieval (`GetAllCustomMilestones()`, `GetAssociatedBoard()`, etc.). For the time being, I will not get into each and every one of them.

In the next post, I will talk about the part of the code that **keeps track of the progress of each player towards a Milestone** and the checks before they're allowed to buy them.