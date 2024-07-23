# Tick Talk
## ***What is gameloop?***

Game loop is the "time" in game. The bare bones of a game is a loop that waits for input from the player (see old text games that wait for player input) and when it gets input it returns a result and asks for input again.

In real time games (LIKE IN UE) that loop is run as long as the game runs and while it does wait for player input, it also continues to update the game and graphics. So player input doesn't block the program.
(for example an enemy hits the player even when they're AFK).

```
while (true)
{
  processInput();
  update();
  render();
}

```

*This loop runs even when the player doesn't give input*

The loop runs as fast as your CPU allows it to. It's also it's job to to make sure to keep the loop at a constant time, but we'll get more into that in *delta time.*

Since every time we run an iteration of the game loop we also render the graphics (AKA a single frame), we can measure how fast the game runs by the very known parameter FPS (Frames Per Second, the more technical brother of First Person Shooter). So as you probably know, the faster the game runs, the higher the FPS.

Also if you some how think your game doesn't need a game loop pattern, take into account that if you want any VFX or SFX to play while your game is waiting for player input, you need a game loop that runs without player input blocking it ✨.

Sources for this chapter:
https://gameprogrammingpatterns.com/game-loop.html#a-world-out-of-time

## ***What is tick?***

Tick is our definition for a single iteration inside the game loop. Y'all know it as Event Tick in engine.
This event is called once per frame, and is affected by the game's performance/FPS (as my movement code sadly was).

We'll talk about when to use tick and when we'd prefer to use a timer later on but generally you want to avoid using *Event Tick* and use another alternative of it instead (see chapter "When to use tick and when to use timer?").

Sources for this chapter:
https://www.youtube.com/watch?v=GyJVYB3IzGAhttps://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/Ticking/

## ***What is Delta Time?***

**Delta time** is the time elapsed between the last frame and the one preceding it.

### How Delta Time is Calculated

Delta time is calculated by:

1. Getting the real time at the start of the iteration from the OS (or engine).
2. Subtracting it from the real time at the end of the iteration.

The result means the longer the frame took, the higher the delta time.

We get the results in seconds, though the result is often in the milliseconds range.
You can get the delta time of specific FPS by doing $1/FPS=DeltaTime$.

For example:
$1/120=0.0083333$
$1/60=0.0166666$ 16millisec
$1/30=0.0333333$
$1/8=0.125$ *(The minimum you can set with t.maxFPS)*

### The Frame Delay

We use the previous frame's difference as our current frame is still being calculated. This results in a 1-frame delay for delta time, but over time, it evens out.

### When to Multiply by Delta Time

We should multiply by delta time only for actions that occur over time (and for physics impulses). This ensures that time-dependent processes run consistently regardless of frame rate.

### Example: Using Delta Time in a Game Loop

Here is an example of how to use the delta time concept to prevent a game loop from running too fast on a strong machine:

```
while (true) {`
  `double start = getCurrentTime();`
  `processInput();`
  `update();`
  `render();`

  `sleep(start + MS_PER_FRAME - getCurrentTime());`
`}

```

*We basically wait it out if we finished too early. (That's what she said?)*

### ***Why Do We Need Delta Time?***

Tick does *not* reflect real time and can vary across different machines, potentially stalling when loading something heavy. This inconsistency can make the game look choppy or overly fast to developers and players.

To ensure consistency, if we have a time-dependent action (e.g., moving an actor from point A to B in a certain time), we multiply it by delta time to make it **independent** of FPS. Without this, the action would depend on tick rate, leading to varying results depending on FPS.
(see example 4:30ish https://www.youtube.com/watch?v=GyJVYB3IzGA).

*This chapter was organized, (not written!), by ChatGPT. I have proof read it but I might've missed something.*

Sources for this chapter:
https://www.youtube.com/watch?v=yGhfUcPjXuEhttps://www.youtube.com/watch?v=GyJVYB3IzGAhttps://forums.unrealengine.com/t/when-using-addforce-do-i-need-to-scale-the-force-by-the-delta-seconds-value/353657/

## ***What is tick groups?***

So we know what tick is by this point (I hope), and now we come into reality and as all things, not all ticks are born equal😔.

In Unreal Engine we have "tick groups" that decide **when** inside the update/iteration of the gameloop we should call the tick event and update our classes.

Generally this is separated by physics; before, during and after, with an additional tick at the last moment in the frame.

By default actors are set to TG_PrePhysics.
UMG auto manages its tick by default (and inherits from it's spawner).

| Tick Group | Engine Activity |
| --- | --- |
| TG_PrePhysics | Beginning of the frame. |
| TG_DuringPhysics | Physics simulation has begun by the time this step is reached. Simulation may finish and update the engine's physics data at any time while ticking this group, or after all group members have ticked. |
| TG_PostPhysics | Physics simulation is complete and the engine is using the current frame's data by the time this step begins. |
| n/a | Process latent actions, tick the world timer manager, update cameras, update level streaming volumes and streaming operations. |
| TG_PostUpdateWork | Called at the latest possible moment inside the tick |
| n/a | Handle deferred spawning of actors created earlier in the frame. Finish the frame and render. |
| note that n/a is 2 other tick groups epic games planned to add but gave up on it. It's still in the docs but those are irrelevant to us right now. |  |

| Tick Group | When to use it |
| --- | --- |
| TG_PrePhysics | You should set your class to use this tick group if they're to interact with physics. This will make the actor's movement be factored into the physics simulation. <br>When using this tick group the simulation data we interact with is 1 frame old (since this is the start of the iteration/update). |
| TG_DuringPhysics | This is the tick group where the physics simulation is run. <br>The simulation can end at any point during this tick and it is generally recommended to use this tick group only for things that you won't care if they're 1 frame off like HUD, mini map or updating inventory displays. |
| TG_PostPhysics | You should set your class to this tick group only for things that should happen after everything finished moving and are as they should be in the frame rendered. For example movement traces (ahem cannons) where a 1 frame lag will be very noticeable. |
| TG_PostUpdateWork | This tick group is called after TG_PostPhysics and after cameras has been updated. This is commonly used for particle systems where we want to run the logic after everything has already been set. This can also be used to run after everything has run and that we want to be absolutely sure that everything is frame matching (like two characters trying to grab each other on the same frame in a fighting game). |

https://www.youtube.com/watch?v=xwR402KlYT0

https://www.youtube.com/watch?v=xwR402KlYT0

### ***What is tick Dependency?***

If we know we want an actor or component to tick only after a certain actor / component has ticked (for example we need data to pass from one component to an actor before the actor runs), we can skip using tick groups for specific cases and just add ✨tick dependency✨.

`AddTickPrerequisiteActorAddTickPrerequisiteComponent`

see BP example here
https://www.youtube.com/watch?v=JbfiolyLpIQ

We just need to input an actor or a component reference we want to depend on before ticking and that's it. We can also remove it but generally this is only for very specific cases so I don't think you'll run into it often.

Sources for this chapter:
https://dev.epicgames.com/documentation/en-us/unreal-engine/actor-ticking-in-unreal-engine?application_version=5.4#tickgrouporderhttps://www.youtube.com/watch?v=JbfiolyLpIQhttps://www.youtube.com/watch?v=xwR402KlYT0

## ***What is substep/timestep?***

Since we want our physics to be calculated separately than the rest of the game to get accurate results we can set the "physics" tick in unreal to a different max value than the rest of the game to make sure we get accurate results that don't affect render time if we go over the expected frame rate. (for example we plan our game to run at 60FPS but a player runs it in 120FPS).

![[MaxPhysicsDeltatime.png]]
*You can get to this by Project Settings > Engine > Physics.*

By setting Max Physics Delta Time we set a maximum FPS value we want our simulation to run at. *(expressed in milliseconds, here 0.016 = 60FPS. You can find the delta time by dividing 1 in the FPS amount; here it's: 1/60)*. If our game runs on a faster FPS than our set limit then ***we clamp the physics simulation time down to 60FPS***. This way we avoid scenarios where our game physics run comically fast on stronger machines. (Not that it ever happened to me😭 not at all.. 😭).

### *Now we get to actually what is substepping*

Substepping/Timestepping is calling smaller (sub) ticks into our physics tick in order to achieve smoother results from physics when we go below our expected FPS target. For example you set your physics simulations to run ok in the range of 50-60FPS but you also want your physics to work if the game runs in 30FPS.

![[SubsteppingInEngine.png]]
*You can get to this by Project Settings > Engine > Physics.*

So we go into project settings and enable substepping, but what do the settings mean?

| Setting | What it does |
| --- | --- |
| Substepping Async | Whether to substep the async phsyics simulation.<br> unsure exactly why this is needed ATM. Taken from UE docs |
| Max Substep Delta Time | This value is set in milliseconds and it means that we won't go beyond this value when we do the timestepping. This is like the Max Physics Delta Time, we set this as an upper bound to which we plan our game to support. For example if we set it to 0.016 it means we won't get above 60FPS in our physics simulation when we enable substepping. |
| Max Substeps | This value sets how many substeps we can do. Meaning, when we go below 60FPS and we're in the range of 30-60FPS, UE will put 2 substeps with the duration deltatime/2 to ensure we keep up with the expected physics FPS. If we go below 30FPS UE will put 3 substeps with the duration deltatime/3 and so on. So the higher the number, the lower the FPS we can support. |

If we substep too many times and go faster than the desired delta time, we'll wait until we reach the desired delta time again.
![[Substepping-Timelines-1-615x402.png]]
*Here you can see at the bottom the timesteps waiting for the max delta time as they passed it ([aclockworkberry.com](http://aclockworkberry.com/))*

Since substepping adds more computation to the CPU of the game if the CPU is slow it can result in computational overhead (AKA a lagging rabbit hole where we keep lagging and can never catch up) if we don't pay attention to the values used (esp the max substeps as that sets the limit to the computational overhead).

Sources for this chapter:
https://www.aclockworkberry.com/unreal-engine-substepping/https://avilapa.github.io/post/framerate-independent-physics-in-ue4/

## ***How to work with physics and tick? (AKA slightly less FPS dependent physics)***

So, we learned how to use substeps! 🎉
But now when we add our physics simulation to tick everything still stays the same, so wtf?

Well... Event Tick is only called on the frame rate rendering tick and it's delta time is also related to the frame rendering tick. So where ***is*** our precious substeps

*The following part (until the next horizontal line) is out dated and wrong. I’m leaving it here for the benefit of the doubt -Emanuel*

---

We'll need to make our own event for it in C++ for our physics using actor.

```cpp
public:
  // Event called every physics tick and sub-step.
  UFUNCTION(BlueprintNativeEvent)
  void PhysicsTick(float SubstepDeltaTime);
  virtual void PhysicsTick_Implementation(float SubstepDeltaTime);

  // Custom physics Delegate
  FCalculateCustomPhysics OnCalculateCustomPhysics;
  void CustomPhysics(float DeltaTime, FBodyInstance* BodyInstance);

```

*This should be in the header file. Here we define a physics tick and an implementation of it that we'll actually use.*

```cpp
OnCalculateCustomPhysics.BindUObject(this, &AExampleActor::CustomPhysics);

```

*We subscribe to the delegate in the class constructor like this. ==unsure of the need for the delegate.==*

### Make sure to replace `AExampleActor` with your actor name!

Then, we need to actually add the custom physics each frame (meaning, in `Tick()`) to the root RootComponent body instance. Here `box_component` is the root.

```cpp
  // Add custom physics on RootComponent's BodyInstance
  if (box_component_->GetBodyInstance() != NULL)
  {
    box_component_->GetBodyInstance()->AddCustomPhysics(OnCalculateCustomPhysics);
  }

```

> Note: Víctor Ávila Parcet says "There is a newer way to handle substepping in UE4 4.15, which I believe it works per scene, rather than for each actor, but I have not tried this yet."
> 
> 
> Since I couldn't find it ==*currently*== we'll use this setup per actor and just add that to the base of our physics needing actors hierarchy.
> 

And now we can actually declare our *PhysicsTick()*!

```cpp
  void AHoverVehicle::PhysicsTick_Implementation(float SubstepDeltaTime)
  {
    // Physics calculations go here...
  }

  void AHoverVehicle::CustomPhysics(float DeltaTime, FBodyInstance* BodyInstance)
  {
    PhysicsTick(DeltaTime);
  }

```

*Just remember to replace `AHoverVehicle` with the name of your actor.*

---

Now all the reading and writing of physics data needs to be done to the BodyInstance of the object we're working on. As the BodyInstance stays up to date on physics steps while the actual root component is only updated once per Tick().

Example on how to use BodyInstance:

```cpp
// Getting the transformation matrix of the object
FTransform WorldTransform = box_component_->GetBodyInstance()->GetUnrealWorldTransform_AssumesLocked();

// World Location
FVector Location = WorldTransform.GetLocation();

// Getting the forward, right, and up vectors
FVector Forward = WorldTransform.GetUnitAxis(EAxis::X);
FVector Right   = WorldTransform.GetUnitAxis(EAxis::Z);
FVector Up      = WorldTransform.GetUnitAxis(EAxis::Y);

```

Now applying forces also requires us to specify we use substepping

```cpp
// Adding forces
box_component_->GetBodyInstance()->AddForce(Forward, false);
box_component_->GetBodyInstance()->AddTorque(Right, false, true);
box_component_->GetBodyInstance()->AddForceAtPosition(-Forward, StartPoint, false);

```

> Note that there is an extra parameter in each of the functions called bAllowSubstepping. ~~Contrary to what it suggests, you must set it to false in order for physics to work correctly~~. *This is fixed in UE5.4, you should set it to true and it should work.*
> 

> Note: As you might know, you don’t need to multiply physics forces by the Delta Time*, as UE does that for you.
> 

**Add Torque cannot be multiplied with delta time, your best bet is to play around with substepping with this one to get a certain rage of FPS where torque will behave almost the same.*

### Notes about requirements:

*we need to have a rigidbody as our root i.e. any Component extending UPrimitiveComponent*

//shouldnt use this lel
*(We'll also probably need to enable smoothed frame rate range according to https://avilapa.github.io/post/framerate-independent-physics-in-ue4/) I will need to test this more thoroughly*.
![[SmoothedFPS.png]]

Sources for this chapter:
https://avilapa.github.io/post/framerate-independent-physics-in-ue4/https://www.aclockworkberry.com/unreal-engine-substepping/

update for UE5.1
I honestly think that you can't access substepping any more as a client and it can only be exposed through FGearVehicle or another async physics plugin. The delegate mentioned is uncalled no matter what and it seems outdated af.
https://itscai.us/blog/post/ue-physics-framework/https://forums.unrealengine.com/t/physics-sub-stepping/3565/30?u=heyoletsgo9

## ***When to use tick and when to use timer?***

This is a loaded question that you'll see a lot of fighting over in the internet (and IRL if you're fun and feral).

Generally the guideline is:

> "If something always has to be done use tick. Anything else you want to do relatively infrequently is for timer".  - Peachmage, a friend of mine who's a AAA senior UE dev.
> 

And now more into detail 'cause quoting a friend is equal to nothing lol.

### So when should you use *Event* Tick?

Basically never.. You should only use it for stuff that *has* to be updated every frame as long as the game runs.

For example:

- Custom character movement
- Dynamic camera animations
- Dynamic character audio changes
- Procedural animations
- Feeding actor positions to a post-process material //its better not involve cpu and gpu

These are stuff that need to be updated per frame and having them separated of framerate will cause stuttering.

> Event based mentality
> 
> - If your implementation’s start and end can be measured, and it’s not the game’s start and end, then your stuff should be event-based.
> If your script is contextual within your game world, it is event-based.
> If the instigator of your action is known, that action is event-based.
> -Hristo Enchev

> "movement should be tick based as you want to move every frame otherwise you'll render 2+ frames with the same position, move, then render again for a few frames and it will look like you're stuttering." - Peachmage
> 

### For all tick uses other than the options mentioned above;

You should use Localized Tick that can be triggered by events instead of running all the time. You can achieve that using a ***Timeline set to looping.***

If your Event Tick has a branch, you should switch it to a Localized Tick, and instead of changing a bool (as you would have one for your branch), you should call an event to Start/End your tick like in the image below.
![[LocalizedTickTimeline.webp]]
*This saves us overhead execution and gives greater control than boring Event Tick.*

![[TimelineSettingExample.png]]
*You can open your Timeline and set it to loop and select a tick group. This gives you much more control on how and when your tick is run compared to Event Tick.*
//check if it's bound tick interval - tell konsta

### Stuff that has to use tick but doesn't have to be *Event* Tick are:

- Physics forces (excluding Impulses)
- Value interpolation (lerp, slerp, interp, ease, etc)

> Though we covered physics in an earlier chapter with substepping, if you don't use substepping, you should consider using a localized tick instead.
> 

### For anything else you should use a Timer.

Timers offer control over the interval time your function/event would be called at and should be used for stuff like:

- Weapon shots
- Data recordings
- Ability cooldowns.

Though! it's important to remember; ***Timers are dependent on FPS.***
If you set the `Time` value lower than delta time, the timer *will* call the event multiple times, but it will be fired in a burst in 1 frame and *will not* be evenly spaced out. This is a waste of execution that can be executed once with a higher delta time instead.

![[SetTimerByEvent.webp]]
*Since UE 5.4 there's an option to limit the execution to be `Max Once Per Frame` to avoid bursting execution and wasting CPU.*

### When do timers tick?

Timers tick with the TickManager (you call it in C++ to use timers), and TickManager ticks between the tick groups PostPhysics and PostUpdateWork. So take it into account, you shouldn't use physics with timers.

### Regarding UMG,

they automatically handle ticking. You can disable them from ticking in class settings. But! you should still avoid binds as much as you can.

If you want them to ***stop*** ticking when game is paused, you need to create your own C++ tick/base widget class.
(https://forums.unrealengine.com/t/umg-pause/319032/7)

According to Hristo Enchev

> "Using the event tick of the widget and manipulating multiple objects is more performant than running the same code in multiple binds."
> 
> 
> (https://medium.com/hri-tech/tick-101-implementation-approach-and-optimization-tips-c6be10b3e092).
> 

### ***For better performance you should disable tick in everything that shouldn't use tick to save execution time.***

![[UMGTicks.webp]]

### Some extra notes:

I didn't cover everything about tick here, you can see in the advanced tick settings here there's also `Tick Even when Paused` that UMG can inherit from. You can also set interval for your actor ticks, and plenty of other things you can dive a lot more into.
![[TickAdvancedOptions.png]]
//do timers tick when paused? - tell konsta
//expand more on tick interval
There's also Mono Tick and options to spread your execution evenly in a tick.
I left those as extras with starting links for you to dive more into them if you so wish.

Sources for this chapter:
https://medium.com/hri-tech/tick-101-implementation-approach-and-optimization-tips-c6be10b3e092https://forums.unrealengine.com/t/tick-vs-timers/118402/5https://forums.unrealengine.com/t/timer-lets-you-set-tick-interval-so-what-is-the-default-tick-interval-of-an-event-tick/48421/9https://www.tomlooman.com/unreal-engine-cpp-timers/https://forums.unrealengine.com/t/keeping-widget-blueprint-ticking-when-paused/260087/4https://forums.unrealengine.com/t/what-is-more-efficient-tick-or-timer/439943

## ***Disclaimer:***

A lot of the images here were taken from the sources I linked.
I'm unsure about my understanding of some of the subjects covered here and some things might require more testing.

If you find any mistakes you can contact me through Discord or Linkedin and (politely please) let me know.
Discord: ayenneuman
Linkedin: https://www.linkedin.com/in/emanuel-nm/

## ***Extra: What is mono tick?***

https://forums.unrealengine.com/t/released-mono-ticking-improving-tick-performance-in-a-few-minutes/151262/4

## ***Extra: Spreading a loop in a tick?***

https://forums.unrealengine.com/t/a-couple-of-useful-macros-loops-per-tick/31890/23

![[ForEachLoopPerTick.png]]

![[ForLoopPerTick.png]]

## ***Sources:***

### Gameloop

https://gameprogrammingpatterns.com/game-loop.html#a-world-out-of-time

### Tick

https://www.youtube.com/watch?v=GyJVYB3IzGAhttps://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/Ticking/

### Delta time

https://www.youtube.com/watch?v=yGhfUcPjXuEhttps://www.youtube.com/watch?v=GyJVYB3IzGA

### Tick groups

https://www.youtube.com/watch?v=xwR402KlYT0[https://dev.epicgames.com/documentation/en-us/unreal-engine/actor-ticking-in-unreal-engine?application_version=5.4#tickgrouporderhttps://dev.epicgames.com/documentation/en-us/unreal-engine/actor-ticking-in-unreal-engine?application_version=5.4#tickgrouporder](https://dev.epicgames.com/documentation/en-us/unreal-engine/actor-ticking-in-unreal-engine?application_version=5.4#tickgrouporder)https://www.youtube.com/watch?v=JbfiolyLpIQ

### Substepping

https://www.aclockworkberry.com/unreal-engine-substepping/https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/Physics/Substepping/

### Physics and tick

https://avilapa.github.io/post/framerate-independent-physics-in-ue4/https://www.aclockworkberry.com/unreal-engine-substepping/

### When to use tick or timer

https://medium.com/hri-tech/tick-101-implementation-approach-and-optimization-tips-c6be10b3e092https://forums.unrealengine.com/t/tick-vs-timers/118402/5https://forums.unrealengine.com/t/timer-lets-you-set-tick-interval-so-what-is-the-default-tick-interval-of-an-event-tick/48421/9https://www.tomlooman.com/unreal-engine-cpp-timers/https://forums.unrealengine.com/t/keeping-widget-blueprint-ticking-when-paused/260087/4https://forums.unrealengine.com/t/what-is-more-efficient-tick-or-timer/439943

## ***Tick interval:***

You can set how fast an actor/component will tick with tick interval. You can't set it to a value smaller than delta time 'cause then it will just tick with FPS when you set the value too low, unlike timers with `Max Once Per Frame` turned off.
When setting a new tick interval during run time your tick interval will only update on the next tick.
Setting the tick interval to 0 will make the actor/component tick every frame (default).

(when I say component I mean *actor* component here)

According to this source, if you try to shorten the tick interval during runtime it's recommended to disable tick, shorten the interval and activate it again to prevent waiting for the next tick? I'm not sure about this info tbh and IMO if you get to this scenario you should probably rethink the way you do things.
https://forums.unrealengine.com/t/changing-actor-tick-interval-while-running/331964/7?u=heyoletsgo9

https://www.youtube.com/watch?v=Axz20Fj_Kp8https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/GameFramework/AActor/SetActorTickInterval

Another source for this chapter, but this one has a lot more details, would be worth reading thru all of this. This also covers Task Graphs! (and is outdated about the physics tick 😭)
https://www.casualdistractiongames.com/post/2018/12/10/unreal-tick-functions-delta-time-and-the-task-graph

### ***Tick interval vs timer:***

Timers will tick even when the actor's tick is disabled in class defaults. Meaning if your actor shouldn't tick all the time you can just disable tick and use a timer with the interval you wanted instead.
https://www.youtube.com/watch?v=T9Kzq4SvrAA

Recommended here that you should use timers instead of tick interval, and if you don't need your actor to tick all the time it's better to just use timers (timers tick even if the actor's tick is disabled).
*Take this with a grain of salt as this links to the forum discussion I found this info at, no official sources, though it does make sense...*https://forums.unrealengine.com/t/tick-interval-and-performance/150254/2

### ***Timers over tick:***

> Timers have several benefits over Tick.
> 
> 
> They can skip condition checks when not needed as one class keeps track of all timers running.
> 
> All timers are listed in a sorted list so they will always be executed in the same order unlike Ticking Actors in the same group.
> 

> Use Tick for ticking every frame with or without interruptions.
> 
> 
> Use Timers for everything else.
> https://www.reddit.com/r/unrealengine/comments/gtxy6j/tickinterval_vs_timer/
> Do note that timers are called after post physics so movement should still be on tick (without interval!).
> 

![[ComapringTickvsTimelinevsTimer.webp]]
https://gameinspired-mail.medium.com/ue4-ticks-timelines-and-timers-9c7333952331

Tick (even with interval) has a greater performance cost even when it's not called, unlike timers and timeline that can be deactivated.

### ***Conclusion?***

Tick interval works like timers but you can set it's tick group.
Generally if you want something to loop not on frame (aka custom tick interval), the tick group becomes meaningless as you lose the accuracy you want either way. (I didn't find anything that says tick interval is better than a timer. I did find usage where they mix both options (link below) but tick interval could still be replaced with a second timer instead).

So, my conclusion is that tick interval is a cool option to make things tick less often and save some performance. But if you want to have better control and performance, just turn off ticking in the class defaults and use multiple timers as they give greater control *when* a class should tick and how often (you can also avoid bursting with timers since UE5.4), (and avoid going any lower than 0.1 - 0.05 as that's the average delta time).

(this is the link below)
https://www.cbgamedev.com/blog/quick-dev-tip-74-ue4-ue5-optimising-tick-rate

## ***Timelines and timers, extra info***

Timelines are **actor components**, you can find them as variables in the component section in BP. Since they're components they have independent tick than the owning actor. Meaning they tick normally even when the actor's tick is disabled and when there's a tick interval on the owning actor.

Since timelines are components, you can set them to tick when paused or set a tick interval for them. You can do that by dragging the timeline component reference variable and set it's setting like you would for any other component.

(the source for this section is me experimenting with the engine)

Timers on the other hand will not run when the game is paused. However they are still independent from the owning actor as they are part of `FTimerManager` and owning actor's tick interval will not affect them.

https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-timers-in-unreal-engine

## ***Advanced Tick() Functionality: Tickables & Multi-Tick (Extra)***

Multi tick and stuff. You can make non component/actors tick and they'll tick after TG_PostUpdate.
https://dev.epicgames.com/community/learning/tutorials/D7P8/unreal-engine-advanced-tick-functionality-tickables-multi-tick

what is FPS drops category?
what is Music Playback Percent?

| Type | Affected by Time Dilation | Affected by FPS | Affected By Pause Game | FPS drops? | Notes |
| --- | --- | --- | --- | --- | --- |
| Event Tick | yes | yes | can be set not to |  | Can use delta time to offset FPS. Can be set to run while paused. |
| Gameplay Timer | yes | yes if interval time is lower than delta time | yes |  | Can't be set to an interval lower than delta time. |
| Timeline | yes | yes | can be set not to |  | Can be set to run while paused and set with a custom tick interval. |
| Audio Time | no | no | yes |  | TBH IDK why it's called audio time.<br>Returns time in seconds since world was brought up for play, IS stopped when game pauses, NOT dilated/clamped. |
| Game Time | yes | no | yes |  | Returns time in seconds since world was brought up for play. |
| Real Time In Seconds | no | not really. see note in accurate real time below | no |  | Returns time in seconds since world was brought up for play, does NOT stop when game pauses, NOT dilated/clamped. |
| Accurate Real Time In Seconds | no | no | no |  | Returns time in seconds since the application was started. Unlike the other time functions this is accurate to the exact time this function is called instead of set once per frame. |
| Unpaused Time | yes | no | no |  | Returns time in seconds since world was brought up for play, adjusted by time dilation and IS NOT stopped when game pauses. |
| Music Playback Percent |  |  |  |  |  |
| Now | no | no | no |  | Was called Current OS Time previously?<br>Returns the time from the computer as integer struct. |
