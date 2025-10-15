### 0. System Requirements
Nylon currently requires Godot 4.2 or greater to function. The ideal Godot version for Nylon may be bumped in the future, check the release notes for details.

Nylon has no hardware requirements.

### 1. Downloading Nylon
Nylon can be purchased and downloaded from Itch.

https://theduriel.itch.io/nylon

Note that Nylon uses an approximation of Semmantic versioning. X.Y.Z, Major.Minor.BugFix. Minor releases are meant to be backwards compatible with earlier versions, and will generally support drag and drop updating. Read the release notes for details.

### 2. Extracting
Nylon is distributed in .zip format. Extract it with your OS or favorite extraction program. Ex. 7zip.

### 3. Overview of the Folders
The extracted archive will contain a number of folders:

Nylon/ - The Nylon Core files
addons/NylonEditorHelper/ - An optional Editor Plugin

Some releases of Nylon may also include:
NylonContent/ - A template directory with examples

### 4. Installing Nylon in your Project
Copy the Nylon/ directory to the root directory of your Godot Game project.

Optionally do the same with the addons/ and NylonContent/ directory.

Note: It is possible to install Nylon/ to a different subdirectory within your project, however you may be required to edit Nylon.gd to gain full functionality. Keep reading to find out why.

### 5. Configuring Nylon
To enable Nylon within your Godot project, open the Project Settings -> Autoloads and add Nylon/Nylon.gd as an autoload with the name 'Nylon'.

Optionally also enable the Editor Plugin.

### 6. Additional Configuration
Nylon includes a configurable custom resource. This resource is located at Nylon/default_config.tres

Opening this resource will present you with a number of configuration options. By default no changes are required.

Nylon will additionally attempt to load a configuration override. The default location for which is NylonContent/nylon_config.tres

This file is optional, but you should prefer editing it instead of the one found within Nylon/, to make updating easier.

It is possible to change the expected locations of either file by editing Nylon/ConfigPaths.gd directly. These have been separated out to work nicer with version control.


### Expected Errors

The first time you open Godot after installing Nylon, and after adding the autoload, Godot will spam you with a significant number of errors. This is expected. Please restart the editor, after adding the autoload and plugin. Godot will resolve them the next time you open the project.
