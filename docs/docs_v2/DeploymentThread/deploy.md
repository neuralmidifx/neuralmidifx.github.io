---
layout: default
title: deploy() Method
parent: 3. Deployment Thread (DPL)
has_children: false
nav_order: 60
permalink: /docs/V2_1_0/DeploymentThread_DPL/deploy_method/
---

# deploy() Method
{: .no_toc }

{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Structure of the deployment process

The deployment process is done in the `PluginDeploymentThread` class, which is located in the 
[PluginCode/deploy.h](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/master/PluginCode/deploy.h) file.

In here you can define any member variables or methods you need. 

However, your main focus will be on the `deploy` method, which is called on a per-event basis and is responsible for the
deployment process. This is the method that you will be using to interact with the `NeuralMidiFX` wrapper.

```

class PluginDeploymentThread: public DeploymentThread {
public:
    // initialize your deployment thread here
    PluginDeploymentThread():DeploymentThread() {

    }

    // this method runs on a per-event basis.
    // the majority of the deployment will be done here!
    std::pair<bool, bool> deploy (
        std::optional<MidiFileEvent> & new_midi_event_dragdrop,
        std::optional<EventFromHost> & new_event_from_host,
        bool gui_params_changed_since_last_call,
        bool new_preset_loaded_since_last_call,
        bool new_midi_file_dropped_on_visualizers,
        bool new_audio_file_dropped_on_visualizers) override {

        // flags to keep track if any new data is generated and should
        // be sent to the main thread
        bool newPlaybackPolicyShouldBeSent{false};
        bool newPlaybackSequenceGeneratedAndShouldBeSent{false};

        cnt += 1;
        cout << "PluginDeploymentThread::deploy() called " << cnt << " times" << endl;
        // cout << "PluginDeploymentThread::deploy() called" << endl;

        // make sure to set these flags to true if you want to send new data to the main thread
        return {newPlaybackPolicyShouldBeSent, newPlaybackSequenceGeneratedAndShouldBeSent};
    }

private:
    // add any member variables or methods you need here
    int cnt = 0;
};

```

## Quick Guide for the deploy() Method
```c++
std::pair<bool, bool> DeploymentThread::deploy(
        std::optional<MidiFileEvent> & new_midi_event_dragdrop,
        std::optional<EventFromHost> & new_event_from_host,
        bool gui_params_changed_since_last_call,
        bool new_preset_loaded_since_last_call,
        bool new_midi_file_dropped_on_visualizers,
        bool new_audio_file_dropped_on_visualizers) {
        
    /*              IF YOU NEED TO PRINT TO CONSOLE FOR DEBUGGING,
     *                  YOU CAN USE THE FOLLOWING METHOD:
     *                      PrintMessage("YOUR MESSAGE HERE");
     */

    // flags to keep track if any new data is generated and should
    // be sent to the main thread
    bool newPlaybackPolicyShouldBeSent{false};
    bool newPlaybackSequenceGeneratedAndShouldBeSent{false};

    // =================================================================================
    // ===         1.a. ACCESSING GUI PARAMETERS
    // Refer to:
    // https://neuralmidifx.github.io/docs/V2_1_0/datatypes/GuiParams#accessing-the-ui-parameters
    // =================================================================================



    // =================================================================================
    // ===         1.b. ACCESSING REALTIME PLAYBACK INFORMATION
    // Refer to:
    // https://neuralmidifx.github.io/docs/V2_1_0/datatypes/RealtimePlaybackInfo
    // =================================================================================



    // =================================================================================
    // ===         2. ACCESSING INFORMATION (EVENTS) RECEIVED FROM HOST
    // Only events specified in `Configs_HostEvents.h` are received here
    // Refer to:
    //      https://neuralmidifx.github.io/docs/V2_1_0/DeploymentThread_DPL/SpecifyHostEvents
    // 
    // The events are received one by one wrapped in an `EventFromHost` object
    // Refer to:
    //  https://neuralmidifx.github.io/docs/V2_1_0/datatypes/EventFromHost
    // =================================================================================
    // !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    /* Warning!
         * DO NOT USE realtime_playback_info for input preparations here, because
         * the notes are most likely registered prior to NOW and stored in the queue for access
         * As Such, for this part, use only the information provided within the received
         * new_event_from_host object. */
    // !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!


    // =================================================================================
    // ===         3. ACCESSING MIDI FILES MANUALLY DROPPED ON VISUALIZERS
    // Refer to:
    // https://neuralmidifx.github.io/docs/V2_1_0/datatypes/MidiVisualizersData
    // =================================================================================

    
    // =================================================================================
    // ===         4. ACCESSING AUDIO FILES MANUALLY DROPPED ON VISUALIZERS
    // Refer to:
    // https://neuralmidifx.github.io/docs/V2_1_0/datatypes/AudioVisualizersData
    // =================================================================================
        
    
    // =================================================================================
    // ===         5. ACCESSING NEW PRESET LOADED
    // Refer to:
    // https://neuralmidifx.github.io/docs/v1_0_0/datatypes/PresetEvent
    // =================================================================================
    
    
    // =================================================================================
    // ===         6. ACCESSING NEW MIDI FILE DROPPED ON VISUALIZERS
    // Refer to:
    // https://neuralmidifx.github.io/docs/v1_0_0/datatypes/MidiFileEvent
    // =================================================================================
    
    
    // =================================================================================
    // ===         7. LOADING THE MODEL
    // Refer to:
    // https://neuralmidifx.github.io/docs/V2_1_0/DeploymentThread_DPL/LoadModel
    // =================================================================================
    // Try loading the model if it hasn't been loaded yet
    if (!isModelLoaded) {
        load("drumLoopVAE.pt");
    }


    // =================================================================================
    // ===         8. Inference
    // =================================================================================
    if (/* your condition to check if you should run inference */) {

        /* ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
         * Use DisplayTensor to display the data if debugging
         * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ */
        // DisplayTensor(some_torch_tensor, "INPUT");

        if (isModelLoaded)
        {   
            // =================================================================================
            // ===       Prepare your input
            // =================================================================================
            // Prepare The Input To Your Model
            // example: let's assume the pytorch model has some method that you want to
            // use for inference, e.g.:
            //          def SomeMethod(self, x: float, y: torch.Tensor)
            //             ....
            //             return a, b

            // Step A: To Prepare the inputs, first create a vector of torch::jit::IValue
            // e.g.:
            // >> std::vector<torch::jit::IValue> inputs;

            // Step B: Push the input parameters
            // Remember that if you need data passed from the ITP thread,
            // you can access it from the model_input struct
            // e.g.:
            // >> inputs.emplace_back(float(Slider1)); // this is the x parameter
            // >> inputs.emplace_back(model_input.tensor1); // this is the y parameter
            // =================================================================================

            // =================================================================================
            // ===        Run Inference
            // =================================================================================
            // Get the method for inference or use the forward method
            // auto SomeMethod = model.get_method("SomeMethod");
            // auto outs = SomeMethod(inputs);
            // auto outs = model.forward(inputs);

            // =================================================================================
            // ===        Prepare the model_output struct and return true to signal
            // ===        that the output is ready to be sent to PPP
            // =================================================================================
            // auto outs_as_tuple = outs.toTuple()->elements();
            // auto first_returned_tensor = outs_as_tuple[0].toTensor();
            // auto second_returned_tensor = outs_as_tuple[1].toTensor();

        }
    } 
    
    // =================================================================================
    // ===         9. Add Extracted Generations to Playback Sequence
    // Refer to:
    // https://neuralmidifx.github.io/docs/V2_1_0/datatypes/PlaybackSequence
    // =================================================================================

    // =================================================================================
    // ===         10. At least once, before sending generations,
    //                  Specify the PlaybackPolicy, Time_unit, OverwritePolicy
    // Refer to:
    // https://neuralmidifx.github.io/docs/V2_1_0/datatypes/PlaybackPolicy
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
    
    // =================================================================================
    // ===         11. Return the flags to notify main thread 
    //                 whether new sequence is ready to be received for playback and also 
    //                 whether how to handle the playback sequence
    // =================================================================================
    // MUST Notify What Data Ready to be Sent
    // If you don't want to send anything, just set both flags to false
    // If you have a new playback policy, set the first flag to true
    // If you have a new playback sequence, set the second flag to true
    return {newPlaybackPolicyShouldBeSent, newPlaybackSequenceGeneratedAndShouldBeSent};
     
}
```