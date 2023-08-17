---
layout: default
title: Deployment Threads
nav_order: 8
has_children: false
permalink: /docs/A0_DeploymentThreads
---

# Deployment Threads
{: .no_toc }

---

NeuralMidiFx dedicates three threads respectively for preparing the inputs of a model [`InputTensorPreparator` thread, a.k.a `ITP`]({{ site.baseurl }}/docs/DeploymentThreads/ITP),
running inference [`Model` thread, a.k.a `MDL`]({{ site.baseurl }}/docs/DeploymentThreads/MDL), and reformatting the generations into MIDI messages for playback 
via the host  [`PlaybackPreparator` thread, a.k.a `PPP`]({{ site.baseurl }}/docs/DeploymentThreads/PPP).

In these threads, the wrapper provides a set of utilities for easily receiving and sending information from/to the 
threads via the previously implemented inter-thread communication pipelines.

```mermaid
graph TD
    
    %% DAW
    DAW_2["Incoming Midi \n and \n DAW Status \n (Tempo, Meter, Position, ...)"]
    DAW_3["Parameters \n Edited, Audtomated \n via DAW/GUI"]
    DAW_5["MIDI Files \n Dragged/Dropped \n into Plugin"]
    GUI_MIDIIN["Visualize \n Incoming MIDI \n from Host \n on GUI"]
    GUI_MIDIIN2["Visualize \n Dropped MIDI \n on GUI"]
    

    NMFX1["NeuralMidiFx"]
    DAWOut["Play \n Generations \n via DAW"]
    GUI_MIDI["Drag Out \n Generations \n via GUI"]
    GUI_MIDI2["Visualization \n Generations \n on GUI"]
    NMFX2["NeuralMidiFx"]

    %% DeploymentThreads 
    subgraph DeploymentThreads[" "]

        subgraph X["Deployment Threads"]
            %% ITP Thread
            subgraph ITPThread["ITP_Deploy.cpp"]
                C1["Process/Tokenize \n Events and Parameters \n "] --> |"Ready For Inference?"|C2["Wrap in ModelInput\n and \n  Send to Next Thread"]
            
            end
            
            %% MDL Thread
            subgraph MDLThread["MDL Thread"]
                D1[Run Inference] --> D2["Wrap in ModelOutput\n and \n  Send to Next Thread"]
            end
            
            %% PPP Thread
            subgraph PPPThread["PPP Thread"]
                
                E1[Extract Generations]-->|append or overwrite \n with note information|E2[Update PlaybackSequence]
                E1[Extract Generations]-->|specify how to interpret \n the sequence |E3[Update PlaybackPolicy]
            end
        end
        subgraph gui_params["gui_params \n Available in \n ITP, MDL, PPP"]
            
        end
    
    end
        

   
    NMFX1 -.-> |GuiParams Structure| gui_params

    DAW_2 -.->|"Accessed  \n according to \n Configs_HostEvents.h "| NMFX1
    DAW_3 -.-> |"Access Info \n specified in \n Configs_HostEvents.h "|NMFX1
    DAW_5 -.-> |"(Optional) \n if specified in \n Configs_GUI.h"|NMFX1
    NMFX1 -.->|EventFromHost Structure| ITPThread
    NMFX1 -.->|MidiFileEvent Structure| ITPThread

    ITPThread -.->|ModelInput Structure \n Specified in Model_Input.h| MDLThread
    MDLThread -.->|ModelOutput Structure \n Specified in Model_Output.h| PPPThread
    PPPThread -.->|PlaybackPolicy, PlaybackSequence| NMFX2
    NMFX2 -.->|MIDI| DAWOut
    NMFX2 -.->|"(Optional) \n if specified in \n Configs_GUI.h"| GUI_MIDI
    NMFX2 -.->|"(Optional) \n if specified in \n Configs_GUI.h"| GUI_MIDI2
    NMFX2 -.->|"(Optional) \n if specified in \n Configs_GUI.h"| GUI_MIDIIN
    NMFX2 -.->|"(Optional) \n if specified in \n Configs_GUI.h"| GUI_MIDIIN2


    style DeploymentThreads fill:#ada1 
    style X fill:#ada1


```