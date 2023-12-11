---
layout: default
title: Deploy() Method
parent: 5. Playback Preparation
grand_parent: V1.0.0 Documentation
has_children: true
nav_order: 2
permalink: /v1_0_0/DeploymentStages/PPP/Deploy
---

# Deploy() Method
{: .no_toc }

{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

{: .note }
> For this Stage of Deployment, You should modify the following file
> 
> [PPP_Deploy.cpp](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/master/NeuralMidiFXPlugin/NeuralMidiFXPlugin/PPP_Deploy.cpp){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

---

## Description
mention that they should have already added model to project (refer to Model Preparation section)

In this method, you receive the `ModelOutput` struct received from the `MDL` thread. Moreover, you have access to the
parameters of the plugin. In here you should extract the notes to be played back as well as specify how the playback
should occur. 

{: .important}
> Methods/Variables Available:
>   - `model_output` : The `ModelOutput` struct received from the `MDL` thread 
>   - `gui_params` : The `GuiParams` struct containing the parameters of the plugin
>   - `realtimePlaybackInfo` : The `RealtimePlaybackInfo` struct containing the realtime playback information 
>   - `playbackSequence` : The `PlaybackSequence` struct that you should fill in this method and send to the `PPP` thread
>   - `playbackPolicy` : The `PlaybackPolicy` struct that you should fill in this method and send to the `PPP` thread

This method is called whenever:
 - A `ModelOutput` is received from the `MDL` thread
 - Any of the parameters are changed

In order to notify the wrapper whether the `PlaybackSequence` and/or `PlaybackPolicy` structs have been filled, you should
return a {`bool`, `bool`} pair. The first `bool` indicates whether the `PlaybackSequence` struct has been filled and the
second `bool` indicates whether the `PlaybackPolicy` struct has been filled. 


## Code

```c++
std::pair<bool, bool> PlaybackPreparatorThread::deploy(bool new_model_output_received, bool did_any_gui_params_change) {

    // =================================================================================
    // ===         1.a. ACCESSING GUI PARAMETERS
    // Refer to:
    // https://neuralmidifx.github.io/datatypes/GuiParams#accessing-the-ui-parameters
    // =================================================================================


    // =================================================================================

    // ---------------------------------------------------------------------------------
    // --- ExampleStarts ------ ExampleStarts ------ ExampleStarts ---- ExampleStarts --
    // in this example, whenever the slider is moved, I resend a sequence of generations
    // to the processor thread (NMP)
    bool testFlag{false};
    auto newPlaybackDelaySlider = gui_params.getValueFor("Generation Playback Delay");
    if (PlaybackDelaySlider != newPlaybackDelaySlider) {
        PrintMessage("PlaybackPreparatorThread: PlaybackDelaySlider changed from" +
        std::to_string(PlaybackDelaySlider) + " to " + std::to_string(newPlaybackDelaySlider));
        PlaybackDelaySlider = newPlaybackDelaySlider;
        testFlag = true;
    }
    // --- ExampleEnds -------- ExampleEnds -------- ExampleEnds ------ ExampleEnds ----
    // ---------------------------------------------------------------------------------

    // =================================================================================
    // ===         1.b. ACCESSING REALTIME PLAYBACK INFORMATION
    // Refer to:
    // https://neuralmidifx.github.io/datatypes/RealtimePlaybackInfo#accessing-the-realtimeplaybackinfo
    // =================================================================================

    // =================================================================================
    // ===         2. ACCESSING INFORMATION (EVENTS) RECEIVED FROM HOST
    // Refer to:
    //  https://neuralmidifx.github.io/datatypes/EventFromHost
    // =================================================================================

    // ---------------------------------------------------------------------------------
    // --- ExampleStarts ------ ExampleStarts ------ ExampleStarts ---- ExampleStarts --
    auto onsets = model_output.tensor1; // I'm not using this tensor below, just an e.g.

    // ...
    // --- ExampleEnds -------- ExampleEnds -------- ExampleEnds ------ ExampleEnds ----
    // ---------------------------------------------------------------------------------

    // =================================================================================
    // ===         3. Add Extracted Generations to Playback Sequence
    // Refer to:
    // https://neuralmidifx.github.io/datatypes/PlaybackSequence
    // =================================================================================

    // =================================================================================
    // ===         4. At least once, before sending generations,
    //                  Specify the PlaybackPolicy, Time_unit, OverwritePolicy
    // Refer to:
    // https://neuralmidifx.github.io/datatypes/PlaybackPolicy
    // =================================================================================

    // -----------------------------------------------------------------------------------------
    // ------ ExampleStarts ------ ExampleStarts ------ ExampleStarts ------ ExampleStarts -----
    bool newPlaybackPolicyShouldBeSent{false};
    bool newPlaybackSequenceGeneratedAndShouldBeSent{false};

    if (testFlag) {
        // 1. ---- Update Playback Policy -----
        // Can be sent just once, || every time the policy changes
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
        newPlaybackPolicyShouldBeSent = true;

        // 2. ---- Update Playback Sequence -----
        // clear the previous sequence || append depending on your requirements
        playbackSequence.clear();
        playbackSequence.clearStartingAt(0);

        // I'm generating a single note to be played at timestamp 4 && delayed by the slider value
        int channel{1};
        int note = rand() % 64;
        float velocity{0.3f};
        double timestamp{0};
        double duration{1};

        // add noteOn Offs (time stamp shifted by the slider value)
        playbackSequence.addNoteOn(channel, note, velocity,
                                   timestamp + newPlaybackDelaySlider);
        playbackSequence.addNoteOff(channel, note, velocity,
                                    timestamp + newPlaybackDelaySlider + duration);
        // or add a note with duration (an octave higher)
        playbackSequence.addNoteWithDuration(channel, note+12, velocity,
                                             timestamp + newPlaybackDelaySlider, duration);

        newPlaybackSequenceGeneratedAndShouldBeSent = true;
    }
    // --- ExampleEnds -------- ExampleEnds -------- ExampleEnds ------ ExampleEnds ----
    // ---------------------------------------------------------------------------------

    // MUST Notify What Data Ready to be Sent
    // If you don't want to send anything, just set both flags to false
    // If you have a new playback policy, set the first flag to true
    // If you have a new playback sequence, set the second flag to true
    return {newPlaybackPolicyShouldBeSent, newPlaybackSequenceGeneratedAndShouldBeSent};
}
```