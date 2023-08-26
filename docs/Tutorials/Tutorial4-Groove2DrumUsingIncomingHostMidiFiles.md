---
layout: default
title: 4 - Groove to Drum (Real-Time) 
nav_order: 4
has_children: false
parent: Tutorials
permalink: /Tutorials/3_Groove2DrumRealTime/
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

## Specifying What Information We Need from DAW

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

As a refresher, let's watch the following video to remind ourselves what information can be received from the DAW. 













## InputTensorPreparatorThread::deploy()
This is the first time we are using the `InputTensorPreparatorThread` class as until now
none of the tasks required an input to be prepared before running inference.

To prepare the input tensor, let's take a quick look at the python script which was serialized
for encoding an input groove into a tensor:

```python

    def encode(self, groove, density):
        """ Encodes a given input sequence of shape (batch_size, seq_len, embedding_size_src) into a latent space
        of shape (batch_size, latent_dim)

        :param groove: the input sequence of shape (batch_size, 32, 3) where 32 is the number of steps
        
        :return: mu, log_var, latent_z (each of shape [batch_size, latent_dim])
        """
```

What can be seen above is that we will create three tensors for `hits`, `velocities`, and `offsets`, each of shape `[1, 32, 1]`.
The `groove` information will then be placed in the third voice (closed hihat channel) of the tensor. 

{:. note}
> Remember that the ITP Deploy() method is called every time a new MIDI event arrives or whenever the GUI parameters change.
> Moreover, the method is called **one event at a time**.  

### Declaring Required Local Variables in `ITPData`
Given that we will repeatedly update `hits`, `velocities`, and `offsets` tensors upon arrival of new MIDI events, 
we will need to create these tensors somewhere outside the `deploy()` function. 
To this end, we will start with modifying the `ITPData` struct in [Configs_CustomStructs.h](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/3_Groove2DrumUsingMidiFile/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CustomStructs.h).

```c++
            // Configs_CustomStructs.h
            
            // rest of the code ...   
            
            struct ITPData
            {                
                torch::Tensor hits = torch::zeros({1, 32, 1}, torch::kFloat32);
                torch::Tensor velocities = torch::zeros({1, 32, 1}, torch::kFloat32);
                torch::Tensor offsets = torch::zeros({1, 32, 1}, torch::kFloat32);
                
                // rest of the code ...   
            };
            
            // rest of the code ...   
```

### Declaring What Data to be Sent to Model Thread
Once the input tensors are ready, we will need to send them to the model thread.
That said, instead of sending the tensors themselves, we can send a concatenated version of them (see the python script above).

To this end, we will modify the `ModelInput` struct in [Configs_CustomStructs.h](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/3_Groove2DrumUsingMidiFile/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CustomStructs.h).

```c++
            // Configs_CustomStructs.h
            
            // rest of the code ...   
            
            struct ModelInput {
                torch::Tensor hvo = torch::zeros({1, 32, 3}, torch::kFloat32);
                
                // rest of the code ...
            };
```

### Accessing the MIDI File
Let's start with accessing a dropped MIDI file!

As mentioned in the [documentation](https://neuralmidifx.github.io/docs/Plugin#midi-file-drag-and-drop), the plugin will
provide the Midi data one event at a time. To access these, we will modify the [`ITP_Deploy.cpp`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/3_Groove2DrumUsingMidiFile/NeuralMidiFXPlugin/NeuralMidiFXPlugin/ITP_Deploy.cpp)
accordingly:

```c++
// ITP_Deploy.cpp

bool InputTensorPreparatorThread::deploy(
    std::optional<MidiFileEvent> & new_midi_event_dragdrop,
    std::optional<EventFromHost> & new_event_from_host,
    bool gui_params_changed_since_last_call) {

    bool SHOULD_SEND_TO_MODEL_FOR_GENERATION_ = false;

    // =================================================================================
    // ===         ACCESSING INFORMATION (EVENTS) RECEIVED FROM
    //                Mannually Drag-Dropped Midi Files
    // Refer to:
    // https://neuralmidifx.github.io/datatypes/MidiFileEvent
    // =================================================================================
    
    // check if there is a new midi file event
    if (new_midi_event_dragdrop.has_value())
    {
        PrintMessage(new_midi_event_dragdrop->getDescription().str());
    }

    return SHOULD_SEND_TO_MODEL_FOR_GENERATION_;
}

```
<img src="{{ site.baseurl }}/assets/gifs/tut3/MidiEventsPrinted.gif">

### Preparing the Input Tensor (Groove)
The groove in this context is basically a flattened version of a polyphonic pattern. Moreover, the groove
is only based on the onsets of the notes.

What this means is that for each Midi event arrived, we will:

1. check if NoteOn
2. Figure out the closest step to the onset of the note
3. Calculate the offset of the note
4. Set the corresponding step in the `hits` tensor to 1
5. Set the corresponding step in the `velocities` tensor to the velocity of the note
6. Set the corresponding step in the `offsets` tensor to the offset of the note
7. For overlapping notes, use the loudest one

As a result, the `deploy()` method will look like this:

```c++

    // ITP_Deploy.cpp

    // rest of the code ...
    
    if (new_midi_event_dragdrop.has_value()) {

        if (new_midi_event_dragdrop->isFirstMessage()) {
            // clear hits, velocities, offsets
            ITPdata.hits = torch::zeros({1, 32, 1}, torch::kFloat32);
            ITPdata.velocities = torch::zeros({1, 32, 1}, torch::kFloat32);
            ITPdata.offsets = torch::zeros({1, 32, 1}, torch::kFloat32);
        }

        if (new_midi_event_dragdrop->isNoteOnEvent()) {
            auto ppq  = new_midi_event_dragdrop->Time(); // time in ppq
            auto velocity = new_midi_event_dragdrop->getVelocity(); // velocity
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

        // if all messages have been received, send to model for generation
        if (new_midi_event_dragdrop->isLastMessage()) {
            
            model_input.hvo = torch::concat(
                {ITPdata.hits, ITPdata.velocities, ITPdata.offsets}, 2);
            DisplayTensor(model_input.hvo, "model_input.hvo");
            
            SHOULD_SEND_TO_MODEL_FOR_GENERATION_ = true;
        }
    }
    
    // rest of the code ...
```
<img src="{{ site.baseurl }}/assets/gifs/tut3/MidiEventsProcessed.gif">

## ModelThread::deploy()
Now, let's move on to the Model Thread.

For this part, we will need to modify the `deploy()` method in the [`MDL_Deploy.cpp`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/3_Groove2DrumUsingMidiFile/NeuralMidiFXPlugin/NeuralMidiFXPlugin/MDL_Deploy.cpp)

### Encoding the Groove with the Density Value

#### Accessing the Density Value
As mentioned earlier, in addition to the groove, the model also accepts a density value. 
We have already added a slider for this to the GUI. Now, we just need to access it

```c++

// MDL_Deploy.cpp

bool ModelThread::deploy(bool new_model_input_received,
                         bool did_any_gui_params_change) {

    /*              IF YOU NEED TO PRINT TO CONSOLE FOR DEBUGGING,
    *                  YOU CAN USE THE FOLLOWING METHOD:
    *                      PrintMessage("YOUR MESSAGE HERE");
    */

    // flag to indicate if a new pattern has been generated and is ready for transmission
    // to the PPP thread
    bool newPatternGenerated = false;

    // =================================================================================
    // ===         0. LOADING THE MODEL
    // =================================================================================
    // Try loading the model if it hasn't been loaded yet
    if (!isModelLoaded) {
        load("drumLoopVAE.pt");
    }


    // =================================================================================
    // ===              ACCESSING GUI PARAMETERS
    // Refer to:
    // https://neuralmidifx.github.io/datatypes/GuiParams#accessing-the-ui-parameters
    // =================================================================================
    auto density = gui_params.getValueFor("Density");
    PrintMessage("Density: " + std::to_string(density));
    
    return newPatternGenerated;
}

```
<img src="{{ site.baseurl }}/assets/gifs/tut3/DensityVal.gif">

#### encode()

Now, we need to prepare the inputs to the encode method. Then, we can use the prepared inputs and encode the 
groove and the density value into a latent vector.

```c++
// MDL_Deploy.cpp

bool ModelThread::deploy(bool new_model_input_received,
                         bool did_any_gui_params_change) {

    /*              IF YOU NEED TO PRINT TO CONSOLE FOR DEBUGGING,
    *                  YOU CAN USE THE FOLLOWING METHOD:
    *                      PrintMessage("YOUR MESSAGE HERE");
    */

    // flag to indicate if a new pattern has been generated and is ready for transmission
    // to the PPP thread
    bool newPatternGenerated = false;

    // =================================================================================
    // ===         0. LOADING THE MODEL
    // =================================================================================
    // Try loading the model if it hasn't been loaded yet
    if (!isModelLoaded) {
        load("drumLoopVAE.pt");
    }

    // get the density value
    auto density = gui_params.getValueFor("Density");

    // get latest groove received from ITP
    auto hvo = model_input.hvo;

    // preparing the input to encode() method
    std::vector<torch::jit::IValue> enc_inputs;
    enc_inputs.emplace_back(hvo);
    enc_inputs.emplace_back(torch::tensor(density, torch::kFloat32));

    // get the encode method
    auto encode = model.get_method("encode");

    // encode the input
    auto encoder_output = encode(enc_inputs);

    // get latent vector from encoder output
    auto latent_vector = encoder_output.toTuple()->elements()[2].toTensor();

    // Display the latent vector
    DisplayTensor(latent_vector, "Latent Vector");

    return newPatternGenerated;
}
    
```

<img src="{{ site.baseurl }}/assets/gifs/tut3/Encoded.gif">


#### Decoding the Latent Vector

We have done the hard part. Now, we just need to reuse the decoding scripts from the previous tutorial.

```c++

// MDL_Deploy.cpp

    // previous code ...

    // Prepare other inputs
    auto voice_thresholds = torch::ones({9 }, torch::kFloat32) * 0.5f;
    auto max_counts_allowed = torch::ones({9 }, torch::kFloat32) * 32;
    int sampling_mode = 0;
    float temperature = 1.0f;

    // Prepare above for inference
    std::vector<torch::jit::IValue> inputs;
    inputs.emplace_back(latentVector);
    inputs.emplace_back(voice_thresholds);
    inputs.emplace_back(max_counts_allowed);
    inputs.emplace_back(sampling_mode);
    inputs.emplace_back(temperature);

    // Get the scripted method
    auto sample_method = model.get_method("sample");

    // Run inference
    auto output = sample_method(inputs);

    // Extract the generated tensors from the output
    auto hits = output.toTuple()->elements()[0].toTensor();
    auto velocities = output.toTuple()->elements()[1].toTensor();
    auto offsets = output.toTuple()->elements()[2].toTensor();

    // wrap the generated tensors into a ModelOutput struct
    model_output.hits = hits;
    model_output.velocities = velocities;
    model_output.offsets = offsets;

    // Set the flag to true
    newPatternGenerated = true;

    // Rest of the code ...

```

Assuming that the PPP thread is exactly the same as the previous tutorial, we can see that the plugin is
now working as expected:

<img src="{{ site.baseurl }}/assets/gifs/tut3/Final.gif">