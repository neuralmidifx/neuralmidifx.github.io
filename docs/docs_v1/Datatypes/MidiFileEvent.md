---
layout: default
title: MidiFileEvent
nav_order: 4
has_children: false
parent: Data Types
permalink: /docs/v1_0_0/datatypes/MidiFileEvent
grand_parent: V1.0.0 Documentation
---

# MidiFileEvent
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Description

In case, you have enabled the dragging of MIDI files into the plugin (in `Configs_GUI.h`), whenever a MIDI 
file is dragged into the plugin, it is converted into a `MidiFileEvent` and is provided to you in the `ITP` thread.

These data received from the MIDI file are wrapped in the `MidiFileEvent` data type and are provided to you in the `ITP` thread.

{: .note }
> In `ITP_Deploy.cpp`, these events are passed on to you sequentially (whenever available) 
> using the `new_midi_event_dragdrop` variable of the Deploy() method.


## 1. Checking if available

Similar to the `EventFromHost` data type, the `new_midi_event_dragdrop` variable is an optional variable, 
as a result before using it, you should always check if it is available

```c++
if (new_midi_event_dragdrop.has_value()) {
    // ... 
}
```


## 2. Identify the type of event

There are different types available in the `MidiFileEvent` data type.

| Event Type               | Check Method                        | Description |
|--------------------------|-------------------------------------|-------------|
| isFirstMessage    | `new_midi_event_dragdrop->isFirstMessage()`  | True if this is the first message of the MIDI file. |
| isLastMessage     | `new_midi_event_dragdrop->isLastMessage()` | True if this is the last message of the MIDI file. |
|isNoteOnEvent      | `new_midi_event_dragdrop->isNoteOnEvent()` | True if this is a Note On event. |
|isNoteOffEvent     | `new_midi_event_dragdrop->isNoteOffEvent()` | True if this is a Note Off event. |
|isCCEvent          | `new_midi_event_dragdrop->isCCEvent()` | True if this is a Control Change event. |

## 3. Timing of MIDI events

The timings of MIDI events in a loaded file are automatically available in terms of number of quarter notes.
That said, using the sample rate and the tempo of the host, you can easily access these in terms of seconds or samples.

### Quarter Notes

```c++
        auto time_quarterNotes = new_midi_event_dragdrop->Time();
```

### Samples or Seconds
```c++
        //   First Get the sample rate and tempo of the host 
        auto realtime_playback_info = realtimePlaybackInfo->get();
        auto sample_rate = realtime_playback_info.sample_rate;
        auto qpm = realtime_playback_info.qpm;
        
        //    Then Get the time of the event in seconds or samples as follows
        auto time_in_seconds = new_midi_event_dragdrop->Time(sample_rate, qpm).inSeconds();
        auto time_in_samples = new_midi_event_dragdrop->Time(sample_rate, qpm).inSamples();
```

## Example

```c++
if (new_midi_event_dragdrop.has_value()) {
        auto time_quarterNotes = new_midi_event_dragdrop->Time();
        auto time_in_seconds = new_midi_event_dragdrop->Time(sample_rate, qpm).inSeconds();
        auto time_in_samples = new_midi_event_dragdrop->Time(sample_rate, qpm).inSamples();
        auto isFirst = new_midi_event_dragdrop->isFirstMessage();
        auto isLast = new_midi_event_dragdrop->isLastMessage();
        auto isNoteOn = new_midi_event_dragdrop->isNoteOnEvent();
        auto isNoteOff = new_midi_event_dragdrop->isNoteOffEvent();
        if (isNoteOn || isNoteOff) {
            auto noteNumber = new_midi_event_dragdrop->getNoteNumber();
            auto velocity = new_midi_event_dragdrop->getVelocity();
        }
        auto isCC = new_midi_event_dragdrop->isCCEvent();
        if (isCC) {
            auto ccNumber = new_midi_event_dragdrop->getCCValue();
        }

        PrintMessage(new_midi_event_dragdrop->getDescription().str());
        PrintMessage(new_midi_event_dragdrop->getDescription(sample_rate,
                                                             qpm).str());

        if (isLast) { // here I'm requesting forward pass only after entire midi file is received
            SHOULD_SEND_TO_MODEL_FOR_GENERATION_ = true;
        }
    }
```