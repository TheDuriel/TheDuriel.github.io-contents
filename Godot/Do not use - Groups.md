> [!info] Heads Up!
> A banned pattern is a programming or design pattern, or engine feature, which I have banned from use in all my own projects. Usually this is first and foremost, because I wasn't using them to begin with.
> Despite the name, you should feel free to still use these features for: Leaning, prototyping, gamejams, and maybe even smaller games and vertical slices. Get the job done first. Just be aware that by using these things, you invite more work.
## What are groups?
Groups are a Godot feature that allows any Node to be tagged and retrieved globally. They are one of several ways to circumvent the SceneTree hirarchy in an effort to offer convenience to users.

## How are they used?
Groups are added to Nodes using String names, either in code or via the editor interface. The latter of which helpfully displays a list of all groups present in the project, in an effort to fight one of the downsides of the system.

A Node that is added to a group, registers itself inside an invisible array within SceneTree. From where it can be retrieved using get_nodes_in_group()

Since this function creates a copy of the internal array. A number of helper functions exist to avoid the overhead from doing that. Like call_group() which takes in a String name for a method that then gets called on all group members.

## Why are they banned?

* **Magic Strings**
Groups are exclusively accessed using case sensitive Strings. Making a typo or misremembering the name of a group, results in silent faults because get_nodes_in_group() will simply return an empty array.

* **Untyped**
Groups are generic, and thus can not be typed. get_nodes_in_group() returns an array of Nodes. There is no way to validate what type those nodes are. call_group() takes this one step further, by taking a string for the name of a group, and a string for the name of a method that, hopefully, exists on every object. If it does not, it silently fails.

* **Invisible**
While the improvement of the editor UI, by adding a list of 'global groups', has helped. Groups mostly remain invisible. They can not be seen in the runtime inspector. Making it impossible to tell what groups exist, or what is in them, when troubleshooting.

* **Redundant**
All uses of groups can be replaced with a manual re-implementation of the same thing. Doing so carries the advantage of eliminating all three previous negative points about groups.

---

## Consider the following boilerplate alternative:

```gdscript
class Entity

func _init() -> void:
	Entities.add_entity(self)
```

```gdscript
# An autoload, or member of an Autoload
# Provides functions for interacting with entities
# This is the S in ECS ;P
class Entities

signal entity_added(entity: Entity)

var _entitiy_instances: Array[Entity] = []

func add_entity(entity_instance: Entity) -> void:
	if not entity_instance in _entitiy_instances:
		_entitiy_instances.append(entity_instance)
		# This is another thing groups can't do.
		entity_added.emit(entity_instance)
```