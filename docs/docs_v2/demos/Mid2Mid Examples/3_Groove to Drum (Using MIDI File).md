---
layout: default
title: Groove to Drum (Using MIDI File)
nav_order: 3
has_children: false
parent: MidToMid
grand_parent: Demos
permalink: /docs/V2_1_0/Demos/MidToMid/Groove_to_Drum_(Using_MIDI_File)
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
As mentioned [here](https://neuralmidifx.github.io/docs/V2_1_0/Installation#step-2-edit-plugin-name-and-description), we need
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


### Declaring Required Local Variables 

We will need to declare the following local variables in the `deploy.h` method:

```c++

    // ...

private:

    // density of tge pattern to be generated
    float density = 0.5f;

    // following will be used for the inference
    // contains the above information in a single tensor of shape (1, 32, 27)
    torch::Tensor groove_hvo = torch::zeros({1, 32, 27}, torch::kFloat32);

    // !! Same as the previous two demos !!
    // add any member variables or methods you need here
    torch::Tensor latent_vector = torch::zeros({1, 128}, torch::kFloat32);
    torch::Tensor voice_thresholds = torch::ones({ 9 }, torch::kFloat32) * 0.5f;
    torch::Tensor max_counts_allowed = torch::ones({ 9 }, torch::kFloat32) * 32;
    int sampling_mode = 0;
    float temperature = 1.0f;
    std::map<int, int> voiceMap;

    torch::Tensor hits;
    torch::Tensor velocities;
    torch::Tensor offsets;

            
```

In the above code, we have also declared a tensor called `latent_vector` which will be used to store the latent vector.

Moreover, the `groove_hvo` tensor is a concatenation of `groove_hits`, `groove_velocities`, and `groove_offsets` tensors.



### deploy()

#### Accessing / Processing the Dropped MIDI File
Let's start with accessing a dropped MIDI file!

As mentioned in the [documentation](https://neuralmidifx.github.io/docs/V2_1_0/datatypes/MidiVisualizersData#accessing-the-content-of-a-dropped-midi-file), the plugin will
provide the Midi data in the form of a vector of MidiFileEvents.

To get the events, we will first check for the `new_midi_file_dropped_on_visualizers` variable in the `deploy()` method, 
and then access the events as follows:

```c++
        auto new_sequence = midiVisualizersData->get_visualizer_data("MidiDropWidget");
        if (new_sequence != std::nullopt) {
            // get and process the events
        }
``` 

As such, we will implement the following method to process the events:

```c++
// ...
        deploy(...) {
        // ... 
            
        // check if a new midi file has been dropped on the visualizers
        bool shouldEncodeGroove = updateInputFromMidiFile();
        
        // ...
        }
        
private:
    // ...
    
    // checks if a new midi file has been dropped on the visualizers and
    // updates the input tensor accordingly
    bool updateInputFromMidiFile() {
        // check if a midi file has been dropped on the visualizers
        auto new_sequence = midiVisualizersData->get_visualizer_data("MidiDropWidget");

        // return false if no new sequence is available
        if (new_sequence == std::nullopt) {
            return false;
        }
        
        // clear out existing groove input
        auto groove_hits = torch::zeros({1, 32, 1}, torch::kFloat32);
        auto groove_velocities = torch::zeros({1, 32, 1}, torch::kFloat32);
        auto groove_offsets = torch::zeros({1, 32, 1}, torch::kFloat32);
        for (const auto& event : *new_sequence) {
            if (event.isNoteOnEvent()) {
                // PrintMessage(event.getDescription().str());
                auto ppq  = event.Time(); // time in ppq
                auto velocity = event.getVelocity(); // velocity
                auto div = round(ppq / .25f);
                auto offset = (ppq - (div * .25f)) / 0.125 * 0.5 ;
                auto grid_index = (long long) fmod(div, 32);
                
                // check if louder if overlapping
                if (groove_hits[0][grid_index][0].item<float>() > 0) {
                    if (groove_velocities[0][grid_index][0].item<float>() < velocity) {
                        groove_velocities[0][grid_index][0] = velocity;
                        groove_offsets[0][grid_index][0] = offset;
                    }
                } else {
                    groove_hits[0][grid_index][0] = 1;
                    groove_velocities[0][grid_index][0] = velocity;
                    groove_offsets[0][grid_index][0] = offset;
                }
            }

        }

        // stack up the groove information into a single tensor
        groove_hvo = torch::concat(
            {
                groove_hits,
                groove_velocities,
                groove_offsets
            }, 2);

        // ignore the operations (I'm just changing the formation of the tensor)
        groove_hvo = torch::zeros({1, 32, 27});
        groove_hvo.index_put_(
            {torch::indexing::Ellipsis, 2},
            groove_hits.index({torch::indexing::Ellipsis, 0}));
        groove_hvo.index_put_(
            {torch::indexing::Ellipsis, 11},
            groove_velocities.index({torch::indexing::Ellipsis, 0}));
        groove_hvo.index_put_(
            {torch::indexing::Ellipsis, 20},
            groove_offsets.index({torch::indexing::Ellipsis, 0}));
        
        return true;
    }
    
```

#### Encode the Groove Information into a latent vector, and decode it into a drum sequence 

Whenever the groove or the density is changed, we will need to encode the groove information into the latent vector.

To do so, we will implement the following method:


```c++
// ...
        deploy(...) {
        // ... 
            
        // encode the groove if necessary
        if (shouldEncodeGroove) {
            if (isModelLoaded) {
                encodeGroove();
                generatePattern();
            }
        }
        
        // ...
        }
        
private:
    // ...
    
    // encodes the groove into a latent vector using the encoder
    void encodeGroove() {
        // preparing the input to encode() method
        std::vector<torch::jit::IValue> enc_inputs;
        enc_inputs.emplace_back(groove_hvo);
        enc_inputs.emplace_back(torch::tensor(
                                    density,
                                    torch::kFloat32).unsqueeze_(0));

        // get the encode method
        auto encode = model.get_method("encode");

        // encode the input
        auto encoder_output = encode(enc_inputs);

        // get latent vector from encoder output
        latent_vector = encoder_output.toTuple()->elements()[2].toTensor();
    }
    
    // decodes a random latent vector into a pattern < --- Similar to previous tutorials
    void generatePattern() {
        // ...   
    }
    
```

### Final `deploy()` Method

```c++
// this method runs on a per-event basis.
    // the majority of the deployment will be done here!
    std::pair<bool, bool> deploy (
        std::optional<MidiFileEvent> & new_midi_event_dragdrop,
        std::optional<EventFromHost> & new_event_from_host,
        bool gui_params_changed_since_last_call,
        bool new_preset_loaded_since_last_call,
        bool new_midi_file_dropped_on_visualizers,
        bool new_audio_file_dropped_on_visualizers) override {


        // Try loading the model if it hasn't been loaded yet
        if (!isModelLoaded) {
            load("drumLoopVAE.pt");
        }

        // Check if voice map should be updated
        bool voiceMapChanged = false;
        if (gui_params_changed_since_last_call) {
            voiceMapChanged = updateVoiceMap();
        }

        // check if a new midi file has been dropped on the visualizers
        bool shouldEncodeGroove = updateInputFromMidiFile();

        // check if density has been updated
        if (gui_params.wasParamUpdated("Density")) {
            density = gui_params.getValueFor("Density");
            shouldEncodeGroove = true;
        }

        // encode the groove if necessary
        if (shouldEncodeGroove) {
            if (isModelLoaded) {
                encodeGroove();
                generatePattern();
            }
        }

        // if the voice map has changed, or a new pattern has been generated,
        // prepare the playback sequence
        if ((voiceMapChanged || shouldEncodeGroove) && isModelLoaded) {
            preparePlaybackSequence();
            preparePlaybackPolicy();
            return {true, true};
        }

        // your implementation goes here
        return {false, false};
    }
```


The complete code for the deploy method can be seen in [PluginCode/deploy.h](https://github.com/neuralmidifx/Mid2Mid_Grv2DrmMidi/blob/master/PluginCode/deploy.h)
We can see that the plugin is
now working as expected:

<img src="{{ site.baseurl }}/assets/gifs/demo3/final.gif">