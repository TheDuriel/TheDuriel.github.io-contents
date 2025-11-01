Saving a game can be difficult, loading a saved state even more so. This article tries to give an overview of my approach to saving, using a top down hierarchy and single pass instantiation process.

* Saving and Loading gets easier the better structured your project is.
* It gets even easier, when you plan around what you are willing to save.

For example, all games will need to save the players progress in completing the game. But do you need to save their exact position in the world? Their progress within an individual level? The state of a combat sequence?

These questions are both a question of game design, and a question of architecture. Making a combat system that can be paused, saved, and loaded, can involve more planning and effort than one that can't be. It's simpler to load a known checkpoint, than to track all possible changes in a game.

Regardless. The process of saving and loading a game can be roughly broken down into four steps.

# 1. Serializing the Game State
The first step is by far the most complex. Saving the wrong information in the wrong structure, can severely complicate the re-instantiating of the game state. Or make it outright impossible.

### The Pointer Problem
The biggest risk here is the "pointer problem". What if you have an Object A, which has a reference to Object B? How do we load A and restore this pointer? When a reference to an object is not something that can be stored.

Identifiers come to the rescue here.

```gdscript
class_name Cable
# Imagine a cable connecting an Outlet to a Machine
# The player can edit place Cables in the world, and define these connections.

var connection_a: ElectricObject
var connection_b: ElectricObject
# These are direct object pointers. And thus, can not be saved.

# However we can make it possible to save or recreate them, by using an ID as a proxy.
# You can generate an ID in a variety of ways. My Utilities repository contains a SimpleIDGenerator as an example.

# Example using IDs:

var connection_a_id: String = "outlet_a"
var connection_b_id: String = "machine_a"
var connection_a: ElectricObject:
	get: return World.get_object(connection_a_id)
var connection_b: ElectricObject:
	get: return World.get_object(connection_a_id)
```

Now you can store the connection IDs instead of the object reference itself.

`World.get_object()` is little more than a function that returns an object by an ID. World in this case, is an object which keeps a list of all objects and their IDs, to enable this. Of course, that list will also need to be saved.

### Top Down Saving
The principal idea at play is that we want to create a dictionary/struct of all the saveable data in our game. Using the same format as the object hierarchy of our game.

To this end, we begin by implementing a get_save_data() function on any object with saveable data. This function collects the required data from the object members, and packages it to be returned to the primary save system.

```gdscript
class_name House

var id: String = "house_a"
var furniture: Array[Furniture] = [a..., b..., c...]

# In the future, we will use Structs instead of Dictionaries.
func get_save_data() -> Dictionary:
	var d: Dictionary = {}
	d.dict_type = "HouseData"
	d.house_id = id
	
	var f: Array[Dictionary] = []
	for furniture_object: Furniture in furniture:
		f.append(furniture_object.get_save_data())
	
	d.furniture = f
	
	return d
```

Note how we generate an array of save data for the furniture of this house. This will allow us to instantiate the furniture later. It also means that the save file takes on a tree-like structure.

```gdscript
Save Dictionary {
	World.get_save_data()
		House.get_save_data()
			Chair.get_save_data()
			Couch.get_save_data()
			Fridge.get_save_data()
				Apple.get_save_data()
	}
```


### Using the Pointer Problem for Good
But if we are using ID's. Then shouldn't it be possible to simplify saving, and flatten the hierarchy? Yes.

If we assume that a top level object like World, already stores a list of all Objects within the world with all their IDs. Then **most of the time**, the hierarchy will look like this instead:

```gdscript
Save Dictionary {
	World.get_save_data()
		[house_a, house_b, ...]
		[chair_a, chair_b, ...]
		[apple_a, apple_b, ...]
		}
```

And instead of storing the data of their children, an  object only needs to store the ID.

Meaning that our house example would look like this:
```gdscript
class_name House

var id: String = "house_a"
var furniture_ids: Array[String] = [chair_a, couch_a, fridge_a,...]

# In the future, we will use Structs instead of Dictionaries.
func get_save_data() -> Dictionary:
	var d: Dictionary = {}
	d.dict_type = "HouseData"
	d.house_id = id
	d.furniture_ids = furniture_ids
	return d
```

# 2. Storing the Data
There are several things to consider when storing save data. Like, where to store your files, what to name them, or what additional information to include.

Personally I prefer the following approach:

```gdscript
class_name SaveFile

func save_game_to_disk(file_path: String)
	var f: FileAccess = FileAccess.open(file_path, FileAccess.WRITE)
	f.store_var(get_metadata())
	f.store_var(World.get_save_data())
	f.close()


func get_metadata() -> Dictionary:
	var d: Dictionary = {}
	d.user_profile = App.get_user_profile()
	d.time = Time.get_unix_time()
	return d
```

The final file will be a **binary** file containing two dictionaries.

The first dictionary contains meta information about the save. The username, the time, perhaps even a screenshot in Image format.

The second dictionary contains a big "blob" of our save data.

Why the split?
To make it easier to filter save files without loading their entire contents.

### What about those user profiles, and managing save paths?
I leave those as an exercise to you. Maybe your game doesn't bother with profiles. Maybe there is only a single autosave. Maybe you want a skyrim-like list of hundreds of save files.

My advice would be to use Folders to differentiate user profiles and playthroughs, and to tailor your implementation to the specific needs of your game.

### But I want to use JSON!!!
Using JSON is very simple if you modify your code as such:
```gdscript
class_name SaveFile

func save_game_to_disk(file_path: String)
	var f: FileAccess = FileAccess.open(file_path, FileAccess.WRITE)
	f.store_line(to_json(get_metadata()))
	f.store_var(to_json(World.get_save_data()))
	f.close()
```
Done.

But why would you want to use json?...
It may occasionally be useful for debugging. I suppose.

But consider that:
* JSON is not **required** to store data.
* JSON requires you to assume that all floats and ints stored, are floats. 1.
* JSON requires you to decompose vector types to x and y components. 1.
* JSON does not allow storing statically typed dictionaries and arrays.
* JSON is much slower to generate and load the larger your file gets.
* JSON isn't actually very human readable.
* JSON is incapable of storing actual objects. 2.

1. Godot has made improvements to its JSON parser to allow for some Godot types to be stored 'more reliably'. At the cost of compatibility with other JSON format reader libraries.

2. The above method that generates dictionaries for every object, can just as well serialize entire Godot objects. I do not recommend this. But all it would take is to annotate saveable properties of such objects with the correct @export flag, and to enable object serialization in the store_var() call, as an additional argument. Again, I do not recommend this. You are on your own if you do.

# 3. Retrieving the Data
This is by far the easiest step.

```gdscript
func load_save_data(file_path: String) -> void:
	var f: FileAccess = FileAccess.open(file_path, FileAccess.READ)
	var metadata: Dictionary = f.get_var()
	var save_data: Dictionary = f.get_var()
	f.close()
	# The first call to get_var() already moved the read header to the start of the second dictionary.
	# A function that only reads the metadata of a save file, only calls get_var() one time.
```

**But JSON?**
Replace the get_var() calls with get_line() and parse the resulting string as JSON and back into a dictionary. If you accounted for every caveat of using JSON, that will work just fine.

# 4. Re-Instantiating the Game State
Thanks to our top down hierarchical structure, re-instantiating the game state is a relatively simple affair.

### 1. Clear the current state.
To begin with, we must clear out the previous state and return the game to a blank state.

The simplest way to do this is to implement a clear() function in each top level object (autoload) and to discard the current scene.

The job of the clear() function is to, clear all member properties and reset them to their initial state.

In other cases, simply deleting the object and creating a new instance of it, will achieve the same result.

### 2. Apply the save data.
Now that the game state is cleared, we apply the stored save data. Lets use our World and House object from earlier as an example.

```gdscript
class_name World

func apply_save_data(d: Dictionary) -> void:
	assert(d.dict_type == "Worldata")
	for object_data: Dictionary = d.objects:
		var wo: WorldObject = WorldObject.new()
		wo.apply_save_data(object_data)
		add_object(wo)
```

```gdscript
class_name House

var id: String = "house_a"
var furniture_ids: Array[String] = [chair_a, couch_a, fridge_a,...]

func apply_save_data(d: Dictionary) -> void:
	assert(d.dict_type == "HouseData")
	id = d.house_id
	
	# We would call the same functions that the game already uses during gameplay.
	for furniture_id: String in d.furniture_ids:
		add_furniture(furniture_id
```