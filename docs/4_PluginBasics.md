---
layout: default
title: Plugin Basics
nav_order: 4
---

# Plugin Basics
{: .no_toc }

In this section, we will provide a brief guide on general VST plugin architecture and how a plugin interacts with the host
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

[Get started now](#getting-started){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

[View it on GitHub][Just the Docs repo]{: .btn .fs-5 .mb-4 .mb-md-0 }

---

## Getting Started
{: #getting-started }

Virtual Studio Technology (VST) is a software interface that integrates software audio synthesizers and effect plugins with audio editors and recording systems.

### Plugin Architecture

#### Components
A VST plugin generally consists of the following components:

- **Audio Processor:** This handles all the audio processing tasks, such as synthesizing audio signals or applying audio effects.
- **User Interface (UI):** This is the graphical representation that allows users to interact with the plugin.
- **Parameters:** These are the variables that can be manipulated by the user or the host to alter the behavior of the plugin.

#### Formats
VST plugins come in two primary formats:

- **VST2:** The older format, widely supported but now deprecated.
- **VST3:** The latest version, with improved performance and functionality.

### Interaction with the Host

VST plugins operate within a host application. Here's how they interact:

1. **Loading:** The host loads the plugin and initializes it.
2. **Processing:** The host sends audio data to the plugin, which processes it and returns the processed audio.
3. **Parameter Control:** The host can send and receive parameter data to and from the plugin to control its behavior.
4. **Saving and Loading Presets:** Users can save and load presets, either through the host or the plugin's UI.

### Building Your First VST Plugin

Building a VST plugin requires a combination of audio programming skills and knowledge of the VST architecture.

1. **Choose a Framework:** Popular frameworks like JUCE or iPlug2 can simplify the development process.
2. **Design the Audio Processor:** Implement the desired audio processing functionality.
3. **Create the UI:** Design the graphical user interface for user interaction.
4. **Test in Various Hosts:** Ensure compatibility with different host applications.

[Learn More about VST Development](https://www.yourlink.com){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

[Source Code on GitHub][Your GitHub Repo]{: .btn .fs-5 .mb-4 .mb-md-0 }

