# i stands for instance

I use the lower case i to indicate a Class which can be instanced. This applies to GDScript only of course, where the lowercase i is not used in any other convention. (There are no plans to implement Interface in GDScript.)

Consider the following example of a standard class:

```gdscript
class_name MyNode
extends Node

func _init() -> void:
	name = "MyNode"
```

This class is instantiated using new()

```gdscript
var instance: MyNode = MyNode.new()
```

However, what if this class was actually representing a Scene? Imagine you were to create a new scene with this class as the Root. Using new() would fail to account for this, as it will create only the instance of MyNode, but fail to create the entire Scene.

For this, we use the i prefix, and ideally, a factory pattern. Consider:

```gdscript
class_name iMyScene
extends Node

var property: int = 0

static func make_instance(parameter: int) -> iMyScene:
	var i: iMyScene = load("res://UIDPATH").instantiate()
	i.property = parameter
	return i

func _init() -> void:
	# Static Initialization can stay in init
	name = "MyScene"
```

There are several things happening here.

1. We are using `i` to indicate that this Class represents a Scene and that you can not use new()
2. We are providing a factory method, that can take additional arguments for configuring the scene.
3. By using an UID path instead of a file path, this remains independent and self contained.

You can move the class and scene files without the risk of any dependency breaking in the process. And will never need to interact with the path within your code. No magic strings required.