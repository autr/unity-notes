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

# Compiling com.unity.webrtc


Be sure to check out the 2.1.3 branch and have Visual Studio 2017 installed as well as 2019. We use a combination of BuildScripts and notes from the guide here:

https://github.com/Unity-Technologies/com.unity.webrtc/blob/develop/Plugin~/README.md

Checkout latest release:

```
git checkout --track origin/release/2.1.3
```

To blitz existing state, clean / reset HEAD (careful):

```
git reset --hard HEAD && git clean -d -f
```

Reset a file: `git checkout @ -- myfile.ext`

## [ignore] Recompiling 


Run the build script from the repository root:

```
cd com.unity.webrtc
"BuildScripts~\build_plugin.cmd"
```

Build project files:

```
# 2017

rmdir /S /q build64
cmake . -G "Visual Studio 15 2017" -A x64 -B "build64"
cmake --build build64 --config Release
cmake --build build64 --config Debug

# 2019 bb  

rmdir /S /q build64
cmake . -G "Visual Studio 16 2019" -A x64 -B "build64"
cmake --build build64 --config Release
cmake --build build64 --config Debug
```


If you get a gtest error, redo compile instructions:

```
cd "com.unity\webrtc\Plugin~\googletest"
git checkout 2fe3bd994b3189899d93f1d5a881e725e046fdc2
cmake . -G "Visual Studio 15 2017" -A x64 -B "build64"
cmake --build build64 --config Release
mkdir include\gtest
xcopy /e googletest\include\gtest include\gtest
mkdir include\gmock
xcopy /e googlemock\include\gmock include\gmock
mkdir lib
xcopy /e build64\googlemock\Release lib
xcopy /e build64\googlemock\gtest\Release lib
```


Attempts to get Debug solution working:

```

# Debug
cd googletest
rmdir /S /q build64
cmake . -G "Visual Studio 15 2017" -A x64 -B "build64" -DCMAKE_CXX_FLAGS_DEBUG="/MTd /Zi -D_ITERATOR_DEBUG_LEVEL=0"
mkdir lib
cmake --build build64 --config Debug
xcopy /e build64\googlemock\Debug lib
xcopy /e build64\googlemock\gtest\Debug lib
cmake --build build64 --config Release
xcopy /e build64\googlemock\Release lib
xcopy /e build64\googlemock\gtest\Release lib

# reference: https://github.com/Unity-Technologies/com.unity.webrtc/pull/200/commits/3932ae1f8b5951d2c00a4756d0ed627ee19f01d4

```

## Update

Working with current `develop` branch is easiest for getting Debug builds to work (currently using: `4192925d2d3241c66db620ddefc39f2cff1e4f6c`).

```

cd com.unity.webrtc
"BuildScripts~\build_plugin.cmd"
cd Plugin~
cmake . -G "Visual Studio 16 2019" -A x64 -B "build64"
```

Open in VStudio 19 and set Project Properties:

```
# Command:
C:\Program Files\Unity\Hub\Editor\2019.4.9f1\Editor\Unity.exe

# Command Arguments:
-projectPath C:\Users\gilbe\Code\UnityRenderStreaming-glitches
```

Voila.