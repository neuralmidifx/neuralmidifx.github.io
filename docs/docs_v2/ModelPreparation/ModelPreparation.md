---
layout: default
title: 1. Model Preparation
nav_order: 40
has_children: true
permalink: /docs/v2_0_0/ModelPreparation
parent: V2.0.0 Documentation
---

# Model Preparation
{: .no_toc }

In this section we will discuss the steps required to prepare a model for deployment. 
{: .fs-6 .fw-300 }
---

## Serialization of Trained PyTorch Models

We have prepared a guide on how to serialize trained PyTorch models [here]({{site.baseurl}}/docs/v2_0_0ModelPreparation/Serialization/).

{: .warning }
> At the moment, we only support PyTorch models. We are working on adding support for [ONNX](https://github.com/onnx/tutorials) models as well.

## Add a Serialized Model to the Plugin Project

Once you have serialized your model, you can add it to the plugin project using the guide [here]({{site.baseurl}}/docs/v2_0_0ModelPreparation/ImportingYourSerializedModels/).