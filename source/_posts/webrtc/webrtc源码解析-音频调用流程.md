---
title: webrtc源码解析-音频调用流程
date: 2022-01-01 20:30:20
tags: Source
category: WebRTC
---

本文主要记录下我阅读 webrtc 时候的其中关于从音频启动 voice-engine 到通过创建 source 获取到数据的整个流程，
至于如何通过 source 传输 rtp 有待后续阅读分析。

本人阅读的代码在 window 端，因而可能更关于 window 的实现


## create adm

整个过程主要是创建 adm (audio device manager) 的过程，这里通过创建获取到了 window 的音频采集能力

```
api/create_peerconnection_factory.cc:70 CreateModularPeerConnectionFactory
pc/peer_connection_factory.cc:71 PeerConnectionFactory::Create Create
pc/peer_connection_factory.cc:85  ConnectionContext::Create Create
pc/connection_context.cc:126 ChannelManager::Create Create
pc/channel_manager.cc:39 ChannelManager::Init Init
media/base/media_engine.cc:180 CompositeMediaEngine::Init Init
media/engine/webrtc_voice_engine.cc:288 webrtc::AudioDeviceModule::Create Create
media/modules/audio_device/audio_device_impl.cc:77 AudioDeviceModule::CreateForTest CreateForTest
media/modules/audio_device/audio_device_impl.cc:81 CreatePlatformSpecificObjects
	media/modules/audio_device/audio_device_impl.cc:81 AudioDeviceWindowsCore::CoreAudioIsSupported end
media/modules/audio_device/audio_device_impl.cc:110 AttachAudioBuffer
media/modules/audio_device/audio_device_impl.cc:313 AttachAudioBuffer
```

## register callback

通过注册回调的方式将 adm 采集的音频数据，发送给具有 source 接口的回调。这里没有凸显 source，
是因为 source 是在后续创建，后续添加吧！

```
media/engine/webrtc_voice_engine.cc:288 WebRtcVoiceEngine::Init
media/modules/audio_device/audio_device_impl:849 AudioDeviceModuleImpl::RegisterAudioCallback
modules/audio_device/audio_device_buffer.cc:86 AudioDeviceBuffer::RegisterAudioCallback
modules/audio_device/audio_device_buffer.cc:261 int32_t AudioDeviceBuffer::DeliverRecordedData
audio/audio_transport_impl.cc:107 AudioTransportImpl::RecordedDataIsAvailable
audio/audio_transport_impl.cc:176 SendProcessedData
audio/audio_transport_impl.cc:190 SendAudioData(audio_sender)
```

## insertable stream

这里我阅读这里的代码主要原因是为了查看 `insertable stream` 如何实现，目前还在看中

```
peer_connection_factory:190 PeerConnectionFactory::CreatePeerConnectionOrError PeerConnection::Create
peer_connection.cc:408  PeerConnection::Create pc->Initialize
peer_connection.cc:620  PeerConnection::Initialize sdp_handler_
peer_connection.cc:1327 PeerConnection::SetLocalDescription SetLocalDescription
sdp_offer_answer.cc:1118 SdpOfferAnswerHandler::SetLocalDescription DoSetLocalDescription
sdp_offer_answer.cc:1859 SdpOfferAnswerHandler::DoSetLocalDescription ApplyLocalDescription
sdp_offer_answer.cc:1246 SdpOfferAnswerHandler::ApplyLocalDescription UpdateSessionState
sdp_offer_answer.cc::2467 SdpOfferAnswerHandler::UpdateSessionState PushdownMediaDescription
sdp_offer_answer.cc:4169 SdpOfferAnswerHandler::PushdownMediaDescription setLocalContent
channel.cc:278 BaseChannel:SetLocalContent SetLocalContent_w
channel.cc:824 VoiceChannel:SetLocalContent_w UpdateLocalStreams_w
channel.cc:604 BaseChannel:UpdateLocalStreams_w AddSendStream
webrtc_voice_engine.cc:1914 WebRtcVoiceMediaChannel:AddSendStream WebRtcAudioSendStream
webrtc_voice_engine.cc:749 WebRtcAudioSendStream:WebRtcAudioSendStream CreateAudioSendStream
call.cc:749 Call:CreateAudioSendStream AudioSendStream
audio_send_stream.cc:100 AudioSendStream:AudioSendStream
channel_send.cc:886 ChannelSend:SetEncoderToPacketizerFrameTransformer
channel_send.cc:905 ChannelSend:InitFrameTransformerDelegate
```
