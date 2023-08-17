---
layout: default
title: GuiParams
nav_order: 1
has_children: false
parent: Data Types
permalink: /datatypes/GuiParams
---

# GuiParams
{: .no_toc }

---

# Accessing the UI Parameters
The UI parameters can be accessed in any thread using the following methods. To access the UI parameters, 
all you need to reference the element is the name you gave it in the previous step.

```c++
    // =================================================================================
    // ===         1.a. ACCESSING GUI PARAMETERS
    // =================================================================================
    /*
    If you need to access information from the GUI, you can do so by using the
    following methods:
    
           Rotary/Sliders: gui_params.getValueFor([slider/button name])
           Toggle Buttons: gui_params.isToggleButtonOn([button name])
           Trigger Buttons: gui_params.wasButtonClicked([button name])

    If you only need this data when the GUI parameters CHANGE, you can use the
           provided gui_params_changed_since_last_call flag 
    */
    
    auto Slider1 = gui_params.getValueFor("Slider 1");
    auto ToggleButton1 = gui_params.isToggleButtonOn("ToggleButton 1");
    auto ButtonTrigger = gui_params.wasButtonClicked("TriggerButton 1");
    
    // Or get value only when a change occurs
    if (gui_params_changed_since_last_call) {
        auto Slider1 = gui_params.getValueFor("Slider 1");
        auto ToggleButton1 = gui_params.isToggleButtonOn("ToggleButton 1");
        auto ButtonTrigger = gui_params.wasButtonClicked("TriggerButton 1");
    }
```