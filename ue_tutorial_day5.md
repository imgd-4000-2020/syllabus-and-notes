# IMGD4000 - Day #5
Goal: Get started on the [Agent-based simulation assignment](https://github.com/imgd-4000-2020/syllabus-and-notes/blob/master/boids_assignment.md) and continue wrapping our brains around Unreal’s C++ “features”.

## Start
Go ahead and create a new, blank, C++ game project. No need to import the starter content… you can always do that later if you need it and it can shorten compile times to leave it out. Name the project “FlockingGame”; if you choose another name make you change all corresponding code snippets to reflect the correct name of your `GameModeBase` subclass.

## Who’s in charge here?
I’m recommending an architecture similar to the following:

1. A class to store position / velocity of agents and render them
2. A class to run the boids algorithm on all agents in the world
3. Use our GameModeBase subclass to instantiate the flock and drive it via `Tick()`.

With that architecture, we’re only looking at implementing two classes and adding a little bit of code to our GameModeBase subclass… not too bad.  We’ll work our way from the last item to the first in this tutorial.

Our `GameModeBase` subclass is basically running the show. Let’s setup its header file as follows:

```c++
#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "FlockingManager.h"
#include "FlockingGameModeBase.generated.h"

UCLASS()
class FLOCKING_API AFlockingGameModeBase : public AGameModeBase
{
	GENERATED_BODY()
	AFlockingGameModeBase();

	UPROPERTY(EditAnywhere)
	class UStaticMeshComponent * Mesh;

	UPROPERTY() UFlockingManager *Manager;

	virtual void BeginPlay() override;
	virtual void Tick(float DeltaTime) override;
};
```

We’ll add in our `BeginPlay` and `Tick` overrides, and also create a mesh component that we’ll assign via Blueprints and use to represent each individual agent. Finally, we’ll add an instance of our `FlockingManager` class (which we haven’t made yet).

Let’s change over to our game mode’s C++ file.  First, the constructor and includes:

```c++
#include "FlockingGameModeBase.h"

AFlockingGameModeBase::AFlockingGameModeBase() {
    AgentMesh = CreateDefaultSubobject<UStaticMeshComponent>("AgentMesh");
    PrimaryActorTick.bCanEverTick = true;
}  
```

Again, we haven’t created the `FlockingManager` yet, we’ll do that shortly. We create a default mesh component to use, and tell the game mode to tick. 

Next let’s add in our `BeginPlay` method:

```c++
void AFlockingGameModeBase::BeginPlay() {
    Super::BeginPlay();
    UE_LOG(LogTemp, Warning, TEXT("GAMEMODE BEGINPLAY()"));
    Manager = NewObject<UFlockingManager>();
    Manager->Init( GetWorld(), AgentMesh );
}
```

A new call here… the templated `NewObject` method. We can use this to create a new object at any point **after** the constructor to a class. In contrast, the constructor is typically used to initialize objects that can be configured in the editor / blueprints, which is the point of `CreateDefaultSubobject`. 

`NewObject` will handle allocation, initialization, and register the resulting object with the engine so that it can be properly garbage collected.

We’ll look at  `FlockingManger::Init` shortly, but here we can see we’re passing it a pointer to our `UWorld` object, and a pointer to our static mesh component.

Last but not least, we define out `Tick`:

```c++
void AFlockingGameModeBase::Tick( float DeltaTime ) {
    Super::Tick( DeltaTime );
    Manager->Flock();
};
```

Not much to see here, but note that our game mode is calling `Flock()` once per frame .

## The Manager class
Our `FlockingManager` will be a generic `UObject`, a fundamental class in Unreal that almost all game objects (including Actors, Pawns, and Game Modes) inherit from. We could instead make it an `Actor` but this didn’t make sense to me as it doesn’t use many `Actor` features, such as location and orientation. 

Let’s get a stub for this class so we can make sure our game mode additions compile correctly before implementing too much of our manager. The header should be:

```c++
#pragma once

#include "CoreMinimal.h"
#include "FlockingManager.generated.h"

UCLASS()
class FLOCKING_API UFlockingManager : public UObject
{

public:
	GENERATED_BODY()

	void Init( UWorld *world, UStaticMeshComponent *mesh );
	void Flock();

private:
	UWorld *World;	
	bool initialized;
	TArray<class AAgent *> Agents;
};
```

Since we’re not inheriting from `Actor` we need to pass a pointer to our manager on initialization (actors have a method that easily grabs this). We also accept a static mesh component that we’ll use for our agents. 

`Flock` will be the main simulation method that will be called each tick. `initialized` will let us know when we’ve placed our agents into the world, and the `Agents` array will store all of them (side note:  you’ll be using `TArray`s a lot in your games, [best to read up on them now](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/TArrays/index.html) !) Finally, we’ll store a reference to the `UWorld` object that’s passed it our `Init` method in `World`.

The calls to `UCLASS()` and `GENERATED_BODY()` are used for code generation and reflection, to enable UE4 to know about this class and manage associated memory.

OK, let’s add in implementation stubs for our two methods:

```c++
#include "FlockingManager.h"

void UFlockingManager::Init( UWorld *world, UStaticMeshComponent *mesh ) {
	UE_LOG(LogTemp, Warning, TEXT("MANAGER INIT"));
}

void UFlockingManager::Flock() {}
```

Go ahead and see if everything compiles. 

## Hooking up the Game Mode
In order to get our game mode to work, we need to do a couple of things. First, to get it so we can configure out static mesh, we need to make Blueprint subclass of our game mode base. You can do this by selecting the game mode base C++ class in your content browser, right-clicking on it and choosing “Create Blueprint class based on FlockingGameModeBase”.  If you double-click on the resulting blueprint, you can navigate to Flocking Game Mode Base > AgentMesh > Static Mesh > Static Mesh and choose a mesh to represent your agents in the world. Make sure to save and compile the blueprint.

Next, under Main Menubar > Edit > Project Settings > Maps & Modes > Default Modes > Default GameMode choose your new blueprint from the associated dropdown menu.

If you play your game now you should see the “MANAGER INIT” message printed to your Output Log window.

## Getting some agents on the stage
Go ahead and make a new Actor C++ subclass, and name it “Agent”. Our agent will be responsible for updating its position, storing its velocity, and representing itself via a mesh component. Its header should be the following:

```c++

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Agent.generated.h"

UCLASS()
class FLOCKING_API AAgent : public AActor
{
	GENERATED_BODY()
	
public:	
	AAgent();
	void Init( UStaticMeshComponent *mesh, int id );
	UStaticMeshComponent * Mesh;
	FVector Velocity;

protected:
	virtual void BeginPlay() override;

public:	
	virtual void Tick(float DeltaTime) override;
};
```

The only elements we need to add to what Unreal gives us are the `Init()` method signature, and the `Mesh` and `Velocity` members. Everything else remains the same as the defaults for the `Actor` subclass.

Now let’s setup our Agent implementation:

```c++
#include "Agent.h"

AAgent::AAgent(){
	PrimaryActorTick.bCanEverTick = true;
	Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("AgentMesh"));	
	RootComponent = Mesh;
	Velocity = FVector(0.f);
}

void AAgent::BeginPlay(){
	Super::BeginPlay();
}

void AAgent::Init( UStaticMeshComponent *mesh, int id ) {
	UE_LOG(LogTemp, Warning, TEXT("Agent initialized.") );
	Mesh->SetStaticMesh( mesh->GetStaticMesh() );
}

void AAgent::Tick(float DeltaTime){
	Super::Tick(DeltaTime);

	FVector loc = GetActorLocation();
	SetActorLocation( loc + Velocity );
}
```

Our constructor performs some basic initialization that we’ve looked at before. In our `Init` method, we accept a mesh component as an argument and then grab its mesh and assign it to our mesh component.  Our `Tick` function simply updates the current position of our agent based on its velocity. Because the velocity is initialized to 0, at this point our agents won’t be moving.

Almost there! Just need to place our agents into the world.

## Rise, agents, rise
Let’s change the `Init` method of our FlockingManager so that it places agents into the world. We’ll place them in a ring on the XY axis to start. We’ll also create a macro to define the number of agents we’re using.

```c++
#define AGENT_COUNT 10

void UFlockingManager::Init( UWorld *world, UStaticMeshComponent *mesh ) {
    UE_LOG(LogTemp, Warning, TEXT("Manager initialized"));
    
    World = world;
    float incr = (PI * 2.f) / AGENT_COUNT;
    for( int i = 0; i < AGENT_COUNT; i++ ) {
        if( World != nullptr ) {
            FRotator rotation = FRotator();

            FVector location = FVector();
            location.X = FMath::Sin( incr * i ) * 150.f;
            location.Y = FMath::Cos( incr * i ) * 150.f;

            AAgent * agent = World->SpawnActor<AAgent>( location, rotation );
            agent->Init( mesh, i );
            Agents.Add( agent );
        }
    }

    initialized = true;
}
```

We’re using a bit of math to create the initial position information, but don’t worry about that if your trig is rusty. The main thing to note is the three lines that spawn each agent, initialize it, and add it to our `TArray`. 

OK! That’s enough to spawn our agents and get them into the world, updating their position each frame according to their velocity. Your job is to now implement `UFlockingManager::Flock` so that it carries out the  [various rules of Boids](https://github.com/imgd-4000-2020/syllabus-and-notes/blob/master/boids_assignment.md) . Good luck, and remember to ask questions on the Slack!
