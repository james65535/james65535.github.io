---
title: "Dynamic Room Generator for the Unreal Engine"
date: 2023-06-29T08:28:24+10:00
draft: false
tags: ["unreal engine", "c++", "blueprints", "unreal editor", "UE5"]
layout: "Projects"
weight: 5
cover:
  image: "drgen-euw.png"
  hidden: false
---

{{< youtube WAtI9r5XojM >}}

### Brief
To Produce an extensible Map generator for the Unreal Engine with a focus on creating Rooms and related interior/exterior elements.  The tool must be usable by designers who have little to no programming experience.  Additionally, the tool must be easily portable from one project to the next.
### Role
I designed and implemented this tool for work on personal game projects but also to publish on the [Unreal Engine  Marketplace](https://www.unrealengine.com/marketplace/).
### Summary of Elements
This Plugin allows game designers to quickly add and modify Rooms and other level content  on the fly without any C++ or Blueprint programming. With a click of a button or movement of the Gizmo widget, rooms can dynamically resize meshes, make doorways, and place furniture instantly. The Plugin has been developed in C++ but can be extended with Blueprints.

* Rapid Creation of Rooms with auto-sizing walls, meshes, and materials

* Simple Editor Utility for rapid  placement or dynamically adjust each room directly

* Packaged as a plugin with everything needed to run out of the box

* Toggable Doorways with dynamic dimensions for which Data Assets can be used to specify what the door should be and how it should look

* Ability to have Room Themes and Furniture presets using Data Assets

* Easy to extend as Objects and code have been developed to make use of Interfaces and does not hard code any specific classes.

* Magnetised Rooms which can instantly snap to other rooms around it after being resized
