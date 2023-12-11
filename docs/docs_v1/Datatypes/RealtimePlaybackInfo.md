---
layout: default
title: RealtimePlaybackInfo
nav_order: 2
has_children: false
parent: Data Types
permalink: /v1_0_0/docs/datatypes/RealtimePlaybackInfo
grand_parent: V1.0.0 Documentation
---

# RealtimePlaybackInfo
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Description

The RealtimePlaybackInfo data type contains information about the current playback state of the DAW 
(e.g. play, pause, stop, etc.).

This information is available in all threads in a variable called `realtimePlaybackInfo`.

## Accessing the RealtimePlaybackInfo

To get the current playback state, follow these steps:

1. Get the data using the `get()` method

```c++
    auto realtime_playback_info = realtimePlaybackInfo->get();
```

2. All information are now directly available in the `realtime_playback_info` variable:

```c++
auto sample_rate = realtime_playback_info.sample_rate;
auto buffer_size_in_samples = realtime_playback_info.buffer_size_in_samples;
auto qpm = realtime_playback_info.qpm;
auto numerator = realtime_playback_info.numerator;
auto denominator = realtime_playback_info.denominator;
auto isPlaying = realtime_playback_info.isPlaying;
auto isRecording = realtime_playback_info.isRecording;
auto current_time_in_samples = realtime_playback_info.time_in_samples;
auto current_time_in_seconds = realtime_playback_info.time_in_seconds;
auto current_time_in_quarterNotes = realtime_playback_info.time_in_ppq;
auto isLooping = realtime_playback_info.isLooping;
auto loopStart_in_quarterNotes = realtime_playback_info.loop_start_in_ppq;
auto loopEnd_in_quarterNotes = realtime_playback_info.loop_end_in_ppq;
auto last_bar_pos_in_quarterNotes = realtime_playback_info.ppq_position_of_last_bar_start;
```

{: .warning } 
> Not every DAW supports all of these features. 
> If these values are not available in your DAW, they will be set to -1.

{: .warning }
> You should avoid using these values in ITP during your tokenization process. 
> Rather, use the information available in the `EventFromHost` data type.
> 
> The reason for this is that the host status bundled in the `EventFromHost` data type corresponds to
> the time the event was registered, which can very well be different from the realtime playback status.