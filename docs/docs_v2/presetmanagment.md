---
layout: default
title: 4. Preset Management

permalink: /docs/V2_1_0/preset_management
nav_order: 75
---

# Preset Management
{: .no_toc }

Starting from version 2.1.0, NeuralMidiFXPlugin supports saving/loading presets. 

From this version on, you can save/load the UI values of the plugin as well as any tensors you want. There
are 100 preset slots available by default. These presets automatically store/load the UI values whenever needed.
You have full control through the `settings.json` file to specify which UI parameters should (or not) be saved
in the presets. Moreover, you have dedicated methods to save/load tensors alongside the UI values.

{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Preset Panel

On the left of the plugin UI, you will find the preset panel. This panel allows you to save/load presets.

here, if a given preset is not-empty, you will be able to load it by clicking on the name of the preset.

If you want to overwrite, rename, or save a new preset, you can do so by renaming a selected preset, then 
clicking on the save button.

![Preset Panel](/assets/images/preset_panel.PNG)

## Adding UI Parameters to the Preset

This is done by default, however, in many cases, you may want to exclude some parameters from the preset.

While all parameters are saved by default, you can make sure that when a preset is loaded, specific parameters
available in the preset data are not loaded. This can be done simply by using the "exclude_**" fields available 
in the `settings.json` file. For more information about these tags, refer to [2. Parameters and GUI]({{ site.baseurl }}/docs/V2_1_0/ParametersAndGUI) 
in the documentation.

## Tracking Tensors for Saving Presets

Any model related data that is not associated with a graphical element can be saved in the presets as well. 
The only restriction is that the data must be / or wrapped in a `torch::Tensor` object.

To learn how to track tensors for saving presets, refer to [Data Types/CustomPresetDataDictionary]({{ site.baseurl }}/docs/V2_1_0/datatypes/CustomPresetDataDictionary) 
in the documentation.

## Accessing Tensors from a Loaded Preset

Whenever a new preset is loaded, you will be notified via the `new_preset_loaded_since_last_call` flag in the `deploy()` method.

To learn how to read tensors from saved presets, refer to [Data Types/CustomPresetDataDictionary]({{ site.baseurl }}/docs/V2_1_0/datatypes/CustomPresetDataDictionary) 
in the documentation.