Nylon separates the concepts of Logic and Display. This means that you can create and execute NylonScenes (more on those in '3. Your First Scene') entirely without needing to display anything on the screen. This can be useful for networking, or building non-dialogue related systems with Nylon.

This also means that the way you display your dialogue is entirely up to you! See '4. Your First Reader' for details.

For testing purposes Nylon comes with a default NylonReader implementation, this is the NylonTestReader. And it is meant specifically for you to test and see if Nylon was correctly installed, and get something on screen without further effort.

### Usage
The NylonTestReader is available as a Node within the Godot Editor. You can add it to a Scene of your choice. Either search for it as 'NylonTestReader', or find it under Control > Container > PanelContainer > NylonTestReader

**You must set a custom_minimum_size** for the Reader to display. Try 512x512, or use the Anchors and Margin preset to cover the whole screen.

### Testing
If you have installed the example NylonContent/ directory as part of installing Nylon, then you will have a number of example scenes available to you to try things out. To use them:

1. Create a New Script in the same Scene you added the NylonTestReader to.
2. Copy the following code:

```GDScript
extends Node2D

# This is the default File Path
const NYLON_EXAMPLE_SCENE: PackedScene = preload("res://NylonContent/Scenes/Examples/Example.tscn")

func _ready() -> void:
	# Starts the Scene after instantiating it.
	# NylonScenes are NOT added to the SceneTree.
	# Nylon will memory manage them for you.
	Nylon.start_scene(NYLON_EXAMPLE_SCENE.instantiate())
```

3. Run your project.

You should now see the NylonTestReader panel populate with the example scene.

![[Pasted image 20250601012540.png]]