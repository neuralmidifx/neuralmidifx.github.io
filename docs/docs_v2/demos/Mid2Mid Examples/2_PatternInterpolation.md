---
layout: default
title: Pattern Interpolation
nav_order: 2
has_children: false
parent: MidToMid
grand_parent: Demos
permalink: /docs/v2_0_0/Demos/MidToMid/Pattern_Interpolation
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

Before modifying the code, let's think about what we want to do.

We want to have two patterns, A and B, and a slider which interpolates between these two patterns. To do this, 
whenever button A is pressed, we will generate a random latent vector and store it somewhere (`latent_A`). Similarly, whenever
button B is pressed, we will generate another random latent vector and store it somewhere  (`latent_B`). 

Subsequently, we will calculate the actual latent vector to be used for inference by interpolating between these two:

```c++
latent = (1 - slider_value) * latent_A + slider_value * latent_B
```

We are familiar with process of generating a random latent vector and using it for inference. However, this is 
the first time that we need to store a custom value that may be used in the future calls of `deploy()` method. As mentioned
in the [`DPLData`]({{ site.baseurl }}/docs/v2_0_0/datatypes/DPLData#customizable-data-for-use-within-dpl-thread), 
section of the documentation, we can modify the `DPLData` struct to include our custom data. Additionally, we will store the
slider value in the `DPLdata` struct as well to check whether it has changed or since the last call to `deploy()`.

{: .note }
> The deploy method is called on an event-driven basis. That is, it is called whenever there is new information available.
> Moreover, any variable that is declared inside the `deploy()` method will be re-initialized to its default value every time
> the method is called. Therefore, we need to store the values that we want to use in the future calls of the `deploy()` method
> in a struct that is declared outside the `deploy()` method.

To do so, 
let's navigate to [`DeploymentData.h`](https://github.com/neuralmidifx/Mid2Mid_PatternInterp/blob/master/PluginCode/DeploymentData.h)
and modify the `DPLData` struct as follows:

```c++
// DPLData struct in DeploymentData.h

struct DPLData {
    torch::Tensor latent_A;
    torch::Tensor latent_B;
    double interpolate_slider_value{0};
};
```

{: .reminder }
> When building the plugin, the plugin will automatically instantiate the `DPLData` struct and pass it to you as `DPLdata` in the `deploy()` method.


Having done this, we can now modify the `deploy()` method to generate a random latent vector and store it in `DPLdata` whenever
button A or B is pressed. 

Before carrying on, let's print some messages to ensure everything is setup correctly


```c++

if (!isModelLoaded) {
        load("drumLoopVAE.pt");
    }

    bool should_interpolate = false;   // flag to check if we should interpolate

    // =================================================================================
    // ===         1. ACCESSING GUI PARAMETERS
    // Refer to:
    // https://neuralmidifx.github.io/docs/v2_0_0/datatypes/GuiParams#accessing-the-ui-parameters
    // =================================================================================
    // check if the buttons have been clicked, if so, update the DPLdata
    auto ButtonATriggered = gui_params.wasButtonClicked("Random A");
    if (ButtonATriggered) {
        should_interpolate = true;
        PrintMessage("Button A Clicked");
        DPLdata.latent_A = torch::randn({ 1, 128 });
    }
    auto ButtonBTriggered = gui_params.wasButtonClicked("Random B");
    if (ButtonBTriggered) {
        should_interpolate = true;
        PrintMessage("Button B Clicked");
        DPLdata.latent_B = torch::randn({ 1, 128 });
    }

    // check if the interpolate slider has changed, if so, update the DPLdata
    auto sliderValue = gui_params.getValueFor("Interpolate");
    bool sliderChanged = (sliderValue != DPLdata.interpolate_slider_value);
    if (sliderChanged) {
        should_interpolate = true;
        PrintMessage("Slider Changed");
        DPLdata.interpolate_slider_value = sliderValue;
    }
```

<img src="{{ site.baseurl }}/assets/gifs/tut2/gui_test.gif">

Seeing that the GUI is working as expected, we can now implement the interpolation process

To interpolate we need both states to be randomized, as a result, to simplify the code, 
we will ensure that on the first call to `deploy()` both `latent_A` and `latent_B` are randomized.


```c++
    
    // ... previous code
    
    // =================================================================================
    // ===         2. initialize latent vectors on the first call
    // =================================================================================
    if (DPLdata.latent_A.size(0) == 0) {
        DPLdata.latent_A = torch::randn({ 1, 128 });
    }
    if (DPLdata.latent_B.size(0) == 0) {
        DPLdata.latent_B = torch::randn({ 1, 128 });
    }
```

Now, to finish this demo, we need to interpolate between `latent_A` and `latent_B` and store the result in `latent`.

```c++
    
    // ... previous code
    
    // =================================================================================
    // ===         Inference
    // =================================================================================

    bool newPatternGenerated = false;

    if (should_interpolate) {

        if (isModelLoaded)
        {
            // calculate interpolated latent vector
            auto slider_value = DPLdata.interpolate_slider_value;
            auto latent_A = DPLdata.latent_A;
            auto latent_B = DPLdata.latent_B;
            auto latentVector = (1 - slider_value) * latent_A + slider_value * latent_B;
            
            // ... previous code
```

<img src="{{ site.baseurl }}/assets/gifs/tut2/interpolate.gif">


## [Optional] Preset Management

Starting in V2.0.0, we have added the ability to save and load presets. This is useful for saving the state of the plugin
and recalling it later. In the case of plugins, this was already possible via the host. However, in that case, only
the UI parameters were saved. As such, all tensor data were lost and needed to be recalculated. 

With the new preset management system, you have access to a structure called [`CustomPresetData`](https://neuralmidifx.github.io/docs/v2_0_0/datatypes/CustomPresetDataDictionary) which you can use to store 
any custom data that should be tracked/reloaded whenever a preset is saved/loaded. 

In this demo, we will use this feature to save the `A` and `B` latent vectors. To do so, I will simply add the following
lines which notifies the preset manager to take a snapshot of these tensors.

```c++
if (should_interpolate) {

        if (isModelLoaded)
        {
            // ... previous code

            // Backup the data for preset saving
            CustomPresetData->tensor("latent_A", latent_A);
            CustomPresetData->tensor("latent_B", latent_B);

            // ... previous code
```

Moreover, we will also add the following lines to the `deploy()` method to load the tensors from the preset manager
whenever a preset is loaded.

As soon as a preset is loaded, the `new_preset_loaded_since_last_call` flag will be set to `true`. As such, you should
check for this flag and subsequently, load your saved tensors

```c++
    // check if the preset has changed, if so, update the MDLdata
    if (new_preset_loaded_since_last_call) {
        should_interpolate = true;
        auto l_a = CustomPresetData->tensor("latent_A");
        auto l_b = CustomPresetData->tensor("latent_B");
        if (l_a != std::nullopt) {
            DPLdata.latent_A = *l_a;
        }
        if (l_b != std::nullopt) {
            DPLdata.latent_B = *l_b;
        }
        
    }
```

<img src="{{ site.baseurl }}/assets/gifs/demo2/demo2.gif">