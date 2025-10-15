This document serves as a high level overview of how Nylon actually functions. Hopefully this will help you explore the code more easily, in case you ever wish to make modifications or add your own features.

### The primary components of Nylon are:

* Nylon.gd - The Autoload from which everything happens
* Components/Parser.gd - Text Transformation Engine
* Components/Node.gd - The core building block for Scenes
* Components/Message.gd - Message Type definition
* Components/TextStyle.gd - Text appearance Resource definition
* Components/Condition.gd - Scene logic flow Resource definition
* Components/Config.gd - Configuration Resource definition
* Reader/Reader.gd - Visual Layer implementation helper

Every other class is either a subtype or component of one of the above. The Logger and ID generator exist as auxiliary classes.

### The NylonEditorHelper Plugin
This optional plugin enables the following features:

* A basic bbcode editor and preview when selecting text nodes.
* The ability to generate .csv translation files from scenes.

### Logic flow:
1. A writer drafts an outline for a Scene
2. They or a designer using NylonScene and NylonNode types to construct the Scene in the Godot editor.
3. A they or a programmer determines when the Scene is executed in the Game using Nylon.start_scene()
4. The Scene internally steps through its component Nodes, and emits Messages via Nylon.send_message()
5. Messages are received by a Reader or other Game components

### The Scene
Nylon Scenes are built using Nodes, similar to how normal Godot Scenes are built. Instead of representing game objects, entities, ui, or similar however, Nylon Scenes are used as a means to visually program the logical flow of a dialogue scene. Similar to a visual or block coding system.
If you're familiar with RPGMaker, its block scripting works very similar.

### The Message
Once a Nylon Scene is executed, it will begin to send Message objects via the Nylon.message_sent signal.

A Message contains several bits of information depending on its purpose. But will always contain the Node that emitted it, and the Scene it belongs to.

The NylonMessageText Message will contain all the relevant information needed to display text in your game. Including the raw text defined within the Scene, the translation key for this text, and a TextStyle Resource.

### The Reader
The Reader is the player facing side of Nylon, and will typically be created by you according to your desired appearance. The NylonReader object implements several helper signals for making this process easier, but it does itself not show anything.

I am currently adding visual templates to the addon, providing you with a starting point from which to build your own appearances for Nylon.

The Reader's job is to receive NylonMessages, and to process and/or respond to them when needed.

For example, it may receive a Text message and use its TextStyle to determine to spawn a Speech Bubble for a specific Character in a certain color. All, without ever being aware of the inner workings of Nylon itself.

Similarly, Choice Messages contain information about branches in the dialogue that can be presented to the user. And contains a chosen() function that will inform the Nylon Scene about how to proceed.

### The Parser
The parser is an optional component that can be invoked to perform text transformation and substitution with Nylon"Syntax"

Each individual feature of the parser is defined in your Nylon Config file, where features can be added/removed without the need to change any code.

At its most basic, the parser will search the text for the following pattern:
`<start_char><feature_alias><seperator_char><key_b>...<end_char>`
The default characters used are:
`{<feature_alias>|<value_b>...}`
(... is not part of the search pattern, but indicates that you can insert any number of values as long as they are separated by the separator character, |)

When the Parser finds this pattern, it will recursively search for the inner most nested instance, if any, and then iterate through each parser feature.

As an example, the "If" feature, will be invoked if the pattern begins with:
`{if|`
And will successfully complete if the following values match its requirements.
`{if|Example_Scene|example_flag|The flag is true.|The flag is false.}`

Parser Features can run **any** arbitrary code you wish, as long as they either exit or return a string for the parser to insert into the text.

To implement your own feature, copy one of the existing feature files in ParserFeatures/, change the prefix in `_set_prefix()` and implement your desired behavior in `_parse()`. The arguments provided for `_parse()` are the NylonScene node that owns this text, as well as an array consisting of the individual arguments found in the pattern.

You can use the Scene to perform speaker lookup and gain additional information about what is happening. By default Nylon allows for setting the Player character, and up to Two additional characters which participate in the scene. (This can be overridden in each Scene and Sequence node.) Defaults are set in NylonConfig.