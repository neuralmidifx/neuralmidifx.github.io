---
layout: default
title: ModelInput & ModelOutput
nav_order: 5
has_children: false
parent: Data Types
permalink: /datatypes/ModelInputMidiOutput
---

# ModelInput & ModelOutput
{: .no_toc }

{:toc}

---

## ModelInput & ModelOutput

`ModelInput` is a container (struct) for sending data from `ITP` to `MDL` thread.

`ModelOutput` is a container (struct) for sending data from `MDL` to `PPP` thread.

### Modifying the Structures

{: .note }
> For this Stage of Deployment, You should modify the following files
> 
> [Model_Input.h](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/master/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Model_Input.h){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
>
> [Model_Output.h](https://github.com/behzadhaki/NeuralMidiFXPlugin/blob/master/NeuralMidiFXPlugin/NeuralMidiFXPlugin/Model_Output.h){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
>

You can modify the structures as you wish by adding or removing fields. 
However, you must make sure that the `timer` field is not modified.

#### Example

Here I have added a `double` and a `torch::Tensor` to the `ModelInput` struct.

```c++
struct ModelInput {
    torch::Tensor tensor1{};
    double someDouble{};

    // ==============================================
    // Don't Change Anything in the following section
    // ==============================================
    // used to measure the time it takes
    // for a prepared instance to be
    // sent to the next thread
    chrono_timer timer{};
};
```

### Using `ModelInput`

In `ITP` and `MDL` Threads, `ModelInput` is already instantiate as `model_input`.

In `ITP` you can modify the `model_input` as follows:

```c++
    // ... some code here
    model_input.someDouble = 0.5;
    model_input.tensor1 = torch::randn({1, 3, 224, 224});
    // ... some code here
``` 

Whenever ready, a copy of the `model_input` can be sent to `MDL` thread as soon as a `true` boolean is returned from 
[ITP::deploy(...)]({{ site.baseurl }}/DeploymentStages/ITP/Deploy) method.

In `MDL` you can access the `model_input` as follows:

```c++
    // ... some code here
    double someDouble = model_input.someDouble;
    torch::Tensor tensor1 = model_input.tensor1;
    // ... some code here
```

### Using `ModelOutput`

In `MDL` and `PPP` Threads, `ModelOutput` is already instantiate as `model_output`.

In `MDL` you can modify the `model_output` as follows:

```c++
    // ... some code here
    model_output.GeneratedTensor = torch::randn({1, 3, 224, 224});
    // ... some code here
```

Whenever ready, a copy of the `model_output` can be sent to `PPP` thread as soon as a `true` boolean is returned from
[MDL::deploy(...)]({{ site.baseurl }}/DeploymentStages/MDL/Deploy) method.

In `PPP` you can access the `model_output` as follows:

```c++
    // ... some code here
    torch::Tensor GeneratedTensor = model_output.GeneratedTensor;
    // ... some code here
```


