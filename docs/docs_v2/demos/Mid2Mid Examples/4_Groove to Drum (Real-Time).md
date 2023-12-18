---
layout: default
title: Groove to Drum (Real-Time)
nav_order: 4
has_children: false
parent: MidToMid
grand_parent: Demos
permalink: /docs/V2_0_1/Demos/MidToMid/Groove_to_Drum_(Real-Time)
---


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Groove to Drum (Real-Time)
{: .no_toc }

## Description
The model provided in the previous tutorials converts a single voice groove into a drum pattern. In that case,
we used a midi file to extract the groove. However, in this tutorial, we will create a plugin which allows the user
to play the groove in real-time using a midi keyboard.

The code here is very similar to the code in the previous tutorial, so we will only highlight the differences.

## Plugin Name and Description
As mentioned [here](https://neuralmidifx.github.io/docs/V2_0_1/Installation#step-2-edit-plugin-name-and-description), 
we need to specify the name of the plugin as well as some descriptions for it.

To do this, we will modify the [PluginCode/CMakeLists.txt](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmRT/blob/master/PluginCode/CMakeLists.txt) file as follows:

```cmake

project(Demo4 VERSION 0.0.1)

set (BaseTargetName Demo4)

add_definitions(-DPROJECT_NAME="${BaseTargetName}")

juce_add_plugin("${BaseTargetName}"
        # VERSION ...                               # Set this if the plugin version is different to the project version
        # ICON_BIG ...                              # ICON_* arguments specify a path to an image file to use as an icon for the Standalone
        # ICON_SMALL ...
        COMPANY_NAME "DemosMid2Mid"                 # Replace with a tag identifying your name
        IS_SYNTH TRUE                               # There is no MIDI vst3 plugin format, so we are going to assume a midi instrument plugin
        NEEDS_MIDI_INPUT TRUE
        NEEDS_MIDI_OUTPUT TRUE
        AU_MAIN_TYPE kAudioUnitType_MIDIProcessor
        EDITOR_WANTS_KEYBOARD_FOCUS FALSE
        COPY_PLUGIN_AFTER_BUILD TRUE                 # copies the plugin to user plugins folder so as to easily load in DAW
        PLUGIN_MANUFACTURER_CODE Juce                #
        PLUGIN_CODE aaac                             # MUST BE UNIQUE!! If similar to other plugins, conflicts will occur
        FORMATS AU VST3 Standalone
        PRODUCT_NAME "Demo4")           # Replace with your plugin title
```

## GUI

The GUI is very similar to the previous tutorial. The only difference is that we don't need the MidiVisualizer widget
anymore. So, we will modify the [PluginCode/settings.json](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmRT/blob/master/PluginCode/settings.json)
as follows:

```json
                {
                    "name": "RandomGeneration",
                    "sliders": [
                        {
                            "label": "Density",
                            "min": 0.0,
                            "max": 1.0,
                            "default": 0.5,
                            "topLeftCorner": "Fi",
                            "bottomRightCorner": "Tm",
                            "horizontal": true
                        }
                    ],
                    "rotaries": [],
                    "buttons": [],
                    "MidiDisplays": []
                }
```

[Optional] We can also add a real-time midi visualizer to the GUI. To do this, we will enable the `MidiInVisualizer` widget
in the same settings file.

```json
{
          "MidiInVisualizer": {
            "enable": true,
            "allowToDragInMidi": false,
            "visualizeIncomingMidiFromHost": true,
            "deletePreviousIncomingMidiMessagesOnBackwardPlayhead": true,
            "deletePreviousIncomingMidiMessagesOnRestart": true
          }
}
```

## Specifying Host Events

As mentioned in the [documentation](https://neuralmidifx.github.io/docs/V2_0_1/DeploymentThread_DPL/SpecifyHostEvents),
we will modify the [PluginCode/settings.json](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmRT/blob/master/PluginCode/settings.json)
to specify what events are required from the host.

In this example, all we need is incoming midi notes. That is, we don't need any frame buffer information as we only
will be triggering the deployment process on each incoming midi note

```json
{
    "event_communication_settings": {
        "SendEventAtBeginningOfNewBuffers_FLAG": false,
        "SendEventForNewBufferIfMetadataChanged_FLAG": false,
        "SendNewBarEvents_FLAG": false,
        "SendTimeShiftEvents_FLAG": false,
        "delta_TimeShiftEventRatioOfQuarterNote": 0.5,
        "FilterNoteOnEvents_FLAG": false,
        "FilterNoteOffEvents_FLAG": true,
        "FilterCCEvents_FLAG": true
    }
}
```

{: .note }
> In the above, we are filtering out all CC events and NoteOff events. This is because we only need NoteOn events to trigger
> the deployment process. However, this is completely optional, as we do a further check for NoteOn events in the code.


## deploy()

The processing here is very similar to the previous tutorial with the exception that the notes are received one by one
in real-time, rather than all at once from a midi file.

As such the only difference to the [deploy()](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmRT/blob/master/PluginCode/Deploy.cpp) 
function is as follows:

```cpp
    
    // previous code here

    if ((new_event_from_host != std::nullopt) || gui_params_changed_since_last_call) {

        // =================================================================================
        // ===         1. Check (and Process)
        //              incoming events from host
        // = Refer to:
        // https://neuralmidifx.github.io/docs/V2_0_1/datatypes/EventFromHost
        // =================================================================================
        if (new_event_from_host->isFirstBufferEvent()) {
            // clear hits, velocities, offsets
            DPLdata.groove_hits = torch::zeros({1, 32, 1}, torch::kFloat32);
            DPLdata.groove_velocities = torch::zeros({1, 32, 1}, torch::kFloat32);
            DPLdata.groove_offsets = torch::zeros({1, 32, 1}, torch::kFloat32);
        }

        if (new_event_from_host->isNoteOnEvent()) {
            auto ppq  = new_event_from_host->Time().inQuarterNotes(); // time in ppq
            auto velocity = new_event_from_host->getVelocity(); // velocity
            auto div = round(ppq / .25f);
            auto offset = (ppq - (div * .25f)) / 0.125 * 0.5 ;
            auto grid_index = (long long) fmod(div, 32);

            // check if louder if overlapping
            if (DPLdata.groove_hits[0][grid_index][0].item<float>() > 0) {
                if (DPLdata.groove_velocities[0][grid_index][0].item<float>() < velocity) {
                    DPLdata.groove_velocities[0][grid_index][0] = velocity;
                    DPLdata.groove_offsets[0][grid_index][0] = offset;
                }
            } else {
                DPLdata.groove_hits[0][grid_index][0] = 1;
                DPLdata.groove_velocities[0][grid_index][0] = velocity;
                DPLdata.groove_offsets[0][grid_index][0] = offset;
            }
        }
    
    // previous code here
```

<img src="{{ site.baseurl }}/assets/gifs/demo4/final.gif">