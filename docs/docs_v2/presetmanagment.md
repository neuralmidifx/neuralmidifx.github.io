---
layout: default
title: 4. Preset Management

permalink: /docs/V2_1_0/preset_management
nav_order: 75
---

# Preset Management
{: .no_toc }

Starting from version 2.1.0, NeuralMidiFXPlugin supports saving/loading presets. 

From this version on, you can save/load the UI values of the plugin as well as any tensors you want. There
are 100 preset slots available by default. These presets automatically store/load the UI values whenever needed.
You have full control through the `settings.json` file to specify which UI parameters should (or not) be saved
in the presets. Moreover, you have dedicated methods to save/load tensors alongside the UI values.

{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 