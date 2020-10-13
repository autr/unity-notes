# UnityRenderStreaming

Repository for research into ["data-moshing" bug](https://github.com/Unity-Technologies/com.unity.webrtc/issues/205) in Unity Render Streaming. Visibly from missing or malformed keyframes or headers (IDR, PPS, SPS) causing predictive-frames to be [glitchy](https://www.goodreads.com/quotes/649039-whatever-you-now-find-weird-ugly-uncomfortable-and-nasty-about).

Some comparison is made between [Unreal4 PixelStreaming](https://github.com/EpicGames/UnrealEngine/tree/f8f4b403eb682ffc055613c7caf9d2ba5df7f319/Engine/Plugins/Media/PixelStreaming/Source/PixelStreaming)* which also uses the NVIDIA SDK, but without glitches.

*requires signup to Unreal developers github

## Notes

* [Setting up Unity and Unreal4 development environment on PopOS (Linux)](DEV.linux.md)
* [Setting up a development environment for com.unity.webrtc (Windows)](DEV.windows.md)
* [Comparison between Unreal and Unity NVIDIA API settings](COMPARE.md)

## Glossary

* H264 = AVC / MPEG4 Pt.10 (advanced video coding)
* HEVC = H265 / MPEG-H Pt.2 (high efficiency video coding)
* VP8 = alt codec, H264 equivalency (google)
* VP9 = alt codec, H265 equivalency (google)
* GOP = length of GOP (group of pictures)
* RCM = rate control mode (VBR, CBR etc)
* QP = quantization parameter (compression amount)
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

