---
layout: default
title: 5 - Single Thread Implementation
nav_order: 5
has_children: false
parent: Tutorials
grand_parent: V1.0.0 Documentation
permalink: /docs/v1_0_0/Tutorials/5_SingleThread
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Tutorial 5 - Single Thread Implementation
{: .no_toc }

{: .warning }
> In this tutorial, we will be using the same model as the previous tutorials, so make sure you have completed the previous tutorials.

{: .note }
> the source code for this tutorial is available in the tutorials branch of the repository.
> 
> [Tutorial 5 Source Code](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/tutorials/5_SingleThread){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


## Description

If you find the three thread implementation too complicated for a given model you're using, 
you can modify the wrapper to use a single thread implementation instead

In this tutorial, we will show you how to do this by modifying the wrapper as follows:

1. Modifying `ITP`:  Send incoming `MidiFileEvent` and `EventFromHost` to the model using the `ModelInput` struct
2. Modifying  `MDL`: Get the `ModelInput` struct from the `ITP` and send it to the model as is
3. Modifying `PPP`: Get the `ModelOutput` containing all necessary events and Implement the processing here

{: .important }
> 1. Once finished with this tutorial, you can always use this as a template for your own models.
> 
> 2. We'll use this single thread implementation method for the next tutorial 
    

## Plugin Name and Description
As mentioned [here](https://neuralmidifx.github.io/docs/v1_0_0/Installation#step-2-edit-plugin-name-and-description), we need
to specify the name of the plugin as well as some descriptions for it. 

To do this, we will modify the [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/5_SingleThread/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt) file as follows:

```cmake
project(Tutorial5NMFx VERSION 0.0.1)

set (BaseTargetName Tutorial5NMFx)

....

juce_add_plugin("${BaseTargetName}"
        COMPANY_NAME "AIMCTutorials"                
        ... 
        PLUGIN_CODE AZCO                # a unique 4 character code for your plugin                          
        ...
        PRODUCT_NAME "Tutorial5NMFx")           # Replace with your plugin title

```

Once you re-build the cmake project, and re-build the plugin, you should see the name of the plugin change in the DAW:

<img src="{{ site.baseurl }}/assets/images/tutorial5/0.PNG">

Now we are ready to move on to the next step.

## Modifying the GUI

For this tutorial, we don't need the GUI we will make sure there are no components in [`Configs_GUI.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/5_SingleThread/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Configs_GUI.h)
file.


```c++

     // Configs_GUI.h
            
     // rest of the code ...   
            
            namespace Tabs {
        const bool show_grid = false;
        const bool draw_borders_for_components = false;
        const std::vector<tab_tuple> tabList{
            tab_tuple
            {
                "Tab",
                slider_list
                {
                },
                rotary_list
                {
                },
                button_list
                {
                }
            },

        };
    }


    namespace MidiInVisualizer {

        const bool enable = true;
        const bool allowToDragInMidi = true;
        const bool visualizeIncomingMidiFromHost = true;
        const bool deletePreviousIncomingMidiMessagesOnBackwardPlayhead = false;
        const bool deletePreviousIncomingMidiMessagesOnRestart = false;
    }

    namespace GeneratedContentVisualizer
    {
        const bool enable = true;
        const bool allowToDragOutAsMidi = true;
    }

```


<img src="{{ site.baseurl }}/assets/images/tutorial5/1.PNG">


## Passing `HostEvent` and `MidiFileEvent` to `PPP` thread

We will start with modifying the [`CustomStructs.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/5_SingleThread/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CustomStructs.h) 
file as follows:

First, make sure `ITPData` and `MDLData` are empty structs. If not, this won't cause any issues, 
but we'll keep them empty as in this case, we will not be doing any processing in these threads. Rather, we will 
just be using these threads to pass the data directly from `ITP` to `PPP` via `MDL` thread.

```c++
// CustomStructs.h


// Any Extra Variables You need in ITP can be defined here
// An instance called 'ITPdata' will be provided to you in Deploy() method
struct ITPData {

};

// Any Extra Variables You need in MDL can be defined here
// An instance called 'MDLdata' will be provided to you in Deploy() method
struct MDLData {

};
```

Then, we will be modifying the `ModelInput` and `ModelOutput` structs as follows:

```c++
// CustomStructs.h

struct ModelInput {
    
    std::optional<MidiFileEvent> new_midi_event_dragdrop;
    std::optional<EventFromHost> new_event_from_host;

    // ==============================================
    // Don't Change Anything in the following section
    // ==============================================
    chrono_timer timer{};
};

struct ModelOutput {

    std::optional<MidiFileEvent> new_midi_event_dragdrop;
    std::optional<EventFromHost> new_event_from_host;

    // ==============================================
    // Don't Change Anything in the following section
    // ==============================================
    chrono_timer timer{};
};

```

Note that we have added two optional variables to the `ModelInput` and `ModelOutput` structs. These are exactly the same 
as the inputs passed on to the `deploy()` method of `ITP` (see [`ITP_Deploy.cpp`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/5_SingleThread/NeuralMidiFXPlugin/NeuralMidiFXPlugin/ITP_Deploy.cpp)).

Now, we are ready to move on to the next step.

## Modifying `ITP` Thread

In here, we will modify the [`ITP_Deploy.cpp`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/5_SingleThread/NeuralMidiFXPlugin/NeuralMidiFXPlugin/ITP_Deploy.cpp) file.

The objective here is to first make sure if the `deploy()` method was called because of an Event (rather than a GUI change). 
If so, then we will place the received events in the `ModelInput` struct and send it to the `MDL` thread.

```c++
// ITP_Deploy.cpp

bool InputTensorPreparatorThread::deploy(
    std::optional<MidiFileEvent> & new_midi_event_dragdrop,
    std::optional<EventFromHost> & new_event_from_host,
    bool gui_params_changed_since_last_call) {

    // we only pass the data if the method was called because of a new event
    // and not because of a gui parameter change
    if (new_midi_event_dragdrop.has_value() || new_event_from_host.has_value()) {
        model_input.new_event_from_host = new_event_from_host;
        model_input.new_midi_event_dragdrop = new_midi_event_dragdrop;
        return true;
    }

    return false;
}

```

## Modifying `MDL` Thread

In here, we will modify the [`MDL_Deploy.cpp`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/5_SingleThread/NeuralMidiFXPlugin/NeuralMidiFXPlugin/MDL_Deploy.cpp) file.

Similar to the `ITP` thread, we will first make sure if the `deploy()` method was called because of a new received `ModelInput` and 
not because of a GUI change. If so, we will pass the received `ModelInput` to the `PPP` thread.

```c++
// MDL_Deploy.cpp

bool ModelThread::deploy(bool new_model_input_received,
                         bool did_any_gui_params_change) {

    // we only pass the data if the method was called because of a new model_input

    if (new_model_input_received) {
        model_output.new_event_from_host = model_input.new_event_from_host;
        model_output.new_midi_event_dragdrop = model_input.new_midi_event_dragdrop;
        return true;
    }

    return false;
}
```
 
After these modifications are done, we can test whether the required data is correctly passed from `ITP` to `PPP` thread.
To do this, we'll modify [`PPP_Deploy.cpp`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/5_SingleThread/NeuralMidiFXPlugin/NeuralMidiFXPlugin/PPP_Deploy.cpp)
file as follows:

```c++

// PPP_Deploy.cpp

std::pair<bool, bool> PlaybackPreparatorThread::deploy(bool new_model_output_received, bool did_any_gui_params_change) {

    bool newPlaybackPolicyShouldBeSent = false;
    bool newPlaybackSequenceGeneratedAndShouldBeSent = false;

    if (new_model_output_received)
    {
        if (model_output.new_event_from_host) {
            PrintMessage("Received new event from host");
        }

        if (model_output.new_midi_event_dragdrop) {
            PrintMessage("Received new midi event from dragdrop");
        }

    }

    return {newPlaybackPolicyShouldBeSent, newPlaybackSequenceGeneratedAndShouldBeSent};
}

```


<img src="{{ site.baseurl }}/assets/gifs/tut5/AllGood.gif">


## Loading the Model in `PPP` Thread

Loading the model in `PPP` thread is slightly different from the previous tutorials. The reason is that, there is no
dedicated loader nor a dedicated place holder for the model in `PPP` thread. That said, we have the ability to load
the model using the `PPPData` struct in [`CustomStructs.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/5_SingleThread/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CustomStructs.h).
as follows:

```c++
// CustomStructs.h

// assuming that the model is called 'drumLoopVAE.pt' and is available somewhere int the TorchScript folder
struct PPPData {
    torch::jit::script::Module model = load_processing_script("drumLoopVAE.pt");
};
```

To use this model in the `PPP_Deploy.cpp` file, we can simply use the local instance of `PPPData` (called `PPPdata`) as follows:

```c++

// PPP_Deploy.cpp

    // some code ...
    
    PPPdata.model.forward(....);

```

<img src="{{ site.baseurl }}/assets/gifs/tut5/ModelFound.gif">