---
layout: default
title: PlaybackPolicy
nav_order: 7
has_children: false
parent: Data Types
permalink: /datatypes/PlaybackPolicy
---

# PlaybackPolicy
{: .no_toc }

---

## Description

The playback policy specifies how generated events are to be interpreted by the wrapper. 

In the `PPP` thread, this data is already instantiated and can be accessed via the 
`playbackPolicy` variable.

| Aspect               | Options & Description                                                                                                                                                                                                                                                                                            |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Timing Reference** | - **Relative to Now**: Register the current time and play messages relative to that. <br> - **Relative to 0**: Stream should be played relative to absolute 0 time. <br> - **Relative to Playback Start**: Stream should be played relative to the time when playback started.                                   |
| **Time Unit**        | - **Seconds**: Time unit is in seconds. <br> - **PPQ (Pulses Per Quarter note)**: Time unit is in pulses per quarter note. <br> - **Audio Samples**: Time unit is in audio samples.                                                                                                                              |
| **Overwrite Policy** | - **Delete All Events in Previous Stream**: Delete all events in the previous stream and use the new stream. <br> - **Delete All Events After Now**: Delete all events after the current time. <br> - **Keep All Previous Events**: Generations can be played on top of each other, keeping all previous events. |
| **Additional Info**  | - **Clear Generations After Pause/Stop**: Remove generations once playback is paused or stopped. <br> - **Loop Generations Indefinitely**: Loop generations indefinitely until a new one is received. Loop starts at the specified Timing Reference. |                                                            


## 
## Example Usage

This section provides code snippets demonstrating how to set various playback policies.

```cpp
// Can be sent just once OR every time the policy changes Depending on the target task

// playbackPolicy.SetPaybackPolicy_RelativeToNow();  // or
playbackPolicy.SetPlaybackPolicy_RelativeToAbsoluteZero(); // or
// playbackPolicy.SetPlaybackPolicy_RelativeToPlaybackStart(); // or

// playbackPolicy.SetTimeUnitIsSeconds(); // or
playbackPolicy.SetTimeUnitIsPPQ(); // or
// playbackPolicy.SetTimeUnitIsAudioSamples(); // or FIXME Timestamps near zero don't work well in loop mode

bool forceSendNoteOffsFirst{true};
// playbackPolicy.SetOverwritePolicy_DeleteAllEventsInPreviousStreamAndUseNewStream(forceSendNoteOffsFirst); // or
playbackPolicy.SetOverwritePolicy_DeleteAllEventsAfterNow(forceSendNoteOffsFirst); // or
// playbackPolicy.SetOverwritePolicy_KeepAllPreviousEvents(forceSendNoteOffsFirst); // or

// playbackPolicy.SetClearGenerationsAfterPauseStop(false); //
 playbackPolicy.ActivateLooping(4);
// playbackPolicy.DisableLooping();

```