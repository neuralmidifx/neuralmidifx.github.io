---
layout: default
title: 2 - Pattern Interpolation
nav_order: 2
has_children: false
parent: Tutorials
permalink: /Tutorials/2_PatternInterpolation
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Tutorial 2 - Pattern Interpolation
{: .no_toc }

In this tutorial, we will expand on the previous tutorial and add the ability to interpolate between two patterns. 
{: .fs-6 .fw-300 }

{: .note }
> the source code for this tutorial is available in the tutorials branch of the repository.
> 
> [Tutorial 2 Source Code](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/tutorials/2_PatternInterpolation){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

---

{: .important }
> This tutorial assumes that you have completed the previous tutorial and have a working plugin.
> Please refer to the previous tutorial if you have not completed it yet.
> 

<img src="{{ site.baseurl }}/assets/images/tut2.jpg">

## Description
We will be using the same model as the previous tutorial. 
However, instead of having a single button which generates a random pattern, 
we will have two buttons which generate two random patterns. 
Then, we will add a slider which interpolates between these two patterns.

## Plugin Name and Description 
Let's start by changing the name and description of the plugin.

To do this, we will modify the [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/2_PatternInterpolation/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt) file as follows:

```cmake
project(Tutorial2NMFx VERSION 0.0.1)

set (BaseTargetName Tutorial2NMFx)

....

juce_add_plugin("${BaseTargetName}"
        COMPANY_NAME "AIMCTutorials"                
        ... 
        PLUGIN_CODE AZCL                # a unique 4 character code for your plugin                          
        ...
        PRODUCT_NAME "Tutorial2NMFx")           # Replace with your plugin title

```

Once you re-build the cmake project, and re-build the plugin, you should see the name of the plugin change in the DAW:

<img src="{{ site.baseurl }}/assets/images/tutorial2/0.PNG">

Now we are ready to move on to the next step.

---

## Modifying the GUI
As mentioned in the description, we will be adding two buttons and a slider to the GUI. Moreover, we will 
keep the second tab (for midi mappings) as is without any changes.

Let's go back to the [`Configs_GUI.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/2_PatternInterpolation/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Configs_GUI.h)
file and modify it as follows:

```c++
    namespace Tabs {
        const bool show_grid = false;
        const bool draw_borders_for_components = false;
        const std::vector<tab_tuple> tabList{
            tab_tuple
            {
                "RandomGeneration",
                slider_list
                {
                },
                rotary_list
                {
                    rotary_tuple {"Interpolate", 0, 1, 0.0, "Kg", "Pr"},
                },
                button_list
                {
                    button_tuple{"Random A", false, "El", "Iq"},
                    button_tuple{"Random B", false, "Rl", "Vq"},
                }
            },
            
            // Rest as before

```

<img src="{{ site.baseurl }}/assets/images/tutorial2/a.PNG">


## ModelThread::deploy()

Before modifying the code, let's think about what we want to do.

We want to have two patterns, A and B, and a slider which interpolates between these two patterns. To do this, 
whenever button A is pressed, we will generate a random latent vector and store it somewhere (`latent_A`). Similarly, whenever
button B is pressed, we will generate another random latent vector and store it somewhere  (`latent_B`). 

Subsequently, we will calculate the actual latent vector to be used for inference by interpolating between these two:

```c++
latent = (1 - slider_value) * latent_A + slider_value * latent_B
```

We are familiar with process of generating a random latent vector and using it for inference. However, this is 
the first time that we need to store a custom value that may be used in the future calls of `ModelThread::deploy()` method. As mentioned
in the [`Data Types`]({{ site.baseurl }}/datatypes/CustomizableDataTypes#customizable-data-for-use-within-itp-mdl-and-ppp-threads), 
section of the documentation, we can modify the `MDLData` struct to include our custom data. Additionally, we will store the
slider value in the `MDLData` struct as well to check whether it has changed or since the last call to `ModelThread::deploy()`.

To do so, 
let's navigate to [`CustomStructs.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/2_PatternInterpolation/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CustomStructs.h)
and modify the `MDLData` struct as follows:

```c++
// MDLData struct in CustomStructs.h

struct MDLData {
    torch::Tensor latent_A;
    torch::Tensor latent_B;
    double interpolate_slider_value{0};
};
```

{: .reminder }
> When building the plugin, the plugin will automatically instantiate the `MDLData` struct and pass it to you as `mdl_data` in the `ModelThread::deploy()` method.


Having done this, we can now modify the `ModelThread::deploy()` method to generate a random latent vector and store it in `mdl_data` whenever
button A or B is pressed. 

Before carrying on, let's print some messages to ensure everything is setup correctly


```c++
    // ModelThread::deploy() in ModelThread.cpp

if (!isModelLoaded) {
        load("drumLoopVAE.pt");
    }

    bool should_interpolate = false;   // flag to check if we should interpolate

    // =================================================================================
    // ===         1. ACCESSING GUI PARAMETERS
    // Refer to:
    // https://neuralmidifx.github.io/datatypes/GuiParams#accessing-the-ui-parameters
    // =================================================================================
    // check if the buttons have been clicked, if so, update the MDLdata
    auto ButtonATriggered = gui_params.wasButtonClicked("Random A");
    if (ButtonATriggered) {
        should_interpolate = true;
        PrintMessage("Button A Clicked");
        MDLdata.latent_A = torch::randn({ 1, 128 });
    }
    auto ButtonBTriggered = gui_params.wasButtonClicked("Random B");
    if (ButtonBTriggered) {
        should_interpolate = true;
        PrintMessage("Button B Clicked");
        MDLdata.latent_B = torch::randn({ 1, 128 });
    }

    // check if the interpolate slider has changed, if so, update the MDLdata
    auto sliderValue = gui_params.getValueFor("Interpolate");
    bool sliderChanged = (sliderValue != MDLdata.interpolate_slider_value);
    if (sliderChanged) {
        should_interpolate = true;
        PrintMessage("Slider Changed");
        MDLdata.interpolate_slider_value = sliderValue;
    }
```

<img src="{{ site.baseurl }}/assets/gifs/tut2/gui_test.gif">

Seeing that the GUI is working as expected, we can now implement the interpolation process

To interpolate we need both states to be randomized, as a result, to simplify the code, 
we will ensure that on the first call to `ModelThread::deploy()` both `latent_A` and `latent_B` are randomized.


```c++
    // ModelThread::deploy() in ModelThread.cpp
    
    // ... previous code
    
    // =================================================================================
    // ===         2. initialize latent vectors on the first call
    // =================================================================================
    if (MDLdata.latent_A.size(0) == 0) {
        MDLdata.latent_A = torch::randn({ 1, 128 });
    }
    if (MDLdata.latent_B.size(0) == 0) {
        MDLdata.latent_B = torch::randn({ 1, 128 });
    }
```

Now, to finish this tutorial, we need to interpolate between `latent_A` and `latent_B` and store the result in `latent`.

```c++
    // ModelThread::deploy() in ModelThread.cpp
    
    // ... previous code
    
    // =================================================================================
    // ===         Inference
    // =================================================================================

    bool newPatternGenerated = false;

    if (should_interpolate) {

        if (isModelLoaded)
        {
            // calculate interpolated latent vector
            auto slider_value = MDLdata.interpolate_slider_value;
            auto latent_A = MDLdata.latent_A;
            auto latent_B = MDLdata.latent_B;
            auto latentVector = (1 - slider_value) * latent_A + slider_value * latent_B;
            
            // ... previous code
```

<img src="{{ site.baseurl }}/assets/gifs/tut2/interpolate.gif">
