This is how I start a new Godot project.
# Step by Step

1. make a new folder
2. run `git init`
3. run `git lfs install`
4. copy the .gitignore in (see below)
5. copy the .gitattributes in (see below)
6. commit these changes
7. create a new file called `project.godot` (it will be empty)
8. open the folder in godot
9. rename the project in the project settings
10. commit these changes
11. optional: within godot, run the setup script (see below) 
12. commit these changes ;P

# .gitignore
```
# Compiled source #
###################
*.com
*.class
*.dll
*.exe
*.o
*.so

# Packages #
############
*.7z
*.dmg
*.gz
*.iso
*.jar
*.rar
*.tar
*.zip

# Logs and databases #
######################
*.log
*.sql
*.sqlite

# OS generated files #
######################
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Godot-specific ignores #
##########################
.import/
.godot/
.mono/
export.cfg
export_presets.cfg
*.translation
data_*/
mono_crash.*.json
*.tmp

# Project-specific ignores #
############################
!_exports/.gitignore
*.lnk
*.psd
```

# .gitattributes

```
# Images
*.jpeg filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
*.png filter=lfs diff=lfs merge=lfs -text
# Sound
*.ogg filter=lfs diff=lfs merge=lfs -text
*.wav filter=lfs diff=lfs merge=lfs -text
# Scene Files / 3D Models
*.collada filter=lfs diff=lfs merge=lfs -text
*.dae filter=lfs diff=lfs merge=lfs -text
*.fbx filter=lfs diff=lfs merge=lfs -text
*.obj filter=lfs diff=lfs merge=lfs -text
*.gltf filter=lfs diff=lfs merge=lfs -text
*.glb filter=lfs diff=lfs merge=lfs -text
*.bin filter=lfs diff=lfs merge=lfs -text
# Other
*.shape filter=lfs diff=lfs merge=lfs -text
*.mesh filter=lfs diff=lfs merge=lfs -text
*.ico filter=lfs diff=lfs merge=lfs -text
*.afdesign filter=lfs diff=lfs merge=lfs -text
*.ttf filter=lfs diff=lfs merge=lfs -text
*.otf filter=lfs diff=lfs merge=lfs -text
```

# setup script

https://gist.github.com/TheDuriel/4507f6f81ebe4ed0bc082c0e3c220049

Note that this script is an example. You could modify it according to your needs, or make a new one from scratch.

This script does a number of things:
1. It adds a number of submodules I use in my projects.
2. It configures the project settings
3. It creates essential directories