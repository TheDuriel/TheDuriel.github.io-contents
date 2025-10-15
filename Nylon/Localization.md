Nylon supports .csv based localization throughout Godots integrated system.

It is recommended you read the [official documentation](https://docs.godotengine.org/en/stable/tutorials/i18n/internationalizing_games.html) before proceeding.

### Generating Localization Files

1. Install the NylonEditorHelper plugin and enable it.

2. Open the Nylon bottom dock.

3. Set a search path in which to search for NylonScenes. Typically this is res://NylonContent/Scenes/, but you may store your scenes anywhere.

4. Set an output path to a .csv file. The default is res://NylonTranslations.csv

5. Hit generate.

Nylon will recursively search the provided directory for NylonScenes and NylonSequences saved as .tres or .res. It will first check the root node script, then **instance the scene temporarily** to extract the Text and Translation Keys from the scene. Note that this will execute @tool enabled scripts. Nylon classes are written that this isn't an issue. But it may cause errors to pop up in the console.

**Nylon can regenerate the file on top of itself.** If you set the output to an existing .csv file, Nylon will attempt to update and merge existing entries with new ones. This involves marking Nodes that no longer exist as DELETED, and rewriting translation keys for Nodes that were moved around in the scene they originate from. Your translations are preserved in this process. But it is still recommended that you keep backups and use git.

### Creating translatable properties

To enable the automatic collection of translation keys, observe the following pattern:

1. Define two properties. One to hold your text, a second to hold your localization key.

```GDScript
@export var text: String = "the english text you wish to localize"
@export_storage var text_tr_key: String = ""
```
The property that holds your key is tagged as storage only, so it will not be visible in the editor.

And it's name must be the name of the base property that holds your text, with the _tr_key suffix.

2. Generate the translation key in code.

To enable key generation for custom properties, like the one you have defined above, extend the _generate_translation_keys() function.

```GDScript
## Called before the Scene is saved by the Editor.
## Use this to generate unique keys for translation purposes.
func _generate_translation_keys() -> void:
	# Always query the sequence. This will include the scene key if this node belongs to a scene
	text_tr_key = tr_key_prefix + "_TEXT"
```

This is automatically done for the Nodes that Nylon provides for you. When you extend such a node, for example the NylonText node, make sure to override and call super() as such:

```GDScript
func _generate_translation_keys() -> void:
        super() # Call the base function to generate keys for existing properties.
	my_custom_property_tr_key = tr_key_prefix + "_TEXT"
```

### Enabling Localization

1. Load the relevant .csv files. You may need to reimport them with the delimiter set to semicolon ; See the official documentation linked above for details on how to do this.

2. Open your NylonConfig, typically in res://NylonContent/nylon_config.tres and enable localization.

This will automatically localize the NylonMessageText.text property.

You can also access localized text without enabling the config setting, by accessing NylonMessageText.localized_text.