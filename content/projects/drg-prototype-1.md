---
title: "Dynamic Room Generator for the Unreal Engine"
date: 2023-06-29T08:28:24+10:00
draft: false
tags: ["unreal engine", "c++", "blueprints", "unreal editor", "UE5", "plugin"]
layout: "Projects"
weight: 5
cover:
  image: "drgen-euw.png"
  hidden: false
---

{{< youtube WAtI9r5XojM >}}

### Brief
To produce an extensible Map generator for the Unreal Engine with a focus on creating Rooms and related interior/exterior elements.  The tool must be usable by designers who have little to no programming experience.  Additionally, the tool must be easily portable from one project to the next.
### Role
I designed and implemented this tool for work on personal game projects but also to publish on the [Unreal Engine  Marketplace](https://www.unrealengine.com/marketplace/).
### Summary of Elements
This Plugin allows game designers to quickly add and modify Rooms and other level content on the fly without any C++ or [Blueprint](https://docs.unrealengine.com/5.2/en-US/introduction-to-blueprints-visual-scripting-in-unreal-engine/) programming. With a click of a button or movement of the Gizmo widget, Rooms can dynamically resize meshes, make doorways, and place furniture instantly. The Plugin has been developed in C++ but can be extended with Blueprints.

* Rapid Creation of Rooms with auto-sizing walls, [Dynamic](https://docs.unrealengine.com/5.2/en-US/API/Plugins/ModelingComponents/UOctreeDynamicMeshComponent/) and [Static](https://docs.unrealengine.com/5.2/en-US/static-meshes/) meshes, and materials

* Use the simple Editor Utility for rapid placement or dynamically adjust the properties of each Room directly through the Room's [Details](https://docs.unrealengine.com/5.2/en-US/level-editor-details-panel-in-unreal-engine/) panel.

* Packaged as a plugin with everything needed to run out of the box including example profiles setup as [Data Assets](https://docs.unrealengine.com/5.2/en-US/asset-management-in-unreal-engine/)

* Toggable Doorways with dynamic dimensions for which Data Assets can be used to specify what the door should be and how it should look

* Ability to have Room Themes and Furniture presets using Data Assets

* Easy to extend as Objects and code have been developed to make use of Unreal Engine's [Interface API](https://docs.unrealengine.com/5.2/en-US/interfaces-in-unreal-engine/) and does not hard code any specific class inheritance requirements other than the Interface.

* Magnetised Rooms which can instantly snap to other Rooms around it after being resized
