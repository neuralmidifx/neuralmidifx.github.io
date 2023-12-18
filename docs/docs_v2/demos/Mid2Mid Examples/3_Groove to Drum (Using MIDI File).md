---
layout: default
title: Groove to Drum (Using MIDI File)
nav_order: 3
has_children: false
parent: MidToMid
grand_parent: Demos
permalink: /docs/v2_0_0/Demos/MidToMid/Groove_to_Drum_(Using_MIDI_File)
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
> [Tutorial 3 Source Code](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmMidi){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


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
As mentioned [here](https://neuralmidifx.github.io/docs/v2_0_0/Installation#step-2-edit-plugin-name-and-description), we need
to specify the name of the plugin as well as some descriptions for it. 

To do this, we will modify the [PluginCode/CMakeLists.txt](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmMidi/blob/master/PluginCode/CMakeLists.txt) file as follows:

```cmake
project(Demo3 VERSION 0.0.1)

set (BaseTargetName Demo3)

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
        PRODUCT_NAME "Demo3")           # Replace with your plugin title




```

Now we are ready to move on to the next step.

## Modifying the GUI

To customize the GUI we will be modifying the [`settings.json`](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmMidi/blob/master/PluginCode/settings.json)
file.

### UI Components

We will create a tab called `RandomGeneration` which will contain the following UI components:
1. A slider to control the density of the generated drum pattern
2. A `MidiDropWidget` to allow the user to drop in a MIDI file

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
                    "MidiDisplays": [
                        {
                            "label": "MidiDropWidget",
                            "topLeftCorner": "Aa",
                            "bottomRightCorner": "Zh",
                            "allowToDragOutAsMidi": false,
                            "allowToDragInMidi": true,
                            "needsPlayhead": false
                        }
                    ]
                }
```


## Encoding the Input MIDI File

So far, in previous tutorials, we have been working with randomized latent vectors. However, in this tutorial, we will be
using a MIDI file as input.

The first step is to process the MIDI file and extract the groove information. This will be done in the `deploy()` method.

To prepare the input tensor, let's take a quick look at the python script which was serialized
for encoding an input groove into a tensor:

```python

    def encode(self, groove, density):
        """ Encodes a given input sequence of shape (batch_size, seq_len, embedding_size_src) into a latent space
        of shape (batch_size, latent_dim)

        :param groove: the input sequence of shape (batch_size, 32, 27) where 32 is the number of steps
        
        :return: mu, log_var, latent_z (each of shape [batch_size, latent_dim])
        """
```

What can be seen above is that we will create three tensors for `hits`, `velocities`, and `offsets`, each of shape `[1, 32, 1]`.
The `groove` information will then be placed in the third voice (closed hihat channel) of the tensor. 


### Declaring Required Local Variables in `DeploymentData.h.h`
Given that we will repeatedly update `hits`, `velocities`, and `offsets` tensors upon arrival of new MIDI events, 
we will need to create these tensors somewhere outside the `deploy()` function. 
To this end, we will start with modifying the `DeploymentData.h` struct in [DeploymentData.h](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmMidi/tree/master/PluginCode/DeploymentData.h)

```c++
            // DeploymentData.h
            
            
            struct DPLData
            {                
                // groove information
                torch::Tensor groove_hits = torch::zeros({1, 32, 1}, torch::kFloat32);
                torch::Tensor groove_velocities = torch::zeros({1, 32, 1}, torch::kFloat32);
                torch::Tensor groove_offsets = torch::zeros({1, 32, 1}, torch::kFloat32);
            
                // following will be used for the inference
                // contains the above information in a single tensor of shape (1, 32, 27)
                torch::Tensor groove_hvo = torch::zeros({1, 32, 27}, torch::kFloat32);
            
                // Latent vector
                torch::Tensor latent_vector = torch::zeros({1, 128}, torch::kFloat32);
            };
            
```

In the above code, we have also declared a tensor called `latent_vector` which will be used to store the latent vector.

Moreover, the `groove_hvo` tensor is a concatenation of `groove_hits`, `groove_velocities`, and `groove_offsets` tensors.



### deploy()
Let's start with accessing a dropped MIDI file!

As mentioned in the [documentation](https://neuralmidifx.github.io/docs/v2_0_0/datatypes/MidiVisualizersData#accessing-the-content-of-a-dropped-midi-file), the plugin will
provide the Midi data in the form of a vector of MidiFileEvents.

To get the events, we will first check for the `new_midi_file_dropped_on_visualizers` variable in the `deploy()` method, 
and then access the events as follows:

```c++
        auto new_sequence = midiVisualizersData->get_visualizer_data("MidiDropWidget");
        if (new_sequence != std::nullopt) {
            // get and process the events
        }
``` 

Once we have the events processed, we will use the `encode` method as follows:

```c++
            // preparing the input to encode() method
            std::vector<torch::jit::IValue> enc_inputs;
            enc_inputs.emplace_back(DPLdata.groove_hvo);
            enc_inputs.emplace_back(torch::tensor(
                                        density,
                                        torch::kFloat32).unsqueeze_(0));

            // get the encode method
            auto encode = model.get_method("encode");

            // encode the input
            auto encoder_output = encode(enc_inputs);

            // get latent vector from encoder output
            DPLdata.latent_vector = encoder_output.toTuple()->elements()[2].toTensor();
```

The complete code for the deploy method can be seen in [PluginCode/Deploy.cpp](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmMidi/blob/master/PluginCode/Deploy.cpp)
We can see that the plugin is
now working as expected:

<img src="{{ site.baseurl }}/assets/gifs/demo3/final.gif">