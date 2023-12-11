---
layout: default
title: Importing Your Serialized Models
has_children: false
parent: 1. Model Preparation
grand_parent: V1.0.0 Documentation
permalink: /v1_0_0/docs/docs/ModelPreparation/ImportingYourModel/
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

## Step 1. Add your serialized model to the project

Once you have your model serialized, you should add it to [`TorchScripts/Models`](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/master/TorchScripts) 
folder in the **project directory**.


<img src="/assets/images/TorchScriptFolderStruct.png" width="200" alt="MDLFile">


## Step 2. Reload the cmake project
Once ready, the wrapper will create a local directories and populate them with the content of the project directory.
As a result, once you have your model added to the project directory (as shown in the previous step)
you **MUST** reload the cmake project so as for the content to be copied to the local directory.


In Clion, you can reload your cmake project as shown in the image below:


<img src="/assets/images/cmake_reload.png" width="500" alt="CMAKE Reload Image">


{: .note }
> On Windows, the content will be cloned into `C:\{BaseTargetName}\TorchScripts`.
> 
> On Mac, the content will be cloned into `~/Library/{BaseTargetName}/TorchScripts`
> 
> !Remember! that `{BaseTargetName}` is the name of the target you have specified in the `CMakeLists.txt` file
> in [NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/master/NeuralMidiFXPlugin/NeuralMidiFXPlugin/CMakeLists.txt).
> 

{: .warning }
> For the sake of consistency, we recommend that you **NEVER** manually update the content of the above local directories created by cmake. 
> Rather, **ALWAYS** update the content of the [`TorchScripts`](https://github.com/behzadhaki/NeuralMidiFXPlugin/tree/master/TorchScripts/) folder in the `NeuralMidiFXPlugin` project and reload the cmake project.


