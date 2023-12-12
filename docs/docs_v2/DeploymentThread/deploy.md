---
layout: default
title: deploy() Method
parent: 3. Deployment Thread (DPL)
has_children: false
nav_order: 60
permalink: /docs/v2_0_0/DeploymentThread_DPL/deploy_method/
---

# deploy() Method
{: .no_toc }

{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

```c++
bool DeploymentThread::deploy(
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
    // https://neuralmidifx.github.io/docs/v2_0_0/datatypes/GuiParams#accessing-the-ui-parameters
    // =================================================================================




    // =================================================================================
    // ===         1.b. ACCESSING REALTIME PLAYBACK INFORMATION
    // Refer to:
    // https://neuralmidifx.github.io/docs/v1_0_0/datatypes/RealtimePlaybackInfo#accessing-the-realtimeplaybackinfo
    // =================================================================================




    // =================================================================================
    // ===         2. ACCESSING INFORMATION (EVENTS) RECEIVED FROM HOST
    // Refer to:
    //  https://neuralmidifx.github.io/docs/v1_0_0/datatypes/EventFromHost
    // =================================================================================
    // !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    /* Warning!
         * DO NOT USE realtime_playback_info for input preparations here, because
         * the notes are most likely registered prior to NOW and stored in the queue for access
         * As Such, for this part, use only the information provided within the received
         * new_event_from_host object. */
    // !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!


    // =================================================================================
    // ===         3. ACCESSING INFORMATION (EVENTS) RECEIVED FROM
    //                Mannually Drag-Dropped Midi Files
    // Refer to:
    // https://neuralmidifx.github.io/docs/v1_0_0/datatypes/MidiFileEvent
    // =================================================================================



    
}

```