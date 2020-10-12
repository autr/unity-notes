# Linux with Unity + Unreal 

Notes are for PopOS 20.04 LTS using latest NVIDIA drivers and a GeForce GTX 950 

## Unity 

If using HDRP make sure the latest NVIDIA drivers (with Vulkan) are installed on your system and loaded - otherwise the sample projects will crash. Check the status of your GPU with:

```
nvidia-smi
```

Download and launch Unity Hub - you must then first download an editor version and create a new project through the GUI before you can add an existing project.

Editor and crash logs can be found here:

```
~/.config/unity3d 
~/.config/unity3d/Editor.log 
```

So an additional package for com.unity.webrtc is needed:

```
sudo apt-get install libc++-dev
```

Vulkan hardware streaming should be supported from com.unity.webrtc-2.2 (~October). HDRP is also only supported with Vulkan.

Go to Project Settings then Player and Other Settings to switch between Vulkan or OpenGL.

## Unreal

Installing UnrealEngine / Editor requires linking your Github account and being added to the developers repositories, then building the binary.

Once the repository is cloned, follow the instructions ( ie ./Setup.sh && ./Generate.sh && make ) - which takes approx 1 hour and 60GB. 

The make command referenced in the Quick Start however is wrong, so use:

```
make UE4Editor -j6
```
