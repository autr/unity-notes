# Unity Render Streaming Fix-Quest

Repository for resarch into fixing a Unity Render Streaming bug that causes "data-moshing" when there is slight packet loss - ie. missing or malformed keyframes (i-frames) causing subsequent p-frames to be glitchy.

Some comparison is made between Unreal4 PixelStreaming which also uses the NVIDIA SDK, but without glitches.

Based on the following discussions, the problem is most likely SPS / PP. 


## Notes

Setting up Unity and Unreal4 development environment on PopOS (Linux)
Setting up a development environment for com.unity.webrtc (Windows)
Review of Unreal Engine 4's PixelStreaming (Windows)
Comparison between Unreal and Unity NVIDIA encoder (Windows)

## Glossary

H264 = 
HEVC = H265
VP8 = 
GOP = length of GOP (group of pictures)
RCM = rate control mode
PPS = 
I-FRAME (intra-frame) = 
IRD-FRAME ()


## Other Resources

