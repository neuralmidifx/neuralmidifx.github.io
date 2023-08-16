---
layout: default
title: Deployment Threads
nav_order: 8
has_children: true
permalink: /docs/DeploymentThreads
---

# Deployment Threads
{: .no_toc }

---

NeuralMidiFx dedicates three threads respectively for preparing the inputs of a model (`InputTensorPreparator` thread, a.k.a `ITP`), 
running inference (`Model` thread, a.k.a `MDL`), and reformatting the generations into MIDI messages for playback 
via the host  (`PlaybackPreparator` thread, a.k.a `PPP`).

In these threads, the wrapper provides a set of utilities for easily receiving and sending information from/to the 
threads via the previously implemented inter-thread communication pipelines. 

```mermaid
graph TD
    
    
    %% ITP Thread
    subgraph ITPThread["ITP Thread"]
        C1[Specify the information required from the host] --> C2[Implement the deployment process]
        C2 --> C3[Update ModelInput structure]
    end
    
    %% MDL Thread
    subgraph MDLThread["MDL Thread"]
        D1[Implement the deployment process] --> D2[Update ModelOutput structure]
    end
    
    %% PPP Thread
    subgraph PPPThread["PPP Thread"]
        E1[Implement the deployment process] --> E2[Specify the PlaybackPolicy]
        E2 --> E3[Specify the PlaybackSequence]
    end
    
    %% DeploymentThreads 
    subgraph DeploymentThreads["Deployment Threads"]
    end
    
    %% Vertical Placement of Groups
    DeploymentThreads -.-> ITPThread
    DeploymentThreads -.-> MDLThread
    DeploymentThreads -.-> PPPThread
    
    ITPThread -->|model_input| MDLThread
    MDLThread -->|model_output| PPPThread
```