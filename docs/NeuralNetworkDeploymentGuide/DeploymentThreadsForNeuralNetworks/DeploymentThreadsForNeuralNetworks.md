
---
layout: default
title: Deployment Threads for Neural Networks
nav_order: 30
has_children: true
permalink: /docs/NeuralNetworkDeploymentGuide/DeploymentThreadsForNeuralNetworks
---

# Deployment Threads for Neural Networks

This section delves into the threading architecture designed to manage various parallel processing tasks within the plugin. The deployment threads include:

## Input Preprocessing Threads

Responsible for preparing the input tensors required for the neural network model.

- **Purpose and Functionality**: Conversion of raw MIDI data into a format suitable for the neural network.
- **Input and Output**: Handles input data and prepares output tensors.
- **Interaction with Other Components**: Coordinates with other threads and components.

## Neural Network Model Threads

Manages the execution of the neural network model.

- **Model Loading**: Load pre-trained models for inference.
- **Inference Process**: Perform real-time inference with the neural network.
- **Result Handling**: Process and forward the inference results.

## Post-processing and Playback Threads

Handles data post-processing and prepares it for playback.

- **Data Preparation**: Convert the model's output into a format suitable for playback.
- **Playback Configuration**: Configure playback settings and controls.
- **Real-time Processing**: Ensure smooth real-time processing and playback.

## Customization and Extending Threads

Provides guidance on customizing and extending thread functionalities to match specific research needs.

- **Creating Custom Threads**: Instructions to create custom threads for specific tasks.
- **Integrating with Existing Threads**: Guidance on integrating custom functionalities.

[Next: Drag-and-Drop Features](/docs/NeuralNetworkDeploymentGuide/DragAndDropFeatures)

