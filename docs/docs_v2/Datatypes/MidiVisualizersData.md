---
layout: default
title: MidiVisualizersData
nav_order: 4
has_children: false
parent: Data Types
permalink: /docs/V2_0_1/datatypes/MidiVisualizersData
---

# MidiVisualizersData
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Description

The content of MidiVisualizers placed in the tabs will be modifiable/accessible in the `deploy()` method using
the `midiVisualizersData` instance.

You can update the content whenever the midi sequence changes. Moreover, when the user drops a midi file into the
widget, the content of the midi visualizers will be provided to you in the `deploy()` method.

### Accessing the content of a dropped midi file
The content of the dropped midi file, will be parsed into a vector of [`MidiFileEvent`]({{site.baseurl}}/docs/V2_0_1/datatypes/MidiFileEvent)s.

As soon as a midi file is dropped on **ANY** of the visualizers, the `deploy()` method will be called with the
`new_midi_file_dropped_on_visualizers` flag set to `true`. As such, check for this flag to be `true`, and then
access the content of the dropped midi file:

```c++
    if (new_midi_file_dropped_on_visualizers) {
        # get the ids of the visualizers that have new content
        # these ids will be the same as the ones specified in the settings.json file
        auto ids =
            midiVisualizersData->get_visualizer_ids_with_user_dropped_new_sequences();
        for (const auto& id : ids) {           
            auto new_sequence = midiVisualizersData->get_visualizer_data(id);
            if (new_sequence != std::nullopt) {
                for (const auto& event : *new_sequence) {
                    cout << event.getDescription().str() << endl;
                }
            }
        }
    }
```

### Updating the content of a visualizer
You can update the content of a visualizer using the unique `id` of the visualizer specified in the `settings.json` file.

```c++
    midiVisualizersData->clear_visualizer_data("MidiDisplay 1");
    midiVisualizersData->displayNoteOn("MidiDisplay 1", 32, 0.1, 0.5);
    midiVisualizersData->displayNoteOff("MidiDisplay 1", 32, 0.5);
    midiVisualizersData->displayNoteWithDuration("MidiDisplay 1", 31, 0.1, 0.5, 0.9);
```