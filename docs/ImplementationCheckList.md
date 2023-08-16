---
layout: default
title: Implementation Check List
nav_order: 5
---

# Implementation Check List
{: .no_toc }

Use this checklist to make sure that you have implemented all the necessary steps to deploy your model!

{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


- [ ] [Model Preparation]({{ site.baseurl }}{% link docs/ModelPreparation.md %})
  - [ ] [Serialize your PyTorch model]({{ site.baseurl }}{% link docs/ModelPreparation/Serialization.md %})
  - [ ] Place the model in "TorchScripts/MDL/" folder (within the project) 
  - [ ] Reload Cmake Project (see note below)
- [ ] Parameters
  - [ ] Identify the parameters
  - [ ] Specify the GUI layout
  - [ ] Specify MIDI Visualizers (if any)
- [ ] ITP Thread
  - [ ] Specify the information required from the host
  - [ ] Implement the deployment process
  - [ ] Update ModelInput structure
- [ ] MDL Thread
  - [ ] Implement the deployment process
  - [ ] Update ModelOutput structure 
- [ ] PPP Thread
  - [ ] Implement the deployment process
  - [ ] Specify the PlaybackPolicy
  - [ ] Specify the PlaybackSequence

{: .note}
> Anytime you update the content of the "TorchScripts" folder, you need to reload the CMAKE project.
> 
> <img src="/assets/images/cmake_reload.png" width="500" alt="CMAKE Reload Image">


```mermaid
graph TD

    %% Your Model
    subgraph ModelPreparation["Model Preparation"]
        A1[Serialize your PyTorch model] --> A2[Place the model in 'TorchScripts/MDL/' folder]
        A2 --> A3[Reload Cmake Project]
    end

    %% Parameters
    subgraph Parameters
        B1[Identify the parameters] --> B2[Specify the GUI layout]
        B2 --> B3[Specify MIDI Visualizers]
    end

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

    %% Vertical Placement of Groups
    ModelPreparation --> Parameters
    Parameters --> ITPThread
    ITPThread --> MDLThread
    MDLThread --> PPPThread

```
