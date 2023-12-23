---
layout: default
title: Data Types 
nav_order: 90
has_children: true
permalink: /docs/V2_1_0/datatypes


---

# Data Types
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


## Description

There are a number of data types that you will be commonly using in the code. Some of these data types are available in 
all of the deployment threads and some are only available in specific threads. This page will give you an overview of
the data types. You can find more information about each data type in the corresponding subsections.

## Customizable Data Types
### Data Types that You Can Modify As You Wish

| Data Type | Description                                                                       | API                                                                       |
|-----------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| CustomPresetDataDictionary   | A structure holding any required torch tensor to be stored alongside a preset | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/CustomPresetDataDictionary) |

### Data Types for Interacting with the GUI Midi and Audio Visualizers

| Data Type | Description                                  | API                                             |
|-----------|----------------------------------------------|-------------------------------------------------|
|MidiVisualizersData| A structure attached to all midi visualizers | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/MidiVisualizersData)                      |
|AudioVisualizersData| A structure attached to all audio visualizers | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/AudioVisualizersData)                      |

## Non-Customizable Data Types
### Data Types Available in All Threads

| Data Type           | Description                                                                               | API                                          |
|---------------------|--------------------------------------------------------------------------------------------|----------------------------------------------|
| GuiParams           | Contains the status and value of the specified daw parameters                              | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/GuiParams) |
| realtimePlaybackInfo| Contains the status of the daw in real time                                                | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/RealtimePlaybackInfo)                     |


| Data Type           | Description                                                                                | API                                             |
|---------------------|--------------------------------------------------------------------------------------------|-------------------------------------------------|
| EventFromHost       | All per-buffer data received from host are wrapped in this datatype for easy access         | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/EventFromHost) |
| MidiFileEvent       | Contains the information in a given midi file manually drag/dropped into GUI               | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/MidiFileEvent)                        |
| PlaybackSequence    | Contains the generated data to be played back or visualized                                | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/PlaybackSequence)                        |
| PlaybackPolicy      | Specifies how generated content sent to the wrapper are to be interpreted                  | [Here]({{site.baseurl}}/docs/V2_1_0/datatypes/PlaybackPolicy)                        |

Please check the API links for a more detailed description of each common data type.

