# Purpose
User Profiles solve two problems. They allow for multiple sets of user preferences to be saved globally to the profile. And they allow savegames to be grouped under a profile.

You can either use Use Profiles to represent **people** or use them to represent **playthroughs**.

# Features

User Profiles are stored in the Games appdata location. Typically `user://profiles/`. This path is defined in `UserProfiles.gd`

Each Profile maintains a global data storage. This allows for storing settings that apply to that profile/playthrough. Like enabling tutorials, subtitles, or other settings that don't need to be global to the entire application.

Each Profile maintains a directory of savegames, and its own backup history for them. See [[Saving Games]]

User Profiles are accessed via `App.profiles`.

The currently active profile is accessible via `App.profiles.current`

A default profile called `default` is re-created each time the game is run. If you do not wish to engage with profiles, then you may safely do so. All save data will be stored under `user://profiles/default/`.

# Important functions

UserProfiles.create_profile() will create a profile with that name.

UserProfiles.set_profile_as_active() will set a profile as active. Make sure to call this after creating a new profile.

UserProfiles.delete_profile_forever() will permanently delete a profile and all save data associated with it.

UserProfiles.save_all_user_data() will save all user data for all existing profiles. Use this after you change a setting like tutorials_enabled in your project. Note: This does not create any 'save games', those are separate. see [[Saving Games]]

# Using User Data

This is an example of how to save and read a tutorials_enabled flag:

```gdscript
# Sets the tutorial_enabled flag to false.
App.profiles.current.user_data.stage_bool("tutorial_enabled", false)

# Write this change to disk.
App.profiles.current.save_user_data()

# Retrieve the value from the current profile. Including a default if it is not found.
var tutorial_enabled: bool = App.profiles.current.user_data.get_bool("tutorial_enabled", true)
```

For more details on how to interact with the `UserProfile.user_data` object, see [[Saving Games]]. Note: In simple games that do not need individual save files, you can rely on user_data only as your save game.