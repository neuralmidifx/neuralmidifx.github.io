---
layout: default
title: PlaybackSequence
nav_order: 6
has_children: false
parent: Data Types
permalink: /docs/v1_0_0/datatypes/PlaybackSequence
grand_parent: V1.0.0 Documentation
---

# PlaybackSequence
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Description

This data type is a wrapper for the generated content and contains the following fields.

In the `PPP` thread, this data is already instantiated and can be accessed via the 
`playbackSequence` variable. 

{: .note }
> The timing of the events added to the sequence can be any unit you want.
> That said, you should make sure that the [PlaybackPolicy]({{site.baseurl}}/docs/v1_0_0/datatypes/PlaybackPolicy) is set to the same unit.
>

## Usage

At any time you can either clear the existing content or add new content to the sequence.

### Clearing the Sequence

To clear the sequence, call the `clear()` method.

```cpp
playbackSequence.clear();
```

Alternatively, you can also clear the sequence starting at a specific time.

```cpp
playbackSequence.clearStartingAt();
```


### Adding NoteOn, NoteOff, or Full Note Messages to the Sequence

To add a note to the sequence, call the `addNoteOn()` or `addNoteOff()` or `addNoteWithDuration()` methods.

| Method | Description                                           |Note Range|Velocity Range| Time Unit                                         |
|--------|-------------------------------------------------------|----------|--------------|---------------------------------------------------|
|addNoteOn(int channel, int noteNumber, float velocity, double time)| Adds a NoteOn message to the sequence                 | 0-127 | 0-1 |see Note above |
|addNoteOff(int channel, int noteNumber, float velocity, double time)| Adds a NoteOff message to the sequence                | 0-127 | 0-1 |see Note above |
| addNoteWithDuration(int channel, int noteNumber, float velocity, double time, double duration)| Adds a NoteOn and NoteOff message to the sequence starting at the specified time and ending at the specified time + duration | 0-127 | 0-1 |see Note above |


### Example

Here, I'm clearing the sequence, then adding a random note on to be played at timestamp 4 with a note off at 1 units later.
Moreover, I'm adding a note with duration (an octave higher) to be played at timestamp 4 and ending at timestamp 5.

```cpp
        // clear the previous sequence || append depending on your requirements
        playbackSequence.clear();

        // Note Information
        int channel{1};
        int note = rand() % 64;
        float velocity{0.3f};
        double timestamp{0};
        double duration{1};

        // add noteOn Offs (time stamp shifted by the slider value)
        playbackSequence.addNoteOn(channel, note, velocity, timestamp );
        playbackSequence.addNoteOff(channel, note, velocity, timestamp + duration);
        
        // add a note with duration (an octave higher)
        playbackSequence.addNoteWithDuration(channel, note+12, velocity, timestamp , duration);
```