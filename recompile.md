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

## Instructions

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

# 2019 bb    

rmdir /S /q build64
cmake . -G "Visual Studio 16 2019" -A x64 -B "build64"
cmake --build build64 --config Release
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
