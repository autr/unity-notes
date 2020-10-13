# UnityRenderStreaming

Repository for resarch into fixing a Unity Render Streaming [bug that causes "data-moshing"](https://github.com/Unity-Technologies/com.unity.webrtc/issues/205) when there is packet loss - ie. missing or malformed keyframes (idr-frame, PPS, SPS) causing subsequent predictive-frames to be [glitchy](https://www.goodreads.com/quotes/649039-whatever-you-now-find-weird-ugly-uncomfortable-and-nasty-about).

Some comparison is made between [Unreal4 PixelStreaming](https://github.com/EpicGames/UnrealEngine/tree/f8f4b403eb682ffc055613c7caf9d2ba5df7f319/Engine/Plugins/Media/PixelStreaming/Source/PixelStreaming)* which also uses the NVIDIA SDK, but without glitches. [NVIDIA API settings](https://github.com/EpicGames/UnrealEngine/blob/f8f4b403eb682ffc055613c7caf9d2ba5df7f319/Engine/Source/Runtime/AVEncoder/Private/Microsoft/Windows/NvVideoEncoder.cpp)* however are relatively identical, whereas adaptive bitrate / FPS logic is custom for each implmentation.

*requires signup to Unreal developers github

## Notes

* [Setting up Unity and Unreal4 development environment on PopOS (Linux)](DEV.linux.md)
* [Setting up a development environment for com.unity.webrtc (Windows)](DEV.windows.md)
* [Review of Unreal Engine 4's PixelStreaming (Windows)](UNREAL.md)
* [Comparison between Unreal and Unity NVIDIA encoder (Windows)](COMPARE.md)

## Glossary

* H264 = AVC / MPEG4 Pt.10 (advanced video coding)
* HEVC = H265 / MPEG-H Pt.2 (high efficiency video coding)
* VP8 = H264 competitor (google)
* VP9 = H265 comeptitor (google)
* GOP = length of GOP (group of pictures)
* RCM = rate control mode
* PPS = [picture parameter set](https://www.quora.com/What-are-SPS-and-PPS-in-video-codecs)
* SPS = [sequence parameter set](https://www.quora.com/What-are-SPS-and-PPS-in-video-codecs)
* I-FRAME = intra frame
* IDR-FRAME = [instantaneous decoder refresh frame](https://streaminglearningcenter.com/articles/everything-you-ever-wanted-to-know-about-idr-frames-but-were-afraid-to-ask.html)
* P-FRAME = predictive frame 
* B-FRAME = bi-directional predictive frame


## Other Resources

* https://webrtcforthecurious.com/
* https://www.html5rocks.com/en/tutorials/webrtc
* https://blog.mozilla.org/webrtc/the-evolution-of-webrtc/
* https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API
* https://webrtchacks.com
* https://webrtcweekly.com