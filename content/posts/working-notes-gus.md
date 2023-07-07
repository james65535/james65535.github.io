---
title: "Working Notes - Graphics Settings Menu in UE5"
date: 2023-07-07T15:11:30+10:00
tags: ["Graphics", "C++", "Screen Resolution", "UE5", "Widgets", "Blueprints"]
cover:
  image: "/posts/screenres/screenresuiunexpandedsm.png"
  hidden: false
draft: false
---

# Working Notes
*These 'Working Notes' blog entries are daily notes from engineering work I've done which I believe may be useful for others.  These notes are not meant to be best recommended practice and are subject to change over time as I develop new approaches*.

## Unreal Engine 5 - In Game Graphics Settings
Currently I'm working on a UI for a game and will need to present to the user a menu UI for graphics settings.  The following will provide information to help people get started in the right direction with using the native Game User Settings (GUS) system in UE.

We'll use C++ and Unreal Engine's reflection system to handle most of the work, and let Widget Blueprints sort out the presentation to the player.  Please note, I've omitted logging and failure case logic for the sake of brevity in these examples.

1. Create a INI file for GUS with the Game Config folder, example name: `DefaultGameUserSettings.ini`
2. Populate the file with the GUS header info and potentially any default settings you would like:
```
  [/Script/Engine.GameUserSettings]
  ResolutionSizeX=1024
  ResolutionSizeY=768
  FullscreenMode=2
```
3. We'll need to consider the configuration flow for the following aspects:

* Presenting Screen Resolution Information
* Applying Player Selected Screen Resolution
* Storing Game User Settings Containing Selected Screen Resolution
* Retrieving Game User Settings with Selected Screen Resolution

The Storing and Retrieving elements will allow us to save the settings across Game restarts and the like since this information will go to and come from disk.

4. Retrieve - Depending on your preference you can place the following code where it fits within your project.  This quick example uses the player's HUD and overrides BeginPlay.
```cpp
//GameHUD.h
protected:
	virtual void BeginPlay() override;
```
```cpp
// GameHUD.cpp Includes
#include "GameHUD.h"

#include "GameFramework/GameUserSettings.h"
#include "Kismet/KismetSystemLibrary.h"
```

```cpp
// GameHUD.cpp
void AGameHUD::BeginPlay()
{
	Super::BeginPlay();

	/** Load and validate Game User Settings for Graphics Options */
	UGameUserSettings* GameUserSettings = UGameUserSettings::GetGameUserSettings();
	/** Deserialise settings from Disk - Ex: DefaultGameUserSettings.ini */
	GameUserSettings->LoadSettings();
	/** Overwrite any garbage with default settings if need be */
	GameUserSettings->ValidateSettings();
}
```
5. Creating - We'll create a method within our HUD which can be used by Widget Blueprints where desired to populate a data structure with Screen Resolution options along with a friendly name which can be displayed to the user.
```cpp
// GameHUD.h
public:

	/**
	 * Sets the given Screen Resolution
	 * @param InScreenRes The IntPoint with which to set the Screen Resolution
	 * @param bConfirmed Set to true to store the desired setting to disk
	 */
	UFUNCTION(BlueprintCallable, Category = "Graphics Options")
	void SetScreenRes(FIntPoint InScreenRes, bool bConfirmed);
```

```cpp
// GameHUD.cpp
void AGameHUD::CreateScreenResOpts(UPARAM(ref) TMap<FString, FIntPoint>& ScreenResOpts)
{
	/** Use Kismet Library to retrieve a list of Screen Resolutions */
	TArray<FIntPoint> SupportedScreenResolutions;
	UKismetSystemLibrary::GetSupportedFullscreenResolutions(SupportedScreenResolutions);

	/** Clear out data container since we are unsure of what is in there */
	ScreenResOpts.Empty();

	/** Iterate over possible Screen Resolutions and populate the data container */
	for (FIntPoint SupportedScreenResolution : SupportedScreenResolutions)
	{
		/** Derive a KeyName which is human friendly, ex: '1024 x 768' */
		FString KeyName = FString::Printf(TEXT("%i x %i"), SupportedScreenResolution.X, SupportedScreenResolution.Y);
		/** Use Emplace instead of Add to overwrite duplicate keys just in case they occur */
		ScreenResOpts.Emplace(KeyName, SupportedScreenResolution);
	}
}
```
6. Applying - Next we'll create the method used by the Widget Blueprint to set the Screen Resolution. A couple things to note, if allowing for multiple Graphics Settings, it may be better to set this up with a Struct that gets passed in by reference with the desired settings.

```cpp
// GameHUD.h
public:
	/**
	 * Sets the given Screen Resolution
	 * @param InScreenRes The IntPoint with which to set the Screen Resolution
	 * @param bOverrideCommandLine Should the Game User Settings override conflicting command line settings
	 */
	UFUNCTION(BlueprintCallable, Category = "Graphics Options")
	void SetScreenRes(FIntPoint InScreenRes, bool bOverrideCommandLine);
```


```cpp
// GameHUD.cpp
void AGameHUD::SetScreenRes(FIntPoint InScreenRes,  bool bOverrideCommandLine)
{
	if(UGameUserSettings* GameUserSettings = UGameUserSettings::GetGameUserSettings())
	{
		GameUserSettings->SetScreenResolution(InScreenRes);
		/** We need to apply the settings before they take effect  */
		GameUserSettings->ApplyResolutionSettings(bOverrideCommandLine);
	}
}
```
7. Storing - We'll need to serialise the Game User Settings to disk so that players do not need to keep specifying their desired resolution each time they play the game.  This can used in conjunction with logic within the Widget to give a constrained time within which they need to 'confirm' the new setting so they don't get stuck with a resolution so bad they can't navigate the UI to set it back.

```cpp
// GameHUD.h
public:
	/**
	 * Stores the Game User Settings to Disk
	 * @param bOverrideCommandLine Should the Game User Settings override conflicting command line settings
	 */
	UFUNCTION(BlueprintCallable, Category = "Graphics Options")
	void ConfirmGameUserSettings(bool bOverrideCommandLine);
```
```cpp
// GameHUD.cpp
void AGameHUD::ConfirmGameUserSettings(bool bOverrideCommandLine)
{
	if(UGameUserSettings* GameUserSettings = UGameUserSettings::GetGameUserSettings())
	{
		GameUserSettings->ApplySettings(bOverrideCommandLine);
	}
}

```

8. With the C++ work done, we'll switch gears and work on the presentation layer using Widgets, and this time Blueprint, although c++ is equally valid.  First, we'll create a quick UI in the designer tab and use a ComboBox String to hold our Screen Resolution options.

![UI Widget for Screen Resolution Change](/posts/screenres/GUS-UIWidget.png)

9. Next we'll pop over to the Event Graph. To start, we'll create a few member variables and some helper functions.  Create the following variables:

* RevertTime\<float\>
* RevertTimerHandle\<Timer Handle\>
* PreviousScreenRes\<Int Point\>
* ScreenResolutions\<Map\<String, Int Point\>\>

10. First helper function `SetComboBoxSelectedOption`. This will save us duplicating code for each time the ComboBox value is changed.  You'll want to create a local variable `ScreenRes`\<String\n and an input variable `InSelectedScreenRes`\<String\>.

![Helper Function for Setting ComboBox Selected Item](/posts/screenres/populatecomboboxselectoptionfunc.png)

11. Second helper function `ConfirmScreenRes`, this will give the player failsafe if they select a bad screen resolution and cannot change back through the UI:

![Helper Function for Reverting to Previous Screen Res](/posts/screenres/populatecomboboxselectoptionfunc.png)

12. Create a function called PopulateScreenResComboBox.  We'll grab the HUD of the player and use our new methods to populate the ComboBox in the UI.  We'll also get the current screen resolution and set that as the default selected value of the ComboBox.  Screen shots are of the same function split over two pictures for the sake of clarity.

![Populate Screen Resolutions ComboBox Part 1](/posts/screenres/populatecomboboxfunc.png)

![Populate Screen Resolutions ComboBox Part 2](/posts/screenres/populatecomboboxfuncpt2.png)


13. We're almost done. Create another function called SetScreenResolution.  Set an input parameter for the function as InSelectedRes\<String\>.  We've added a timer here which will call the Revert Function upon expiration.  In a moment we'll add an event for On-Click for the confirm button which will destroy that timer and save the Game User Settings to disk.

![Set the Screen Resolution to Selected Value](/posts/screenres/setscreenresfunc.png)

14. In the designer view, go to the ComboBox and at the bottom of the Details pane, hit the binding next to `On Selection Changed`.

15. Also, in the designer view, go to the Confirm button and at the bottom of the Details pane, hit the binding next to `Clicked`.

16. Finally, Finally! In the Event Graph we wire up our funcs to three events:

* Event Construct -> PopulateScreenResComboBox
* On Selection Changed -> SetScreenResolution, Selected Item -> In Selected Res
* Clicked -> Confirm Screen Res

![Set the Screen Resolution to Selected Value](/posts/screenres/eventgraph.png)

17. Now, make sure your HUD Class is selected in your GameMode then run the game as standalone and bring up your UI.

![Set the Screen Resolution to Selected Value](/posts/screenres/hudclass.png)

18. Hopefully your should have something somewhat like the following.  Try changing the resolution to make sure it works, let the timer expire to see if it auto defaults back to the previous resolution and the ComboBox selected values update accordingly.  Next change the resolution again, hit confirm.  Quit the game, then come back in and check to see if the selected screen resolution has been loaded.

![Set the Screen Resolution to Selected Value](/posts/screenres/screenresuiunexpanded.png)

![Set the Screen Resolution to Selected Value](/posts/screenres/screenresuiexpanded.png)
