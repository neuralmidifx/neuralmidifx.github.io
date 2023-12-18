---
layout: default
title: DPLData
nav_order: 2
has_children: false
parent: Data Types

permalink: /docs/V2_0_1/datatypes/DPLData
---

# Customizable Data Types
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Customizable Data for use within `DPL` Thread

If you need any data to be reused within the deploy method of each thread, you can add them to any of the 
`DPLData` structs in the `DeploymentData.h` file.

The content of this struct can be accessed from within the `deploy` method of each thread using the `DPLdata` instance.


{: .note }
> If you need to use a custom torch script in the threads for any reason,
> you can add the following variable to these structs:
> ```c++
> 
>   // assuming that you have your script in a file called "my_script.pt" located 
>   // in the `TorchScripts/ProcessingScripts` folder
>   
>   struct DPLData {
> 
>      // ... some code here
> 
>     torch::jit::script::Module my_script = load_processing_script("my_script.pt");
> 
>   };
> '''