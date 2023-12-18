---
layout: default
title: AudioVisualizersData
nav_order: 5
has_children: false
parent: Data Types

permalink: /docs/V2_1_0/datatypes/AudioVisualizersData
---

# AudioVisualizersData
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Description

The content of AudioVisualizersData placed in the tabs will be modifiable/accessible in the `deploy()` method using
the `audioVisualizersData` instance.

You can update the content whenever the audio content changes. Moreover, when the user drops an audio file into the
widget, the content of the audio visualizers will be provided to you in the `deploy()` method.

### Accessing the content of a dropped audio file
The content of the dropped midi file, will be provided to you as a pair of juce::AudioBuffer<float> and a double
where the first corresponds to the audio data and the second to the sample rate.

As soon as an audio file is dropped on **ANY** of the visualizers, the `deploy()` method will be called with the
`new_audio_file_dropped_on_visualizers` flag set to `true`. As such, check for this flag to be `true`, and then
access the content of the dropped audio file:

```c++
    if (new_audio_file_dropped_on_visualizers) {
        auto ids = audioVisualizersData->get_visualizer_ids_with_user_dropped_new_audio();
        for (const auto& id : ids) {
            auto new_audios = audioVisualizersData->get_visualizer_data(id);
            if (new_audios != std::nullopt) {
                auto [audio, fs] = *new_audios;
                cout << "Audio File Dropped on Visualizer: " << id << endl;
                cout << "Audio File Length In Samples: " << audio.getNumSamples() << endl;
            }
        }
    }
```

### Updating the content of a visualizer
You can update the content of a visualizer using the unique `id` of the visualizer specified in the `settings.json` file.

Make sure the audio data displayed is placed in a `juce::AudioBuffer<float>` instance. 

```c++
    audioVisualizersData->clear_visualizer_data("AudioDisplay 1");
    float fs = 44100;
    audioVisualizersData->display_audio("AudioDisplay 1", audioBuffer, 44100.0f);
```

#### Quick Guide on [`juce::AudioBuffer<float>`](https://docs.juce.com/master/classAudioBuffer.html)

In the following example, we will create a sine wave of 10 seconds duration

```c++
    # specify the sample rate
    float fs = 44100;
    
    # specify the duration of the audio in samples
    int duration_samples = int(fs * 10.0f);
    
    # create an empty audio buffer of 1 channel, 441000 samples (10 seconds) duration
    juce::AudioBuffer<float> audioBuffer(1, duration_samples);  
    
    // populate audioBuffer with 10 cycles of a sine wave
    for (int i = 0; i < duration_samples; i++) {
        audioBuffer.setSample(0, i, std::sin(i * 2 * M_PI / fs));
    }
```