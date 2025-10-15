# Features
Save games are associated to a UserProfile, see [[User Profiles]] for details on profile management.

Saves are stored in the `user://profiles/<the_profile>/` directory.

Saves are named sequentially using unix time stamps to maintain their creation order.

There are three 'types' of savegames. Generic, Autosave, and Quicksave.

When creating a save game, backups are automatically created, and each type of savegame can maintain a (un)limited amount of historical saves.

# Important Functions

These are some important functions for managing save games. Importantly the responsibility for creating new save files lies with the UserProfile class, usually accessed via `App.profiles.current`, while managing the data contained within the file is handled via `App.save`, which is a helper class for this purpose.

## Gathering Save Data
`App.save.gather_data_to_save()` is a helper function for gathering data to stage. Specifically, this function will query every Autoload in your project for a `get_save_data()` function, which must return a dictionary.

Example:
```GDScript
# We are about to make a quicksave.
# 1. Collect the game state
App.save.gather_data_to_save()
# 2. Store the data
App.save.save_to_quicksave()
```

Example of an Inventory Autoload which can be saved using this method. Note how it calls into its children objects to gather their save data in a recursive fashion.
```GDScript
class_name Inventory

func get_save_data() -> Dictionary:
	var d: Dictionary = {}
	d.dict_type == "inventory_save_data"
	var item_data: Array[Dictionary] = []
	for item: Item in items:
		item_data.append(item.get_save_data())
	d.item_data = item_data
	return d
```

## Applying Sava Data
`App.save.apply_sava_data()` is a similar helper function, which will call the `apply_save_data()` function on all Autoloads that possess it.

Example:
```GDScript
# We are about to loada quicksave.
# 1. Get the list of quicksaves.
var quicksaves: Array[SaveFile] = App.profiles.current.get_quick_saves()
# 2. Pick one to load, the newest will always be last
var save: SaveFile = quicksaves[-1]
# 3. Pull the data from the file into the helper.
App.save.stage_save_file(save)
# 4. Now we can apply the save data.
App.save.apply_save_data()
```

Example of the same inventory class.
```GDScript
class_name Inventory

func apply_save_data(data: Dictionary) -> void:
	if not data.get("dict_type") == "inventory_save_data":
		return # ERROR
	
	for item_data: Dictionary in data.get("item_data", []):
		var item: Item = Item.new()
		item.apply_save_data(item_data)
		items.append(item)
```

## Manual Data Management

If instead of using the gather and apply functions you prefer to put and fetch data yourself. You can instead use the `stage_<type>()` and `get_<type>()` functions in `App.save` implemented in AppSaveData.gd.

```GDScript
# Sets the tutorial_completed flag to true.
App.save.stage_bool("tutorial_completed", true)

# Retrieve the value later.
var tutorial_enabled: bool = App.save.get_bool("tutorial_completed", false)
```

# Creating Save Files

Once you are happy with the data you have gathered and staged. You can use the following methods to quickly save to disk:

`App.save.save_to_file(path)` For manual file management and overriding.

`App.save.save_to_new_file()` Creates a new generic save file.

`App.save.save_to_last_file()` Overrides the last generic save file.

`App.save.save_to_quicksave()` A maximum of 3 quicksaves are kept.

`App.save.save_to_autosave()` A maximum of 3 autosaves are kept.
