The NylonParser is a text processor that takes in the raw text you defined in a NylonNode, typically a Text node, and executes any NylonSyntax found within it to replace it with the corresponding true text.

For example you may use it to insert the name of your character name.
`"Hello {player|name}!"`

Nested patterns are supported.
`{if|scene_id|flag_id|"Yes I've seen {char_a|name}."|"No I don't know them."}`

A detailed explanation of each NylonParserFeature is found within the example scenes provided as part of Nylon and the example project.

It is recommended that large branching is left to their respective NylonNodes.

The Parser is configured in NylonConfig, where you can define the start { end } and separator | characters. You can also replace the parser entirely, and add/remove NylonParserFeatures from it.

The default list of features is:
flag, int, str, if, rng, char

The char feature also implements several aliases:
player, char_a, char_b

### Adding Features

Parser features extend NylonParserFeature and are registered inside your NylonConfig. The parser steps through each feature in order and recursively processes your file. If the output of a feature is itself another valid instance of NylonSyntax, then this too will be executed.

To add a new feature, define a new class extending NylonParserFeature like so, and define the two required virtual functions:


```GDScript
class_name NylonParserFeatureRng
extends NylonParserFeature


func _set_prefix() -> void:
	prefix = "rng"


func _parse(_scene: NylonScene, args: Array[String]) -> String:
	if args.is_empty():
		return "(Error: rng wants > 1 args, got none)"
	
	return args.pick_random()

```

Above you can see the implementation of the rng feature, which returns a random element from the provided list.

`{rng|a|b|c...}`

To define your own functionality:

* Set a unique class name
* Set the prefix in `_set_prefix()`
* Return a string in `_parse()`
* Add your feature script to your NylonConfig

The parse function receives all arguments provided after the prefix. `{rng|<a|b|c...>}` And must return a valid string.

NylonParserFeature instances are owned by the parser and not added to Godots SceneTree, this means they will only have access to autoloads and loaded resoures.

Of course, you may use NylonParserFeature as a wrapper to call functions elsewhere within your project, thanks to Godots first class callables.