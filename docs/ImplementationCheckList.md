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

## Step 1 - Model Preparation
- [ ] [Model Preparation]({{ site.baseurl }}/docs/ModelPreparation)
  - [ ] [Serialize your PyTorch model]({{ site.baseurl }}/docs/ModelPreparation/Serialization)
  - [ ] [Place the model in "TorchScripts/MDL/" folder (within the project)]({{ site.baseurl }}/docs/ModelPreparation/ImportingYourSerializedModels/#step-1-add-your-serialized-model-to-the-project) 
  - [ ] [Reload Cmake Project (see note below)]({{ site.baseurl }}/docs/ModelPreparation/ImportingYourSerializedModels/#step-2-reload-the-cmake-project) 

```mermaid
graph TD

    %% Your Model
    subgraph ModelPreparation["Model Preparation"]
        A1[Serialize your PyTorch model] --> A2[Place the model in 'TorchScripts/MDL/' folder]
        A2 --> A3[Reload Cmake Project]
    end
```

## Step 2 - Parameters

- [ ] [Parameters]({{ site.baseurl }}/docs/GraphicalInterface)
  - [ ] [Identify the parameters]({{ site.baseurl }}/docs/GraphicalInterface/#step-1-available-user-interface-elements)
  - [ ] [Specify the GUI layout]({{ site.baseurl }}/docs/GraphicalInterface/#step-2-define-the-ui-elements)
  - [ ] Specify [MIDI In]({{ site.baseurl }}/docs/GraphicalInterface/#step-4-midi-in-widget) or [MIDI Out]({{ site.baseurl }}/docs/GraphicalInterface/#step-5-midi-out-widget) Visualizers (if any)

```mermaid
graph TD
 %% Parameters
    subgraph Parameters
        B1[Identify the parameters] --> B2[Specify the GUI layout]
        B2 --> B3[Specify MIDI Visualizers]
    end
```

# Step 3 - InputTensorPreparator Thread (ITP)
- [ ] ITP Thread
  - [ ] Specify the information required from the host
  - [ ] Implement the deployment process
  - [ ] Update ModelInput structure

```mermaid
graph TD
    %% ITP Thread
    subgraph ITPThread["ITP Thread"]
        C1[Specify the information required from the host] --> C2[Implement the deployment process]
        C2 --> C3[Update ModelInput structure]
    end
```

# Step 4 - Model Thread (MDL)
- [ ] MDL Thread
  - [ ] Implement the deployment process
  - [ ] Update ModelOutput structure

```mermaid
graph TD


    %% MDL Thread
    subgraph MDLThread["MDL Thread"]
        D1[Implement the deployment process] --> D2[Update ModelOutput structure]
    end
```

# Step 5 - PlaybackPreparator Thread (PPP)
- [ ] PPP Thread
  - [ ] Implement the deployment process
  - [ ] Specify the PlaybackPolicy
  - [ ] Specify the PlaybackSequence

```mermaid
graph TD

    %% PPP Thread
    subgraph PPPThread["PPP Thread"]
        E1[Implement the deployment process] --> E2[Specify the PlaybackPolicy]
        E2 --> E3[Specify the PlaybackSequence]
    end
```

```mermaid
graph TD

    %% PPP Thread
    subgraph PPPThread["PPP Thread"]
        E1[Implement the deployment process] --> E2[Specify the PlaybackPolicy]
        E2 --> E3[Specify the PlaybackSequence]
    end
```
