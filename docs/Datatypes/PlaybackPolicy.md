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

TOC
{:toc}
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

## `playbackPolicy` Methods Summary

| Method                                                                                      | Description                                                                     |
|---------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| `SetPlaybackPolicy_RelativeToNow()`                                                         | Sets the timing specification to be relative to the current time.               |
| `SetPlaybackPolicy_RelativeToAbsoluteZero()`                                                | Sets the timing specification to be relative to absolute 0 time.                |
| `SetPlaybackPolicy_RelativeToPlaybackStart()`                                               | Sets the timing specification to be relative to the time when playback started. |
| `SetTimeUnitIsSeconds()`                                                                    | Sets the time unit to seconds.                                                  |
| `SetTimeUnitIsPPQ()`                                                                        | Sets the time unit to pulses per quarter note (PPQ).                            |
| `SetTimeUnitIsAudioSamples()`                                                               | Sets the time unit to audio samples.                                            |
| `SetOverwritePolicy_DeleteAllEventsInPreviousStreamAndUseNewStream(forceSendNoteOffsFirst)` | Deletes all events in the previous stream and uses the new stream.              |
| `SetOverwritePolicy_DeleteAllEventsAfterNow(forceSendNoteOffsFirst)`                        | Deletes all events after the current time.                                      |
| `SetOverwritePolicy_KeepAllPreviousEvents(forceSendNoteOffsFirst)`                          | Retains all previous events, allowing new generations to be played on top.      |
| `SetClearGenerationsAfterPauseStop(bool)`                                                   | Determines whether to remove generations once playback is paused or stopped.    |
| `ActivateLooping(loopDurationInUserUnit)`                                                   | Activates looping of generations for a specific duration in user unit           |
| `DisableLooping()`                                                                          | Deactivates looping of generations.                                             |

*  forceSendNoteOffsFirst determines whether to send note offs after deleting to ensure no hanging note ons. 
 
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