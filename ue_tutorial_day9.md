# IMGD 4000 - Day ?
• Unreal tips and tricks!
• Presentations: either Thursday or Monday, let me know what works for you. Cem might go today.  

## Python scripting demo
Discussion: Why we would want to script the editor?

### Tutorial:
• First things first… [the python API](https://docs.unrealengine.com/en-US/PythonAPI/index.html)  
• Enable Scripting plugins by going to Edit > Plugins > Scripting. We’ll want to turn on all three that you find there: Editor Scripting Utilities, Python Editor Script Plugin, Sequencer scripting… at the very least get the Python Editor plugin.  
• Try some scripts! You can enter them by choosing Python (REPL) in the the command bar of the Output Log window.  

### Set location of all actors in scene to zero
```python
actors = unreal.EditorLevelLibrary.get_all_level_actors()
for a in actors: 
    loc = a.get_actor_location()
    loc.z = 0
    a.set_actor_location(loc,False,False)
```

### Add a new actor to scene
```python
test = unreal.EditorLevelLibrary().spawn_actor_from_class(
	unreal.StaticMeshActor.static_class(), 
	unreal.Vector(0,0,0), 
	unreal.Rotator() 
)
```

Here’s a simple tutorial showing how you can expose a C++ command to Python scripting: https://www.youtube.com/watch?v=6bg1lTN67HA

## Blueprints debugging tutorial
In this tutorial we’ll walk through debugging a very simple Blueprint. We’ll set up some watch variables, look a the execution stack, and step through the blueprint.

1. Create a new, blank, Blueprints project
2. Choose Blueprints > Open Level Blueprint from main editor menubar.
3. Drag a pin off of the Event BeginPlay control output, and create a new Print String node.
4. Right-click on your Print String node, and choose Add Breakpoint.
5. Hit Play within the level blueprint editor. If you don’t have your Debug view open, choose Window > Debug to bring it up. You should be able to see the call stack, and also resume operation. Go ahead and stop debugging once you’ve confirmed this.
6. Drag the control pin off of your Print String node, and add a second Print String.
7. Drag the In String input off your new Print String node, and create an Append String node. Enter a value of “test” 
8. Create a new string variable for your blueprint and drag it into the editor. Connect it to input B of your Append String node. 
9. In your second Print String node (the one with the Append String node connected) right-click on the In String input and choose Watches > Watch this value.
10. Hit Play in the Blueprint editor. Debugging stops on the first PrintString value; note that are watched variable is currently empty (make sure your debug window is open!). Hit the Frame Skip button in the Debug window. The control flow jumps to our second Print String node, and now we can see our variable has been populated.

For more info see [the Unreal documentation on debugging](https://docs.unrealengine.com/en-US/Engine/Blueprints/UserGuide/Debugging/index.html)
