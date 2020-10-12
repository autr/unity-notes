# Unreal4

## NvVideoEncoder.cpp


Public functions:

```

	const TCHAR* GetName() const override;
	const TCHAR* GetType() const override;
	bool Initialize(const FVideoEncoderConfig& InConfig) override;
	void Shutdown() override;
	bool CopyTexture(FTexture2DRHIRef Texture, FTimespan CaptureTs, FTimespan Duration, FBufferId& OutBufferId, FIntPoint Resolution) override;
	void Drop(FBufferId BufferId) override;
	void Encode(FBufferId BufferId, bool bForceKeyFrame, uint32 Bitrate, TUniquePtr<FEncoderVideoFrameCookie> Cookie) override;
	FVideoEncoderConfig GetConfig() const override;
	bool SetBitrate(uint32 Bitrate) override;
	bool SetFramerate(uint32 Framerate) override;
	bool SetParameter(const FString& Parameter, const FString& Value) override;

```

Exposes these base H264 profiles:

```
		case NV_ENC_PARAMS_RC_CONSTQP:
			return TEXT("ConstQP");
		case NV_ENC_PARAMS_RC_VBR:
			return TEXT("VBR");
		case NV_ENC_PARAMS_RC_CBR:
			return TEXT("CBR");
		case NV_ENC_PARAMS_RC_CBR_LOWDELAY_HQ:
			return TEXT("CBR_LOWDELAY_HQ");
		case NV_ENC_PARAMS_RC_CBR_HQ:
			return TEXT("CBR_HQ");
		case NV_ENC_PARAMS_RC_VBR_HQ:
			return TEXT("VBR_HQ");
```

And various frame types:

```
		case NV_ENC_PIC_TYPE_P:
			return TEXT("NV_ENC_PIC_TYPE_P");
		case NV_ENC_PIC_TYPE_B:
			return TEXT("NV_ENC_PIC_TYPE_B");
		case NV_ENC_PIC_TYPE_I:
			return TEXT("NV_ENC_PIC_TYPE_I");
		case NV_ENC_PIC_TYPE_IDR:
			return TEXT("NV_ENC_PIC_TYPE_IDR");
		case NV_ENC_PIC_TYPE_BI:
			return TEXT("NV_ENC_PIC_TYPE_BI");
		case NV_ENC_PIC_TYPE_SKIPPED:
			return TEXT("NV_ENC_PIC_TYPE_SKIPPED");
		case NV_ENC_PIC_TYPE_INTRA_REFRESH:
			return TEXT("NV_ENC_PIC_TYPE_INTRA_REFRESH");
```

Of these, only **NV_ENC_PIC_TYPE_IDR** is referenced again in encoder (FNvVideoEncoder::ProcessFrame).


### ::Initialize


* Switches between NV_ENC_PRESET_LOW_LATENCY_HQ_GUID and NV_ENC_PRESET_HQ_GUID presets
* Slice mode and slice mode data set to 0 due to "immediate visual artefacts"
* Mandatory SPS and PPS is set for each frame
* GOP length is set to a single frame (FPS)
* Jazzy things are disabled (reportSliceOffsets=0 enableSubFrameWrite=0)?

From the NVIDIA API docs:

```
 Application submitting buffers in Encode order must specify
 NV_ENC_PIC_PARAMS :: pictureType
 NV_ENC_PIC_PARAMS_H264 :: displayPOCSyntax
 NV_ENC_PIC_PARAMS_H264 :: refPicFlag
 NV_ENC_INITIALIZE_PARAMS :: enablePTD to 0
 Application submitting buffers in Display order must specify
 NV_ENC_CONFIG ::gopLength
 NV_ENC_CONFIG :: frameIntervalP
 NV_ENC_CONFIG_H264 :: idrPeriod
 NV_ENC_INITIALIZE_PARAMS :: enablePTD to 1
```

```


bool FNvVideoEncoder::Initialize(const FVideoEncoderConfig& InConfig)
{
	check(!bInitialized);

	Config = InConfig;
	ConfigH264 = {};
	ReadH264Settings(Config.Options, ConfigH264); # ?

	UE_LOG(LogAVEncoder, Log, TEXT("FNvVideoEncoder initialization with %u*%d, %u FPS"),
		Config.Width, Config.Height, Config.Framerate);

	// Set initialization parameters
	{
		FMemory::Memzero(NvEncInitializeParams);
		NvEncInitializeParams.version = NV_ENC_INITIALIZE_PARAMS_VER;
		// hardcoded to FullHD for now, if actual resolution is different it will be changed dynamically
		NvEncInitializeParams.encodeWidth = NvEncInitializeParams.darWidth = Config.Width;
		NvEncInitializeParams.encodeHeight = NvEncInitializeParams.darHeight = Config.Height;
		NvEncInitializeParams.encodeGUID = NV_ENC_CODEC_H264_GUID;

		if (Config.Preset == FVideoEncoderConfig::EPreset::LowLatency)
			NvEncInitializeParams.presetGUID = NV_ENC_PRESET_LOW_LATENCY_HQ_GUID;
		else if (Config.Preset == FVideoEncoderConfig::EPreset::HighQuality)
			NvEncInitializeParams.presetGUID = NV_ENC_PRESET_HQ_GUID;
		else
		{
			check(false);
		}

		NvEncInitializeParams.frameRateNum = Config.Framerate;
		NvEncInitializeParams.frameRateDen = 1;
		NvEncInitializeParams.enablePTD = 1;
		NvEncInitializeParams.reportSliceOffsets = 0;
		NvEncInitializeParams.enableSubFrameWrite = 0;
		NvEncInitializeParams.encodeConfig = &NvEncConfig;
		NvEncInitializeParams.maxEncodeWidth = 3840;
		NvEncInitializeParams.maxEncodeHeight = 2160;
		FParse::Value(FCommandLine::Get(), TEXT("NvEncMaxEncodeWidth="), NvEncInitializeParams.maxEncodeWidth);
		FParse::Value(FCommandLine::Get(), TEXT("NvEncMaxEncodeHeight="), NvEncInitializeParams.maxEncodeHeight);
	}
	// Get preset config and tweak it accordingly
	{
		NV_ENC_PRESET_CONFIG PresetConfig;
		FMemory::Memzero(PresetConfig);
		PresetConfig.version = NV_ENC_PRESET_CONFIG_VER;
		PresetConfig.presetCfg.version = NV_ENC_CONFIG_VER;
		Result = NvEncodeAPI->nvEncGetEncodePresetConfig(EncoderInterface, NvEncInitializeParams.encodeGUID, NvEncInitializeParams.presetGUID, &PresetConfig);
		checkf(NV_RESULT(Result), TEXT("Failed to select NVEncoder preset config (status: %d)"), Result);
		FMemory::Memcpy(&NvEncConfig, &PresetConfig.presetCfg, sizeof(NV_ENC_CONFIG));

		NvEncConfig.profileGUID = Config.Preset == FVideoEncoderConfig::EPreset::LowLatency ? NV_ENC_H264_PROFILE_BASELINE_GUID : NV_ENC_H264_PROFILE_MAIN_GUID;

		NvEncConfig.gopLength = NvEncInitializeParams.frameRateNum; // once a sec

		NV_ENC_RC_PARAMS& RcParams = NvEncConfig.rcParams;
		RcParams.rateControlMode = ToNvEncRcMode(ConfigH264.RcMode);

		RcParams.enableMinQP = true;
		RcParams.minQP = { 20, 20, 20 };

		RcParams.maxBitRate = Config.MaxBitrate;
		RcParams.averageBitRate = FMath::Min(Config.Bitrate, RcParams.maxBitRate);

		NvEncConfig.encodeCodecConfig.h264Config.idrPeriod = NvEncConfig.gopLength;

		// !!!

		// configure "entire frame as a single slice"
		// seems WebRTC implementation doesn't work well with slicing, default mode 
		// (Mode=3/ModeData=4 - 4 slices per frame) produces (rarely) grey full screen or just top half of it. 
		// it also can be related with our handling of slices in proxy's FakeVideoEncoder
		if (Config.Preset == FVideoEncoderConfig::EPreset::LowLatency)
		{
			NvEncConfig.encodeCodecConfig.h264Config.sliceMode = 0;
			NvEncConfig.encodeCodecConfig.h264Config.sliceModeData = 0;
		}
		else
		{
			NvEncConfig.encodeCodecConfig.h264Config.sliceMode = 3;
			NvEncConfig.encodeCodecConfig.h264Config.sliceModeData = 1;
		}

		// let encoder slice encoded frame so they can fit into RTP packets
		// commented out because at some point it started to produce immediately visible visual artifacts on players
		//NvEncConfig.encodeCodecConfig.h264Config.sliceMode = 1;
		//NvEncConfig.encodeCodecConfig.h264Config.sliceModeData = 1100; // max bytes per slice

		// repeat SPS/PPS with each key-frame for a case when the first frame (with mandatory SPS/PPS) 
		// was dropped by WebRTC
		NvEncConfig.encodeCodecConfig.h264Config.repeatSPSPPS = 1;

		// maybe doesn't have an effect, high level is chosen because we aim at high bitrate
		NvEncConfig.encodeCodecConfig.h264Config.level = Config.Preset==FVideoEncoderConfig::EPreset::LowLatency ? NV_ENC_LEVEL_H264_52 : NV_ENC_LEVEL_H264_51;

	}

}

```

### ::ProcessFrame

Flag **bForceKeyFrame** comes from base class encoder (see )

Packet object structure:

```
{

	Timestamp,
	Duration,
	Video: {
		bKeyFrame,
		FrameAvgQP,
		Width,
		Height,
		Framerate
	},
	EncodeStartTs,
	EncodeFinishTs
}
```



```
void FNvVideoEncoder::ProcessFrame(FFrame& Frame)
{
	check(Frame.State.Load() == EFrameState::Encoding);

	FOutputFrame& OutputFrame = Frame.OutputFrame;

	FAVPacket Packet(EPacketType::Video);
	NV_ENC_PIC_TYPE PicType;
	// Retrieve encoded frame from output buffer
	{
		SCOPE_CYCLE_COUNTER(STAT_NvEnc_RetrieveEncodedFrame);

		NV_ENC_LOCK_BITSTREAM LockBitstream;
		FMemory::Memzero(LockBitstream);
		LockBitstream.version = NV_ENC_LOCK_BITSTREAM_VER;
		LockBitstream.outputBitstream = OutputFrame.BitstreamBuffer;
		LockBitstream.doNotWait = NvEncInitializeParams.enableEncodeAsync;

		_NVENCSTATUS Result = NvEncodeAPI->nvEncLockBitstream(EncoderInterface, &LockBitstream);
		checkf(NV_RESULT(Result), TEXT("Failed to lock bitstream (status: %d)"), Result);

		PicType = LockBitstream.pictureType;

		// !!!

		checkf(PicType == NV_ENC_PIC_TYPE_IDR || Frame.InputFrame.bForceKeyFrame==false, TEXT("key frame requested by but not provided by NvEnc. NvEnc provided %d"), (int)PicType);
		Packet.Video.bKeyFrame = (PicType == NV_ENC_PIC_TYPE_IDR) ? true : false;
		Packet.Video.FrameAvgQP = LockBitstream.frameAvgQP; 

		// !!!

		Packet.Data = TArray<uint8>(reinterpret_cast<const uint8*>(LockBitstream.bitstreamBufferPtr), LockBitstream.bitstreamSizeInBytes);
		Result = NvEncodeAPI->nvEncUnlockBitstream(EncoderInterface, Frame.OutputFrame.BitstreamBuffer);
		checkf(NV_RESULT(Result), TEXT("Failed to unlock bitstream (status: %d)"), Result);
	}

	Frame.EncodingFinishTs = FTimespan::FromSeconds(FPlatformTime::Seconds());

	Packet.Timestamp = Frame.InputFrame.CaptureTs;
	Packet.Duration = Frame.InputFrame.Duration;
	Packet.Video.Width = Frame.InputFrame.Texture->GetSizeX();
	Packet.Video.Height = Frame.InputFrame.Texture->GetSizeY();
	Packet.Video.Framerate = NvEncInitializeParams.frameRateNum;
	Packet.Timings.EncodeStartTs = Frame.EncodingStartTs;
	Packet.Timings.EncodeFinishTs = Frame.EncodingFinishTs;

#if NVENC_VIDEO_ENCODER_DEBUG
	{
		FFrameTiming Timing;
		Timing.Total[0] = (Frame.CopyBufferFinishTs - Frame.CopyBufferStartTs).GetTotalMilliseconds();
		Timing.Total[1] = (Frame.EncodingStartTs - Frame.CopyBufferStartTs).GetTotalMilliseconds();
		Timing.Total[2] = (Frame.EncodingFinishTs - Frame.CopyBufferStartTs).GetTotalMilliseconds();

		Timing.Steps[0] = (Frame.CopyBufferFinishTs - Frame.CopyBufferStartTs).GetTotalMilliseconds();
		Timing.Steps[1] = (Frame.EncodingStartTs - Frame.CopyBufferFinishTs).GetTotalMilliseconds();
		Timing.Steps[2] = (Frame.EncodingFinishTs - Frame.EncodingStartTs).GetTotalMilliseconds();
		Timings.Add(Timing);
		// Limit the array size
		if (Timings.Num()>1000)
		{
			Timings.RemoveAt(0);
		}
	}
#endif

	UE_LOG(LogAVEncoder, VeryVerbose, TEXT("encoded %s ts %lld, %d bytes")
		, ToString(PicType)
		, Packet.Timestamp.GetTicks()
		, (int)Packet.Data.Num());

	{
		SCOPE_CYCLE_COUNTER(STAT_NvEnc_OnEncodedVideoFrameCallback);
		OnEncodedVideoFrame(Packet, MoveTemp(Frame.OutputFrame.Cookie));
	}

	Frame.State = EFrameState::Free;
}
```