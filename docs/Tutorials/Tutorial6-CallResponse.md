---
layout: default
title: 6 - Call and Response
nav_order: 6
has_children: false
parent: Tutorials
permalink: /Tutorials/6_CallResponse
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Tutorial 6 - Call and Response
{: .no_toc }

{: .warning }
> In this tutorial, we will be using the same model as the previous tutorials, so make sure you have completed the previous tutorials.
> Also, in this excercise, we will focus on a single thread implementation (as explained in [Tutorial 5](https://neuralmidifx.github.io/docs/Tutorials/5_SingleThreadImplementation)),
> 

{: .note }
> the source code for this tutorial is available in the tutorials branch of the repository.
> 
> [Tutorial 6 Source Code](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/tutorials/6_CallResponse){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


## Description

The objective here is to build a call and response plugin with the following assumptions:

1. The user starts playing a sequence of notes
2. Once the user stops playing for 2 seconds, we move on to generate a response
3. To generate the response:
   1. we will chop up the performance into non-overlapping 2 bar chunks
   2. Then, we will generate a drum pattern for each 2bar chunk
   3. Finally, we will concatenate the generated drum patterns to create the response

4. If the user starts playing again, we will stop playback of generations immediately and wait for the user to stop playing again

## Plugin Name and Description
As mentioned [here](https://neuralmidifx.github.io/docs/Installation#step-2-edit-plugin-name-and-description), we need
to specify the name of the plugin as well as some descriptions for it. 

To do this, we will modify the [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/tutorials/6_CallResponse/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt) file as follows:

```cmake
project(Tutorial6NMFx VERSION 0.0.1)

set (BaseTargetName Tutorial6NMFx)

....

juce_add_plugin("${BaseTargetName}"
        COMPANY_NAME "AIMCTutorials"                
        ... 
        PLUGIN_CODE AZCP                # a unique 4 character code for your plugin                          
        ...
        PRODUCT_NAME "Tutorial6NMFx")           # Replace with your plugin title

```


Once you re-build the cmake project, and re-build the plugin, you should see the name of the plugin change in the DAW:

<img src="{{ site.baseurl }}/assets/images/tutorial6/0.PNG">

Now we are ready to move on to the next step.
