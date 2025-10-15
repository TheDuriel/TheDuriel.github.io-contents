Once you have created your Scene, you need to display it to the user. The way you do this is entirely up to you! Unlike other dialogue addons for Godot, Nylon does not prescribe how your game has to look.

You could make JRPG style boxes with character portraits. You could make a Visual Novel. You could copy classical CRPGs, Twine games, and more! You could even switch styles mid dialogue.

This guide will give you an overview on how to process NylonMessages to construct your own Reader from.

For the sake of example, the style here will be a JRPG style. With one primary text box, and a dynamic list of buttons for choices. We will be ignoring graphics, sounds, and animations.

### NylonMessages

NylonMessage is a RefCounted object which is emitted by the Nylon.message_sent signal. This will typically be the only point of communication between your game and Nylon.

Messages inform your game about things that happen within Nylon. Many Nylon Nodes will emit messages. For example:

NylonScene will inform you about when a Scene has started, ended, and in what way that happened.

NylonSequence will inform you about authorship.

NylonText will tell you about text being displayed.

NylonChoice will tell you to offer the user a choice, typically as a button.

### Handling Messages

The core function of your custom Reader will be a callback for the Nylon.message_sent signal.

To get started, create a new Scene 'MyReader.tscn' with a MarginContainer root node.

Add the following script:

```GDScript
extends Control


func _ready() -> void:
	Nylon.message_sent.connect(_on_message_sent)


func _on_message_sent(message: NylonMessage) -> void:
	# Ignore handled messages.
	if message.handled:
		return
	
	# Your own code goes here.
	
	# This is optional.
	# Setting a message to handled will filter it.
	# This may be useful if you have your reader split across different classes
	# Or have multiple of them.
	message.handled = true

```

Then add the following block of if statements in the 'Your own code goes here' space:

```GDScript
	# Your own code goes here.
	if message is NylonMessageScene:
		pass
	if message is NylonMessageSequence:
		pass
	if message is NylonMessageText:
		pass
	if message is NylonMessageContinue:
		pass
	if message is NylonMessageChoice:
		pass
```

This is the basic set of messages a Reader will receive and should be capable of handling. We will now implement each type of message in turn.

Starting with

### NylonMessageScene

NylonMessageScene has two pertinent blocks of information:

Authorship. With a scene and author name, which you may display if you wish and can be set in the NylonScene Node.

State. Which indicates what happened to the scene.

Start by extending the if statement from earlier with your own function:

```GDScript
	if message is NylonMessageScene:
		_message_scene(message)


func _message_scene(m: NylonMessageScene) -> void:
	match m.status:
		NylonScene.STATUS.NONE:
			# Invalid, you did an oopsie.
			pass
		NylonScene.STATUS.RUNNING:
			# Scene is processing nodes.
			pass
		NylonScene.STATUS.STARTED:
			# Scene has been started.
			pass
		NylonScene.STATUS.COMPLETED:
			# Scene has ran out of nodes, or stop has been called.
			pass
		NylonScene.STATUS.PAUSED:
			# Scene is waiting for a Choice or similar.
			pass
		NylonScene.STATUS.TRANSITIONED:
			# Scene has finished, but is going to start a new scene.
			pass

```

This match statement covers all possible states a Scene may be in. You'll note that not all of them are immediately relevant.

For example, paused and transitioned are mainly used by Nylon itself internally. But may be useful for you to play different transition animations.

For now our focus is to: Open and Close the Reader when a Scene starts or stops.

To do this we trim the the match statement to the relevant branches:

```GDScript
func _message_scene(m: NylonMessageScene) -> void:
	match m.status:
		NylonScene.STATUS.STARTED:
			# Scene has been started.
			visible = true
		NylonScene.STATUS.COMPLETED:
			# Scene has ran out of nodes, or stop has been called.
			# This also means that no new scenes will start after this one.
			visible = false

```

In the future, you may want to trigger fade in/out animations, or whatever effects you want.

### NylonMessageSequence

NylonMessageSequence is practically identical to NylonMessageScene. It does not have a status property, leaving only the authorship information also present in NylonMessageScene.

Handling Sequence messages is not required for your Reader to function, and you can safely skip this step.

One use for Sequence messages is to display the title and author of the current scene/sequence at the top of the screen like some Twine games might do.

To achieve this, consider adding one or more labels and set their contents from these messages.

```GDScript
func _message_sequence(m: NylonMessageSequence) -> void:
	scene_title_label.text = m.name # In NylonMessageScene messages this is the m.title property.
	scene_author_label.text = m.author
```

### NylonMessageText

NylonMessageText contains text as well as contextual information about how to display it.

Depending on your needs you may choose to ignore most of these. For example, the 'append mode' property is only useful if you plan to append text to one continuous scrolling box, rather than displaying one text box per text node.

To handle text, you first need a RichTextLabel in your Reader. Add a PanelContainer, and then a RichTextLabel, and export some property to access the latter.

Example:
![[Pasted image 20250601012728.png]]

The minimum required code here is as follows:
```GDScript
@export var text_label: RichTextLabel


func _message_text(m: NylonMessageText) -> void:
	if m.style is NylonTextStyleNovel:
		if m.style.clear_previous:
			text_label.clear()
	
	text_label.text = m.text
```

If you are planning on having one continuous text scroll. Like for example in a CRPG like Pillars of Eternity or Divinity the Original Sin, or a Twine game, then you will want to handle the append mode provided by the default NylonTextStyle that accompanies the message.

The default NylonTextStyle contains no properties. It is designed for you to be extended as needed.

The `res://Nylon/TextStyles/` directory contains example base classes for Paragraph style Novels, Speech Bubbles, and Inter Titles.

For our example here, we make the assumption that you are using the Novel style with the following properties:

```GDScript
## Paragraph Mode
enum APPEND_MODE {
	APPEND, ## Appends text to previous text.
	NEW_LINE, ## Appends text on a new line.
	NEW_PARAGRAPH ## Starts a new paragraph.
	}
## The MODE to use.
@export var append_mode: APPEND_MODE = APPEND_MODE.NEW_PARAGRAPH
@export var clear_previous: bool = false
```


Here is an example of what that could look like:

```GDScript

# You can handle different styles in the same reader by checking for them.
if not m.style is NylonTextStyleNovel:
    text_label.text = m.text
    return

# We now know the style is a Novel style and has the required properties.
match m.style.append_mode: # NylonMessageText.style contains a NylonTextStyle Resource.
	NylonTextStyleNovel.APPEND_MODE.APPEND:
		text_label.append_text(m.text)
	
	NylonTextStyleNovel.APPEND_MODE.NEW_LINE:
		if text_label.text.ends_with("\n"):
			text_label.append_text(m.text)
		else:
			text_label.newline()
			text_label.append_text(m.text)
	
	NylonTextStyleNovel.APPEND_MODE.NEW_PARAGRAPH:
		if text_label.text.ends_with("\n\n"):
			text_label.append_text(m.text)
		if text_label.text.ends_with("\n"):
			text_label.newline()
			text_label.append_text(m.text)
		else:
			text_label.newline()
			text_label.newline()
			text_label.append_text(m.text)
```

Another way of handling append modes is to create a new RichTextLabel in a ScrollContainer for each new Paragraph.

### NylonMessageContinue

The Continue message is the request for user input before the Scene continues processing.

You may want to add a NylonContinue node at each point in your scene where you want to pause and wait for the user. Or you could change the NylonConfig property which makes all NylonText nodes request a continue.

This is especially useful if you are making a JRPG style dialogue like in this basic example. Since without waiting for the users continue, all text would play back instantly.

To change this, find your NylonConfig file, by default located in NylonContent/nylon_config.tres and change the corresponding property within it.

To handle this message then, you will need to manage user input.

To keep things simple, we will display a Continue button.

Edit your scene hierarchy to look like this:

![image](https://github.com/TheDuriel/Nylon-Documentation/assets/44248915/6a519cd6-1ad2-4be9-bc80-c16c4ac3f03e)

Buttons is a VBoxContainer, make sure you can access it with a property. Then add new functions as follows:

```GDScript
@export var button_container: VBoxContainer


func _message_continue(m: NylonMessageContinue) -> void:
	_clear_buttons()
	
	# Add Continue Button
	# You could use a Scene here for your own custom button.
	var b: Button = Button.new()
	b.text = "Continue"
	b.pressed.connect(_on_continue_pressed.bind(m)) # Binds the message to the button
	
	button_container.add_child(b)


func _on_continue_pressed(m: NylonMessageContinue) -> void:
	# Tell Nylon to continue processing the scene.
	m.choose()
	_clear_buttons()


func _clear_buttons() -> void:
	# Clear previous buttons.
	if button_container.get_child_count() > 0:
		for child: Node in button_container.get_children():
			child.queue_free()
		await get_tree().process_frame

```

This manages the creation of new buttons, adding them to our container, and finally informing Nylon that the user has continued.

What is happening under the hood is that the NylonScene which has requested the Continue input using the NylonMessageContinue, is waiting for the message to emit the 'continued' signal. You emit this signal with the m.choose() function. Or via m.continued.emit()

Note that the NylonScene can not complete until it is continued.

Incidentally, when a NylonScene pauses to wait for user input, a NylonMessageScene will be emitted with the corresponding status.

### NylonMessageChoice

A Choice is a narrative branch in your scene. A NylonChoice node may continue playing a different Sequence within the same scene, or start a new scene all together.

The basic implementation of choices works as follows:

```GDScript
func _message_choice(m: NylonMessageChoice) -> void:
	# Add choice Button
	# You could use a Scene here for your own custom button.
	var b: Button = Button.new()
	b.text = m.label
	b.pressed.connect(_on_choice_pressed.bind(m)) # Binds the message to the button
	
	if not m.condition_fulfilled:
		b.disabled = true
		
		if m.hidden_if_unfulfilled:
			b.visible = false
			# You could also skip adding it.
	
	button_container.add_child(b)


func _on_choice_pressed(m: NylonMessageChoice) -> void:
	# Tell Nylon to continue processing the scene.
	m.choose()
	_clear_buttons()

```

This is practically identically to handling the Continue case, but with multiple buttons.

The message also contains information about whether or not the choice is actually available, in case of some condition not being met, and if to display it anyways.

### More Stuff!

Nylon has a number of different messages that are not covered here. Additionally the example project available on itch contains many more nodes and an entire implementation of a custom reader you may consider looking at for further examples!
