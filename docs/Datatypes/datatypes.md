---
layout: default
title: Data Types 
nav_order: 90
has_children: true
permalink: /datatypes
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

| Data Type           | Description                                                      | Usage/Availability | API                                             |
|---------------------|------------------------------------------------------------------|--------------------|-------------------------------------------------|
| ModelInput          | A structure holding any required data to be sent from ITP to MDL | ITP and MDL        | [Here]({{site.baseurl}}/datatypes/CustomizableDataTypes)                        |
| ModelOutput         | A structure holding any required data to be sent from MDL to PPP | MDL and PPP        | [Here]({{site.baseurl}}/datatypes/CustomizableDataTypes)                        |
| ITPData             | A structure holding any required data to be used in ITP Thread   | ITP Only           | [Here]({{site.baseurl}}/datatypes/CustomizableDataTypes)                        |
|MDLData             | A structure holding any required data to be used in MDL Thread   | MDL Only           | [Here]({{site.baseurl}}/datatypes/CustomizableDataTypes)                        |
|PPPData             | A structure holding any required data to be used in PPP Thread   | PPP Only           | [Here]({{site.baseurl}}/datatypes/CustomizableDataTypes)                        |

## Non-Customizable Data Types
### Data Types Available in All Threads

| Data Type           | Description                                                                                | Usage/Availability      | API                                          |
|---------------------|--------------------------------------------------------------------------------------------|-------------------------|----------------------------------------------|
| GuiParams           | Contains the status and value of the specified daw parameters                              | ITP, MDL, PPP           | [Here]({{site.baseurl}}/datatypes/GuiParams) |
| realtimePlaybackInfo| Contains the status of the daw in real time                                                | ITP, MDL, PPP           | [Here]({{site.baseurl}}/datatypes/RealtimePlaybackInfo)                     |

### Data Types Available in Specific Threads

| Data Type           | Description                                                                                | Usage/Availability      | API                                             |
|---------------------|--------------------------------------------------------------------------------------------|-------------------------|-------------------------------------------------|
| EventFromHost       | All per-buffer data received from host are wrapped in this datatype for easy access         | ITP Only                | [Here]({{site.baseurl}}/datatypes/EventFromHost) |
| MidiFileEvent       | Contains the information in a given midi file manually drag/dropped into GUI               | ITP Only                | [Here]({{site.baseurl}}/datatypes/MidiFileEvent)                        |
| PlaybackSequence    | Contains the generated data to be played back or visualized                                | PPP Only                | [Here]({{site.baseurl}}/datatypes/PlaybackSequence)                        |
| PlaybackPolicy      | Specifies how generated content sent to the wrapper are to be interpreted                  | PPP Only                | [Here]({{site.baseurl}}/datatypes/PlaybackPolicy)                        |

Please check the API links for a more detailed description of each common data type.

