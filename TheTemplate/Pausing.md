The Template overrides the default pausing behavior in the following ways:

1. The Main Window (`SceneTree.root`) process mode is set to `Node.PROCESS_MODE_ALWAYS`
	1. This means that, no nodes will ever pause when `SceneTree.set_pause()` is called.
2. The Game.gd Autoload process mode is set to `Node.PROCESS_MODE_PAUSABLE`
	1. This means that, any child of Game will be paused when the SceneTree is paused.
3. The Interface.gd Autoload process mode is set to `Node.PROCESS_MODE_ALWAYS`
	1. This means that Interface will never pause.
	2. To allow pausing your own UI scenes, use `Node.PROCESS_MODE_PAUSABLE` in them.

### How to include Nodes in pausing?
To include a scene that is not a child of the Game Autoload in the pause process, for example your gameplay HUD elements. Use `process_mode = Node.PROCESS_MODE_PAUSABLE` in their init or ready functions.

You may want to do this for all game level menus. While leaving out the Pause Menu itself.

### How to Pause the Game?
The Game autoload provides `Game.pause()` and `Game.resume()` for this purpose. Both functions require an object to be passed, which is the source for the pause. This means that only the object that caused a pause, can undo a pause. If you need to pause from an object that will be deleted during the pause, consider passing Game itself.

The purpose of this argument is to prevent scenarios in which there are multiple objects trying to pause and unpause the game at the same time. And thus undo each others pausing.This way the game can only be resumed, if all pauses are resolved.

> [!warning]
> You should never use the SceneTree pause functions directly. Always use the Game autoload instead.