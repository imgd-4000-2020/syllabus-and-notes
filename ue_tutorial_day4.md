## Goal
Today we will add two features to our project from last class. Both will take advantage of Blueprints, and part of the goal of this class is to spend a bit more time in the various environments Blueprints gives us.

1. We’ll attach a particle system (electricity) to connect our player and our target when we hit the target with a trace.
2. We’ll create a simple GUI to display, for example, player health.

Those two tasks don’t sound like much, but it’s actually a pretty gnarly. Sigh…

**IF YOU HAVEN’T DONE IT YET, PLEASE DOWNLOAD THE CONTENT EXAMPLES PROJECT FROM THE EPIC GAMES LAUNCHER.** You can find it under the “Learn” tab by simply scrolling down.

## Adding a particle system
### Importing our particles
In our [last class](ue_tutorial_day3.md) we added a line trace that checked to see if the “shot” our sphere pawn fired intersected with an enemy cube (spheres are natural enemies of cubes natch).  Now we’re going to add a particle emitter that will be placed when a hit occurs… this effect will create a stream between the player and the target object.

First thing we need to do is “Migrate” our particle effect from our Content Examples project. Open the Content Examples project now. If the content browser window isn’t open, open it with Window > Content Browser > Content Browser 1. In the content browser do a search for particle system. Find the system named “P_electricity_arc”. Right click on it, and then choose Asset Actions > Move > Migrate. You will then see a window depicting all the assets that will be exported; hit “OK. Last but not least you will be asked to choose the destination. Navigate to the “Content” directory from the project we worked on in the last class and hit the “Open” button (or the equivalent for Windows/Linux).

Great! You’ve now added your particle system to the project. Go ahead and open the project from last class now (this will close the Content Examples project).

### Creating a “hit” event for Blueprints
We’re going to edit / dynamically place our particle emitter using Blueprints, but before we can do that we need to create an event that will tell us when our hit test actually strikes the enemy cube. 

We’ll add this “HitSomething” function to the header for our `SpherePawn` class:

```c++
UFUNCTION(BlueprintNativeEvent)
void HitSomething(class UStaticMeshComponent *m);
void HitSomething_Implementation(class UStaticMeshComponent *m);
```

Important (and bizzaro) things to point out, that all relate to the bindings between these C++ classes, the editor, and the reflection engine:

1. The `BlueprintNativeEvent` specifier will enable us to create a node in Blueprints that outputs an event trigger.
2. We need to create two separate functions. One generates our blueprints event, and one contains an implementation for the event. 
3. You must name the implementation the same as the Blueprints event + `_Implementation` or the compiler will throw an error. This is a very specific way to create bindings between C++ / Blueprints / the editor. Ugh.

OK, with that out of the way, we can all our `HitSomething` method from our `Fire` method (assuming we do actually hit something). In the `SpherePawn` C++ file, switch the following line in the `Fire` method:

```c++
hr.GetActor()->Destroy();
```

 to the following:
```c++
HitSomething_Implementation(Cast<UStaticMeshComponent>(hr.GetActor()->GetRootComponent()));
```

The root component of our cube will typically (but not necessarily) be a StaticMeshComponent, so we’ll cast it as such, and then pass it to our `HitSomething_Implementation`, which we’ll define as below:

```c++
void ASpherePawn::HitSomething_Implementation(class UStaticMeshComponent *mesh) {
	HitSomething(mesh);
	UE_LOG(LogTemp, Warning, TEXT("HIT SOMETHING CALLED"));
}
```

Note that calling `HitSomething` (which we defined as a UFUNCTION with the BlueprintNativeEvent specifier) is what actually triggers the event in Blueprints. We have to include that call.

### Into the Blueprint
OK, so now we want to make a Blueprint “subclass” out of our `SpherePawn` class. Go ahead and delete previous instance of `SpherePawn` that we had placed in the scene. Next, right-click on the SpherePawn class in your content browser (or in the Modes > Place > All Classes pane)  and choose “Create Blueprint class based on SpherePawn”.  Name the new blueprint `SpherePawn_BP`.

Go ahead and open up the resulting Blueprint. Click on the “Event Graph” tab to open the visual programming environment. We’re not going to use any of the events they give us, instead we’ll use the one we just added in C++.

Right-click in any of the blank space and then type “Hit” to search through the available nodes. You should see a “Event Hit Something” appear in the dropdown menu; select it. The arrow at the top of this node is the control flow for event processing, while the blue output marked `M` is the mesh that has been hit.

Great! Now let’s spawn our particle emitter. Right-click on the Blueprints canvas again and search for “Spawn Emitter Attached”. Connect the topmost control flow output from our “Event Hit Something” node to the topmost control flow input in our “Spawn Emitter Attached” node. 

Next, we need to select the particle system that will be spawned. Click on the “Emitter Template” input for the spawn node, and then select the “P_electricity_arc” that we migrated to this project earlier.

Almost there! Connect the “M” output to the “Attach to Component” input of our spawn node. Last but not least, set “Location Type” on the spawn node to be “Snap to Target, Including Scale”.

Last step: drag an instance of your `SpherePawn_BP` component into your level. Assign it a mesh of your choosing for representation. Last but not least we need to tell the pawn to “possess”. Make sure the SpherePawn is selected (not the mesh component) and then choose Pawn > Auto Possess Player > Player 0 from the 	Details tree menu.

OK! Compile your blueprint, hit Save, and then hit Play. You should see the particle system placed when you hit the cube with your line trace. It might help to make the cube a bigger target (by scaling it) so it’s easier to hit.

### Orient our particles
Right now, although the particles are placed correctly, there not oriented to create a connection between our player and the cube. Let’s change this. First we need to edit the particle system itself. Find it in the Content Browser (it should be in the ExampleContent folder) and then double click on it to edit. 

Click on the “Source” property of the “beam” emitter in the particle editor. Change its value to be “Emitter”. This means the particles will begin at the source of the emitter. Next click on the “Target” property of the beam emitter and set it to be “User set”. This will enable us to set its value from Blueprints. Save the particle blueprint and close the editor; alternatively, feel free to play around with some of the other properties (color, size, etc.)

Back in the blueprint for our “SpherePawn”, drag the “Mesh” component into our canvas from the Components panel in the upper left corner; this automatically creates a Mesh node that will represent whatever Mesh we assign to our `SpherePawn`. Drag off a connection and release, this will bring up the new node dialog box. Search for `GetWorldLocation`, this node will find the location of the mesh in the world. Once selected, the output of your mesh will be automatically connected to the relevant input (in this case there’s only one, `Target`) in the new node that is created.

OK, our next step is to set the target location of our beam. Drag a pin off of the “Return Value” output of our `GetWorldLocation` node, and then do a search for the “Set Beam Target Point” node; the location will be auto-connected to the “New Target Point” property. 

Connect the control flow output of our Spawn Emitter to our Set Beam Target Point control flow input, and then connect the Return Value (which represents the emitter) of the Spawn Emitter node two the “Target” attribute of the Set Beam Target Point node.  Compile your Blueprint, save it, and hit Play. You should now have correctly oriented particles!

### Set the particles to flow between pawn and enemy
Last but not least we’ll set the particles to flow between the pawn and the enemy. To do this, all we need to do is keep calling Set Beam Target Point with a brief delay.  Drag a pin out of the control flow output from Set Beam Target Point, and do a search for Delay. Set a duration of .05 (in seconds) by typing it into the property field. Last but not least, connect the output of the delay to the control flow input of the Set Beam Target Point. This, in effect, creates a recursive function that calls itself over time, constantly updating the location of the player and thus changing the positioning of the particle beam. 

Save and compile the blueprint, and then test your scene to see the results. Your final blueprint should look something [like this](./images/day4/beam_blueprint.png)


Adding a GUI
We’re going to add a simple parameter to our `SpherePawn` class named `Health`. This will automatically tick down every frame, and we’ll create a HUD with an animated bar to show the current value. We’ll then also add a constructor method to our `GameMode` subclass that will place our GUI on the screen. Last but not least, we’ll make the GUI itself!

### Adding the C++ for our SpherePawn
This one is pretty simple. We’re going to add a property named Health, set its initial value in the `SpherePawn` constructor, and then decrease it slightly on the `Tick` method of `SpherePawn`.

In our `SpherePawn.h` header, add the following public property:

```c++
UPROPERTY(BlueprintReadOnly)
float Health;
``` 

Blueprints will be able to read this value, but only our C++ methods will be able to change it.

In our `SpherePawn` C++ file, set the initial value of `Health` to be `1.f` in the constructor. Change the `Tick` method, to the following:

```c++
void ASpherePawn::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	Health -= .0001f;
}
```

OK, that’s all we need to do here! The SpherePawn now has a Health property that will automatically decrease on each frame, and this property is set to be readable by Blueprints. Now, on to use the Unreal Motion Graphics framework.

### Modifying our build file
Go ahead and open the C# build file for your project, located alongside the rest of your project’s source code. This configures many of the default libraries that are included in your build. Add “UMG” to the first dependency module configuration, so it reads:

```c++
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "UMG" });
```

Next, uncomment the Slate UI line immediately beneath the one you just edited. Nothing else to see here; on to editing our game mode.

### Editing our GameMode subclass
The GameMode is responsible for broadly managing the game, including displaying relevant statistics via HUD, managing win/lose conditions and more. This is the class we’ll use to go ahead and display the GUI we create using the Unreal Motion Graphics editor (UMG).

Go ahead and open up your project’s `GameModeBase` subclass header;  the Unreal project template will have made this for you automatically. Add the following public properties to your class:

```c++
public:
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Health")
	TSubclassOf<class UUserWidget> HUDWidgetClass;
	
	UPROPERTY()
	class UUserWidget * CurrentWidget;

	virtual void BeginPlay() override;
```

Next, add override the `BeginPlay` method in the C++ file and include the UserWidget header file. Make sure to change your project name where appropriate, and don’t remove the A prefix in the class name that denotes this an Actor:	

```c++
#include "YOURPROJECTNAMEGameModeBase.h"
#include "Blueprint/UserWidget.h"
```

```c++
void AYOURPROJECTNAMEHEREGameModeBase::BeginPlay() {
    Super::BeginPlay();

    CurrentWidget = CreateWidget<UUserWidget>(GetWorld(), HUDWidgetClass);
    CurrentWidget->AddToViewport();
}
```

### Create a Blueprint to display our Health.
In the editor content browser, 	right-click and choose Create Advanced Asset > User Interface > Widget Blueprint. Name it “Health HUD”.

In the designer view, layout a simple Progress Bar widget and place an accompanying text label. In the Graph view, create a [blueprint like this image](./images/day4/widget_blueprint.png)

Compile and save this blueprint.

### Link everything together

1. Create a blueprint subclass of our `GameModeBase`. Name it “GameMode_BP”.
2. Double click on it and set the Health > HUDWidgetClass to be our “Health HUD”.
3. Choose Edit > Project Settings > Maps and Modes > Default Modes > Default GameMode and set it to be “GameMode_BP” using the associated dropdown menu.

Try Compiling the C++ code if you haven’t done so in a while, and then run Play. You should see a health bar counting down!
