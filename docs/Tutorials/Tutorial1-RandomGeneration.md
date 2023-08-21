---
layout: default
title: 1 - Random Generation
nav_order: 1
has_children: false
parent: Tutorials
permalink: /Tutorials/1_RandomGeneration
---

# Tutorial 1 - Random Generation on Button Press
{: .no_toc }

In this tutorial, we will learn how to unconditionaly generate a random drum loop on a button press.
{: .fs-6 .fw-300 }

{: .note }
> the source code for this tutorial is available in the tutorials branch of the repository.
> 
> [Tutorial 1 Source Code](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/tutorials/1_RandomGenOnButtonPress){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


## Model Description
The model we will be using in this exercise is a `VariationalAutoEncoder` (VAE) trained on the [Groove Midi Dataset](https://magenta.tensorflow.org/datasets/groove).

The model has been trained on 2-bar drum loops in 4/4 time signature. 

The model has already been serialized and is available at `TorchScripts/MDL/drumLoopVAE.pt`

### Input/Output Description
The input and output of the model are both 3 stacked tensors of shape (32, 9) where 32 is the number of 16th notes in a 2-bar segment and 9 is the number of drum instruments. 
The three stacked tensors are as follows:
- `hits`: A binary tensor indicating whether or not a drum instrument is hit at a given time step
- `velocities`: A tensor indicating the velocity of the drum instrument at a given time step
- `offsets`: A tensor indicating the offset of the drum instrument at a given time step

### Methods available 
The model has multiple methods available, some of which we will be using in this tutorial. The python
definitions of these models are as follows:

#### 1. encode 
This method encodes a given input pattern into a latent vector. 
The method returns a number of parameters, however the third parameter is the latent vector we are interested in. 

### 2. sample
This method decodes a given latent vector into an output tensor and returns a `hits`, `velocities`, and `offsets` tensors describing the output pattern.

The input to this method requires a number of additional parameters, however, for the purposes of this tutorial, 
we will not be discussing them.

---

## Plugin Development

### 1. Creating A Graphical User Interface (GUI)

As discussed in the [Graphical Interface]({{site.baseurl}}/docs/ParametersAndGUI) section of the documentation, 
to prepare the interface we will need to figure out what UI elements we need as well as how we want to organize them!

For this tutorial, all we need is a single button which will trigger the generation of a random pattern. As such, 
we will modify the `Configs_GUI.h` file as follows:

```c++
namespace Tabs {
        const bool show_grid = true;
        const bool draw_borders_for_components = true;
        const std::vector<tab_tuple> tabList{
            tab_tuple
            {
                "RandomGeneration", // use any name you want for the tab
                slider_list
                {
                },
                rotary_list
                {
                },
                button_list
                {
                    button_tuple{"Randomize", false, "Kh", "Pm"}
                }
            }
        };
    },
    
     namespace MidiInVisualizer {
        const bool enable = false;
        // ....
    }, 
    
    namespace GeneratedContentVisualizer
    {
        const bool enable = true;
        const bool allowToDragOutAsMidi = true;
    }
    
```
{: .note}
> 1. To be able to easily re-position the button on the Gui, Initially we will setting the `show_grid` and `draw_borders_for_components` to true.
> once we have positioned the button, we can set these to false to remove the grid and borders.
> 2. We don't need any drag in features for this tutorial, so we will disable the `MidiInVisualizer` tab.
> 3. We still want to visualize the generations and allow the user to drag them out as midi, so we will enable the `GeneratedContentVisualizer` tab.

Once rendered, we can see that the button is positioned in the bottom right corner of the GUI. However, 
