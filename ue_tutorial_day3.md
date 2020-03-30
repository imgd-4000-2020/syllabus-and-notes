## Goal: 
Extend our example from last class to include a hit test that destroys a simple cube. 

## Concepts
This is a very short tutorial, just to make sure we’re keep our heads in some code while we start planning our projects. Last time we made a simple example of a projectile that flew off into space. However, for many types of of “shots” it’s better to be near-instantaneous… think of a laser, for example.  In these cases we want to simply see if a hit occurs at the moment the shot is **fired**.  Unreal enables us to do this with **traces**.

A trace in Unreal is the equivalent of a `Raycast` object in other game systems. We’ll project a line from one point to another. see if any objects are struck in the process, and return the first one we hit that **blocks** (does not allow objects to pass through it). Although you can [define custom filters for the types of objects you want to perform hit tests with](https://www.unrealengine.com/en-US/blog/collision-filtering) in this case we’ll use one of UE4’s built-in filters, which checks against all objects that are visible. 

## Code
Again, this starts with the [code from the last tutorial](./ue_tutorial_day2.md). We will primarily be replacing our the `SpherePawn::fire` method. But first we need to include a helper that will let us see where we’re shooting, so that we can visually identify if hits should occur. Include the following at the top of `SpherePawn.h`:

```c++
#include "DrawDebugHelpers.h"
```

This gives us a variety of different shapes that we can dynamically draw in our world at different locations and with different orientations. In this case we’ll be using lines. Now go to our `.fire` method.

We need to define a start point and an endpoint for our line. 	Our code already defines a starting location at the beginning of our fire method, let’s also augment its X component:

```c++
FVector start = GetActorLocation();
start.Z += 100.f;
start.Y += 100.f;
start.X += 150.f;
```

Next, we want to define an end position. This will simply be our start + (actorForward * distance). We already looked at getting the actor’s current forward direction in last class:

```c++
float distance = 1000.f
FVector fv = GetActorForwardVector();
FVector end = ((fv * distance) + start);
```

Now let’s draw a line using a function from our draw debug helpers:

```c++
DrawDebugLine(GetWorld(), start, end, FColor::Red, false, 2.f, 0, 5.f);
```

The first four parameters are probably self-explanatory. The fifth(currently `false`) dictates whether the line is persistent. The sixth is the amount of time the line is on screen, then depth priority, and last but not least line thickness.

If you compile and play now, you should have some shooting lines that appear.

### The hit test

OK, let’s find out if something is hit. We need to create a couple of structs to pass to our hit test.

```c++
FCollisionQueryParams cqp;
FHitResult hr;
```

We’ll use the default query parameters that our `cqp` struct gives us, while our `hr` struct will be populated with the results of our hit test.

The call to our hit test is:

```c++
GetWorld()->LineTraceSingleByChannel(hr, start, end, ECC_Visibility, cqp);
```

…where `ECC_Visibility` defines that trace filter that we want to apply to our collision test; in this case, is the object visible. Again, the results of our hit test will be place in the `FHitResult` struct, `hr`.

Now let’s look and see if a hit was found on a blocking object:

```c++
if( hr.bBlockingHit == true ) {	
    if( hr.GetActor() != this ) {
        UE_LOG(LogTemp, Warning, TEXT("HIT! %s"), *hr.GetActor()->GetName() );
        hr.GetActor()->Destroy();
    }
}
```

If you have any code related to spawning projectiles from the last class left in `.fire` go ahead and move it now. 

OK! If you hit compile and then play, we should be getting the lines drawn, but no other results. Go ahead and drag a Cube actor (or any other shape) out into the world in a place where the rays will be able to strike it. Try it again, and you should see the instance name of the object presented in the message console. The cube should also be destroyed.
