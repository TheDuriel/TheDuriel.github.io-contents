One current limitation of Nylon is that a NylonScene can not be paused and resumed at a later date. Scaffolding for this feature is in place and it may be officialized later.

This means that to save your game, you should ensure that all NylonScenes have completed first. Or, if your game is in the Visual Novel style, that the player has reached a 'checkpoint' at which the current scene is concluded, and a new scene can be started.

### What can be saved?
Nylon currently saves Scene Data Only. All scene variables can be saved and restored at a later time.

Nylon.get_scene_data() will return a Dictionary of ready to save data. No conversion required. This dictionary only contains String Keys, and bool, int, and string, values.

Nylon.load_scene_data() will accept this same Dictionary and apply it.

The specific structure of this dictionary is:

```
{
	scene_id = {
		flags = {
			flag_id = flag_value
		}
		ints = {
			int_id = flag_value
		}
		strings = {
			int_id = flag_value
		}
	}
}
```

It is ensured that this dictionary can be converted to and from Json should that be desired.


### What's a Scene ID?

A scene_id is a string key that associates a NylonScene and its Data.

For example, the SetFlag node may set the "has_eaten" flag to "true" for the scene within which it is running.

There are two types of scene_id's:
An automatically generated scene_id is set to all scenes when you first create the NylonScene Node. And a manually created id can be used to override this.

You may use this to let scenes share data. For example, if you group all your castle scenes to use the 'castle' ID, all NylonNodes accessing scene variables will access the same pool of data. Letting you build interconnected scenes easily.

### Additional Features

NylonScene has two flags which may be toggled to determine how it treats its data.

The 'persist' flag is enabled by default, and determines whether or not data is to be included in Nylon.get_scene_data()

Disabling this will make it so that closing out your game will delete all data for this scene.

The 'unique' flag will override the scene_id when the Scene is executed. Ensuring that it will not share data with any other scene, even itself. As an example, you may have a scene that is a random event in the forest, but which should start over fresh each time.