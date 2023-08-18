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

If you need to access information from the GUI, you can do so by using the
following methods:

```c++
    Rotary/Sliders: gui_params.getValueFor([slider/rotary name])
    Toggle Buttons: gui_params.isToggleButtonOn([button name])
    Trigger Buttons: gui_params.wasButtonClicked([button name])

e.g.:
    auto Slider1 = gui_params.getValueFor("Slider 1");
    auto ToggleButton1 = gui_params.isToggleButtonOn("ToggleButton 1");
    auto ButtonTrigger = gui_params.wasButtonClicked("TriggerButton 1");
```

If you only need this data when the GUI parameters CHANGE, you can use the
provided gui_params_changed_since_last_call flag 

```c++
    if (gui_params_changed_since_last_call) {
        auto Slider1 = gui_params.getValueFor("Slider 1");
        auto ToggleButton1 = gui_params.isToggleButtonOn("ToggleButton 1");
        auto ButtonTrigger = gui_params.wasButtonClicked("TriggerButton 1");
    }
```

{: .warning }
> You should be careful about the following:
> 
> 1. The name of the parameter is **case sensitive**
>
> 2. The name of the parameter is the same as the name you gave it in the GUI
> 
> 3. You should use the right method for the right type of parameter (slider, toggle button, trigger button). 
> That is, you should use `getValueFor` for sliders/rotaries and `isToggleButtonOn` for toggle buttons, and `wasButtonClicked` for trigger buttons.
