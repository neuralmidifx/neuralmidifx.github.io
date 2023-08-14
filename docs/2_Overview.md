---
layout: default
title: Overview and Architecture
nav_order: 2
---

# Overview & Architecture
{: .no_toc }

Explain what this section is about
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

[View it on GitHub][repo]{: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

--- 

## Purpose and Overview of the Wrapper

This guide is designed to assist researchers in deploying generative neural network models of symbolic music as VST3 plugins. Utilizing this wrapper, you can easily integrate your research models into the VST3 framework without needing expertise in VST development.

{: .note }
> **Target Audience:** The primary audience for this guide includes researchers and practitioners working in the field of generative symbolic music, particularly those focusing on neural network-based models.

## Your Responsibilities vs. the Wrapper's Responsibilities
The proposed wrapper, NeuralMidiFx, fully automates the deployment processes that are specific to plugin development frameworks. However, tasks related to the model are not fully automated as they are highly dependent on the specific task at hand. Nonetheless, knowing that the stages involved in the inference process can be distinctly divided into three separate procedures (Input Preparation, Model Inference, and Output Preparation/Post-Processing), NeuralMidiFx provides dedicated threads for each of these procedures, with specific access points made available to the researcher. This allows for greater control and customization of the model-related tasks, while still providing an easy and streamlined process for plugin development.

![img.png](/assets/images/responsibilities.png)

## How the Wrapper Facilitates VST3 Plugin Development

With this wrapper, you can:

- Integrate serialized [PyTorch](https://pytorch.org/cppdocs/) neural network models with ease.
- Customize the user interface for control and interaction.
- Implement/optimize real-time processing and playback.

## Architecture
![img.png](/assets/images/architecture.png)
---

[Previous: Home]({{site.baseurl}}/){: .btn .fs-5 .mb-4 .mb-md-0 }
[Next: Installation]({{site.baseurl}}/docs/3_Installation){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

--- 
[Return to main website]({{site.baseurl}}/).
---

[repo]: https://github.com/behzadhaki/NeuralMidiFXPlugin
