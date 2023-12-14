---
title: "SPY vs SPY - UE5 Remake of an 80s Classic"
date: 2023-11-29T08:28:24+10:00
draft: false
tags: ["unreal engine", "c++", "blueprints", "unreal editor", "UE5", "spyvsspy", "nes"]
layout: "Projects"
weight: 5
cover:
  image: "spy-bar.png"
  hidden: false
---

{{< youtube icn96Uj7HN0 >}}

### Brief
A personal project to remake the 80s classic SPY Vs Spy by which was a two player split screen competitive game.  This project is a redevelopment using modern game development levering Unreal Engine 5 and allows for online multi-player head to head play. A gameplay overview view can be found here: https://www.youtube.com/watch?v=icn96Uj7HN0
### Role
The project is a solo development effort thus far covering game design, systems architecture, development, and asset work unless noted below in the Third Party Attribution section.
### Summary of Elements
The gameplay element sees the Spy frantically searching rooms and environments for spy materials and weapons before making their hasty exit. The only thing in their way is the opposing spy who is looking to do the same. So ensues a tactical game of placing various traps in parts of the rooms where the other Spy is likely to look for spy materials, and arming yourself for the inevitable moment where both Spies burst into the same room.

Features include:

* Online head to head gameplay using server/client architecture.
* Rooms and world elements dynamically appear and disappear upon Spy Entry / Exit to heighten intensity of gameplay.
* Advanced Materials and Material functions enable Room vanishing warp effects to align with the direction of player travel.
* An item and inventory system for weapons, traps, and mission materials which allow for both players and furniture (such as desks, cabinets, etc...) to be pilfered by the players.
* Network optimised combat system yielding faithful hit detection even for players with high ping (Testing included online play with 600ms delta).
* Dev tooling to allow game designers to create new weapons and traps using Data Assets and not have to perform any programming.
* Dev Tooling hooks into player animation state transitions to allow for codeless customisation of hit detection zones and attack timings.
* Weapon and trap assets also allow for custom visual / Sound FX and Hit / Death animation sequences without having to modify any other assets.
* Character Spy model features Chaos Physics for cloth simulation on Spy coats to add to the immersion of the game.
* A deterministic seeding system per map handles random item placement across the world without needing to replicate data between server and clients or leverage Remote Procedure Calls.
* A Network manager system optimises state replication in a unified manner to remove network delays from having numerous actors as part of the replication graph, thus increasing network performance.


### Third Party Attribution
Assets:
* LukHash(https://www.lukhash.com/) - Remix of SPY vs SPY (1984) by Nick Scarim - Music Video (https://www.youtube.com/watch?v=sVtJGobSZKM)
* Google - Work Sans Font
* Epic Games - Manny Skeletal Mesh & Animations
* Finlay Pearston - Brutalist Pack: various static meshes and materials
* Michal Cavrnoch - Gangster Hat Static Mesh and Material
* A-Mod Studio - EP Master Materials 
* Blender3d(On Sketchfab) - Simple C4-Bomb Static Mesh and Material
* JPG(On Sketchfab) - Prom suit
* Digital Dressmaker(On Sketchfab) - 1830s Frock Coat
* Deoking - Punch2 Sound Recording
* Derplayer - Explosion_big_01 Sound Recording
* Ellbellekc - Step Sound Recording
* Ihitokage - Swish5 Sound Recording
* Macsat-rd - Greasy Bacon Sound Recording

Special thanks to Cliff Sharif and friends at CG Spectrum for love and support and specifically Anu and Pace for support and play testing.  Thanks goes you to Tranek, Tech Art Aid, Ryan Shmidt, and Chris Murphy for having immensely helpful learning material content published online which aided me in this work.