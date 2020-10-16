# UnityRenderStreaming

Repository for research into ["data-moshing" bug](https://github.com/Unity-Technologies/com.unity.webrtc/issues/205) in Unity Render Streaming. Visibly from missing or malformed keyframes or headers (IDR, PPS, SPS) causing predictive-frames to be [glitchy](https://www.goodreads.com/quotes/649039-whatever-you-now-find-weird-ugly-uncomfortable-and-nasty-about). A comparison is made with [Unreal4 PixelStreaming](https://github.com/EpicGames/UnrealEngine/tree/f8f4b403eb682ffc055613c7caf9d2ba5df7f319/Engine/Plugins/Media/PixelStreaming/Source/PixelStreaming) (requires [signup](https://www.unrealengine.com/en-US/ue4-on-github) to Unreal github) which also uses NVIDIA SDK, but does not have this bug.

## Notes

* [Setting up Unity and Unreal4 development environment on PopOS (Linux)](DEV.linux.md)
* [Setting up a development environment for com.unity.webrtc (Windows)](DEV.windows.md)
* [Comparison between Unreal and Unity NVIDIA API settings](COMPARE.md)

## Glossary

Codecs:

* H264 = AVC / MPEG4 Pt.10 (advanced video coding)
* HEVC = H265 / MPEG-H Pt.2 (high efficiency video coding)
* VP8 = alt codec, H264 equivalency (google)
* VP9 = alt codec, H265 equivalency (google)

Properties:

* GOP = length of GOP (group of pictures)
* RCM = rate control mode (VBR, CBR etc)
* QP = quantization parameter (compression amount)
* PPS = [picture parameter set](https://www.quora.com/What-are-SPS-and-PPS-in-video-codecs)
* SPS = [sequence parameter set](https://www.quora.com/What-are-SPS-and-PPS-in-video-codecs)
* I-FRAME = intra frame
* IDR-FRAME = [instantaneous decoder refresh frame](https://streaminglearningcenter.com/articles/everything-you-ever-wanted-to-know-about-idr-frames-but-were-afraid-to-ask.html)
* P-FRAME = predictive frame 
* B-FRAME = bi-directional predictive frame
* NALU = network abstraction layer units

Error Resilience Mechanisms:

* NACK = [negative acknowledgement](https://www.callstats.io/blog/2015/10/30/error-resilience-mechanisms-webrtc-video)
* FIR = [full intra request](https://www.callstats.io/blog/2015/10/30/error-resilience-mechanisms-webrtc-video)
* PLI = picture loss indication (see `webrtc::RTCRtpStreamStats.pliCount`, [see also](https://stackoverflow.com/questions/64285619/webrtc-how-to-force-pli-packet-sent-from-javascript-web-app-client))
* SLI = slice loss indication (see `webrtc::RTCRtpStreamStats.sliCount`)
* LTR = long term reference pictures ([mysterious undocumented NVIDIA setting](https://forums.developer.nvidia.com/t/hevc-ltr-does-it-work-or-not/48623))

## NVIDIA API

Official recommendations:

```
Low-latency use cases like game-streaming, video conferencing etc.

Ultra-low latency or low latency Tuning Info
Rate control mode = CBR
Multi Pass – Quarter/Full (evaluate and decide)
Very low VBV buffer size (e.g. single frame = bitrate/framerate)
No B Frames
Infinite GOP length
Adaptive quantization (AQ) enabled**
Long term reference pictures***
Intra refresh***
Non-reference P frames***
Force IDR***

*: Recommended for low motion games and natural video.

**: Recommended on second generation Maxwell GPUs and above.

***: These features are useful for error recovery during transmission across noisy mediums.
```

Elsewhere:

```
High quality Tuning Info
‣ Rate control mode = CBR
‣ Medium VBV buffer size (1 second)
‣ B Frames*
‣ Look-ahead
```


* https://docs.nvidia.com/video-technologies/video-codec-sdk/pdf/NVENC_VideoEncoder_API_ProgGuide.pdf
* https://on-demand.gputechconf.com/gtc/2014/presentations/S4654-detailed-overview-nvenc-encoder-api.pdf
* https://docs.nvidia.com/video-technologies/video-codec-sdk
* http://developer.download.nvidia.com/compute/nvenc/v4.0/NVENC_AppNote.pdf
* https://docs.nvidia.com/video-technologies/video-codec-sdk/nvenc-video-encoder-api-prog-guide
* https://github.com/rigaya/NVEnc

## Unreal 

Unreal implementation references:

* [NvEncoder.cpp](unreal-NvEncoder.cpp) (see lines: [407](NvEncoder.cpp#L407) / [742](NvEncoder.cpp#L854) / [742](NvEncoder.cpp#L854))
* [NvEncoder.cpp](unreal-VideoEncoder.cpp) (see lines: [193](VideoEncoder.cpp#L193) / [269](VideoEncoder.cpp#L269))

## Other Resources

* [Handling Packet Loss in WebRTC](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/41611.pdf)
* https://webrtcforthecurious.com/
* https://www.html5rocks.com/en/tutorials/webrtc
* https://blog.mozilla.org/webrtc/the-evolution-of-webrtc/
* https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API
* https://webrtchacks.com
* https://webrtcweekly.com

