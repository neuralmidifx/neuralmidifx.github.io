---
layout: default
title: 6 - Call and Response
nav_order: 6
has_children: false
parent: Tutorials
permalink: /Tutorials/6_CallResponse
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Tutorial 6 - Call and Response
{: .no_toc }

{: .warning }
> In this tutorial, we will be using the same model as the previous tutorials, so make sure you have completed the previous tutorials.
> Also, in this excercise, we will focus on a single thread implementation (as explained in [Tutorial 5](https://neuralmidifx.github.io/docs/Tutorials/5_SingleThreadImplementation)),
> 

{: .note }
> the source code for this tutorial is available in the tutorials branch of the repository.
> 
> [Tutorial 6 Source Code](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/tutorials/6_CallResponse){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


## Description

The objective here is to build a call and response plugin with the following assumptions:

1. The user starts playing a sequence of notes
2. Once the user stops playing for 2 seconds, we move on to generate a response
3. To generate the response:
   1. we will chop up the performance into non-overlapping 2 bar chunks
   2. Then, we will generate a drum pattern for each 2bar chunk
   3. Finally, we will concatenate the generated drum patterns to create the response

4. If the user starts playing again, we will stop playback of generations immediately and wait for the user to stop playing again

## Plugin Name and Description
As mentioned [here](https://neuralmidifx.github.io/docs/Installation#step-2-edit-plugin-name-and-description), we need
to specify the name of the plugin as well as some descriptions for it. 

To do this, we will modify the [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/6_CallResponse/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt) file as follows:

```cmake
project(Tutorial6NMFx VERSION 0.0.1)

set (BaseTargetName Tutorial6NMFx)

....

juce_add_plugin("${BaseTargetName}"
        COMPANY_NAME "AIMCTutorials"                
        ... 
        PLUGIN_CODE AZCP                # a unique 4 character code for your plugin                          
        ...
        PRODUCT_NAME "Tutorial6NMFx")           # Replace with your plugin title

```


Once you re-build the cmake project, and re-build the plugin, you should see the name of the plugin change in the DAW:

<img src="{{ site.baseurl }}/assets/images/tutorial6/0.PNG">

Now we are ready to move on to the next step.

## Specifying Required Information from the host

For specifying what information we need from the DAW, we will be modifying the [`Configs_HostEvents.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/6_CallResponse/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Configs_HostEvents.h)

We need to decide which of the following events are required from the host

| Event Type           | Check Method                        | Description                                                                 |
|----------------------|-------------------------------------|-----------------------------------------------------------------------------|
| FirstBufferEvent*    | `new_event_from_host->isFirstBufferEvent()`  | Sent at the beginning of the host start.                                    |
| PlaybackStoppedEvent* | `new_event_from_host->isPlaybackStoppedEvent()` | Sent when the host stops the playback.                                      |
| NewBufferEvent       | `new_event_from_host->isNewBufferEvent()` | Sent at the beginning of every new buffer or when qpm, meter, etc. changes. |
| NewBarEvent          | `new_event_from_host->isNewBarEvent()` | Sent at the beginning of every new bar.                                     |
| NewTimeShiftEvent    | `new_event_from_host->isNewTimeShiftEvent()` | Sent every N QuarterNotes (as specified in the configs file                 |
| NoteOnEvent          | `new_event_from_host->isNoteOnEvent()` | Sent when a note is played.                                                 |
| NoteOffEvent         | `new_event_from_host->isNoteOffEvent()` | Sent when a note is stopped.                                                |
| CCEvent              | `new_event_from_host->isCCEvent()` | Sent for Control Change events.                                             |

* -> Always sent by the host

The most obvious one is the `NoteOnEvent`, which we will need to use for creating the inputs to the model. As such,
we should make sure that these event are not filtered out by the wrapper

```cpp

// Configs_HostEvents.h

namespace event_communication_settings {
    // ....
    
    // Filter Note On Events if you don't need them
    constexpr bool FilterNoteOnEvents_FLAG{false};        <--- Must be false
}
```

One more thing to consider here is that we need to keep track of the time since last played note. This is because we
need to know when the user has stopped playing for 2 seconds. To do this, we will request the `NewBufferEvent` from the host.
This event is sent as soon as a new buffer is provided to the plugin. This way we will always be able to keep track of 
time

```cpp

// Configs_HostEvents.h

namespace event_communication_settings {
    // ....
   
    // set to true, if you need to send the metadata for a new buffer to the ITP thread
    constexpr bool SendEventAtBeginningOfNewBuffers_FLAG{true};       <--- Must be true
    constexpr bool SendEventForNewBufferIfMetadataChanged_FLAG{false}; <--- Must be false (otherwise sent on tempo, meter, etc. changes)
}
```

{: .warning }
> whenever the above flag is set, you need to also double check the `SendEventForNewBufferIfMetadataChanged_FLAG` flag.
> This flag is used to specify whether the `NewBufferEvent` should always be sent or should be sent 
> only when the metadata (tempo, meter, ... ) changes.

Now we are ready to move on to the next step.

To check that the above settings are correct, let's print some information to the console. 

Given that we are working with a single thread implementation, we will be using the `PPP` thread only! So to print the 
information, we will be updating the [`PPP_Deploy().cpp`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/6_CallResponse/NeuralMidiFXPlugin/NeuralMidiFXPlugin/PPP_Deploy.cpp) file. 

```cpp

// PPP_Deploy().cpp

std::pair<bool, bool> PlaybackPreparatorThread::deploy(bool new_model_output_received, bool did_any_gui_params_change) {

    bool newPlaybackPolicyShouldBeSent = false;
    bool newPlaybackSequenceGeneratedAndShouldBeSent = false;

    if (new_model_output_received)
    {
        // check if any host data is received from model thread via model_output
        auto new_event_from_host = model_output.new_event_from_host;
        if (new_event_from_host.has_value()) {

            if (new_event_from_host->isFirstBufferEvent() ||
                new_event_from_host->isPlaybackStoppedEvent() ||
                new_event_from_host->isNewBufferEvent()) {
                PrintMessage(
                    "Time (sec) = " +
                    std::to_string(new_event_from_host->Time().inSeconds()) +
                    ", (Quarter Note) = " +
                    std::to_string(new_event_from_host->Time().inQuarterNotes()) +
                    ", (Samples) = " +
                    std::to_string(new_event_from_host->Time().inSamples())
                    );
            }
        }
        
    }

    return {newPlaybackPolicyShouldBeSent, newPlaybackSequenceGeneratedAndShouldBeSent};
}

```

<img src="{{ site.baseurl }}/assets/gifs/tut6/TimePrint.gif">