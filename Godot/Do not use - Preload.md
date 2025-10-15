**Preloading is done when the scene is instanced.**

This is false.

**Preloading is done when the Class (script file) is loaded.**

This is correct.

### Lets look at the consequences of preloading.

First. We assume that preloading is used to prevent a Resource from being unloaded.

This is false. Because Resources already remain cached as long as any reference remains. Additionally, you can use the additional arguments of ResourceLoader.load() to force a Resource to remain cached, even if it is not referenced anywhere in your project.

Second. We assume that preloading is useful to prevent hiccups from loading when we instance a large number of objects.

This is false. A Resource once loaded does not behave any different if it was preloaded. It will not vanish between for loop iterations and need to be reloaded either. Instancing 200 projectiles using load and using preload, does not make any difference. (Inlining either call into the loop is an equally bad idea.)

Third. We assume that preloading prevents hiccups from loading a large Resource during gameplay.

This is false. Preloading moves the cost of loading a Resource, to the time at which the Resource **containing** the preload statement is loaded. Instead of invoking a load for the first Resource, and then later for the second Resource. You have now added both of them together. Increasing the time it takes for the first Resource to load.

### The actual problem: Startup time

1. preloads occur when the Class Script containing it is loaded.
2. Class Scripts with a class_name are loaded when the game starts up.
3. preloads are recursive, preloading a scene containing a script containing a preload, will load all of those Resources.
4. @export behaves in most practical scenarios, as if you are using preloading

What does this mean?

1. The user starts the game.
2. The Godot splash screen appears.
3. The game loads the class database, eg. All Scripts with a class_name
4. The game loads all preloaded resources.
5. The game loads all preloaded resources inside preloaded resources.
6. ...
7. The game loads all @exported resources inside preloaded resources.
8. The game loads all @exported and preloaded resources inside preloaded resources.
9. ...
10. Oh no.
11. The game has been frozen and not responding to user input until this process is finished. Possibly taking seconds, or minutes.
12. The game loads and instances all Autoloads.
13. The game loads all @exported and preloaded resources inside all Autoloads.
14. ...
15. The game instances all autoloads.
16. The game loads the main scene. (AND SO ON)
17. The game instances the main scene.
18. The game finally becomes responsive again and starts playing.

### Takeaways:

1. Preloads should be small files, that do not contain nested resources.
2. Named classes should not preload() unless 1. is true.
3. Autoloads should not preload() unless 1. is true.
4. The main scene should not preload() unless 1. is true.
5. **If 1. is true, you can also load() and it won't make a difference.**
6. If the files are big, you've just caused a load stutter.

### Solutions:

Threaded loading.

Check out my utilities repo for a reasonably universal threaded resource loader implementation.

https://github.com/TheDuriel/DurielUtilities/tree/main/ContentProvider