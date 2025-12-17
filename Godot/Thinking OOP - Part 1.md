> [!info] Heads Up!
> This article is part of a longer series on Object Oriented, and Component Driven, systems architecture in Godot.
> Part 1 starts with an example on how to get more out of Classes in the context of an Inventory and Item system.
> Part 2 and onwards, expands this item system to make use of Components and data driven design.
> This series should act as a primer, not gospel. And I recommend reading actual C++/C# text books if you disagree with how I use certain terminology in this series.

Composition is a buzzword often thrown around. And rarely with much context or explanation. Typically together with the popular Godot mantra:

**Call Down, Signal Up.**

Certainly most often people will be more than happy to explain that composition solves 'problems' that it 'decouples' or does some other things. But there are scant few resources about the efficacy and especially the **thought process** behind how to apply the concept.

For this article, I consider Composition in the context of game programming using object oriented languages. This text will be applicable to GDScript and C# both. Note that C# makes available some methods of composition that are unique to it. Eg. Interfaces. But this article is not about such specifics.

Additionally, I'd like to footnote a difference between: Using Classes, Using Composition, and Using Components. This article will try to work through the first two concepts in a breezy manner. But if you are not yet comfortable with Classes, most of this may not be immediately actionable to you.

## The Problems and Goals
First, lets define the problems that composition tries to solve. Or in other words, the goals that it wants to achieve.

1. We want to de-duplicate our code.
2. We want to be open ended in our architecture.
3. We want to stay flexible in our game design.

De-duplication is the simplest of concepts. Why write a line of code twice when you don't have to?
Composition allows us to create the concept of a component, which holds a piece of code that is required in several places.

Open ended architecture means that we avoid situations in which 'adding' or 'refactoring' become difficult.
Breaking complex code into smaller pieces, allows us to consider less code at a time. It reduces the cognitive load required to reason about the changes you want to make, and it makes doing those changes easier by reducing the number of steps it takes to do so.

Flexibility in design refers to the ability to add on and change the game itself.
Adding new features, changing how a feature behaves, or in fact trying out new permutations of features all benefit from 'open ended architecture'. But composition also allows you to put power into the designers hands. Rather than relying on the programmer for every single change. A well composed system is also an architecture that is easy to explore and tweak.

> [!warning] It should be noted...
> ... that the goal of applying composition principles is not to reduce the number of lines of code you have to write, by itself.
> Indeed in many cases, you will find situations in which you are writing additional boilerplate. A task that can seem tedious and redundant. But which however by its nature, will enhance the readability, structure, safety, or modifiability of your project. Indeed, sometimes more code is better than less. Though in most cases, you will write less code in total, I still ask you to trust me on this.

But, how do we actually **think** with composition?

## Lesson 1: Using More Classes
The first exercise on the road towards making use of Components. Is the splitting of larger classes into several smaller ones. This frequently builds the basis of "Entity Component" structures, which in some cases expands further into "Entity Component System". On its own however, this has little do with Components as described further into the article. But I think it important to mention here as a primer.

> [!info] There's a difference...
> Component design is often conflated with the popular `ECS or Entity Component System` programming paradigm. I will **not** talk about ECS in this article.
> ECS is useful, and I recommend you check out articles on it in the context of lower level systems programming. (There's some good GDC talks to act as flashy introductions.)

A typical example of using more Classes, is to replace 'dumb' data containers, with smarter ones. In C# you are provided with Structs, and in GDScript we will often default to the use of a Dictionary as data containers. However, both of these have a downside: You can not attach logic to them. They only contain the values themselves. Acting upon these values, can quickly became a hassle.

### Simple Inventory
Consider the following scenario, in which we want to keep a list of Items in a Container/Inventory. The naïve approach may look something like this:

```gdscript
class_name Container #Imagine a Crate or Backpack
extends RefCounted

var _items: Dictionary[String, int] = {} # item_id, durability


func has_item(item_id: String) -> bool:
	return item_id in _items


func get_quantity(item_id: String) -> int:
	return _items.get(item_id, 0)


func add_item(item_id: String, durability: int) -> void:
	# This sytem assumes you can only have one of each item id. Yes that's probably very bad design.
	_items[item_id] = _items.get(item_id, durability)


func remove_item(item_id: String) -> void:
	if item_id in _items:
		_items.erase(item_id)


func reduce_durability(item_id: String, ammount: int) -> void:
	_items.set(item_id, _items.get(item_id, 0) - ammount)
	if _items[item_id] <= 0:
		_items.erase(item_id)
```

### Inventory Worries
So far so good. Lets expand this system so we can track if an item can break or not.

```gdscript
class_name Container #Imagine a Crate or Backpack
extends RefCounted

# We get the choice between storing a Dictionary with item_id : item_data structure, or an array.
# The array adds the need to 'search' for items, which is suboptimal. But keeps us from having
# to have two places to track the id, which I find more comfortable.
var _items: Array[Dictionary] = []


func has_item(item_id: String) -> void:
	for data: Dictioanry in _item_data:
		if data.item_id == item_id:
			return true
	return false


func get_item(item_id: String) -> Dictionary:
	for data: Dictionary in _item_data:
		if data.id = item_id.
			return data
	# Returning an empty dictionary can cause bugs.
	# A has_item() call, or verifying the dictionary contents, should be done.
	return {}


func add_item(new_item_data: Dictionary) -> void:
	var item_id: String = new_item_data.item_id
	# We assume our system is still limited to one copy of each item id.
	# And add their durability together. This is looking more wrong than before
	if not new_item_data in _item_data:
		_item_data.append(new_item_data)


func remove_item(item_id: String) -> void:
	for item_data: Dictionary in _item_data:
		if _item_data.id = item_id:
			_item_data.erase(item_data)


func reduce_durability(item_id: String, ammount: int) -> void:
	for item_data: Dictionary in _item_data:
		if _item_data.id = item_id:
			_item_data.durability -= ammount
			if item_data.durability <= 0 and _item_data.can_break:
				_item_data.erase(item_data)

```

That works? But what if we add more attributes to items? And where does this item_data dictionary come from? How do we make sure that when we call add_item() it's in the right format? How do we patch things? What if the game design changes and broken items should remain at 0 durability? Every time a new consideration is added, this class will simply grow... and grow... and grow...

### Inventory Solution
The solution is straight forward.

1. We identify that using a Dictionary as a data container is unsuitable for our needs. It fails to be statically typed, can not be safely accessed, and we already are struggling to add new features to our items. Despite only having two. (Though of course, the above is not a perfect example of a dictionary based implementation. But these problems will only keep growing.)
2. We replace the dictionary with a class.

```gdscript
class_name Item
extends RefCounted

signal broken(item)

# We are making use of the Property + Field pattern here.
# In C# the property would be private
var id: String #  Field
	get: return _id
var _id: String:

var durability: int:
	set(value): _durability = value
	get: return _quantity
var _durability: int:
	set(value):
		if not _can_break:
			return
		_durability = max(value, 0)
		if _durability <= 0:
			broken.emit(self)

var can_break: bool:
	get: return _can_break
var _can_break

# This helper property allows us to quickly check if an item is broken.
# You can no longer mess up this if check, no matter how often you have to check across your project.
var is_broken: bool:
	get: return _durability <= 0 and _can_break

# I don't enjoy the new_ prefix. But it keeps things from being ambiguous.
# Data Driven Design would resolve this issue. Keep an eye out for an article on that!
func _init(new_id: String, new_durability: int, new_can_break: bool = false) -> void:
	_id = new_id
	_durability_ = new_durability
	_can_break = new_can_break
```

Read more: [Example of a Property + Field.](https://stackoverflow.com/questions/295104/what-is-the-difference-between-a-field-and-a-property#:~:text=Properties%20expose%20fields.)

**First reaction?** Wow that's a lot of code to do the job of a dictionary with 3 keys.

**Second reaction?** Oh neat, we can now use setters and getters to add behavior to our items without cluttering the container class

And our container now looks like this:

```gdscript
class_name Container
extends RefCounted

signal item_added(item: Item)
signal item_removed(item: Item)

var _items: Array[Item] = [] # Oh hey, static typing appears.


func has_item(id: String) -> bool:
	for item: Item in _items:
		if item.id == id:
			return true


func get_item(id: String) -> Item:
	for item: Item in _items:
		if item.id == id:
			return item


func add_item(item: Item) -> void:
	if has_item:
		return
	
	_items.append(item)
	item.broken.connect(_on_item_broken)
	
	item_added.emit(item)


func remove_item(item: Item) -> void:
	if not has_item():
		return
	
	_items.erase(item)
	
	item_removed.emit(item)


func _on_item_broken(item: Item) -> void:
	if GameRules.items_delete_when_broken:
		remove_item(item)

```

That's so much less code! Despite being longer in total (thanks to the addition of the signals the previous examples were missing), we're writing less lines. And each function accomplishes a very simple task only. There's no pulling double or triple duty. And almost all item operations can now be done directly to the item itself. **Most importantly:** Adding new features to an item, be it properties or behavior, is now done in Item, not Container.

And we snuck in a clean implementation of a configurable game rule as a bonus. It's placed in the Container rather than the Item, because we don't want Item to need to know about which Container it is in.

## Part 2... Loading...

The next parts will introduce:
1. Data Oriented Design
	1. Using Resources properly.
2. Static Components
	1. Keeping Items simple.
3. Dynamic Components
	1. Making Items expandable.
4. Oh no we're making an ARPG aren't we?
	1. Yes I like Diablo 2 a lot.
5. Model View Controller
	1. Because Item is RefCounted, and that's not a Node!
6. Order doesn't matter.
	1. These points will not be covered in this order. I think.