---
title: "Tantrumn - A Multi-Player Network Game"
date: 2023-06-230T08:28:24+10:00
draft: false
tags: ["unreal engine", "c++", "blueprints", "game", "UE5"]
layout: "Projects"
cover:
  image: "tantrumn-bar.png"
  hidden: false
---

### Brief
Tantrumn is a racing game (on foot, not car) developed as a solo school project which is part of the [CGSpectrum](https://www.cgspectrum.com/) Game Programming Curriculum.  The game was designed to accommodate single player, multi-player split view, and networked multi-player modes.  The player(s) can compete against each other or AI bots.
### Role
The design and guidance was provided by the CGSpectrum curriculum along with code examples.  My role was to implement the game with scope to make any changes I deemed fit.  Changes I introduced spanned the gamut of Unreal Engine capabilities such as a custom [Character Movement Component](https://docs.unrealengine.com/5.2/en-US/understanding-networked-movement-in-the-character-movement-component-for-unreal-engine/) (detailed below), level layouts, Mesh / Audio / Material assets, AI behaviour, and UI elements.  The implementation was done primarily in C++ with [Blueprints](https://docs.unrealengine.com/5.2/en-US/introduction-to-blueprints-visual-scripting-in-unreal-engine/) used for prototyping functionality, scripting some aspects of [Widgets](https://docs.unrealengine.com/5.2/en-US/creating-widgets-in-unreal-engine/) used as part of the UI, and level specific gameplay elements. 
### Summary of Elements
The game includes player levels, [Kinematics](https://en.wikipedia.org/wiki/Kinematics) test levels, custom [Game Modes](https://docs.unrealengine.com/5.2/en-US/game-mode-and-game-state-in-unreal-engine/) which can match gameplay to the corresponding levels.  All gameplay supports network play whether via dedicated server and clients, or local LAN play.  Detail on additional implementations included in the game as follows:

* The custom Character Movement Component extends Epic's implementation to include additional modes of travel.  These modes are efficiently replicated via [Bit Flags](https://en.wikipedia.org/wiki/Bit_field) by extending the Network Prediction system used for lag compensation (reducing the [rubberbanding](https://www.dictionary.com/browse/rubberbanding) effect prevalent in many older online games), specifically [FSavedMove](https://docs.unrealengine.com/5.2/en-US/API/Runtime/Engine/GameFramework/FSavedMove_Character/). The end result being minimal [Server Corrections](https://docs.unrealengine.com/5.2/en-US/understanding-networked-movement-in-the-character-movement-component-for-unreal-engine/#customizingnetworkedcharactermovement) for client side movement predictions.


* Impacts have the ability to stun characters (players and AI).  The magnitude of impact determines the duration of stun and strength of controller vibration. Additionally, the stun state controls animation behaviour by triggering corresponding [Animation Montages](https://docs.unrealengine.com/5.2/en-US/animation-montage-in-unreal-engine/).

### Third Party Attribution
* Character Model and majority of character animations come from the Unreal Engine [Hour of Code](https://www.unrealengine.com/marketplace/en-US/product/unreal-engine-hour-of-code) pack
* Some snippets of code from CGSpectrum's example content has been re-utilised in this implementation
* Some Meshes and Materials from Unreal's [Starter Content Pack](https://docs.unrealengine.com/5.2/en-US/assets-and-content-packs-in-unreal-engine/) have been utilised.
