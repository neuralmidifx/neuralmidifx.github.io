---
layout: default
title: Installation
permalink: /docs/Installation
nav_order: 30
---

# Installation
{: .no_toc }

Explain what this section is about
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Step 1: Cloning the Repository

The first step is to clone the repository.
    
```bash
git clone https://github.com/behzadhaki/NeuralMidiFXPlugin.git
```

## Step 2: Edit Plugin Name and Description

First, open the `CMakeLists.txt` file at [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/dev/windows/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt)

Modify the following lines with the description of your plugin:

```cmake
set (BaseTargetName NeuralMidiFXPlugin)

...

juce_add_plugin("${NeuralMidiFXPlugin}"
        COMPANY_NAME "MyCompanyTBD"                 # Replace with a tag identifying your name
        ....
        PLUGIN_CODE NMFx                             # MUST BE UNIQUE!! If similar to other plugins, conflicts will occur
        PRODUCT_NAME "NeuralMidiFXPlugin")           # Replace with your plugin title
```

## Step 3: Install CMake

CMake is a cross-platform tool that allows you to build, test and package software.
You can download the latest version of CMake from [here](https://cmake.org/download/).

## Step 4: Install an IDE 

I highly recommend using [CLion](https://www.jetbrains.com/clion/) as your IDE. 
The project has been developed and tested using CLion.

## Step 5. Configure the Project Settings 
If you decide to use CLion, go ahead with the following step. 
Otherwise, you will need to configure your IDE manually to the provided settings

Go to Settings -> Build, Execution, Deployment -> CMake and add the following to the CMake options:

![](/assets/images/cmake_settings.png)

{: .note}
> When debugging set it to `Debug` and when building the plugin set it to `Release`.
> that said, watch out for the following warning

{: .warning}
> In windows, when torch is built in debug mode, it will not allow for loading serialized models.
> Therefore, you will need to build torch in release mode (Exactly as the figure above)


## Step 6. Setting up the Run Configuration
On the top right corner of CLion, click on `Edit Configurations...`

![](/assets/images/run_config1.png)

Make sure the settings are as follows:

![](/assets/images/run_config2.png)

## Step 7. Attaching host to the IDE
You can debug the plugin while running in a host. This is extremely useful for debugging and testing purposes.
To do so, go back to the `Edit Configurations...`. Then, click on `Executable > Custom Executable` and select
the host executable (see Example below).

![](/assets/images/run_cofig3.png)