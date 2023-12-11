---
layout: default
title: B. Deploy() Method
parent: 3. Input Preparation
grand_parent: V1.0.0 Documentation
has_children: true
nav_order: 2
permalink: /docs/v1_0_0/DeploymentStages/ITP/Deploy
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
> [ITP_Deploy.cpp](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/master/NeuralMidiFXPlugin/NeuralMidiFXPlugin/ITP_Deploy.cpp){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

---


## Description

In this method, you should prepare the `ModelInput` struct and send it to the `MDL` thread. In other words, 
any preprocessing (tokenization, ...) that you want to do on the input data should be done here. 

{: .note}
> The deploy method is called whenver:
>  - A HostEvent is received
>  - A Midi File is dropped on the plugin
>  - Any of the parameters are changed

{: .important}
> Whenever you think that the data is ready for inference, you should return `true` from this method.
> Otherwise, you should return `false`. 

## Code

```c++
bool InputTensorPreparatorThread::deploy(
    std::optional<MidiFileEvent> & new_midi_event_dragdrop,
    std::optional<EventFromHost> & new_event_from_host,
    bool gui_params_changed_since_last_call) {
    /*              IF YOU NEED TO PRINT TO CONSOLE FOR DEBUGGING,
     *                  YOU CAN USE THE FOLLOWING METHOD:
     *                      PrintMessage("YOUR MESSAGE HERE");
     */

    /* A flag like this one can be used to check whether || not the model input
        is ready to be sent to the model thread (MDL)*/
    bool SHOULD_SEND_TO_MODEL_FOR_GENERATION_ = false;

    // =================================================================================
    // ===         1.a. ACCESSING GUI PARAMETERS
    // Refer to:
    // https://neuralmidifx.github.io/docs/v1_0_0/datatypes/GuiParams#accessing-the-ui-parameters
    // =================================================================================


    // =================================================================================


    // =================================================================================
    // ===         1.b. ACCESSING REALTIME PLAYBACK INFORMATION
    // Refer to:
    // https://neuralmidifx.github.io/docs/v1_0_0/datatypes/RealtimePlaybackInfo#accessing-the-realtimeplaybackinfo
    // =================================================================================


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

    // =================================================================================
    // ===         3. ACCESSING INFORMATION (EVENTS) RECEIVED FROM
    //                Mannually Drag-Dropped Midi Files
    // Refer to:
    // https://neuralmidifx.github.io/docs/v1_0_0/datatypes/MidiFileEvent
    // =================================================================================


    // =================================================================================


    // =================================================================================
    // ===         4. Sending data to the model thread (MDL) if necessary
    // =================================================================================

    // A.  Place necessary data in `model_input` object


    // B.  Return true if updated data should be sent to the model thread (MDL)


    /* All data to be sent to the model thread (MDL) should be stored in the model_input
            object. This object is defined in the header file of this class.
            The class ModelInput is defined in the file model_input.h && should be modified
            to include all the data you want to send to the model thread.

         Once prepared && should be sent, return true from this function! Otherwise,
         return false. --> NOTE: This is necessary so that the wrapper can know when to
         send the data to the model thread. */

    if (SHOULD_SEND_TO_MODEL_FOR_GENERATION_) {
        // Example:
        //      If should send to model, update model_input && return true
        model_input.tensor1 = torch::rand({1, 32, 27}, torch::kFloat32);
        model_input.someDouble = 0.5f;
        /*  ... set other model_input fields */

        // Notify ITP thread to send the updated data by returning true
        return true;
    } else {
        return false;
    }
    // =================================================================================
}

```