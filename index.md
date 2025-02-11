---
layout: default
title: Home
nav_order: 1
description: "A Wrapper/Template for Deploying Generative Neural Network Models of Symbolic Music As VST Plugins"
permalink: /
---

# NeuralMidiFx
{: .fs-9 }

---
## About
Welcome to the documentation for NeuralMidiFx [^1], a wrapper/template designed to simplify the deployment of AI-based generative music systems as VST3 plugins. Our goal with NeuralMidiFx is to bridge the gap between AI research and end-users who wish to harness them for performance or composition.

Many researchers find themselves challenged by the technical complexity and costs associated with deploying generative music systems within traditional production and composition workflows.

The complexity and resources often required for deploying these systems in meaningful and useful ways have left many unable to bridge this divide. Unfortunately, these barriers have hindered both the evaluation of these systems and the broader engagement with them, limiting their potential impact and accessibility.

That's where NeuralMidiFx comes in. This wrapper/template is our effort to make the deployment of neural network-based symbolic music generation systems as VST3 plugins a bit more approachable. We've aimed to design it in such a way that even those with minimal familiarity with plugin development can navigate the process.

We hope that NeuralMidiFx can serve as a helpful tool for those looking to explore the potential of AI in music without getting entangled in the technical details. We acknowledge that it's just one step towards making these exciting technologies more accessible, and we're eager to learn from your experiences and feedback as we continue to refine and improve this tool. 

{: note }
See this [video](https://www.youtube.com/watch?v=xcq-VWo0Y6U) for more information about the motivation behind this project.

<iframe width="560" height="315" src="https://www.youtube.com/embed/xcq-VWo0Y6U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## Citing NeuralMidiFx

If you use NeuralMidiFx in your research, please cite it as follows:

```bibtex
@article{HakiNeuralMidiFx,
	author = {Haki, Behzad and Lenz, Julian and Jorda, Sergi},
	journal = {AIMC 2023},
	note = {https://aimc2023.pubpub.org/pub/givwzz98},
	title = {NeuralMidiFx: A {Wrapper} {Template} for {Deploying} {Neural} {Networks} as {VST3} {Plugins}},
}
```

## Source Code

[View it on GitHub][repo]{: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

{: .warning }
> Please note that this project is still in development and is subject to change. We will be updating the documentation as we continue to refine the wrapper/template.


## System Requirements and Compatibility
The wrapper is compatible with the following operating systems:

- Windows 
- macOS (Intel Only - Arm support coming soon)

{: .warning }
>No CUDA support is provided at the moment. We will be looking into this in the future.


## Acknowledgements


## About the project

NeuralMidiFx is &copy; 2022-{{ "now" | date: "%Y" }} by [Behzad Haki](http://behzadhaki.com).

### License

### Contributing

---

[Next: Overview & Architecture]({{site.baseurl}}/docs/2_Overview){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

----

[1^]: Haki B, Lenz J, Jorda S. NeuralMidiFx: A Wrapper Template for Deploying Neural Networks as VST3 Plugins. AIMC 2023 . Available from: https://aimc2023.pubpub.org/pub/givwzz98

[repo]: https://github.com/behzadhaki/NeuralMidiFXPlugin
