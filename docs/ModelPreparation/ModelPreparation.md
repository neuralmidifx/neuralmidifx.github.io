---
layout: default
title: Model Preparation
nav_order: 6
has_children: true
permalink: /docs/ModelPreparation
---

# Model Preparation
{: .no_toc }

In this section we will discuss the steps required to prepare a model for deployment. 
{: .fs-6 .fw-300 }
---

## Serialization of Trained PyTorch Models

We have prepared a guide on how to serialize trained PyTorch models [here]({{site.baseurl}}/docs/ModelPreparation/Serialization/).

{: .warning }
> At the moment, we only support PyTorch models. We are working on adding support for [ONNX](https://github.com/onnx/tutorials) models as well.

## Add a Serialized Model to the Plugin Project

Once you have serialized your model, you can add it to the plugin project using the guide [here]({{site.baseurl}}/docs/ModelPreparation/ImportingYourSerializedModels/).