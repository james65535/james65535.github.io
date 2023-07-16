---
title: "Working Notes - The Replicated Cake is a Lie"
date: 2023-07-16T08:29:35+10:00
tags: ["Networking", "C++", "Replication", "UE5", "NetDriver", "Push Model"]
cover:
  image: "/posts/repisalie/repisalie.png"
  hidden: false
draft: false
---

**TLDR: FDoRepLifetimeParams bIsPushBased doesn't do what you think it does**

# Working Notes
*These 'Working Notes' blog entries are daily notes from engineering work I've done which I believe may be useful for others.  These notes are not meant to be best recommended practice and will probably change over time as I learn new things or come up with better tricks...*

### Assumptions
In my process of learning Unreal Engine, I've looked at a number of different game code bases developers have been kind enough to put online for others to check out.  My extreme thanks to them, they're the real heroes!  

Before I begin, a quick word on terminology.  I'll be using the word 'State' to refer to Data in a very generic sense, you can think of this as a variable, an object, or whatever it is in your program that holds data.  Going beyond that, and outside the scope of this article, State is actually a pretty good way to reason about complex systems.  If you're interested in knowing more then I highly recommend this research paper from David Harel on state charts: [Statecharts: A Visual Formalism for Complex Systems](https://www.sciencedirect.com/science/article/pii/0167642387900359)

### Initiating State Transfer
One of the things I took note of, given my distributed systems background, is how people are handling state replication for games.  Two common methods for initiating state transfer are Polling and Push mechanisms.  This basically comes down to whether you want the provider or client to initiate potential state transfer.  Typically, the Push method is used by the provider to inform the client of new state and to send it.  Usually this happens when the developer's code has chosen for this to happen, for instance when the existing State transitions to a new state, `uint32 Number = 65534;Number++;`, for example.

Polling is when you want the client to initiate State transfer.  The difference here is that the client doesn't know when State has changed, if it did, then that probably meant the provider did a Push and there's no need to pull! So the client connects to the provider and asks if there is new State to be had, if not then the client will wait for some determined period of time and check again, and check again, and check again until it gets some new State.  There's a million different ways to perform this depending on the task at hand.

### Unreal Replication Setup
In the Unreal Engine there are a number of systems to handle State transfer and replication is one of them.  This is where the Provider(or Server in this case), will tell the clients there is new State and what that new State is.  Historically, this has been done by the Server repeatedly checking to see if State marked for replication has been changed and if it has then it will initiate State transfer.  This is commonly setup in actors like follows:

```cpp
// ExampleActor.h
UPROPERTY(Replicated)
FString SomeString;

// ExampleActor.cpp
#include "Net/UnrealNetwork.h"

AExampleActor::AExampleActor()
{
	bReplicates = true;
}

void AExampleActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	/** Simple Rep */
	FDoRepLifetimeParams SharedParams;
	SharedParams.Condition = COND_SkipOwner;

	DOREPLIFETIME_WITH_PARAMS_FAST(AExampleActor, SomeString, SharedParams);
}
```


The `FDoRepLifetimeParams` struct has three members to it: 

* `Condition` - Constraints for to whom this State should replicate to, ex: COND_SkipOwner.
* `RepNotifyCondition` - Constraints for when to replicate this state and if a RepNotify callback has been specified then the client will run that upon receiving the State transfer.
* `bIsPushedBased` - The culprit at hand, more below.

### Unreal Engine Push Model
So, `bIshPushedBased`, if your IDE provides syntax hints, you'll see a comment refer to pushmodel.h.  That header file has a decently sized preamble comment talking about a new method for replication in the Unreal Engine and comparing it to old methods, I highly recommend reading it.  The gist is that the historical model would "Poll" it's internal State constantly looking for changes, then replicate if it found a replicated State which has changed.  This can be computationally taxing for Servers with many players or lots of different State set to replicate.  The Push Model advocates for a more "needs based approach", again check the pushmodel.h file for more info.

The problem at hand is that in many Game projects I see people use the `bIsPushedBased` member to indicate they want their state to take part in the PushModel way of doing things.  BUT they haven't put in place the required elements for that to even happen.  So while they have set `bIsPushedBased` to true, their State is still being polled and still being replicated on every change.

### Example Scenario
Consider a scenario where having a bit more control over when State is replicated would be helpful.  Let's say you have a multi-player racing game.  As each player passes the finish line, you want the game to update the Results Array containing the results of each player, then once all the players have finished, display the contents of the Results Array and process some other game related logic on the clients.  The trick here is for the client not to use the Results Array until all players have finished so that all results can be considered for whatever the game logic requires, such as displaying to screen.  Some approaches could be:

1. Replicate the Results Array every time it has been modified and have the clients polling the server using a repeated timer. Checking for when the race is finished and the client can consider the Results finalised and ready to be displayed.
2. Similar to the above but instead of having the clients poll, have the Server initiate a client based Remote Procedure Call (RPC) invoking the processes on the clients for handling the finalised race results.
3. Use the Push Model and only replicate the Results Array to clients once all players have finished the race.  A client based RepNotify can trigger logic for them to process game logic on the final results.

Option 3 is the easiest to implement, requires the least amount of code, and arguably may be the most performant option available.  However, without configuring your game to use it, your Results Array will be replicating every time it has been modified, not when your code has asked it to do so!

### What You'll Need to Do
So what needs to be done?  A few things, but read all items before proceeding:

1. Add `NetCore` to your Build.cs file in `PublicDependencyModuleNames`.
2. In the game config folder, add the following to DefaultEngine.ini
```
[SystemSettings]
net.IsPushModelEnabled=1
```
3. Now, go back through your code base and fix all your code which had been using `bIsPushedBased` and is now completely broken. Let me expand on this below.

Since you have chosen PushModel based replicated State transfer, you have greater power over when to replicate, and of course we know with great power comes great responsibility!  To exercise that power you'll need to explicitly state when to replicate (*note: for Blueprints this is different, go reach pushmodel.h*). You do this by calling the suitable macro from the list below:

* `MARK_PROPERTY_DIRTY()`
* `MARK_PROPERTY_DIRTY_UNSAFE()`
* `MARK_PROPERTY_DIRTY_FROM_NAME()`
* `MARK_PROPERTY_DIRTY_STATIC_ARRAY()`
* `MARK_PROPERTY_DIRTY_STATIC_ARRAY_INDEX()`
* `MARK_PROPERTY_DIRTY_FROM_NAME_STATIC_ARRAY()`
* `MARK_PROPERTY_DIRTY_FROM_NAME_STATIC_ARRAY_INDEX()`
* `MARK_PROPERTY_DIRTY_FROM_NAME_STATIC_ARRAY_INDEX_AND_ACTUALLY_I_REALLY_WOULD_LIKE_SOME_CAKE()`

Ok, maybe not that last one &#128517; Example:

```cpp
// ExampleActor.cpp

/** Option 1 */
MARK_PROPERTY_DIRTY_FROM_NAME(AExampleActor, SomeStringVar, this);

/** Option 2 */
SomeStringVar = "Time to Replicate";
MARK_PROPERTY_DIRTY_FROM_NAME(AExampleActor, SomeStringVar, this);

/** Option 3 */
MARK_PROPERTY_DIRTY_FROM_NAME(AExampleActor, SomeStringVar, this);
SomeStringVar = "Time to Replicate";
```
So that's it, you are now taking advantage of the "new" Push Model for State replication... No, for real this time!  Just don't forget to go and read pushmodel.h!
