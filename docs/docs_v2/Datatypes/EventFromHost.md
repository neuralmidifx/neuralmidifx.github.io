---
layout: default
title: EventFromHost
nav_order: 8
has_children: false
parent: Data Types
permalink: /docs/V2_1_0/datatypes/EventFromHost

---

# EventFromHost
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---


## Description

As mentioned before the data received from the host are provided in a per-buffer basis.
For each buffer multiple information can be available:

1. DAW Status at the start of the buffer: Play, Stop, Record, Loop, Tempo, Time Signature, Buffer Position
2. MIDI Events within the buffer: Note On, Note Off ...

These data received from the host are wrapped in the `EventFromHost` data type and are provided to you in the `ITP` thread.

{: .note }
> In `ITP_Deploy.cpp`, these events are passed on to you sequentially (whenever available) 
> using the `new_event_from_host` variable of the Deploy() method.
 

{: .note}
> As mentioned before, you can specify which of these events you want to receive in the `ITP` thread 
> 
> To do so, you need to specify the event_communication_settings in the `settings.json`
> file. To learn more about this process, visit [this page](({{site.baseurl}}/docs/V2_1_0/DeploymentStages/ITP/HostEvents)


## 1. Checking if available

The `new_event_from_host` variable is an optional variable, as a result before using it, you should always check if it is available

```c++
if (new_event_from_host->has_value()) {
    // ... 
}
```
## 2. Identify the type of event

There are different types available in the `EventFromHost` data type. 

| Event Type               | Check Method                        | Description                                                                 |
|--------------------------|-------------------------------------|-----------------------------------------------------------------------------|
| FirstBufferEvent         | `new_event_from_host->isFirstBufferEvent()`  | Sent at the beginning of the host start.                                    |
| PlaybackStoppedEvent     | `new_event_from_host->isPlaybackStoppedEvent()` | Sent when the host stops the playback.                                      |
| NewBufferEvent           | `new_event_from_host->isNewBufferEvent()` | Sent at the beginning of every new buffer or when qpm, meter, etc. changes. |
| NewBarEvent              | `new_event_from_host->isNewBarEvent()` | Sent at the beginning of every new bar.                                     |
| NewTimeShiftEvent        | `new_event_from_host->isNewTimeShiftEvent()` | Sent every N QuarterNotes (as specified in the settings.json file           |
| NoteOnEvent              | `new_event_from_host->isNoteOnEvent()` | Sent when a note is played.                                                 |
| NoteOffEvent             | `new_event_from_host->isNoteOffEvent()` | Sent when a note is stopped.                                                |
| CCEvent                  | `new_event_from_host->isCCEvent()` | Sent for Control Change events.                                             |

<video width="85%" preload="auto" muted controls>
    <source src="{{ site.baseurl }}/assets/videos/BufferHostEvents.mp4" type="video/mp4"/>
</video>

## 3. Accessing the information

Regardless of the type, the following information is always available within the received event:

| Information                     | Access Method                                                                                                                       |
|---------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| **Event Time (Various Units)**      | `new_event_from_host->Time().inSeconds()`, `new_event_from_host->Time().inSamples()`, `new_event_from_host->Time().inQuarterNotes()` |
| QPM (Tempo)                     | `new_event_from_host->qpm()`                                                                                                        |
| Time Signature                  | `new_event_from_host->numerator()`, `new_event_from_host->denominator()`                                                            |
| Playback Status                 | `new_event_from_host->isPlaying()`, `new_event_from_host->isRecording()`                                                            |
| Buffer Start Time               | `new_event_from_host->BufferStartTime().inSeconds()`                                                                                |
| Buffer End Time (Seconds)       | `new_event_from_host->BufferEndTime().inSamples()`                                                                                  |
| Buffer End Time (Quarter Notes) | `new_event_from_host->BufferEndTime().inQuarterNotes()`                                                                             |
| Looping Status                  | `new_event_from_host->isLooping()`                                                                                                  |
| Loop Start and End Times  (Quarter Notes)       | `new_event_from_host->loopStart()`, `new_event_from_host->loopEnd()`                                                                |
| Number of Bars Elapsed          | `new_event_from_host->barCount()`                                                                                                   |
| Last Bar Position               | `new_event_from_host->lastBarPos().inSeconds()`, `new_event_from_host->lastBarPos().inSamples()`, `new_event_from_host->lastBarPos().inQuarterNotes()`,                                               |

The following information is available for `NoteOnEvent` and `NoteOffEvent`:

| Information               | Access Method                           |
|---------------------------|-----------------------------------------|
| Note Number               | `new_event_from_host->getNoteNumber()`      |
| Velocity                  | `new_event_from_host->getVelocity()`        |
| Channel                   | `new_event_from_host->getChannel()`         |

The following information is available for `CCEvent`:

| Information               | Access Method                           |
|---------------------------|-----------------------------------------|
| CC Number                 | `new_event_from_host->getCCNumber()`        |
| Value                     | `new_event_from_host->getCCValue()`           |
| Channel                   | `new_event_from_host->getChannel()`         |


## Example

```c++
   if (new_event_from_host.has_value()) {

        if (new_event_from_host->isFirstBufferEvent()) {

        } else if (new_event_from_host->isPlaybackStoppedEvent()) {

        } else if (new_event_from_host->isNewBufferEvent()) {

        } else if (new_event_from_host->isNewBarEvent()) {

        } else if (new_event_from_host->isNewTimeShiftEvent()) {

        } else if (new_event_from_host->isNoteOnEvent()) {
            auto note_n = new_event_from_host->getNoteNumber();
            auto vel =new_event_from_host->getVelocity();
            auto channel = new_event_from_host->getChannel();
            auto time = new_event_from_host->Time().inSeconds();
        } else if (new_event_from_host->isNoteOffEvent()) {
            auto note_n = new_event_from_host->getNoteNumber();
            auto vel =new_event_from_host->getVelocity();
            auto channel = new_event_from_host->getChannel();
        } else if (new_event_from_host->isCCEvent()) {
        }
    }
```
