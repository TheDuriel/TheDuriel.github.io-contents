The AppSettings system provides a means of defining global settings that are applied, regardless of the active user profile.

These are mainly used for, window management, audio settings, and language or accessibility settings.

### Adding a new Setting
The easiest way to add a new setting is to copy an existing one. For example `res://App/AppSettings/DefaultSettings/WindowBorderless.gd` represents a simple boolean on/off toggle setting.

Creating a new setting in detail however is done as follows:

1. Create a new script and save it in `res://Conent/AppSettings/`
2. Extend AppSetting
3. Implement the abstract functions as indicated

### Example Setting
```gdscript
class_name AppSettingBlood
extends AppSetting
## Blood Setting
##
## Toggles Blood on or off for all users

## Default is no blood
const DEFAULT: bool = false


# These are the abstract functions defined in AppSetting
# You must define each according to the comments provided in AppSetting
func _init_value_type() -> int: return TYPE_BOOL
func get_cfg_section() -> String: return "parental"
func get_cfg_key() -> String: return "blood"
func get_default() -> Variant: return DEFAULT

# These four are provided as type safe alternatives for their unsafe counterpart in AppSetting
# They are optional, but recommended, note that you need to adjust the typing accodingly
func set_current(value: bool) -> void: _current = value
func get_current() -> bool: return _current
func set_staged(value: bool) -> void: _staged = value
func get_staged() -> bool: return _staged

# Also abstract.
func apply_setting() -> void:

	# This is an example of how to skip applying a setting
	# on a platform that it does not apply to.
	# Or to force it on.
	if App.settings.platform == AppSettings.PLATFORMS.NINTENDO:
		return
	
	# Here you would apply the setting. Assuming it needs to be.
	# While making the assumption that a blood VFX would query this setting to determine if it should be used or not.
	
	
	# In this example, we will toggle an imaginary blood sound effect channel.
	AudioServer.set_bus_mute(AudioServer.get_bus_index("blood"), not get_current())
```
