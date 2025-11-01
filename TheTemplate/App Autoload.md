The App Autoload is the heart of the Template. It manages the core systems that are generalized to all games, handles the game startup, and handles the main application state.

All App level systems are found in `res://App/`

Configuring App is done via the config file found at `res://Configs/AppConfig.tscn`

The App Autoload is found at `res://App/Core/App.gd`

### What happens when I run my project?

The first thing you will notice is that the provided "default scene" or starting scene for the project, was configured to be `res://App/Boot/Boot.tscn`

This scene is intentionally left empty. And you do not need to add on to it, nor should you. It's only purpose is to run the ready function in `res://App/Boot/Boot.gd` after all other setup processes are completed.

**Godot Startup:**
1. Godot Initializes
2. Godot applies the Project Settings
3. Godot Instances all Autoloads and Plugins
4. Godot Instances the Templates Boot Scene
5. Boot.gd runs `_ready()`

**Boot.gd Startup:**
1. Call `randomize()`
2. Load the active UserProfile - See [[User Profiles]]
3. Initialize the AppStateBoot state

The AppStateBoot state is configured in the AppConfig file mentioned above.

The default version of this application state, will try to load the Example game project provided in `res://App/EXAMPLES/` starting with the splash screen example, and proceeding to the main menu example.

### What happens in the App Autoload when I run my project?
The App autoload is initialized in step 3 of the Godot startup detailed above.

During this step, both `_init()` and `_ready()` will be executed on all autoloads.

**During initialization:**
1. The AppConfig file is loaded from the Config directory.
	1. This provides settings for all following systems.
2. The AppSettings are initialized, and current User Settings are loaded.
	1. This includes things like the window mode, screen, audio, and language.
3. The UserProfiles are discovered, and the default profile is loaded.
	1. This is then later set to the active profile by Boot.gd

**During ready:**
1. The Main Windows process mode is set to always.
	1. This will prevent SceneTree.set_pause() from affecting it and any of its contents.
	2. This means you will not be able to pause any Template components.
	3. The Game.gd Autoload manages its own pause state. See [[Pausing]]
2. SceneTree.auto_accept_quit is set to false.
	1. All subsequent quit requests are then handled in `App._on_quit_request()`
	2. By default it will immediately quit. But you may override this function to add your own quit behavior. Eg to save the game first.
3. The AppSettings are applied.
	1. During init, the settings were only loaded.
	2. This is done, since some settings are not safe to apply until `_ready()`
4. The Default User Profile is set.
	1. It is created if missing.
5. AppStateNone is entered.
	1. This state does nothing.
	2. The first actual state is set by Boot.gd afterwards.