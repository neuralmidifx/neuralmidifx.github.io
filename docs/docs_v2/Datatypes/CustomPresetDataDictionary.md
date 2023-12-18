---
layout: default
title: CustomPresetDataDictionary
nav_order: 3
has_children: false
parent: Data Types
permalink: /docs/V2_1_0/datatypes/CustomPresetDataDictionary
---

# CustomPresetDataDictionary
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
There are 100 preset slots available by default. These presets automatically store/load the UI values. 

That said, you may want to store some tensors along side these UI values. 

As a result, you can use the `CustomPresetDataDictionary` struct to store any tensor data you want along side the UI values.

## Tracking Tensors for Saving Presets
To track the content of a given tensor that should be stored in a preset, do the following in the deploy() method

```c++
    auto some_tensor = torch::rand({1, 1, 1, 1});
    CustomPresetData->tensor("input_tensor", some_tensor);
    
    auto second_tensor = torch::rand({128});
    CustomPresetData->tensor("second_tensor", second_tensor);
```

Once a new preset is saved, the content of these tensors will be saved along side the UI values. 

## Accessing Tensors from a Loaded Preset
once a new preset is loaded, you will be notified via the `new_preset_loaded_since_last_call` flag. 

You can then access the content of the stored tensors as follows:

#### Approach 1: Iterate over all tensors
```c++
    if (new_preset_loaded_since_last_call) {
        for (const auto& pair : CustomPresetData->tensors()) {
            const std::string& key = pair.first;
            const torch::Tensor& t = pair.second;
        }
    }
```

#### Approach 2: Access a specific tensor using the specified key
```c++
    if (new_preset_loaded_since_last_call) {
       
        auto t = CustomPresetData->tensor(key);
        if (t != std::nullopt) {    <-- if key doesnt exist, it will return std::nullopt, so ALWAYS check for this
            auto value = *t;
        }

    }
```