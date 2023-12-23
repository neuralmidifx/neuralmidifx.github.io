---
layout: default
title: 2. Parameters and GUI
nav_order: 50

permalink: /docs/V2_1_0/ParametersAndGUI
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


## Available User Interface Elements
The wrapper allows for automatic rendering of the plugin's graphical interface. All you need to do, is to
decide what sort of UI elements you would like to have in your plugin.

{: .note }
> Available UI elements are:
> 1. Labels
> 2. Lines
> 4. Sliders (horizontal and vertical)
> 5. Rotary Knobs
> 6. Toggle/Trigger Buttons
> 7. Combo Boxes
> 8. Midi Visualizer
> 9. Audio Visualizer
> 10. Triangle Slider Area


Once you have decided what sort of UI elements you would like to have in your plugin, continue with the following steps.


{: .note }
> The UI elements can be organized in tabs. Each tab can have any number of sliders, rotary knobs and buttons.

{: .important }
> All UI elements must have a unique label. This label is used to access the value of UI elements in the code.
> In the case of Midi/Audio Visualizers, this label is also used to update the content of the visualizer.

## Define the UI Elements

In `Settings.json`, you can define the UI elements that you would like to have in your plugin.

```json
{
    "UI": {
        "resizable": true,
        "maintain_aspect_ratio": true,
        "width": 1000,
        "height": 800,

        "Tabs": {
            "show_grid": false,
            "draw_borders_for_components": false,
            "tabList": []
        },

        "MidiInVisualizer": {},

        "GeneratedContentVisualizer": {}

    },

    "StandaloneTransportPanel": {}

}


```

![img.png](/assets/images/GUILayoutAnnotated.png)

### Dimension and Resizability

Specify the initial dimensions of the plugin window by setting `width` and `height` to the desired values.

```json
{
    "UI": {
        "resizable": true,
        "maintain_aspect_ratio": true,
        "width": 1000,
        "height": 800
    }
}

```

The above can be read as follows:

```text
        "resizable" --> If set to true, the plugin window will be resizable using the bottom right corner
        "maintain_aspect_ratio" --> If set to true, the aspect ratio of the plugin window will be maintained when resized
        "width" --> The initial width of the plugin window
        "height" --> The initial height of the plugin window
```

### Standalone Transport Panel

The standalone transport panel is the panel that is shown when the plugin is run in standalone mode.

To show the standalone transport panel, set `show_standalone_transport_panel` to `true`. To hide the standalone transport panel, set `show_standalone_transport_panel` to `false`.
moreover, use the `exclude_**` fields to make sure these parameters are not loaded from a preset.

```json
{
        "StandaloneTransportPanel": {
        "enable": true,
        "disableInPluginMode": true,
        "exclude_tempo_from_presets": true,
        "exclude_time_signature_from_presets": true,
        "exclude_record_button_from_presets": true,
        "exclude_play_button_from_presets": true
         }
}
```

If you only want the standalone transport panel to be shown when the plugin is run in standalone mode, set `disableInPluginMode` to `true`. If you want the standalone transport panel to be shown in both standalone and plugin mode, set `disableInPluginMode` to `false`.

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
 {
            "tabList": 
            [
                {
                    "name": "Tab 1",
                    "labels": [ ],
                    "lines": [ ],
                    "sliders": [ ],
                    "rotaries": [ ],
                    "buttons": [ ],
                    "labels": [ ],
                    "lines": [ ],
                    "MidiDisplays": [ ]
                    "AudioDisplays": [ ]
                    "triangleSliders": [ ]
                }
            ]
}
```
### Adding Labels

To add a label, add a new entry to the `labels` array. 

```json
{
    "labels": [
        {
            "label": "Groove Velocity",
            "topLeftCorner": "Oe",
            "bottomRightCorner": "Qf",
            "font_size": 10
        }
    ]  
}

```

The above can be read as follows:

```text
        
        "label" --> The label of the label (will be used to set/get the value of the label)
        "topLeftCorner" --> The top left corner of the label (in the grid)
        "bottomRightCorner" --> The bottom right corner of the label (in the grid)
        "font_size" --> The font size of the label
```

### Adding Lines

To add a line, add a new entry to the `lines` array. 

```json
{
    "lines": [
        {
            "start": "Gg",
            "end": "Hh"
        }
    ]  
}

```
### Adding Sliders
To add a slider, add a new entry to the `sliders` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/V2_1_0/datatypes/GuiParams)

```json
{
    "sliders": 
    [
        {
            "label": "Slider 1",       
            "min": 0.0,                  
            "max": 1.0,                 
            "default": 0.5,             
            "topLeftCorner": "Ll",      
            "bottomRightCorner": "Pp",
            "horizontal": true,
            "show_label": true,
            "info": "This is a slider", 
            "exclude_from_presets": false
        }
    ]  
}

```
The above can be read as follows:

```text
        
        "label" --> The label of the slider (will be used to set/get the value of the slider)
        "min" --> The minimum value of the slider
        "max" --> The maximum value of the slider
        "default" --> The default value of the slider
        "topLeftCorner" --> The top left corner of the slider (in the grid)
        "bottomRightCorner" --> The bottom right corner of the slider (in the grid)
        "horizontal" --> Whether or not the slider is horizontal (default is vertical)
        "show_label" --> Whether or not to show the label of the slider
        "info" --> The info of the slider (will be shown when hovering over the slider)
        "exclude_from_presets" --> if true, this value will not change when loading a preset
```

### Adding Rotary Knobs
To add a rotary knob, add a new entry to the `rotaries` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/V2_1_0/datatypes/GuiParams)

```json
{
    "rotaries": 
    [
        {
            "label": "Rotary 1",        
            "min": 0.0,                 
            "max": 1.0,                 
            "default": 0.5,             
            "topLeftCorner": "Ll",      
            "bottomRightCorner": "Pp", 
            "show_label": true,
            "info": "This is a rotary knob",
            "exclude_from_presets": false
        }
    ]
}
```

The above can be read as follows:

```text
            
            "label" --> The label of the rotary knob (will be used to set/get the value of the rotary knob)
            "min" --> The minimum value of the rotary knob
            "max" --> The maximum value of the rotary knob
            "default" --> The default value of the rotary knob
            "topLeftCorner" --> The top left corner of the rotary knob (in the grid)
            "bottomRightCorner" --> The bottom right corner of the rotary knob (in the grid)
            "show_label" --> Whether or not to show the label of the rotary knob
            "info" --> The info of the rotary knob (will be shown when hovering over the rotary knob)
            "exclude_from_presets" --> if true, this value will not change when loading a preset
```
### Adding Buttons
#### Adding Click/Trigger Buttons
To add a click button, add a new entry to the `buttons` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/V2_1_0/datatypes/GuiParams)

```json
 {
    "buttons": [
                    {
                        "label": "Button 1",
                        "isToggle": true,
                        "topLeftCorner": "Be",
                        "bottomRightCorner": "Dg",
                        "show_label": false, 
                        "info": "..."
                    }
              ]
}
```


#### Adding Toggle Switch Buttons
To add a toggle button, add a new entry to the `buttons` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/V2_1_0/datatypes/GuiParams)

```json
 {
    "buttons": [
                    {
                        "label": "Toggle Button 1",
                        "isToggle": true,
                        "topLeftCorner": "Be",
                        "bottomRightCorner": "Dg",
                        "show_label": false, 
                        "info": "...",
                        "exclude_from_presets": true
                    }
              ]
}
```
In the case of buttons, the `exclude_from_presets` parameter is used 
only for toggle buttons. 

For both of the above, you can interpret the parameters as follows:

```text
            "label" --> The label of the button (will be used to set/get the value of the button)
            "isToggle" --> Whether or not the button is a toggle button
            "topLeftCorner" --> The top left corner of the button (in the grid)
            "bottomRightCorner" --> The bottom right corner of the button (in the grid)
            "show_label" --> Whether or not to show the label of the button
            "info" --> The info of the button (will be shown when hovering over the button)
            "exclude_from_presets" --> if true, this value will not change when loading a preset
```

### Adding Combo Boxes
To add a combo box, add a new entry to the `comboBoxes` array. 

To access the values of these elements, refer to [GuiParams]({{site.baseurl}}/docs/V2_1_0/datatypes/GuiParams)

```json
{
  "comboBoxes": [
        {
            "label": "ComboBox 1",
            "topLeftCorner": "Wv",
            "bottomRightCorner": "Zz",
            "items": [ "Item 1", "Item 2", "Item 3" ],
            "info": "ComboBox 1 Info",
            "exclude_from_presets": false

        }
    ]
}
```

The above can be read as follows:

```text
            "label" --> The label of the combo box (will be used to set/get the value of the combo box)
            "topLeftCorner" --> The top left corner of the combo box (in the grid)
            "bottomRightCorner" --> The bottom right corner of the combo box (in the grid)
            "items" --> The items of the combo box
            "info" --> The info of the combo box (will be shown when hovering over the combo box)
            "exclude_from_presets" --> if true, this value will not change when loading a preset
```

### Adding in-tab Midi Visualizers
To add a MIDI visualizer, add a new entry to the `MidiDisplays` array. 

To access/set the values of these elements, refer to [MidiVisualizersData]({{site.baseurl}}/docs/V2_1_0/datatypes/MidiVisualizersData)

```json
{
    "MidiDisplays": [
        {
            "label": "MidiVisualizerA",
            "topLeftCorner": "Av",
            "bottomRightCorner": "Gz",
            "allowToDragOutAsMidi": true,
            "allowToDragInMidi": false,
            "needsPlayhead": true,
            "PlayheadLoopDurationQuarterNotes": 8.0,
            "info": "Pattern A Visualization"
        }
    ]
}
```

In case, you want to show that the content is looped, specify 'PlayheadLoopDurationQuarterNotes', otherwise, 
the playhead will not be looped.

The above can be read as follows:

```text
            "label" --> The label of the MIDI visualizer (will be used to set/get the value of the MIDI visualizer)
            "topLeftCorner" --> The top left corner of the MIDI visualizer (in the grid)
            "bottomRightCorner" --> The bottom right corner of the MIDI visualizer (in the grid)
            "allowToDragOutAsMidi" --> If true, the user can drag out the MIDI data as a MIDI file
            "allowToDragInMidi" --> If true, the user can drag in a MIDI file
            "needsPlayhead" --> If true, the playhead will be shown
            "PlayheadLoopDurationQuarterNotes" --> The duration of the loop in quarter notes (if specified)
            "info" --> The info of the MIDI visualizer (will be shown when hovering over the MIDI visualizer)
```


### Adding in-tab Audio Visualizers
To add an audio visualizer, add a new entry to the `AudioDisplays` array. 

If you want to show that the content is looped, specify 'PlayheadLoopDurationSamples', otherwise,
the playhead will not be looped.

To access/set the values of these elements, refer to [AudioVisualizersData]({{site.baseurl}}/docs/V2_1_0/datatypes/AudioVisualizersData)


```json
{
    "AudioDisplays": [
            {
                "label": "AudioDisplay",
                "topLeftCorner": "Aa",
                "bottomRightCorner": "Kz",
                "allowToDragOutAsAudio": true,
                "allowToDragInAudio": true,
                "needsPlayhead": true,
                "PlayheadLoopDurationQuarterNotes": 8.0,
                "info": "Audio Visualization"
            }
    ]
}
```

The above can be read as follows:

```text
            "label" --> The label of the audio visualizer (will be used to set/get the value of the audio visualizer)
            "topLeftCorner" --> The top left corner of the audio visualizer (in the grid)
            "bottomRightCorner" --> The bottom right corner of the audio visualizer (in the grid)
            "allowToDragOutAsAudio" --> If true, the user can drag out the audio data as a WAV file
            "allowToDragInAudio" --> If true, the user can drag in a WAV file
            "needsPlayhead" --> If true, the playhead will be shown
            "info" --> The info of the audio visualizer (will be shown when hovering over the audio visualizer)
```
## Adding Triangle Slider Widgets

This is a triangular area where you can drag a point around. The position of the point is then used to set two sliders.
This component can be used for a bilinear interpolation between three values.

To add a triangle slider, add a new entry to the `triangleSliders` array.

To access/set the values of these elements, refer to [TriangleSliderData]({{site.baseurl}}/docs/V2_1_0/datatypes/TriangleSliderData)

```json
{
    "triangleSliders": [
        {
            "label": "Interpolation Area",
            "topLeftCorner": "Di",
            "bottomRightCorner": "Nt",
            "DistanceFromBottomLeftCornerSlider": "Interpolate",
            "HeightSlider": "Follow",
            "ensure_equilateral": true,
            "info": "Area in which the interpolation happens"
        }
    ]
}
```

The above can be read as follows:

```text
            "label" --> The label of the triangle slider (will be used to set/get the value of the triangle slider)
            "topLeftCorner" --> The top left corner of the triangle slider (in the grid)
            "bottomRightCorner" --> The bottom right corner of the triangle slider (in the grid)
            "DistanceFromBottomLeftCornerSlider" --> The label of the slider that controls the distance from the bottom left corner
            "HeightSlider" --> The label of the slider that controls the height of the triangle
            "ensure_equilateral" --> If true, the triangle will be drawn as an equilateral triangle, otherwise, bottom edge will be
            almost flushed the bottom of the area and the top edge will be almost flushed to the top middle point of the area
            "info" --> The info of the triangle slider (will be shown when hovering over the triangle slider)
```

To get/set the values of the triangle slider, use the "DistanceFromBottomLeftCornerSlider" and "HeightSlider" labels, 
respectively, and use the typical slider accessors.

## MIDI In/Out Widgets 
These widgets behave differently than the above widgets. These should be only used to visualize incoming or 
outgoing MIDI messages from the plugin

A MIDI In widget can be added to the bottom of the plugin to allow for the user to drop a MIDI file into the plugin.
Moreover, the widget can be used to visualize incoming MIDI messages. To enable this widget, modify the following line in 

```json
{
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
}
```

To access the content of a dragged in MIDI file, refer to [MidiFileEvent]({{site.baseurl}}/docs/V2_1_0/datatypes/MidiFileEvent)



