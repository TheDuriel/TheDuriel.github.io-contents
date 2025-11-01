This page replaces "howtoscenetree.png" and "howtoscenetree2.png" If you know them. Delete them. @export reigns supreme in Godot 4.

![[Pasted image 20250625170135.png]]
# A Node
So you want to access the properties or functions of another Node?

### In the same Scene
If you can see the node in the same scene as the one from which you are trying to access it. Use @export

```gdscript
class_name House

@export var door: Door

func _ready()
	door.open()
```

Then assign the Door to the exported property.

### In another Scene
If both scenes are siblings within the same parent scene. Say, there is a LightSwitch and a Light inside the same House, then you can use @export like you would if they were in the same scene.

If the target is not available in the same scene. But is hidden within a subscene, you may be able to use the parent scene as the communication relay.

```gdscript
class_name World

@export var house: House
@export var switch: Switch

func _ready() -> void:
	switch.target = house.light
```

```gdscript
class_name House

@export var light: Light
```

```gdscript
class_name Switch

var target: Switchable

func _on_switch_toggled(enabled: bool) -> void:
	target.enabled = enabled
```

### Somewhere else entirely
If there is no scene hierarchy through which the objects can easily access each other. Consider using an Autoload as the relay. I prefer ID based structures for situations in which there are **many of the same kind of thing.** In other scenarios, simply having a property in an autoload you assign to, can work fine. (Like keeping a Global.player reference to the player object. Or the Interface.)

```gdscript
class_name Interactibles

var _interactibles: Dictionary[String, Interactible] = {} # ID : Object

func add_interactible(interactible: Interactible) -> void:
	# Adds to the dictionary and performs any needed initialization.

func get_interactible(id: String) -> Interactible:
	# Returns the object if it exists.
```

```gdscript
class_name Switch

@export var target_id: String
var target: Interactible:
	get: return Interactibles.get_interactible(target_id)

func _on_switch_toggled(enabled: bool) -> void:
	target.enabled = enabled
```

### Dynamically created
When the object is dynamically created, then the easiest thing is to just keep the reference around instead of throwing it away.

```gdscript
class_name House

var front_door: Door

func _ready() -> void:
	front_door = Door.new()
	add_child(front_door


func _on_lever_activated() -> void:
	front_door.open()
```

# A Resource
So you want to access the properties of a Resource?

### From a File (.tres, .res, etc.)
Loading a resource will return an instance of it. No matter how many times you load a resource, you will always get the same Instance. This is because each Resource File represents a specific Instance.

`load("PlayerConfig.tres")` will always return the same resource.

```gdscript
class_name House

var house_config: HouseConfig = load("HouseConfig.tres")
# Alll Houses will have the same resource.
# No matter which House changes this resource.
# All Houses will access the same resource, and see the same changes.
```

When using @export to apply a Resource to a Class. You can use the "local to scene" toggle in the Resource itself. To automatically run the following code snipped, and thus get each house a Unique copy.

**This is what the 'local to scene' setting actually does.**
```gdscript
func _init() -> void:
	if house_config.local_to_scene:
		house_config = house_config.duplicate()
```

### From a Node
Access the Resource as a member property.

```gdscript
class_name ResourceManipulator

@export var node_with_resource: Node
var the_resource: Resource = node_with_resource.the_resource
```

### Shared between Nodes
The best way to access a Resource that is used in many different places. Is to **save it as a file**, and then load() it.

# A File? A Class?
Not sure? Lets examine some options.

### Static Things
To access `const` and `enum` properties of a Class Script. Give it a name.

```gdscript
class_name CarConstants

enum COLOR {BLUE, RED}
const SPEED: int = 100
```

```gdscript
class_name RaceCar

@export var color: CarConstants.COLOR
@export var speed: CarConstants.SPEED
```

### Dynamic Things
Make a new instance of the named class.

```gdscript
class_name CarStats

enum COLOR {RED, BLUE}
var color: Color = COLOR.RED

const SPEED: int = 100
var speed: int = 100
```

```gdscript
class_name RaceCar

@export var speed_override: int = 500:
	set(value):
	speed_override = value
	car_stats.speed = speed_override

var car_stats: CarStats = CarStats.new()
# This is unique to this specific car
```