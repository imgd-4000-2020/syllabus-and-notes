## Tech Workshop
Goal: Create a very small world with a floating mesh that represents the player using an Pawn subclass. Configure InputController to use WASD keys for movement. Assign spacebar to “Fire” other instances of a separate “Projectile” class. Use a UPROPERTY value to configure speed of projectiles.

### Creating the Project / Level
We’re going to spend another day starting “from scratch” in Unreal. Although you will probably start your final project using a template, I’m hopeful that going through this process will help give you a better understanding of how everything is tied together.

1. Create a new blank game project that uses C++ and includes the starter content pack.
2. Create a new blank level.
3. From the Modes panel, drag a Basic > Plane mesh into your scene. Increase its X & Y scale to 20. Add a material of your choosing to it (I’ll use “Chrome”)
4. Drag a Directional Light into the scene
5. Drag a Visual Effects > Atmospheric Fog into the scene

OK, you should now have a lit scene ready to go… next thing to do is make our “Player”.

### Creating Our Pawn subclass (the player) and configuring movement
In Unreal, the `Pawn` class is used to represent the player and any NPCs / AI in the game. We’ll create a subclass of  `Pawn` and setup movement control via the WASD keys. We’ll also configure a camera to “follow” the pawn. 

1. In UE4, choose File > Create C++ class, and select Pawn. Name the class “SpherePawn”.
2. Add a `UPROPERTY` named `Mesh` that will hold the mesh representing our player. Make the property public in the new pawns header file:
```c++
	UPROPERTY(EditAnywhere)
	class UStaticMeshComponent * Mesh;
```
Remember that by using the UPROPERTY decorator we’ll be able to define this value from within UE4 itself.

3. Add two protected methods to the header:
```c++
void MoveForward( float Amount );
void MoveRight( float Amount );
```

4. UE4 has a global input manager that’s pretty simple to get started with. We want to configure our W key to move forward, our S key to move backwards, and our A_D keys to move left_right. Select Edit > Project Settings > Engine > Input to start this process.
5. In the Input settings menu, click the + button next to “Axis Mappings”, which are for continuous controls. Name the new control “MoveForward”. Assign the W key to this with a Scale value of 1. Create a new mapping and assign the S key to this with a value of -1. In our code this will ensure that S moves in the opposite direction of W.
6. Do the same thing for the A/D keys but to a new control named “MoveRight”.
7. While we’re here, let’s go ahead and create an action for the `Fire` method we’ll add to our Pawn in the next section. Create a new “Action Mapping” (these are for discrete events like buttons presses) and name it “Fire”. Map the spacebar to this action.
8. Let’s add a new component that will help us control the movement of our Pawn. There’s a variety of components for this, but we’ll use the `FloatingPawnMovement` component. Add the include to our Pawn’s header file:
```c++
#include "GameFramework/FloatingPawnMovement.h"
```
… and then create a protected member:
```c++
UFloatingPawnMovement * PawnMovement;
```

10. OK, let’s jump into our `.cpp` file for the Pawn.  Add the following two implementations:
```c++
void ASpherePawn::MoveForward( float Amount ) {
	PawnMovement->AddInputVector( GetActorForwardVector() * Amount );
}

void ASpherePawn::MoveRight( float Amount ) {
	PawnMovement->AddInputVector( GetActorRightVector() * Amount );
}
```

Then, in our constructor, instantiate `PawnMovement` and our `Mesh`:
```c++
ASpherePawn::ASpherePawn(){
	PrimaryActorTick.bCanEverTick = true;
	PawnMovement = CreateDefaultSubobject<UFloatingPawnMovement>("PawnMovement");
  Mesh = CreateDefaultSubobject<UStaticMeshComponent>("Mesh");
}
```

Last bit of C++. We need to bind the commands we setup in the projects input manager to our `MoveForward` and `MoveRight` functions. The Pawn class has a virtual method that gives us access to the `InputComponent`:

```c++
void ASpherePawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	PlayerInputComponent->BindAxis("MoveForward", this, &ASpherePawn::MoveForward);
	PlayerInputComponent->BindAxis("MoveRight", this, &ASpherePawn::MoveRight);	
}
```

11. OK, drag a copy of our `SpherePawn` into our level. Assign it a mesh in its details view. 
12. We need to tell UE4 that we want our `SpherePawn` instance to “possess” the main camera. You can do this by selecting the SpherePawn in the level, and then selecting “Player 0” from the Details > Pawn > Auto Possess Player.
13. Compile & Play. You should be able to navigate the world using WASD… although you can’t actually see the mesh you assigned to the player at this point.

### Creating our camera
We need to create a camera that will follow our player at a certain offset. 
1. Add our camera as a protected member to the header of our `SpherePawn` class.
```c++
UPROPERTY(EditAnywhere)
class UCameraComponent * Camera;
```
2. We _forward-declared_ our `UCameraComponent` in our header, which means we need to include it in our `.cpp` file: `#include "Camera/CameraComponent.h"`
3. Add our camera instantiation / setup the the `SpherePawn` constructor:
```c++
ASpherePawn::ASpherePawn(){
	PrimaryActorTick.bCanEverTick = true;
	PawnMovement = CreateDefaultSubobject<UFloatingPawnMovement>("PawnMovement");

	Mesh = CreateDefaultSubobject<UStaticMeshComponent>("Mesh");
	Camera = CreateDefaultSubobject<UCameraComponent>("CameraComponent");

	Camera->SetRelativeLocation(FVector(-500.f,0.f,0.f));
	Camera->SetupAttachment(Mesh);
}
```
This assigns an offset (which can be change in the UE4 editor under the Camera details of SpherePawn) and tells the camera to follow our mesh.
4. Go ahead and hit Compile and then Play in UE4. You should see that the camera follows the mesh now.

### Creating Our Projectile subclass
We’ll make a simple Actor to define our projectiles. These Actors will then be spawned (inside of our `SpherePawn` code) whenever the spacebar is pressed.
1. Create a new C++ class and make it an Actor. Name it “ProjectileActor”.
2. We only need to add two members to the header file. Make them both public, although we’ll probably only access the `Speed`parameter externally in this demo:
```c++
public:	
	AProjectileActor();
	class UStaticMeshComponent * SphereMesh;
	float Speed;
```
3. There’s some kinda tricky code to instantiate a new Sphere. We’ll do that in our constructor, in addition to initializing our `Speed` property. 
```c++
AProjectileActor::AProjectileActor()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
	SphereMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("SphereMesh"));

	static ConstructorHelpers::FObjectFinder<UStaticMesh>SphereMeshAsset(TEXT("StaticMesh'/Engine/BasicShapes/Sphere.Sphere'"));
	SphereMesh->SetStaticMesh(SphereMeshAsset.Object);

	Speed = 10.f;

	RootComponent = SphereMesh;
}
```

Note the last line… by specifying the `SphereMesh` as our `RootComponent` (a property every `Actor` has) we ensure that this mesh will be the primary representation of our Actor and be placed at the top of the component hierarchy.
5. Next let’s define our a tick method for our projectiles, that will control how they move once they have been spawned:
```c++
void AProjectileActor::Tick(float DeltaTime){
	Super::Tick(DeltaTime);
	FVector forward = GetActorForwardVector();
	SetActorLocation( GetActorLocation() + forward * Speed );
}
```
Hopefully this is fairly straightforward (hah, pun). We get the forward vector (the direction our projectile is facing… this will be set when the projectile is spawned) and then move it in that direction scaled by our `Speed` property.

All done! Make sure it compiles correctly, and then move on to firing it from our `SpherePawn` player.

### Firing our projectile subclass
1. First, we want to create a `Fire` method for our player. In our header, just add the following as a protected member:
```c++
void Fire();
```
2. Next, we need to bind the fire event of our input component to our `Fire`method. Add the following line to the `SpherePawn` `SetupPlayerInputComponent` method:
```c++
PlayerInputComponent->BindAction("Fire", IE_Pressed, this, &ASpherePawn::Fire );
```
3. Add initialization for our `ProjectileSpeed` property to the end of our `SpherePawn` constructor:
```c++
  ProjectileSpeed = 10.f;
```
5. Last but not least, we add our `Fire` method to our implementation:
```c++
void ASpherePawn::Fire() {
	FVector loc = GetActorLocation();
	loc.Z += 100.f;
	loc.Y += 50.f;
	
	AProjectileActor *a = GetWorld()->SpawnActor<AProjectileActor>( loc, GetActorRotation() );

	a->Speed = ProjectileSpeed;
}
```

4. Compile and Play. You should be able to fire via the spacebar now.  You can also adjust the speed of the projectiles using the `SpherePawn` details view in UE4.
