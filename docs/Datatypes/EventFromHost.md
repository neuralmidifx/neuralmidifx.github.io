---
layout: default
title: EventFromHost
nav_order: 3
has_children: false
parent: Data Types
permalink: /datatypes/EventFromHost
---

# EventFromHost
{: .no_toc }

---

# Description

As mentioned before the data received from the host are provided in a per-buffer basis.
For each buffer multiple information can be available:

1. DAW Status at the start of the buffer: Play, Stop, Record, Loop, Tempo, Time Signature, Buffer Position
2. MIDI Events within the buffer: Note On, Note Off ...

These data received from the host are wrapped in the `EventFromHost` data type and are provided to you in the `ITP` thread.

# Step 1. Identify the type of event

There are many different types available in the `EventFromHost` data type. 






{: .note}
> As mentioned before, you can specify which of these events you want to receive in the `ITP` thread 
> 
> To do so, you need to specify the [`Configs_HostEvents.h`](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/master/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Configs_HostEvents.h)
> file. To learn more about this process, visit [this page](({{site.baseurl}}/DeploymentStages/ITP/HostEvents)
