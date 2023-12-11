---
layout: default
title: 4 - Groove to Drum (Real-Time) 
nav_order: 4
has_children: false
parent: Tutorials
grand_parent: V1.0.0 Documentation
permalink: /v1_0_0/docs/Tutorials/4_RealTimeGroove2Drum
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Tutorial 4 - Real-Time Groove to Drum Accompaniment 
{: .no_toc }

{: .warning }
> In this tutorial, we will be modifying the previous tutorial to allow for real-time groove to drum accompaniment.
> In this case, instead of relying on a user-provided MIDI file, we will be using the incoming MIDI events from the DAW.

{: .note }
> the source code for this tutorial is available in the tutorials branch of the repository.
> 
> [Tutorial 4 Source Code](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/tutorials/4_Groove2DrumRealTime){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


## Description

In this tutorial, we will be modifying the previous tutorial to allow for real-time groove to drum accompaniment.
That is, instead of relying on a user-provided MIDI file, we will be using the incoming MIDI events from the DAW.
As a result, we will mostly be modifying the `ITP` thread.

In this case, will need to do the following:

1. Modify the plugin name and description
2. Modify the GUI (Optional)
3. Specify what information we need from DAW (`Configs_HostEvents.h`)
4. Modify the `ITP` thread to prepare the input tensor (`InputTensorPreparatorThread::deploy()`)

## Plugin Name and Description
As mentioned [here](https://neuralmidifx.github.io/docs/Installation#step-2-edit-plugin-name-and-description), we need
to specify the name of the plugin as well as some descriptions for it. 

To do this, we will modify the [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/4_Groove2DrumRealTime/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt) file as follows:

```cmake
project(Tutorial4NMFx VERSION 0.0.1)

set (BaseTargetName Tutorial4NMFx)

....

juce_add_plugin("${BaseTargetName}"
        COMPANY_NAME "AIMCTutorials"                
        ... 
        PLUGIN_CODE AZCN                # a unique 4 character code for your plugin                          
        ...
        PRODUCT_NAME "Tutorial4NMFx")           # Replace with your plugin title

```

Once you re-build the cmake project, and re-build the plugin, you should see the name of the plugin change in the DAW:

<img src="{{ site.baseurl }}/assets/images/tutorial 4/0.PNG">

Now we are ready to move on to the next step.

## Modifying the GUI

We will be reusing the exact same GUI as the previous tutorial. That said, we will be slightly modify it in two ways:

- Disable drag/drop of MIDI files
- Enable Visualization of Incoming MIDI Events

To customize the GUI we will be modifying the [`Configs_GUI.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/4_Groove2DrumRealTime/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Configs_GUI.h)
file.


```c++

            // Configs_GUI.h
            
            // rest of the code ...   
            
            namespace MidiInVisualizer {

                const bool enable = true;
                const bool allowToDragInMidi = false;
                const bool visualizeIncomingMidiFromHost = true;

                // rest of the code ...   
            
            }
            
```

Having done this, we can see that midi files can no longer be dragged into the plugin:

<img src="{{ site.baseurl }}/assets/gifs/tut4/NoMidiIn.gif">

Also, we can see that any incoming MIDI events from the DAW are now visualized in the GUI:

<img src="{{ site.baseurl }}/assets/gifs/tut4/MidiInRT.gif">

## Host Events

{: .note }
> If you are not familiar with the concepts of per buffer processing in plugins,
> refer to [this guide]({{ site.baseurl }}/docs/PluginBasics/#processor) before continuing.
> 

For specifying what information we need from the DAW, we will be modifying the [`Configs_HostEvents.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/4_Groove2DrumRealTime/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Configs_HostEvents.h)

{: .note }
> 1. The events received from host are wrapped in a `EventFromHost` struct (see Documentation [here]({{ site.baseurl }}/datatypes/EventFromHost))  
> 
> 2. The received information from the DAW are provided sequentially, that is, every time a new event is received,
> the `ITP Deploy()` method is called with the new information. As an example, if the DAW sends 10 events, the `ITP Deploy()`
> method will be called 10 times, each time with a new event.
>

### Available Information from DAW


There are different types available in the `EventFromHost` data type. 

| Event Type               | Check Method                        | Description                                                                 |
|--------------------------|-------------------------------------|-----------------------------------------------------------------------------|
| FirstBufferEvent         | `new_event_from_host->isFirstBufferEvent()`  | Sent at the beginning of the host start.                                    |
| PlaybackStoppedEvent     | `new_event_from_host->isPlaybackStoppedEvent()` | Sent when the host stops the playback.                                      |
| NewBufferEvent           | `new_event_from_host->isNewBufferEvent()` | Sent at the beginning of every new buffer or when qpm, meter, etc. changes. |
| NewBarEvent              | `new_event_from_host->isNewBarEvent()` | Sent at the beginning of every new bar.                                     |
| NewTimeShiftEvent        | `new_event_from_host->isNewTimeShiftEvent()` | Sent every N QuarterNotes (as specified in the configs file                 |
| NoteOnEvent              | `new_event_from_host->isNoteOnEvent()` | Sent when a note is played.                                                 |
| NoteOffEvent             | `new_event_from_host->isNoteOffEvent()` | Sent when a note is stopped.                                                |
| CCEvent                  | `new_event_from_host->isCCEvent()` | Sent for Control Change events.                                             |

<video width="85%" preload="auto" muted controls>
    <source src="{{ site.baseurl }}/assets/videos/BufferHostEvents.mp4" type="video/mp4"/>
</video>


### Specifying What Information We Need from DAW

In this exercise, we will only be needing NoteOn events. To specify this we will modify
the `Configs_HostEvents.h` as follows:

```c++

// Configs_HostEvents.h
        
namespace event_communication_settings {
    // set to true, if you need to send the metadata for a new buffer to the ITP thread
    constexpr bool SendEventAtBeginningOfNewBuffers_FLAG{false};
    constexpr bool SendEventForNewBufferIfMetadataChanged_FLAG{false};     // only sends if metadata changes
    
    // set to true if you need to notify the beginning of a new bar
    constexpr bool SendNewBarEvents_FLAG{false};
    
    // set to true EventFromHost for every time_shift_event ratio of quarter notes
    constexpr bool SendTimeShiftEvents_FLAG{false};
    constexpr double delta_TimeShiftEventRatioOfQuarterNote{0.5}; // sends a time shift event every 8th note
    
    // Filter Note On Events if you don't need them
    constexpr bool FilterNoteOnEvents_FLAG{false};
    
    // Filter Note Off Events if you don't need them
    constexpr bool FilterNoteOffEvents_FLAG{false};
    
    // Filter CC Events if you don't need them
    constexpr bool FilterCCEvents_FLAG{false};
}

            
```

Now, we're ready to move on to the `ITP`'s `deploy()` method to specify how we want to handle the incoming events.

## InputTensorPreparatorThread::deploy()

The processing in here is very similar to the previous tutorial with some slight modifications.

The `HostEvent` data are provided to the `deploy()` method as an optional object called `new_event_from_host`. Let's check 
if we are receiving any events from the host:

```c++
// ITP_Deploy.cpp

bool InputTensorPreparatorThread::deploy(
    std::optional<MidiFileEvent> & new_midi_event_dragdrop,
    std::optional<EventFromHost> & new_event_from_host,
    bool gui_params_changed_since_last_call) {

    bool SHOULD_SEND_TO_MODEL_FOR_GENERATION_ = false;

    // check that the deploy() method was called because of a new midi event
    if (new_event_from_host.has_value()) {
        PrintMessage("new_event_from_host received");
        PrintMessage(new_event_from_host->getDescription().str());
    }

    return SHOULD_SEND_TO_MODEL_FOR_GENERATION_;
}
```


<img src="{{ site.baseurl }}/assets/gifs/tut4/HostEventsReceivingCorrectly.gif">


Now, we need to do  to decide whether we want to take any actions when `FirstBufferEvent` is received.
In this case, let's clear the groove and start from scratch. That is, everytime the playback restarts, we assume that
the session starts from scratch.

```c++
// ITP_Deploy.cpp

    // check that the deploy() method was called because of a new midi event
    if (new_event_from_host.has_value()) {
        if (new_event_from_host->isFirstBufferEvent()) {
            // clear hits, velocities, offsets
            ITPdata.hits = torch::zeros({1, 32, 1}, torch::kFloat32);
            ITPdata.velocities = torch::zeros({1, 32, 1}, torch::kFloat32);
            ITPdata.offsets = torch::zeros({1, 32, 1}, torch::kFloat32);
        }
    }
```

Now, we are ready to handle the `NoteOnEvent`s to update the groove. A NoteOn event works similarly to a `MidiFileEvent`. 
So, this part is very similar to the previous tutorial. 

```c++
// ITP_Deploy.cpp

        if (new_event_from_host->isNoteOnEvent()) {
            auto ppq  = new_event_from_host->Time().inQuarterNotes(); // time in ppq
            auto velocity = new_event_from_host->getVelocity(); // velocity
            auto div = round(ppq / .25f);
            auto offset = (ppq - (div * .25f)) / 0.125 * 0.5 ;
            auto grid_index = (long long) fmod(div, 32);

            // check if louder if overlapping
            if (ITPdata.hits[0][grid_index][0].item<float>() > 0) {
                if (ITPdata.velocities[0][grid_index][0].item<float>() < velocity) {
                    ITPdata.velocities[0][grid_index][0] = velocity;
                    ITPdata.offsets[0][grid_index][0] = offset;
                }
            } else {
                ITPdata.hits[0][grid_index][0] = 1;
                ITPdata.velocities[0][grid_index][0] = velocity;
                ITPdata.offsets[0][grid_index][0] = offset;
            }
        }
        
```

Last thing to do is to return `true` if we want to send the data to the model for generation. In this case, we need to 
decide how often we want to send the data to the `MDL` thread. Let's send the everytime the groove changes. That is,
everytime we receive a `HostEvent` we will send the data to the `MDL` thread.

```c++
// ITP_Deploy.cpp

    bool InputTensorPreparatorThread::deploy(
    std::optional<MidiFileEvent> & new_midi_event_dragdrop,
    std::optional<EventFromHost> & new_event_from_host,
    bool gui_params_changed_since_last_call) {

    bool SHOULD_SEND_TO_MODEL_FOR_GENERATION_ = false;

    // check that the deploy() method was called because of a new midi event
    if (new_event_from_host.has_value()) {
        SHOULD_SEND_TO_MODEL_FOR_GENERATION_ = true;        <--- send to model
        
        if (new_event_from_host->isFirstBufferEvent()) {
            // clear hits, velocities, offsets
            ITPdata.hits = torch::zeros({1, 32, 1}, torch::kFloat32);
            ITPdata.velocities = torch::zeros({1, 32, 1}, torch::kFloat32);
            ITPdata.offsets = torch::zeros({1, 32, 1}, torch::kFloat32);
        }
        
        if (new_event_from_host->isNoteOnEvent()) {
            auto ppq  = new_event_from_host->Time().inQuarterNotes(); // time in ppq
            auto velocity = new_event_from_host->getVelocity(); // velocity
            auto div = round(ppq / .25f);
            auto offset = (ppq - (div * .25f)) / 0.125 * 0.5 ;
            auto grid_index = (long long) fmod(div, 32);

            // check if louder if overlapping
            if (ITPdata.hits[0][grid_index][0].item<float>() > 0) {
                if (ITPdata.velocities[0][grid_index][0].item<float>() < velocity) {
                    ITPdata.velocities[0][grid_index][0] = velocity;
                    ITPdata.offsets[0][grid_index][0] = offset;
                }
            } else {
                ITPdata.hits[0][grid_index][0] = 1;
                ITPdata.velocities[0][grid_index][0] = velocity;
                ITPdata.offsets[0][grid_index][0] = offset;
            }
        }
    }

    return SHOULD_SEND_TO_MODEL_FOR_GENERATION_;
}

```

Assuming that the `MDL` and `PPP` threads are the same as the previous tutorial, we can see that the plugin is
now working as expected:

<img src="{{ site.baseurl }}/assets/gifs/tut4/FinalA.gif">


----

## Alternative: Generating At Specific Times Only

In the previous part, what we did was that we generated a new pattern as soon as we received a new `HostEvent`.

However, we might want to generate a new pattern only at specific times. For example, we might want to generate a new
pattern Every Two Quarter Notes. 

To do this, we need to first modify the `Configs_HostEvents.h` file to specify that we want a `HostEvent` to be sent every quarter note

```c++
// Configs_HostEvents.h
        
namespace event_communication_settings {
    // same as before ...
    
    // set to true EventFromHost for every time_shift_event ratio of quarter notes
    constexpr bool SendTimeShiftEvents_FLAG{true};                                  <-- set to true
    constexpr double delta_TimeShiftEventRatioOfQuarterNote{0.5};                    <-- set to 2      
    
    // same as before ...
}
```

Then, we add modify the `InputTensorPreparatorThread` as follows:

```c++
// ITP_Deploy.cpp

    // rest of the code same as before ...

    // check that the deploy() method was called because of a new midi event
    if (new_event_from_host.has_value()) {
        if (new_event_from_host->isNewTimeShiftEvent()) {
            SHOULD_SEND_TO_MODEL_FOR_GENERATION_ = true;
        }
        
        // rest of the code same as before ...
        
```

Now, the plugin will generate a new pattern every two quarter notes:

<img src="{{ site.baseurl }}/assets/gifs/tut4/FinalB.gif">