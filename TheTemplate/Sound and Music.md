# Features
The Template provides a simple audio player implementation capable of:

* Playing sound effects, including position audio in 2D and 3D.
* Playing a single looping "music" track, and fading from one to the next.

**This is not intended to be a comprehensive audio solution.** But to provide the building blocks to make something more complex. Do note that if you wish to start using complex audio sequencing, mixing, and responsive behavior, you will most likely want to use a FMOD plugin instead. Though new versions of Godot do provide better integration for adaptive music out of the box.

# Audio Resources
AudioEffect and MusicTrack are two custom Resource types that are provided to allow for pre-configuring sound before playback.

For example it is possible to set a DB offset on the effect, allowing you to adjust the volume of the playback of the individual sound, without the need to edit it again in an external program.

The AudioEffect resource is compatible with all of Godots built in audio stream types. Including random playback, pitch shifting, and more.

The intent is that you create an AudioEffect resource for every effect you want to use, or every configuration of an effect.

This also has the advantage of allowing you to replace the raw sound files, without needing to edit the rest of your project to track down references to it.

# Playing SFX

Playing a sound effect is as simple as: `App.sfx.play(effect_resource)`. To play a positional effect, use the play_2D or play_3D function, and provide a position in world space to place the sound at.

# Playing Music
Use `App.music.play(track_resource, fade_time)` where fade time is the duration over which to linearly fade this track in from the previous (if any).

If the loop flag is set on the raw audio file, it will loop indefinitely until faded to another track.
