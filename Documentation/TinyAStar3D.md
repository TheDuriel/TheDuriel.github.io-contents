This page covers the usage documentation for TinyAStar3D a Godot 4 implementation of a grid based memory minimizing AStar solution.

### Requirements
* Godot v4.7.2.stable.mono.official **or above**
* C# Dev Environment (Optional)

### Installation
* Get the source here: https://github.com/TheDuriel/TinyAStar3D
* Place in any folder of your choice within the project.
* Make sure that your Godot Project already has a C# solution file created. You can do so by creating any C# script from within the editor one time, then deleting it again.
* Press the build button in the editor, top right corner.

![[Pasted image 20260904224521.png]]

### Limitations / Use Case
* TinyAStar3D only supports fixed size grids.
* Cells are connected in cardinal directions only. (That's 6 directions, no diagonals.)
* Cells default to being traversable.
* Cells do not have the ability to store weights or custom neighbors.

### Code Overview

TinyAStar3D includes the following **User facing** classes. These will be the classes you will actively use:
* TinyAStar3DNode - GDScript - Node
	* This Node is used to interface with TinyAStar3D.
	* Manages one TinyAStar3D instance.
	* Handles collecting Volume's, and generating the Grid and AStar instances.
	* Can instance a TinyAStar3DDebugView instance for debug visuals.
* TinyAStar3DVolume - GDScript - Node3D
	* This Node is used to mark areas as traversable (or not) within the grid.

These classes are the **Godot Glue**. You will usually not interact with them.
* TinyAStar3D - C# - RefCounted
	* This is the C# bridge between Godot and BitAStar.
	* Handles creating and interacting with the voxel grid and pathing algorithm.
* TinyAStar3DDebugView - GDScript - Node3D
	* Instanced by TinyAStar3DNode
	* Display blocked cells within a given grid.

These classes are **internal**. You will never interact with them.
* VoxelGrid - C#
	* Single Bit per Cell grid implementation.
	* Tracks the traversable state of a cell and nothing else.
* BitAStar - C#
	* AStar implementation that uses a VixelGrid to generate paths.
	* Uses a simplified heuristic. Cardinal directions only, no weights.
	* Path quality **is** impacted by these facts.


### Getting Started

1. Add a TinyAStar3DNode instance to your scene.

![[Pasted image 20260904224302.png]]

2. Set the world_size representing the playable area in global space.
	* The world is assumed to originate from 0,0,0 and expand in the given direction.
	* A world_size of 64,64,64 means the world is 128,128,128 units in size centered on 0,0,0.
3. Set the grid_size representing the number of grid cells that span the world across each axies.
	* This value should be divisible by 64.
	* Values above 512 are **strongly discouraged.**
	* Memory usage for a size of 448 is roughly 600mb.
	* The minimum is 64.

![[Pasted image 20260904224326.png]]

4. You can now generate paths via TinyAStar3D.get_astar_path(from, to).
	* from and to must be global coordinates (within the world_size)
	* This will return a path in global coordinates
5. To block cells from being visited, you can then add TinyAStar3DVolume nodes.
	* A volume is a box centered on the node.
	* All grid cells within the box will be marked untraversable by default.
	* Volumes can be enabled/disabled at runtime.

![[Pasted image 20260904224417.png]]

Alternatively you can block individual cells manually, by extending TinyAStar3DNode with your own code. And accessing it's `_astar` property to call SetTraversible() directly.


### Using the debug view
Note: To prevent issues with C# error causing Godot to spiral into an infinite non responsive state, debug visualization is disabled each time the editor loses focus.

Debugging untraversable cells is done by enabling the debug_draw. Doing so will immediately generate a new grid, and query all existing volumes in the scene. Untraversable cells will be indicated using red cubes. Traversable cells are not shown.

![[Pasted image 20260904224432.png]]