---
layout: default
title: 3. Deployment Thread (DPL)
has_children: true
nav_order: 60
permalink: /docs/v2_0_0/DeploymentThread_DPL
---

# Deployment Thread (DPL)
{: .no_toc }

{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Deployment Thread (DPL)
This thread is responsible for the deployment process. In here, you will be preparing the inputs to the model, 
run inference, and post-process the outputs. 

The deployment will be done in the `deploy` method of the `DPL` class, which is located in the `Deployment/Deploy.cpp` file.
This method does not run all the time, but rather, it is only triggered whenever any status of the plugin that may
impact the generation process changes. To be more specific, the `deploy` method is triggered whenever any of the following
changes:
- An [EventFromHost]({{ site.baseurl }}/docs/v2_0_0/datatypes/EventFromHost) is received from the host
- A [GUI Parameter]({{ site.baseurl }}/docs/v2_0_0/datatypes/GuiParams) is changed
- An [Audio file]({{ site.baseurl }}/docs/v2_0_0/datatypes/AudioVisualizersData) or 
[MIDI file]({{ site.baseurl/docs/v2_0_0/datatypes/MidiVisualizersData }}) is dropped on any of the plugin's 
[AudiVisualizers]({{ site.baseurl }}/docs/v2_0_0/ParametersAndGUI#audio-visualizer) or 
[MidiVisualizers]({{ site.baseurl }}/docs/v2_0_0/ParametersAndGUI#midi-visualizer) respectively.

Once you process the above information, and subsequently, run inference, you will need to extract the outputs from the model
and specify what sequence and how it should be played back. This is done by wrapping these information in 
[PlaybackSequence]({{ site.baseurl }}/docs/v2_0_0/datatypes/PlaybackSequence) and [PlaybackPolicy]({{ site.baseurl }}/docs/v2_0_0/datatypes/PlaybackPolicy)
respectively.

<object data="https://neuralmidifx.github.io/assets/quickGuide - v2.pdf" width="1000" height="1000" type='application/pdf'></object>

An overview of the "deploy" method is available [here]({{ site.baseurl }}/docs/v2_0_0/DeploymentThread_DPL/deploy_method).