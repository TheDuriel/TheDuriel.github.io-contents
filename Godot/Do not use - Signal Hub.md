A banned pattern is a programming or design pattern, or engine feature, which I have banned from use in all my own projects. Usually this is first and foremost, because I wasn’t using them to begin with.

Despite the name, you should feel free to still use these features for: Leaning, prototyping, gamejams, and maybe even smaller games and vertical slices. Get the job done first. Just be aware that by using these things, you invite more work.

## What is a Signal Hub?
The Signal Hub is a popular pattern in Godot, proposed as a means of decoupling objects and allowing you to sidestep the scene hierarchy for node to node communication. Signals themselves are nothing but delegeate/events from C#, or similar anonymous function call subscription patterns.

## How are they used?
The typical Signal Hub is an Autoload script which contains a list of signals. This publicly exposes all signals to all objects in the engine. Creating one common point of communication.

## Why are they banned?

* Messy
Signals Hubs move connections to one location. Where previously only objects A and B were coupled, now the coupling extends through A -> C -> B, with C being visible to an infinite number of other objects that aren't part of the **responsibilities** of A or B. Or in other words: All the spaghetti, now goes through a single knot. Spitting the plate into two half servings of spaghetti where the ends have all been stuck into the same meat ball.

* Global
As per SOLID principles, making things global should **generally** avoided. A Signal Hub expressly globalizes signals that would otherwise be private to the relevant parties. It thus breaks several of SOLID design practices at the same time.

* Redundant
The Signal Hub is a Band-Aid pattern. While it solves an immediate problem, how to access objects in separate parts of the SceneTree, it can easily be avoided through hierarchical design, dependency injection, and Event/Messaging systems.

---

The following is an humorous example of the effect a Signal Hub has on project organization:
![[Pasted image 20250531173841.png]]

This is an image of a possible Object Hierarchy in a Game:
![[Pasted image 20250531175516.png]]

And these are the underlying Function and Signal calls:
![[Pasted image 20250531175644.png]]

Note how the Player Characters Health Attribute is emitting a signal. And how the Interface's HUD's HealthBar is connecting to the signal. This is possible because Game is a Singleton which is Globally accessible, and which in turn provides an API for accessing the Party, which in turn has exposed the Player Character and its Public member functions.

---

**Things to research:**

Signal Hubs are often confused for Messaging or Even systems, as well as Command Queues.

While these can fulfill similar purposes, there is a distinct different:

Instead of one global location holding dozens of individual signals. You may have one global location holding a single signal.

Consider this example:

```SWIFT
class MessageServer

signal message_sent(message: Message)

func send_message(message: Message) -> void:
	message_sent.emit(message)


class Message

var sender: Object

func _init(_sender: Object) -> void:
	sender = _sender


class MainMenu

func _on_start_game_pressed() -> void:
	var m: StartGameMessage = StartGameMessage.new(self)
	m.character_name = name_field.text
	MessageServer.send_message(m)

class SomethingThatListensToMessagesIDontKnowWhat

func _init()-> void:
	MessageServer.message_sent.connect(_on_message_sent)


func _on_message_sent(m: Message) -> void:
	if m is StartGameMessage:
		print("A new game is starting! Lets do something!")
```
