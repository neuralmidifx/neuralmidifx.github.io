---
layout: default
title: Data Flow
parent: Guides & References
grand_parent: V1.0.0 Documentation
permalink: /docs/v1_0_0/GuidesReferences/DataFlow
nav_order: 2
---

# Deployment Threads
{: .no_toc }
---

NeuralMidiFx dedicates three threads respectively for preparing the inputs of a model [`InputTensorPreparator` thread, a.k.a `ITP`]({{ site.baseurl }}/docs/v1_0_0DeploymentThreads/ITP),
running inference [`Model` thread, a.k.a `MDL`]({{ site.baseurl }}/docs/v1_0_0DeploymentThreads/MDL), and reformatting the generations into MIDI messages for playback 
via the host  [`PlaybackPreparator` thread, a.k.a `PPP`]({{ site.baseurl }}/docs/v1_0_0DeploymentThreads/PPP).

In these threads, the wrapper provides a set of utilities for easily receiving and sending information from/to the 
threads via the previously implemented inter-thread communication pipelines.

<object data="https://neuralmidifx.github.io/assets/quickguide.pdf" width="1000" height="1000" type='application/pdf'></object>
