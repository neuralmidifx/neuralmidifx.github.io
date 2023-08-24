---
layout: default
title: 3 - Groove to Drum 
nav_order: 3
has_children: false
parent: Tutorials
permalink: /Tutorials/3_Groove2DrumUsingMidiFile
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Tutorial 3 - Groove to Drum Accompaniment using MIDI File
{: .no_toc }

{: .warning }
> In this tutorial, we will use the same model as the previous tutorials, so make sure you have completed the previous tutorials.

{: .note }
> the source code for this tutorial is available in the tutorials branch of the repository.
> 
> [Tutorial 3 Source Code](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/tutorials/3_Groove2DrumUsingMidiFile){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


## Description
The model provided in the previous tutorials converts a single voice groove into a drum pattern.
Moreover, it allows for specifying the density of the generated pattern.

In this tutorial, we will create a plugin which allows the user to drop in a MIDI file
containing any pattern (assuming 2-bars and 4/4 time signature).

In this case, the plugin will need to do the following:
1. Enable the user to drop in a MIDI file
2. Analyze the MIDI file and extract the groove
3. Run Inference on the extracted groove
4. Extract the drum pattern from the inference result

## Plugin Name and Description
As mentioned [here](https://neuralmidifx.github.io/docs/Installation#step-2-edit-plugin-name-and-description), we need
to specify the name of the plugin as well as some descriptions for it. 

To do this, we will modify the [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/3_Groove2DrumUsingMidiFile/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt) file as follows:

```cmake
project(Tutorial3NMFx VERSION 0.0.1)

set (BaseTargetName Tutorial3NMFx)

....

juce_add_plugin("${BaseTargetName}"
        COMPANY_NAME "AIMCTutorials"                
        ... 
        PLUGIN_CODE AZCM                # a unique 4 character code for your plugin                          
        ...
        PRODUCT_NAME "Tutorial3NMFx")           # Replace with your plugin title

```

Once you re-build the cmake project, and re-build the plugin, you should see the name of the plugin change in the DAW:

<img src="{{ site.baseurl }}/assets/images/tutorial3/0.PNG">

Now we are ready to move on to the next step.

## Modifying the GUI

To customize the GUI we will be modifying the [`Configs_GUI.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/3_Groove2DrumUsingMidiFile/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Configs_GUI.h)
file.

### Density Slider
On the first tab of the plugin, we will add a slider to control the density of the generated drum pattern.

```cpp
            // Configs_GUI.h
            
            // rest of the code ...          

            tab_tuple
            {
                "Generation Controls",
                slider_list
                {
                },
                rotary_list
                {
                    rotary_tuple {"Density", 0, 1, 0.5, "Kg", "Pr"},
                },
                button_list
                {
                }
            },
             
            // rest of the code ...          
```

### Enabling Drag and Drop
To allow for Drag/Drop features, we will modify `MidiInVisualizer` in the same file as follows:

```c++

            // Configs_GUI.h
            
            // rest of the code ...   
            
            namespace MidiInVisualizer {

                const bool enable = true;
                const bool allowToDragInMidi = true;
                
                // rest of the code ...   
            
            }
            
```

Having done this, the plugin should have the following appearance and also allow for drag/drop of MIDI files:


<img src="{{ site.baseurl }}/assets/gifs/tut3/MidiDrag.gif">

## InputTensorPreparatorThread::deploy()
### Accessing the MIDI File
### Preparing the Input Tensor (Groove)

## ModelThread::deploy()
### 
