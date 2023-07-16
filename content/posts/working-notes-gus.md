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
*These 'Working Notes' blog entries are daily notes from engineering work I've done which I believe may be useful for others.  These notes are not meant to be best recommended practice and will probably change over time as I learn new things or come up with better tricks...*

## Unreal Engine 5 - In Game Graphics Settings
Currently I'm working on a UI for a game and will need to present to the user a menu UI for graphics settings.  The following will provide information to help people get started in the right direction with using the native Game User Settings (GUS) system in Unreal Engine 5.

We'll use C++ and Unreal Engine's reflection system to handle most of the work, and let Widget Blueprints sort out the presentation to the player.  Please note, I've omitted logging and failure case logic for the sake of brevity in these examples.

1. Create an ini file for GUS within the Game Config folder, right next to all the rest of it's little ini friends, example filename: `DefaultGameUserSettings.ini`
2. Populate the file with the GUS header pre-amble and sprinkle in any default settings you would like.  Go easy on the salt, you'll need that for later when you realise how many steps are in this article &#128517;
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

The Storing and Retrieving elements will allow us to save the settings across Game restarts and the like since this information will load to and from disk.

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
	 * Checks for available Screen Resolutions
	 * @param ScreenResOpts A reference to the data container with which to store the Screen Resolutions
	 */
	UFUNCTION(BlueprintCallable, Category = "Graphics Options")
	void CreateScreenResOpts(UPARAM(ref) TMap<FString, FIntPoint>& ScreenResOpts);
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
6. Applying - Next we'll create the method used by the Widget Blueprint to set the Screen Resolution. Something to consider, if allowing for multiple Graphics Settings, it may be better to set this up with a Struct that gets passed in by reference with all the desired settings.

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
7. Storing - We'll need to serialise the Game User Settings to disk so that players do not need to keep specifying their desired resolution each time they play the game.  This can be used in conjunction with logic within the Widget to give a constrained time within which they need to 'confirm' the new setting so they don't get stuck with a resolution so bad they can't navigate the UI to set it back.

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

8. With the C++ work done, we'll switch gears and work on the presentation layer using Widgets, and this time Blueprints, although c++ is equally valid.  First, we'll create a quick UI in the designer tab and use a ComboBox String to hold our Screen Resolution options. Don't worry about the need to fidget with the widget and adjust the content array field within the Details pane.  We'll handle that in code &#128526;

![UI Widget for Screen Resolution Change](/posts/screenres/GUS-UIWidget.png)

9. Next we'll pop over to the Event Graph. To start, we'll create a few member variables and some helper functions.  A quick word on nomenclature, for Blueprint variables I'll use the format `VarName<VarType>` Setup the following variables:

* `RevertTime<float>` Example Default Value: `5.0`
* `RevertTimerHandle<Timer Handle>`
* `PreviousScreenRes<Int Point>`
* `ScreenResolutions<Map<String, Int Point>>`

10. First helper function: `SetComboBoxSelectedOption` This will save us duplicating code for each time the ComboBox value is changed.  You'll want to create a local variable within the function: `ScreenRes<String>` and an input variable `InSelectedScreenRes<String>`

![Helper Function for Setting ComboBox Selected Item](/posts/screenres/populatecomboboxselectoptionfunc.png)

11. Second helper function: `RevertToPreviousScreenRes` this will give the player a fail-safe for if they select a bad screen resolution and cannot change back through the UI:

![Helper Function for Reverting to Previous Screen Res](/posts/screenres/revertfunc.png)

12. Third helper function: `ConfirmScreenRes` Does what it says on the tin!

![Helper Function for Confirming the Player's Screen Res Choice](/posts/screenres/confirmfunc.png)

13. Create a function called `PopulateScreenResComboBox`  We'll grab the HUD of the player and use our new methods to populate the ComboBox in the UI.  We'll also get the current screen resolution and set that as the default selected value of the ComboBox.  Screen shots are from the same function, just split over two pictures for the sake of clarity.

![Populate Screen Resolutions ComboBox Part 1](/posts/screenres/populatecomboboxfunc.png)

![Populate Screen Resolutions ComboBox Part 2](/posts/screenres/populatecomboboxfuncpt2.png)


14. We're almost done. Create another function called `SetScreenResolution`  Set an input parameter for the function as `InSelectedRes<String>`  We've added a timer here which will call the Revert Function upon expiration. A helpful thing to add would be a countdown in the UI using Widget Animations to let the user know how much time they have left to confirm, but you can do that in your own time. In a moment we'll add an event for On-Click for the confirm button which will destroy that timer and save the Game User Settings to disk.

![Set the Screen Resolution to Selected Value](/posts/screenres/setscreenresfunc.png)

15. In the designer view, click on the ComboBox and at the bottom of the Details pane, hit the binding next to `On Selection Changed`

16. Also, in the designer view, click on your Confirm button and at the bottom of the Details pane, hit the binding next to `Clicked`

17. Finally, Finally! In the Event Graph we wire up our funcs to three events:

* Event Construct -> PopulateScreenResComboBox
* On Selection Changed -> SetScreenResolution, Selected Item -> In Selected Res
* Clicked -> Confirm Screen Res

![Event Graph Where We Wire Up Our Funcs to UI Events](/posts/screenres/eventgraph.png)

18. Now, make sure <ins>your HUD Class is selected in your GameMode</ins>, you've come too far now to let something like that trip you up at the last step! Run the game as standalone and bring up your UI.

![Set the Screen Resolution to Selected Value](/posts/screenres/hudclass.png)

19. Hopefully you should have something somewhat like the following.  Try changing the resolution to make sure it works, let the timer expire to see if it auto defaults back to the previous resolution and the ComboBox selected value updates accordingly.  Next change the resolution again, write it down to remember later unless your mind is like a steel trap, mine's like a sieve... of Eratosthenes &#128526; anyways... hit confirm.  Quit the game, then come back in and check to see if the selected screen resolution has been loaded.

![Set the Screen Resolution to Selected Value](/posts/screenres/screenresuiunexpanded.png)

![Set the Screen Resolution to Selected Value](/posts/screenres/screenresuiexpanded.png)

And there you go, a working setup to allow Players to change their screen resolution in ~~5~~ 19 easy steps! &#127881; For further work you can include other graphics settings, don't forget my note in step 6. Create a visual confirm countdown timer in the UI. As well as limit the number of Screen Resolutions options which get listed within the ComboBox, as Tyranny of Choice is a thing.   At any rate, thanks for reading!

Full files:

```cpp
// GamHUD.h
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/HUD.h"
#include "GameHUD.generated.h"

/**
 * 
 */
UCLASS()
class MYSUPERAWESOMEGAME_API AGameHUD : public AHUD
{
	GENERATED_BODY()

protected:
	virtual void BeginPlay() override;

public:

	/**
	 * Checks for available Screen Resolutions
	 * @param ScreenResOpts A reference to the data container with which to store the Screen Resolutions
	 */
	UFUNCTION(BlueprintCallable, Category = "Graphics Options")
	void CreateScreenResOpts(UPARAM(ref) TMap<FString, FIntPoint>& ScreenResOpts);

	/**
	 * Sets the given Screen Resolution
	 * @param InScreenRes The IntPoint with which to set the Screen Resolution
	 * @param bOverrideCommandLine Should the Game User Settings override conflicting command line settings
	 */
	UFUNCTION(BlueprintCallable, Category = "Graphics Options")
	void SetScreenRes(FIntPoint InScreenRes, bool bOverrideCommandLine);

	/**
	 * Stores the Game User Settings to Disk
	 * @param bOverrideCommandLine Should the Game User Settings override conflicting command line settings
	 */
	UFUNCTION(BlueprintCallable, Category = "Graphics Options")
	void ConfirmGameUserSettings(bool bOverrideCommandLine);
	
};

```

```cpp
// GameHUD.cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "AGameHUD.h"

#include "GameFramework/GameUserSettings.h"
#include "Kismet/KismetSystemLibrary.h"

void AGameHUD::BeginPlay()
{
	Super::BeginPlay();

	/** Get the GameUserSettings data container with which our work will depend upon */
	UGameUserSettings* GameUserSettings = UGameUserSettings::GetGameUserSettings();
	/** Deserialise settings from Disk - Ex: DefaultGameUserSettings.ini */
	GameUserSettings->LoadSettings();
	/** Overwrite any garbage with default settings if need be */
	GameUserSettings->ValidateSettings();
}

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

void AGameHUD::SetScreenRes(FIntPoint InScreenRes,  bool bOverrideCommandLine)
{
	if(UGameUserSettings* GameUserSettings = UGameUserSettings::GetGameUserSettings())
	{
		GameUserSettings->SetScreenResolution(InScreenRes);
		/** We need to apply the settings before they take effect  */
		GameUserSettings->ApplyResolutionSettings(bOverrideCommandLine);
	}
}

void AGameHUD::ConfirmGameUserSettings(bool bOverrideCommandLine)
{
	if(UGameUserSettings* GameUserSettings = UGameUserSettings::GetGameUserSettings())
	{
		GameUserSettings->ApplySettings(bOverrideCommandLine);
	}
}

```