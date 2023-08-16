---
layout: default
title: Importing Your Serialized Models
parent: Model Preparation
nav_order: 2
---

# Importing Your Serialized Models
{: .no_toc }

In this guide, we will show you the steps necessary for import your **serialized** model into the plugin.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Getting started

Once you have your model serialized, you can import it into the plugin. 
The way the wrapper works is that whenever the cmake project is built, it will copy the [`TorchScripts`](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/master/TorchScripts/) 
folder to a local directory:

{: .note }
> On Windows, the content will be cloned into `C:\{BaseTargetName}\TorchScripts`.
> 
> On Mac, the content will be cloned into `~/Library/{BaseTargetName}/TorchScripts`
> 
> !Remember! that `{BaseTargetName}` is the name of the target you have specified in the `CMakeLists.txt` file
> in [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/master/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt).