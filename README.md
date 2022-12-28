# Major-Minor-Patch Version System
PlatformIO provides a highly extensible and customizable build system which can be configured through the [platformio.ini file](https://docs.platformio.org/en/latest/projectconf/index.html). While using PlatformIO and working on a project for a client it became evident that there was a need to keep track of firmware versions with a consistent, modular and configurable system built on top of the PlatformIO configuration system.

# How Does It Work?
The system consists of three parts:

1. A custom INI file (`mmp_version_system.ini`) which defines four types of releases for users which are defined in environments/sections.
    * `mmp_release:major`: increments like v0.3.5620 -> v1.0.0
    * `mmp_release:minor`: increments like v0.3.5620 -> v0.4.0
    * `mmp_release:patch`: increments like v0.3.5620 -> v0.3.5621

2. `platformio.ini` where `mmp_version_system.ini` is included in the `[platformio]` section as an `extra_configs` argument, thereby including its contents for usage in user defined environments.
    
        [platformio]
        ; include the mmp_version_system environments
        extra_configs = mmp_version_system.ini 

        [env]
        platform = arduino
        framework = arduino
        board = uno

        [env:release]
        ; extends adds scripts to output version file and update version
        extends = mmp_release

3. `mmp_version_system.py` is added to the list of extra scripts to be run by PlatformIO before the building. The script reads `version.h`, or initializes it based on user input, to get previous version and increment it. It also sets the output firmware file name to match the `custom_fw_prefix` + version number.

# Installation
1. Clone this repository and copy `mmp_version_system.ini` and `mmp_version_system.py` into your PlatformIO project directory, the same directory as `platformio.ini`.

2. Add the following lines to `platformio.ini`:

        [platformio]
        extra_configs = mmp_version_system.ini

3. Create a default release environment in `platformio.ini` with:

        [env:release]
        extends = mmp_release

4. Modify `custom_fw_prefix` in `mmp_version_system.ini` to match your project name. This will be included as the first part of the output binary and elf file names along with the version number.4. Modify `custom_fw_prefix` in `mmp_version_system.ini` to match your project name. This will be included as the first part of the output binary and elf file names along with the version number.

5. Run with `pio run -e release`. You will be prompted the first time to set the starting version number. Every run after that the version number will be incremented by one. Note that the default setting is that new releases will increment the minor version number (vMAJOR.MINOR.PATCH). 

    * To release different versions, repeat step two from above, but change the `extends` field to one of the three release types listed in part one of [the how it works section above](#how-does-it-work?). 
    * Also note that you will need to change `[env:release]` to be something more descriptive/unique like `[env:<something_here>]`. To run this new environment change your build command to `pio run -e <something_here>`.
    * If you want to customize what happens for the different release types, edit them in `mmp_version_system.ini`. Follow PlatformIO's documentation linked above and take a look at their **Advanced Scripting** section for more cool configuration options as well as to better understand how this system works.

# More Usage Tips
* The default environment for PlatformIO is `env`. Any environment in `platformio.ini` with the header `[env:NAME]` will first run what is listed under `[env]`. Thus, if you want the same board definition for all your releases, simply place the platform, framework, and board definitions under `[env]` and your release configurations under separate child headers with the form `[env:NAME]`. HOWEVER, if you want to release for MULTIPLE DEVICES, simply include the desired release type under a new header with the desired configuration.

    * Note that this will increment the version number, potentially creating a divergent version line without reflecting software changes **unless you use mmp_release:manual** which will prompt you to set the version number with each build. _A future feature will include adding a mode for different devices to prevent this._

* Firmware build files can be found in the project directory under: `./.pio/build/<environment_name>/<output_file>.bin`
