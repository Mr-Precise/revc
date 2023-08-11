## revc

In this repository you'll find the fully reversed source code for GTA VC

It has been tested and works on Windows, Linux, MacOS and FreeBSD, on x86, amd64, arm and arm64.  
Rendering is handled either by original RenderWare (D3D8)
or the reimplementation [librw](https://github.com/aap/librw) (D3D9, OpenGL 2.1 or above, OpenGL ES 2.0 or above).

## Installation

- reVC requires game assets to work, so you **must** own [a copy of GTA Vice City (Steam)](https://store.steampowered.com/app/12110/Grand_Theft_Auto_Vice_City/) or [Official store](https://store.rockstargames.com/game/buy-grand-theft-auto-the-trilogy-the-definitive-edition).  
- Build reVC or download the latest build:
  - [revc latest](https://github.com/Mr-Precise/revc/releases/latest)

## Building from Source

#### Install dependensies:
Ubuntu/Debian:
`sudo apt install g++ make cmake pkg-config libglfw3-dev libopus-dev libopusfile-dev libogg-dev libsndfile1-dev libopenal-dev libmpg123-dev`

Clone the repository with  
```bash
git clone --recursive https://github.com/Mr-Precise/revc
```
Then ender into the cloned repository.  
```bash
cd revc
```

#### CMake configure && build
```bash
mkdir build && cd build
cmake .. -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/usr -DLIBRW_PLATFORM=GL3
make -j$(($(nproc) + 1))
```

There are various settings in [config.h](https://github.com/Mr-Precise/revc/blob/main/src/core/config.h), you may want to take a look there.

reVC uses completely homebrew RenderWare-replacement rendering engine; [librw](https://github.com/aap/librw/). librw comes as submodule of re3, but you also can use LIBRW enviorenment variable to specify path to your own librw.

## Improvements

We have implemented a number of changes and improvements to the original game.
They can be configured in `core/config.h`.
Some of them can be toggled at runtime, some cannot.

* Fixed a lot of smaller and bigger bugs
* User files (saves and settings) stored in GTA root directory
* Settings stored in reVC.ini file instead of gta_vc.set
* Debug menu to do and change various things (Ctrl-M to open)
* Debug camera (Ctrl-B to toggle)
* Rotatable camera
* XInput controller support (Windows)
* No loading screens between islands ("map memory usage" in menu)
* Rendering
  * Widescreen support (properly scaled HUD, Menu and FOV)
  * PS2 MatFX (vehicle reflections)
  * PS2 alpha test (better rendering of transparency)
  * Xbox vehicle rendering
  * Xbox world lightmap rendering (needs Xbox map)
  * Xbox ped rim light
  * Xbox screen rain droplets
  * More customizable colourfilter
* Menu
  * More options
  * Controller configuration menu
  * ...
* Can load DFFs and TXDs from other platforms, possibly with a performance penalty
* ...


## License

We don't feel like we're in a position to give this code a license.  
The code should only be used for educational, documentation and modding purposes.  
We do **not encourage** piracy or commercial use.  
Please keep derivate work open source and give proper credit.
