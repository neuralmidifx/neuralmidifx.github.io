---
layout: default
title: Pattern Interpolation
nav_order: 2
has_children: false
parent: MidToMid
grand_parent: Demos
permalink: /docs/V2_1_0/Demos/MidToMid/Pattern_Interpolation
---


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Pattern Interpolation
{: .no_toc }


In this demo, we will expand on the previous demo and add the ability to interpolate between two patterns. 
{: .fs-6 .fw-300 }

{: .note }
> the source code for this demo is available in the demos branch of the repository.
> 
> [Demo 2 Source Code](https://github.com/neuralmidifx/Mid2Mid_PatternInterp){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

---

{: .important }
> This demo assumes that you have completed the previous demo and have a working plugin.
> Please refer to the previous demo if you have not completed it yet.
> 

## Description
We will be using the same model as the previous demo. 
However, instead of having a single button which generates a random pattern, 
we will have two buttons which generate two random patterns. 
Then, we will add a slider which interpolates between these two patterns.

## Plugin Name and Description 
Let's start by changing the name and description of the plugin.

To do this, we will modify the [PluginCode/CMakeLists.txt](https://github.com/neuralmidifx/Mid2Mid_PatternInterp/blob/master/PluginCode/CMakeLists.txt) file as follows:

```cmake
project(Demo2 VERSION 0.0.1)

set (BaseTargetName Demo2)

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
        PLUGIN_CODE aaab                             # MUST BE UNIQUE!! If similar to other plugins, conflicts will occur
        FORMATS AU VST3 Standalone
        PRODUCT_NAME "Demo2")           # Replace with your plugin title

```

Once you re-build the cmake project, and re-build the plugin, you should see the name of the plugin change in the DAW:

<img src="{{ site.baseurl }}/assets/images/mid2mid/demo2/0.PNG">

Now we are ready to move on to the next step.

---

## Modifying the GUI
As mentioned in the description, we will be adding two buttons and a slider to the GUI. Moreover, we will 
keep the second tab (for midi mappings) as is without any changes.

Let's go back to the [`settings.json`](https://github.com/neuralmidifx/Mid2Mid_PatternInterp/blob/master/PluginCode/settings.json)
file and modify it as follows:

```json
            {
                "name": "RandomGeneration",
                "sliders": [],
                "rotaries": [
                    {
                        "label": "Interpolate",
                        "min": 0.0,
                        "max": 1.0,
                        "default": 0.0,
                        "topLeftCorner": "Kg",
                        "bottomRightCorner": "Pr"
                    }
                ],
                "buttons": [{
                    "label": "Random A",
                    "isToggle": false,
                    "topLeftCorner": "El",
                    "bottomRightCorner": "Iq"},
                    {
                        "label": "Random B",
                        "isToggle": false,
                        "topLeftCorner": "Rl",
                        "bottomRightCorner": "Vq"
                    }
                ],
                "MidiDisplays": []
            }

```

<img src="{{ site.baseurl }}/assets/images/demo2/a.PNG">


## DeploymentThread::deploy()

### Adding Variables

The very first thing we need to do is to add the variables used between different iterations of the deploy() function.

As such, we will add the following variables to the `deploy.h` file.

```c++
private:
    // AB Interpolation Parameters
    torch::Tensor latent_A = torch::randn({ 1, 128 });
    torch::Tensor latent_B = torch::randn({ 1, 128 });
    torch::Tensor latentVector;
    double interpolate_slider_value = 0.0;
    
    // ---------------------------------------------------------------------------------------- 
    // !! the following were added in the previous demo! make sure you've read the previous demo !!
    // ----------------------------------------------------------------------------------------
    // add any member variables or methods you need here
    torch::Tensor voice_thresholds = torch::ones({ 9 }, torch::kFloat32) * 0.5f;
    torch::Tensor max_counts_allowed = torch::ones({ 9 }, torch::kFloat32) * 32;
    int sampling_mode = 0;
    float temperature = 1.0f;
    std::map<int, int> voiceMap;

    torch::Tensor hits;
    torch::Tensor velocities;
    torch::Tensor offsets;
```

### Randomizing the Latent Vectors

Next, we will modify the `deploy()` function to randomize the latent vectors when the buttons are clicked or the 
slider is moved

```c++
        // check if buttons were clicked to randomize the pattern
        auto ButtonATriggered = gui_params.wasButtonClicked("Random A");
        if (ButtonATriggered) {
            latent_A = torch::randn({ 1, 128 });
            shouldInterpolate = true;
        }

        // check if buttons were clicked to randomize the pattern
        auto ButtonBTriggered = gui_params.wasButtonClicked("Random B");
        if (ButtonBTriggered) {
            cout << "Random B" << endl;
            latent_B = torch::randn({ 1, 128 });
            shouldInterpolate = true;
        }
        
        // get interpolate slider value if changed
        if (gui_params.wasParamUpdated("Interpolate")) {
            interpolate_slider_value = gui_params.getValueFor("Interpolate");
            shouldInterpolate = true;
        }
```

The `shouldInterpolate` flag is used to check if we should interpolate between the two latent vectors or not in the next step.

### Interpolating the Latent Vectors

We will implement the following method to interpolate between the two latent vectors:

```c++
// previous code

private:
    
    // ...
    
    // interpolation method
    void interpolate() {
        latentVector = (1 - interpolate_slider_value) * latent_A + interpolate_slider_value * latent_B;
    }
    
    // ...
```

### Generating and Playback

Once we have the latent vector, we will be doing the exact same procedure as the previous demo, with one exception. 
Instead of using a random latent vector, we will be using the interpolated latent vector.

```c++
 void generatePatternUsingLatent() {

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
        hits = output.toTuple()->elements()[0].toTensor();
        velocities = output.toTuple()->elements()[1].toTensor();
        offsets = output.toTuple()->elements()[2].toTensor();
    }
```

### Putting it all together

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

        // flag to check if a new latent vector should be interpolated
        bool shouldInterpolate = false;

        // check if new preset was loaded
        if (new_preset_loaded_since_last_call) {
            loadTensorsFromPreset();
            shouldInterpolate = true;
            cout << "New Preset Loaded" << endl;
        }

        // check if buttons were clicked to randomize the pattern
        auto ButtonATriggered = gui_params.wasButtonClicked("Random A");
        if (ButtonATriggered) {
            latent_A = torch::randn({ 1, 128 });
            CustomPresetData->tensor("latent_A", latent_A);
            shouldInterpolate = true;
        }

        // check if buttons were clicked to randomize the pattern
        auto ButtonBTriggered = gui_params.wasButtonClicked("Random B");
        if (ButtonBTriggered) {
            cout << "Random B" << endl;
            latent_B = torch::randn({ 1, 128 });
            CustomPresetData->tensor("latent_A", latent_A);
            shouldInterpolate = true;
        }

        // get interpolate slider value if changed
        if (gui_params.wasParamUpdated("Interpolate")) {
            interpolate_slider_value = gui_params.getValueFor("Interpolate");
            shouldInterpolate = true;
        }

        // Check if voice map should be updated
        bool voiceMapChanged = false;
        if (gui_params_changed_since_last_call) {
            voiceMapChanged = updateVoiceMap();
        }


        // if the voice map has changed, or a new pattern has been generated,
        // prepare the playback sequence
        if ((voiceMapChanged || shouldInterpolate) && isModelLoaded) {
            interpolate();
            generatePatternUsingLatent();
            preparePlaybackSequence();
            preparePlaybackPolicy();
            return {true, true};
        }

        // your implementation goes here
        return {false, false};
    }
```



<img src="{{ site.baseurl }}/assets/gifs/tut2/interpolate.gif">


## [Optional] Preset Management

Starting in V2.0.0, we have added the ability to save and load presets. This is useful for saving the state of the plugin
and recalling it later. In the case of plugins, this was already possible via the host. However, in that case, only
the UI parameters were saved. As such, all tensor data were lost and needed to be recalculated. 

With the new preset management system, you have access to a structure called [`CustomPresetData`](https://neuralmidifx.github.io/docs/V2_1_0/datatypes/CustomPresetDataDictionary) which you can use to store 
any custom data that should be tracked/reloaded whenever a preset is saved/loaded. 

In this demo, we will use this feature to save the `A` and `B` latent vectors. To do so, I will simply add the following
lines which notifies the preset manager to take a snapshot of these tensors.

```c++
        // check if buttons were clicked to randomize the pattern
        auto ButtonATriggered = gui_params.wasButtonClicked("Random A");
        if (ButtonATriggered) {
            latent_A = torch::randn({ 1, 128 });
            CustomPresetData->tensor("latent_A", latent_A);  // <--- this line
            shouldInterpolate = true;
        }

        // check if buttons were clicked to randomize the pattern
        auto ButtonBTriggered = gui_params.wasButtonClicked("Random B");
        if (ButtonBTriggered) {
            cout << "Random B" << endl;
            latent_B = torch::randn({ 1, 128 });
            CustomPresetData->tensor("latent_B", latent_B); // <--- this line
            shouldInterpolate = true;
        }
```

Moreover, we will also add the following lines to the `deploy()` method to load the tensors from the preset manager
whenever a preset is loaded.

As soon as a preset is loaded, the `new_preset_loaded_since_last_call` flag will be set to `true`. As such, you should
check for this flag and subsequently, load your saved tensors

```c++
    
    deploy() {
    
    // ...
        
        // check if new preset was loaded
        if (new_preset_loaded_since_last_call) {
            loadTensorsFromPreset();
            shouldInterpolate = true;
            cout << "New Preset Loaded" << endl;
            
    }
    
private:
    
    // ...
    
        void loadTensorsFromPreset() {
        auto A = CustomPresetData->tensor("latent_A");  // get reference to the tensor (if it exists)
        if (A != std::nullopt)                          // <--- this line checks if the tensor exists in the preset manager
        {
            latent_A = *A;                              //  <--- this line loads the tensor from the preset manager
        }
        auto B = CustomPresetData->tensor("latent_B");
        if (B != std::nullopt)
        {
            latent_B = *B;
        }
    }
    
    // ...
```

<img src="{{ site.baseurl }}/assets/gifs/demo2/final.gif">