---
title: Azure Media Services LiveEvent types | Microsoft Docs
description: This article shows a detailed table that compares LiveEvent types. 
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''

ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 06/13/2019
ms.author: juliako

---
# Live Event types comparison

In Azure Media Services, a  [Live Event](https://docs.microsoft.com/rest/api/media/liveevents) can be one of two types: live encoding and pass-through. 

## Types comparison 

The following table compares features of the Live Event types. The types are set during creation using these properties:

* **LiveEventEncodingType.None** - An on-premises live encoder sends a multiple bitrate stream. The ingested streams passes through the Live Event without any further processing. 
* **LiveEventEncodingType.Standard** - An on-premises live encoder sends a single-bitrate stream to the Live Event and Media Services creates multiple bitrate streams. If the contribution feed is of 720p or higher resolution, the **Default720p** preset will encode a set of 6 resolution/bit rate pairs (details follow later in the article).
* **LiveEventEncodingType.Premium1080p** - An on-premises live encoder sends a single-bitrate stream to the Live Event and Media Services creates multiple bitrate streams. The Default1080p preset specifies the output set of resolution/bit rate pairs (details follow later in the article). 

| Feature | Pass-through Live Event | Standard or Premium1080p Live Event |
| --- | --- | --- |
| Single bitrate input is encoded into multiple bitrates in the cloud |No |Yes |
| Maximum video resolution for contribution feed |4K (4096x2160 at 60 frames/sec) |1080p (1920x1088 at 30 frames/sec)|
| Recommended maximum layers in contribution feed|Up to 12|One audio|
| Maximum layers in output| Same as input|Up to 6 (see System Presets below)|
| Maximum aggregate bandwidth of contribution feed|60 Mbps|N/A|
| Maximum bitrate for a single layer in the contribution |20 Mbps|20 Mbps|
| Support for multiple language audio tracks|Yes|No|
| Supported input video codecs |H.264/AVC and H.265/HEVC|H.264/AVC|
| Supported output video codecs|Same as input|H.264/AVC|
| Supported video bit depth, input, and output|Up to 10-bit including HDR 10/HLG|8-bit|
| Supported input audio codecs|AAC-LC, HE-AAC v1, HE-AAC v2|AAC-LC, HE-AAC v1, HE-AAC v2|
| Supported output audio codecs|Same as input|AAC-LC|
| Maximum video resolution of output video|Same as input|Standard - 720p, Premium1080p - 1080p|
| Maximum frame rate of input video|60 frames/second|Standard or Premium1080p - 30 frames/second|
| Input protocols|RTMP, fragmented-MP4 (Smooth Streaming)|RTMP, fragmented-MP4 (Smooth Streaming)|
| Price|See the [pricing page](https://azure.microsoft.com/pricing/details/media-services/) and click on "Live Video" tab|See the [pricing page](https://azure.microsoft.com/pricing/details/media-services/) and click on "Live Video" tab|
| Maximum run time| 24 hrs x 365 days, live linear | Up to 24 hours|
| Ability to pass through embedded CEA 608/708 captions data|Yes|Yes|
| Support for inserting slates|No|No|
| Support for ad signaling via API| No|No|
| Support for ad signaling via SCTE-35 in-band messages|Yes|Yes|
| Ability to recover from brief stalls in contribution feed|Yes|Partial|
| Support for non-uniform input GOPs|Yes|No – input must have fixed GOP duration|
| Support for variable frame rate input|Yes|No – input must be fixed frame rate. Minor variations are tolerated, for example, during high motion scenes. But the contribution feed cannot drop the frame rate (for example, to 15 frames/second).|
| Auto-shutoff of Live Event when input feed is lost|No|After 12 hours, if there is no LiveOutput running|

## System presets

The resolutions and bitrates contained in the output from the live encoder is determined by the [presetName](https://docs.microsoft.com/rest/api/media/liveevents/create#liveeventencoding). If using a **Standard** live encoder (LiveEventEncodingType.Standard), then the *Default720p* preset specifies a set of 6 resolution/bit rate pairs described below. Otherwise, if using a **Premium1080p** live encoder (LiveEventEncodingType.Premium1080p), then the *Default1080p* preset specifies the output set of resolution/bit rate pairs.

> [!NOTE]
> You cannot apply the Default1080p preset to a Live Event if it has been setup for Standard live encoding - you will get an error. You will also get an error if you try to apply the Default720p preset to a Premium1080p live encoder.

### Output Video Streams for Default720p

If the contribution feed is of 720p or higher resolution, the **Default720p** preset will encode the feed into the following 6 layers. In the table below, Bitrate is in kbps, MaxFPS represents that maximum allowed frame rate (in frames/second), Profile represents the H.264 Profile used.

| Bitrate | Width | Height | MaxFPS | Profile |
| --- | --- | --- | --- | --- |
| 3500 |1280 |720 |30 |High |
| 2200 |960 |540 |30 |High |
| 1350 |704 |396 |30 |High |
| 850 |512 |288 |30 |High |
| 550 |384 |216 |30 |High |
| 200 |340 |192 |30 |High |

> [!NOTE]
> If you need to customize the live encoding preset, please open a support ticket via Azure Portal. You should specify the desired table of resolution and bitrates. Do verify that there is only one layer at 720p, and at most 6 layers. Also do specify that you are requesting a preset for a Standard live encoder.
> The specific values of the bit rates and resolutions may be adjusted over time

### Output Video Streams for Default1080p

If the contribution feed is of 1080p resolution, the **Default1080p** preset will encode the feed into the following 6 layers.

| Bitrate | Width | Height | MaxFPS | Profile |
| --- | --- | --- | --- | --- |
| 5500 |1920 |1080 |30 |High |
| 3000 |1280 |720 |30 |High |
| 1600 |960 |540 |30 |High |
| 800 |640 |360 |30 |High |
| 400 |480 |270 |30 |High |
| 200 |320 |180 |30 |High |

> [!NOTE]
> If you need to customize the live encoding preset, please open a support ticket via Azure Portal. You should specify the desired table of resolution and bitrates. Do verify that there is only one layer at 1080p, and at most 6 layers. Also do specify that you are requesting a preset for a Premium1080p live encoder.
> The specific values of the bit rates and resolutions may be adjusted over time.

### Output Audio Stream for Default720p and Default1080p

For both *Default720p* and *Default1080p* presets, audio is encoded to stereo AAC-LC at 128 kbps, sampling rate of 48 kHz.

## Implicit properties of the live encoder

The previous section describes the properties of the live encoder that can be controlled explicitly, via the preset - such as the number of layers, resolutions, and bitrates. This section clarifies the implicit properties.

### Group of pictures (GOP) duration

The live encoder follows the [GOP](https://en.wikipedia.org/wiki/Group_of_pictures) structure of the contribution feed - which means the output layers will have the same GOP duration. Hence, it is recommended that you configure the on-premises encoder to produce a contribution feed that has fixed GOP duration (typically 2 seconds). This will ensure that the outgoing HLS and MPEG DASH streams from the service also has fixed GOP durations. Small variations in GOP durations are likely to be tolerated by most devices.

### Frame rate

The live encoder also follows the durations of the individual video frames in the contribution feed - which means the output layers will have frames with the same durations. Hence, it is recommended that you configure the on-premises encoder to produce a contribution feed that has fixed frame rate (at most 30 frames/second). This will ensure that the outgoing HLS and MPEG DASH streams from the service also has fixed frame rates durations. Small variations in frame rates are likely to be tolerated by most devices, but your on-premises live encoder should not be dropping frames (eg. under low battery conditions).

### Resolution of contribution feed and output layers

The live encoder is configured to avoid upconverting the contribution feed. As a result the maximum resolution of the output layers will not exceed that of the contribution feed.

For example, if you send a contribution feed at 720p to a Live Event configured for Default1080p live encoding, the output will only have 5 layers, starting with 720p at 3Mbps, going down to 1080p at 200 kbps. Or if you send a contribution feed at 360p into a Live Event configured for Standard live encoding, the output will contain 3 layers (at resolutions of 288p, 216p, and 192p). In the degenerate case, if you send a contribution feed of, say, 160x90 pixels to a Standard live encoder, the output will contain one layer at 160x90 resolution at 200 kbps.

### Bitrate of contribution feed and output layers

The live encoder is configured to honor the bitrate settings in the preset, irrespective of the bitrate of the contribution feed. As a result the bitrate of the output layers may exceed that of the contribution feed. For example, if you send in a contribution feed at a resolution of 720p at 1 Mbps, the output layers will remain the same as in the [table](live-event-types-comparison.md#output-video-streams-for-default720p) above.

## Next steps

[Live streaming overview](live-streaming-overview.md)
