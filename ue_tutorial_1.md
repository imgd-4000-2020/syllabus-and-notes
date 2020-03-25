## Unreal Editor Tutorial #1
Goal: Make a (arguably) prettily lit scene from **scratch**, with some physics, collisions, and C++ hit detection. Along the way we’ll look at some coding in Blueprints, some C++ coding, and the general use of the Unreal Editor.

### Getting started
This tutorial assumes that you have both [Unreal Engine 4.24.3](https://www.unrealengine.com/download) and Visual Studio or VSCode installed. CLion is another option and [there are free educational licenses available](https://www.jetbrains.com/community/education/#students). If you **don’t** have both Unreal and an appropriate editor installed, no worries… just follow along with the screencast and then try things out later (you can actually get through most of today with just the Unreal Editor). Or if you just learn better by watching, feel free to just watch, listen, and ask questions.

We’re going to go ahead and start a new blank game project, and then populate a blank level with lighting, plugins, and meshes to create a simple scene. We’ll then add physics / collisions to the the scene, make a new glowing material using Blueprints, and then add some interaction by writing some (relatively) simple C++.

#### Creating our project / level
Go ahead and launch Unreal Editor (henceforth known as UE4). You should see a window with various templates to select from; choose “Game”. From the resulting menu, choose “Blank” when it asks what type of game to setup for you. Change the programming method from “Blueprints” to “C++”. 

The result Is not very “blank” at all. There’s a bunch of lighting and even a mesh or two setup in the level that Unreal creates by default. That’s OK; we can easily replace this with a truly blank level. But first let’s go ahead and fly around in the default level Unreal has given us. Hit the “Play” button to do this; you can then hit “Escape” to leave the level simulator and return to the editor. The WASD keys and the trackpad will let you navigate while the level simulator is engaged; use the Unreal documentation to get [more information about viewport control](https://docs.unrealengine.com/en-US/Engine/UI/LevelEditor/Viewports/ViewportControls/index.html).  

After you’ve flown around the default level a bit, go ahead and create a new level. Select File > New Level and choose “Blank Level”. Alternatively you can just delete everything the default project level gives you. Tip: If you go to Preferences > General > Loading & Saving > Startup > Load Level at Startup you can select what level is generated when you first create a new project.

### Populating our new level
Levels consist of meshes, lighting, blueprints, and other assets working in tandem to create a scene. Let’s start by adding a simple Cube Mesh to our level. In the editor in the lefthand pane, take Basic > Cube and drag it out into the level. In the Details view that appears on the right (beneath the World Outliner) go ahead and set Transform > Location to be 0,0,0 and set Transform > Scale to be 150, 150, .1. 

OK, we have a mesh that serves as a “ground”, but there are currently no lights in our level. As a result, if you hit the “Play” button, you won’t see anything appear. To correct this, drag Lights > Directional Light out and above our ground mesh. You should see a warning that the lights need to be “rebuilt”; this is basically a shader compilation / code generation step. Go ahead and hit “Build”. When the lighting is built hit "Play” and you should now see your mesh illuminated.

Doesn’t look so great at the moment, does it? We can change things a bit by playing around with controls for the light found in the contextual “Details” view, such as the light intensity (try lowering this significantly, to around 3.) and the light color. Still a bit boring, right?

### Adding a material
Let’s add a material object to our mesh that will determine its color / texture. If you didn’t add the Starter Content pack when you created your project, go to Content Browser (bottom left corner of the editor) > Add New > Add Feature or Content Pack > Content Packs > Starter Content  and then hit “Add to Project”. This will give us a bunch of materials to play around with.

Select our ground mesh, and then click on the dropdown in Details > Material that currently reads “Basic Static Mesh”. Scroll through all the different options you find there and try one or two out. When you’re done exploring, do a search for “Hex” and then choose “M_Tech_Hex_Tile_Pulse”. Once the material is applied things should look a little more interesting.

### Editing a material with Blueprints
OK, let’s add a new mesh to the scene, and then edit its material to change it’s color.

1. Add a Basic > Sphere mesh to your scene.
2. Set the sphere’s Transform > Scale to be 15,15,15,
3. Set the sphere’s Transform > Location to be 0,0,2000

You’ll note that the sphere doesn’t currently have much color, due to its default material and the current lights in the scene. Let’s create and edit a new material for the sphere.

1. With the sphere, selected click on the dropdown in the Details > Materials and select Create New Asset > Material.
2. This will launch Blueprints, the visual programming environment for Unreal. You will currently see one node, which contains the default parameters for a material. We will feed a couple of new values to this node.
3. Right-click on the blueprint. This will bring up a node creation menu where you can select from all the various blueprints nodes. Type “VectorParameter” to enter it into the search field and then hit enter when the “VectorParameter” node is selected. Name the new node “Color”.
4. Drag the topmost output of the new VectorParameter node to the “Base Color” input of our material. Click on the color editor to choose a new color.
5. Right-click on the blueprint and create a new ScalarParameter node, and name it “Metallic”. Connect its output to “Metallic” input of our material.  Set the default for the parameter to be 1. 
6. Last but not least, we need to “Apply” the changes in our Blueprint (later we’ll learn how to change material parameters on-the-fly). Click the apply button and go back to the main editing window.
7. You should see some changes to the sphere. If not, try playing with the intensity of your directional light and its z-positioning (make sure it’s above your sphere).

OK, that’s a basic introduction to playing around in Blueprints. 

### Adding a plugin
Let’s create a beautiful sunrise for our scene, because all our scenes should have beautiful sunrises. This will also give us a chance to see how plugins work. Before we get install the plugin, we need to save the project and our current level. Press the “Save Current” button in the main toolbar, and give your level a name. Then choose File > Save All just to make sure all our project details are saved. 

Here’s a link to the tutorial on [Geographically Accurate Sun Positioning | Unreal Engine Documentation](https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/SunPositioner/index.html) . Complete the first three steps, the last of which will require you to restart the Unity editor. Instead of continuing, follow the steps below (although feel free to come back to this tutorial and experiment later).

In your Editor Modes panel, drag Lights > Sun and Sky into your level. This will blow out your scene with white light, but if you select the SunSky in your World Outliner you’ll be able to to change the current time of day under Details > Time > Solar Time. Set the value to be 6 (for 6AM). Now you should have a nice sunrise.

### Adding some physics
It doesn’t get much easier than this. Select our sphere (which should currently be positioned above our ground). In Details > Physics, check the “Simulate Physics” option and make sure “Enable Gravity” is checked as well. Hit Play.

You should see your sphere fall to the ground and then stop. We can get more interesting movement by slanting the ground. Select your ground mesh, and then set its Transform > Rotation > Y to be 200 degrees. Try hitting play again… we now have a rolling ball.

Let’s add an obstacle. Drag out a Basic > Cube mesh from our placement menu. Position and scale it so that it becomes an obstacle for the rolling movement of our sphere; keep testing until the sphere actually strikes the cube.

### Writing some code
Let’s quickly add a hit test for when our cube is struck.
1. Select the cube in the editor
2. In Details, select “+ Add Component”
3. Choose the Actor Component, and name the new component “Actor Collide”
4. Right-click the new component and select C++ > Open ActorCollide.h
5. We need to paste a signature for our hit test into the header file. Add the following under the first `public` section of the class definition:
```c++
public: 
	// Sets default values for this component's properties
  UNewActorComponent();
// ... paste begins here
	UFUNCTION()
	void OnActorHit(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComponent, FVector NormalImpulse, const FHitResult& Hit );
```

6. Now open up the corresponding .cpp file. We want it to look like this:
```c++

#include "NewActorComponent.h"

// Sets default values for this component's properties
UNewActorComponent::UNewActorComponent()
{
	// Set this component to be initialized when the game starts, and to be ticked every frame.  You can turn these features
	// off to improve performance if you don't need them.
	PrimaryComponentTick.bCanEverTick = true;
}

// Called when the game starts
void UNewActorComponent::BeginPlay()
{
	Super::BeginPlay();
	UE_LOG(LogTemp, Warning, TEXT("begin"));

	AActor * actor = GetOwner();
	UStaticMeshComponent * cube = actor->FindComponentByClass<UStaticMeshComponent>();

    cube->OnComponentHit.AddDynamic(this, &UNewActorComponent::OnActorHit);
	
}

// Called every frame
void UNewActorComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	//UE_LOG(LogTemp, Warning, TEXT("tick"));
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	// ...
}

void UNewActorComponent::OnActorHit(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComponent, FVector NormalImpulse, const FHitResult& Hit ) {
  UE_LOG(LogTemp, Warning, TEXT("HIT!"));
}
```

7. Back in UE4, make sure both the rolling sphere and your cube are generating hit messages. You can find this in Details > Collisions > Simulation Generates Hit Events.

8. Hit Compile
9. Hit Play
10. Celebrate???

### Your lightspeed tour of UE4 comes to an end
OK, that was actually quite a lot. We create some lights, added some meshes, turned on physics, edited a material in Blueprints, generated a C++ component, and got a basic hit test to work. There’s a chance we didn’t get all the way through this on the first day of class… if so we’ll finish up tomorrow.

If you were watching but not following along, part of your homework tonight is to explicitly go and work through this on your own.
