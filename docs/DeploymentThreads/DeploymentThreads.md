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

Deployment threads are the threads for which you are partially responsible.

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
    "Deployment Threads" --> "ITP Thread"
    "Deployment Threads" --> "MDL Thread"
    "Deployment Threads" --> "PPP Thread"
```