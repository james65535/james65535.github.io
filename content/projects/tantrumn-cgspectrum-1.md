---
title: "Tantrumn - A Multi-Player Network Game"
date: 2023-06-230T08:28:24+10:00
draft: false
tags: ["unreal engine", "c++", "blueprints", "game", "UE5"]
layout: "Projects"
cover:
  image: "tantrumn.png"
  hidden: false
---

### Brief
Tantrumn is a solo school project which is part of the [CGSpectrum](https://www.cgspectrum.com/) Game Programming Curriculum.  The game was designed to accomodate single, split view, or networked multi-player racing game.  The player(s) can compete against each other or AI bots.
### Role
The design and guidance was provided by the CGSpectrum curriculum.  My role was to implement the game with scope to make any changes I deemed fit.  These changes include a Custom Character Movement Component (detailed below), level layouts, Mesh/Audio/Materiel assets, AI behaviour, and UI elements.
### Summary of Elements
The game includes player levels, designer kinematics test levels along with custom GameModes which can match gameplay to the corresponding levels.  All gameplay supports network play whether via dedicated server and clients, or local LAN play.  A Customer Character Movement Component was developed to extend Epic's imeplementation to include additional modes of travel.  These modes are efficiently replicated via Bit Flags by extending the Network Prediction system.  The end result being minimal server corrections for client side movement predictions.

* Rapid Creation of Rooms with auto-sizing walls, meshes, and materials
These changes include a Custom Character Movement Component to allow for additional modes of movement to be efficiently replicated across networked systems. 
* Simple Editor Utility for rapid  placement or dynamically adjust each room directly

* Packaged as a plugin with everything needed to run out of the box

* Toggable Doorways with dynamic dimensions for which Data Assets can be used to specify what the door should be and how it should look

* Ability to have Room Themes and Furniture presets using Data Assets

* Easy to extend as Objects and code have been developed to make use of Interfaces and does not hard code any specific classes.

* Magnetised Rooms which can instantly snap to other rooms around it after being resized
