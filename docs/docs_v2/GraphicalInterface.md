---
layout: default
title: 2. Parameters and GUI
nav_order: 50

permalink: /docs/v2_0_0/ParametersAndGUI
---

# Plugin Parameters and Graphical Interface Rendering
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

{: .note }
> For this Stage of Deployment, You should modify the following file
> 
> [Deployment/settings.json](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/releases/v2.0.0/Deployment/settings.json){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


{: .note }
> Use the following pdf to draw out and figure out the coordinates of the UI elements
> 
> <object data="https://neuralmidifx.github.io/assets/LayoutPlanner.pdf" width="400" height="300" type='application/pdf'></object>
> 
> [LayoutPlanner.pdf](https://neuralmidifx.github.io/assets/LayoutPlanner.pdf){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


---


## Step 1: Available User Interface Elements
The wrapper allows for automatic rendering of the plugin's graphical interface. All you need to do, is to
decide what sort of UI elements you would like to have in your plugin.

{: .note }
> Available UI elements are:
> 1. Sliders (horizontal and vertical)
> 2. Rotary Knobs
> 3. Toggle/Trigger Buttons
> 4. Combo Boxes
> 5. Midi Visualizer
> 6. Audio Visualizer

Once you have decided what sort of UI elements you would like to have in your plugin, continue with the following steps.

![img.png](/assets/images/GUI.png)

{: .note }
> The UI elements can be organized in tabs. Each tab can have any number of sliders, rotary knobs and buttons.

{: .important }
> All UI elements must have a unique label. This label is used to access the value of UI elements in the code.
> In the case of Midi/Audio Visualizers, this label is also used to update the content of the visualizer.

## Step 2. Define the UI Elements

In `Settings.json`, you can define the UI elements that you would like to have in your plugin.

```json
    {
    "UI": {
        "Tabs": {
            "show_grid": true,
            "draw_borders_for_components": true,
            "tabList": [
                {
                    "name": "Tab 1",
                    "sliders": [
                        { ... }, { ... }
                    ],
                    "rotaries": [
                         { ... }, { ... }
                    ],
                    "buttons": [
                         { ... }, { ... }
                    ],
                    "MidiDisplays": [
                         { ... }, { ... }
                    ]
                },
              
                {
                    "name": "Tab 1",
                    "sliders": [
                        { ... }, { ... }
                    ],
                    "rotaries": [
                         { ... }, { ... }
                    ],
                    "buttons": [
                         { ... }, { ... }
                    ],
                    "MidiDisplays": [
                         { ... }, { ... }
                    ]
                }
            ]
        },

        "MidiInVisualizer": { .... },
        "GeneratedContentVisualizer": { ... }

    },

    "StandaloneTransportPanel": { ... },

    "VirtualMidiOut": { ... }

}
```

### Showing a Grid (For Design Stage)
To show a grid, set `show_grid` to `true`. To hide the grid, set `show_grid` to `false`.
The grid is useful for aligning the UI elements in the plugin. 
Each point in the grid is a capital letter (column) and a lowercase letter (row).

### Specify if Borders Should be Drawn for the UI Elements
To draw borders for the UI elements, set `draw_borders_for_components` to `true`. 
To hide the borders, set `draw_borders_for_components` to `false`.

### Adding Tabs
To add a tab, add a new entry to the `tabList` array. Each tab has a `name` and a list of UI elements.
The UI elements can be sliders, rotary knobs, buttons, comboboxes or MIDI displays.

```json
            "tabList": 
            [
                {
                    "name": "Tab 1",
                    "sliders": [ ],
                    "rotaries": [ ],
                    "buttons": [ ],
                    "MidiDisplays": [ ]
                },
              
                {
                    "name": "Tab 2",
                    "sliders": [ ],
                    "rotaries": [ ],
                    "buttons": [ ],
                    "MidiDisplays": [ ]
                }
            ]
```


### Adding Sliders
To add a slider, add a new entry to the `sliders` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/v2_0_0/datatypes/GuiParams)

```json
        {
            "label": "Slider 1",       
            "min": 0.0,                  
            "max": 1.0,                 
            "default": 0.5,             
            "topLeftCorner": "Ll",      
            "bottomRightCorner": "Pp"   
        },
        {
            "label": "Slider 2",           
            "min": 0.0,
            "max": 4.0,
            "default": 1.5,
            "topLeftCorner": "Pp",
            "bottomRightCorner": "Tt",
            "horizontal": true        

        }
```
The above can be read as follows:

```text
        {
            "label": "Slider 1",       --> The label of the slider 
            "min": 0.0,                 --> The minimum value of the slider
            "max": 1.0,                 --> The maximum value of the slider
            "default": 0.5,             --> The default value of the slider
            "topLeftCorner": "Ll",      --> The top left corner of the slider
            "bottomRightCorner": "Pp"   --> The bottom right corner of the slider
        },
        {
            "label": "Slider 2",           
            "min": 0.0,
            "max": 4.0,
            "default": 1.5,
            "topLeftCorner": "Pp",
            "bottomRightCorner": "Tt",
            "horizontal": true          --> If true, the slider will be horizontal. 
                                            If false or unspecified, the slider will be vertical.

        }
```

### Adding Rotary Knobs
To add a rotary knob, add a new entry to the `rotaries` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/v2_0_0/datatypes/GuiParams)

```json
        {
            "label": "Rotary 1",        
            "min": 0.0,                 
            "max": 1.0,                 
            "default": 0.5,             
            "topLeftCorner": "Ll",      
            "bottomRightCorner": "Pp"   
        }
```

The above can be read as follows:

```text
        {
            "label": "Rotary 1",        --> The label of the rotary knob
            "min": 0.0,                 --> The minimum value of the rotary knob
            "max": 1.0,                 --> The maximum value of the rotary knob
            "default": 0.5,             --> The default value of the rotary knob
            "topLeftCorner": "Ll",      --> The top left corner of the rotary knob
            "bottomRightCorner": "Pp"   --> The bottom right corner of the rotary knob
        }
```

### Adding Click/Trigger Buttons
To add a click button, add a new entry to the `buttons` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/v2_0_0/datatypes/GuiParams)

```json
        {
            "label": "TriggerButton 1",
            "isToggle": false,              
            "topLeftCorner": "Wv",
            "bottomRightCorner": "Zz"
        }
```

### Adding Toggle Switch Buttons
To add a toggle button, add a new entry to the `buttons` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/v2_0_0/datatypes/GuiParams)

```json
        {
            "label": "ToggleButton 2",
            "isToggle": true,              
            "topLeftCorner": "Wv",
            "bottomRightCorner": "Zz"
        }
```

### Adding Combo Boxes
To add a combo box, add a new entry to the `comboBoxes` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/v2_0_0/datatypes/GuiParams)

```json
        {
            "label": "ComboBox 1",
            "topLeftCorner": "Wv",
            "bottomRightCorner": "Zz",
            "items": [ "Item 1", "Item 2", "Item 3" ]
        }
```

The above can be read as follows:

```text
        {
            "label": "ComboBox 1",      --> The label of the combo box
            "topLeftCorner": "Wv",      --> The top left corner of the combo box
            "bottomRightCorner": "Zz",  --> The bottom right corner of the combo box
            "items": [ "Item 1", "Item 2", "Item 3" ] --> The items shown in the combo box
        }
```

### Midi Visualizer
To add a MIDI visualizer, add a new entry to the `MidiDisplays` array. 

To access/set the values of these elements, refer to [MidiVisualizersData]({{site.baseurl}}/docs/v2_0_0/datatypes/MidiVisualizersData)

```json
        {
            "label": "MidiDisplay 1",
            "topLeftCorner": "Bb",
            "bottomRightCorner": "Jh",
            "allowToDragOutAsMidi": true,   
            "allowToDragInMidi": true,     
            "needsPlayhead": true          
        }
```
 

In case, you want to show that the content is looped, specify 'PlayheadLoopDurationQuarterNotes', otherwise, 
the playhead will not be looped.


```json
        {
            "label": "MidiDisplay 2",
            "topLeftCorner": "Jh",
            "bottomRightCorner": "Ro",
            "allowToDragOutAsMidi": true,
            "allowToDragInMidi": true,
            "needsPlayhead": true,
            "PlayheadLoopDurationQuarterNotes": 4.0 
        }

```

The above can be read as follows:

```text
        {
            "label": "MidiDisplay 1",       --> The label of the MIDI visualizer
            "topLeftCorner": "Bb",          --> The top left corner of the MIDI visualizer
            "bottomRightCorner": "Jh",      --> The bottom right corner of the MIDI visualizer
            "allowToDragOutAsMidi": true,   --> If true, the user can drag out the MIDI data as a MIDI file
            "allowToDragInMidi": true,      --> If true, the user can drag in a MIDI file
            "needsPlayhead": true           --> If true, the playhead will be shown
            "PlayheadLoopDurationQuarterNotes": 4.0 --> The duration of the loop in quarter notes (if specified)
        }
```

### Audio Visualizer
To add an audio visualizer, add a new entry to the `AudioDisplays` array. 

If you want to show that the content is looped, specify 'PlayheadLoopDurationSamples', otherwise,
the playhead will not be looped.

To access/set the values of these elements, refer to [AudioVisualizersData]({{site.baseurl}}/docs/v2_0_0/datatypes/AudioVisualizersData)


```json
        {
            "label": "AudioDisplay 2",
            "topLeftCorner": "Jh",
            "bottomRightCorner": "Ro",
            "allowToDragOutAsAudio": true,
            "allowToDragInAudio": true,
            "needsPlayhead": true,               
            "PlayheadLoopDurationSamples": 44100  
        }
```

The above can be read as follows:

```text
        {
            "label": "AudioDisplay 1",      --> The label of the audio visualizer
            "topLeftCorner": "Bb",          --> The top left corner of the audio visualizer
            "bottomRightCorner": "Jh",      --> The bottom right corner of the audio visualizer
            "allowToDragOutAsAudio": true,  --> If true, the user can drag out the audio data as a WAV file
            "allowToDragInAudio": true,     --> If true, the user can drag in a WAV file
            "needsPlayhead": true           --> If true, the playhead will be shown
            "PlayheadLoopDurationSamples": 44100 --> The duration of the loop in samples (if specified)
        }
```

## MIDI In/Out Widgets 
These widgets behave differently than the above widgets. These should be only used to visualize incoming or 
outgoing MIDI messages from the plugin

A MIDI In widget can be added to the bottom of the plugin to allow for the user to drop a MIDI file into the plugin.
Moreover, the widget can be used to visualize incoming MIDI messages. To enable this widget, modify the following line in 

```json
        "MidiInVisualizer": {
            "enable": true,
            "allowToDragInMidi": true,
            "visualizeIncomingMidiFromHost": true,
            "deletePreviousIncomingMidiMessagesOnBackwardPlayhead": true,
            "deletePreviousIncomingMidiMessagesOnRestart": true
        },
        "GeneratedContentVisualizer": {
            "enable": true,
            "allowToDragOutAsMidi": true
        }
```

To access the content of a dragged in MIDI file, refer to [MidiFileEvent]({{site.baseurl}}/docs/v2_0_0/datatypes/MidiFileEvent)