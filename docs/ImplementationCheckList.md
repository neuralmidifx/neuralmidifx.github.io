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


- [ ] Add Your Model
  - [ ] Serialize your PyTorch model
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

    subgraph YourModel
        A1 --> A2
        A2 --> A3
    end

    subgraph Parameters
        B1 --> B2
        B2 --> B3
    end

    %% Vertical Placement of Groups
    YourModel --> Parameters
    Parameters --> ITPThread
```
