# Setting up development environment for com.unity.webrtc

Follow instructions here - but don't use WSL, Windows Terminal or Power Shell, which will cause strange things to happen - use only Command Prompt;

https://github.com/Unity-Technologies/com.unity.webrtc/blob/develop/Plugin~/README.md

Windows stores ENV variables and PATH here:

```
System Properties > Advanced > Environment Variables > System Variables > PATH
```

When installing Google Test, `cd %SOLUTION_DIR%` means `cd com.unity.webrtc/Plugin~`. Also replace VS Studio v15 / 2016 with v16 / 2019.

NB. Guide means to reference this script when using `%SOLUTION_DIR%`;

https://github.com/Unity-Technologies/com.unity.webrtc/blob/develop/BuildScripts~/build_plugin.cmd

## libwebrtc

The pre-built libwebrtc are broken (missing binaries), so you will either need to build libwebrtc from scratch or download a pre-built binary from elsewhere. 

**Pre-built (don't do this)**

I'm (was) downloading the binary here:

https://github.com/crow-misia/libwebrtc-bin

You must then restructure the folder exactly like so to play nice with CMAKE settings:

```
Plugin~
    webrtc
        include
            ... # etc
        lib
            x64
                webrtcd.lib # renamed Debug lib
                webrtc.lib # renamed Release lib

```

Alas, this lib will be missing the patches from BuildScripts~/patches, so:

**Build libwerbrtc**

To build from scratch, run **BuildScripts~/build_libwebrtc_win.cmd**. If you encourter errors / missing dependencies, install them, then blitz any build generated or downloaded files before you try again: especially **.gclient**

Takes a couple of hours, innit.

**Pre-build (DO THIS)**

Binaries on the Github repo have been updated (and now work);

* https://github.com/Unity-Technologies/com.unity.webrtc/releases/tag/M85
* https://github.com/Unity-Technologies/com.unity.webrtc/blob/develop/BuildScripts~/build_plugin.cmd

## VS Studio 2019

After cmake commands has completed, build64 folder will contain multiple VS 2019 projects. Open `webrtc.vcxproj` and right click > Project Properties to edit debug settings like so;

![vsstudio settings](https://github.com/Unity-Technologies/com.unity.webrtc/blob/develop/Documentation~/images/command_config_vs2017.png)

Switching out UnityRenderStreamingExample for UnityRenderStreaming-glitches etc:

```
# Command:
C:\Program Files\Unity\Hub\Editor\2019.4.9f1\Editor\Unity.exe

# Command Arguments:
-projectPath C:\Users\gilbe\Code\UnityRenderStreaming-glitches
```

