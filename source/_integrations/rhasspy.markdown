---
title: "rhasspy"
description: "Instructions on how to integrate Rhasspy within Home Assistant."
logo: rhasspy.png
ha_category:
  - "Voice"
ha_release: 0.101.3
---

[Rhasspy](https://rhasspy.readthedocs.io) is a highly customizable, private voice assistant that [runs completely offline](#the-zen-of-rhasspy) and [supports many languages](#supported-languages). It generates a personalized speech/intent recognizer from voice commands that [you can create](https://rhasspy.readthedocs.io/en/latest/training/#sentencesini) entirely on your device.

This integration uses the [rhasspy-client](https://pypi.org/project/rhasspy-client/) to communicate with a remote Rhasspy server using its [HTTP API](https://rhasspy.readthedocs.io/en/latest/reference/#http-api).

To get started, you must:

1. [Install a Rhasspy server](https://rhasspy.readthedocs.io/en/latest/installation/)
2. Visit the web interface at http://YOUR-RHASSPY-SERVER:12101 to download a language-specific profile
3. Go to the [Settings tab](https://rhasspy.readthedocs.io/en/latest/usage/#settings-tab) and configure Rhasspy
4. Write your [custom voice commands](https://rhasspy.readthedocs.io/en/latest/training/#sentencesini) and train your profile ("Train" button in the [top bar](https://rhasspy.readthedocs.io/en/latest/usage/#top-bar))

```yaml
# Example configuration.yaml entry
rhasspy:
  api_url: http://YOUR_RHASSPY_SERVER:12101/api/
  register_conversation: true
    
# Rhasspy will register as a conversation agent
conversation:

stt:
  - platform: rhasspy
```

Replace `YOUR_RHASSPY_SERVER` with the hostname of the machine where Rhasspy is running.

## Rhasspy and Home Assistant

There are many different ways you can configure Rhasspy and Home Assistant to work together. It all depends on exactly what you want to do. Below are some common configurations options.

### Just Speech Recognition

If you just want to use Rhasspy for speech recognition, but keep using [Ada](https://github.com/home-assistant/ada) and Home Assistant's [conversation](https://www.home-assistant.io/integrations/conversation/) component (possibly with [Almond](https://www.home-assistant.io/integrations/almond/)), configure the `rhasspy` integration like this:

```yaml
# Just speech recognition (configuration.yaml)
rhasspy:
  api_url: http://YOUR_RHASSPY_SERVER:12101/api/
  register_conversation: false

stt:
  - platform: rhasspy
```

In the Rhasspy [Settings tab](https://rhasspy.readthedocs.io/en/latest/usage/#settings-tab):

1. Uncheck "Listen for wake word on start-up" in the "Rhasspy" section
2. Select "No wake word on this device" in the "Wake Word" section
3. In the "Speech Recognition" section
    * Choose "Do speech recognition with kaldi on this device" if available for your language (choose pocketsphinx otherwise)
    * Check "Open transcription mode (no custom voice commands)" under the "kaldi" or "pocketsphinx" setting you chose
4. Save your settings and restart Rhasspy
5. Follow instructions to download any additional files for Rhasspy

### Speech and Intent Recognition

You can use Rhasspy both for speech and intent recognition (replacing [Almond](https://www.home-assistant.io/integrations/almond/)). In this configuration, [Ada](https://github.com/home-assistant/ada) should still be used.

In Home Assistant:

```yaml
# Speech and intent recognition (configuration.yaml)
rhasspy:
  api_url: http://YOUR_RHASSPY_SERVER:12101/api/
  register_conversation: true
    
# Rhasspy will register as a conversation agent
conversation:

stt:
  - platform: rhasspy
```

In the Rhasspy [Settings tab](https://rhasspy.readthedocs.io/en/latest/usage/#settings-tab):

1. Uncheck "Listen for wake word on start-up" in the "Rhasspy" section
2. Select "No wake word on this device" in the "Wake Word" section
3. In the "Speech Recognition" section
    * Choose "Do speech recognition with kaldi on this device" if available for your language (choose pocketsphinx otherwise)
4. Save your settings and restart Rhasspy
5. Follow instructions to download any additional files for Rhasspy
6. Author your [custom voice commands](https://rhasspy.readthedocs.io/en/latest/training/#sentencesini)
    * Save sentences and train your profile after each change
    
When writing your voice commands, you can use Home Assistant's [built-in intents](https://developers.home-assistant.io/docs/en/intent_builtin.html). Make sure to match the same slot names; for example:

```ini
[HassTurnOn]
entity_names = (kitchen lights | bedroom lights)
turn on the (<entity_names>){name}
```

The `name` slot is required for `HassTurnOn`, and should be the friendly name of one of your Home Assistant entities.

## Conversation Platform

When `rhasspy.register_conversation` is set to `true`, the `rhasspy` integration will forward all sentences to your Rhasspy server for processing (specifically, to `/api/text-to-intent`). If the sentence matches one your custom voice commands, Home Assistant will handle the returned intent.

For example, assume you have the following text in your Rhasspy [sentences.ini file](https://rhasspy.readthedocs.io/en/latest/training/#sentencesini):

```ini
[HassLightSet]
light_name = (kitchen lights) | (bed lights)
color_name = red | green | blue | white
set [the] (<light_name>){name} to (<color_name>){color}
```

When Home Assistants [conversation](https://www.home-assistant.io/integrations/conversation/) integration receives a sentence like "set the kitchen lights to red", Rhasspy will return a `HassLightSet` intent with the `name` and `color` slots set to "kitchen lights" and "red". Because `HassLightSet` is [built into Home Assistant](https://developers.home-assistant.io/docs/en/intent_builtin.html), Home Assistant will automatically handle it!

You can handle custom intents that Home Assistant doesn't know by using the [intent script](https://www.home-assistant.io/integrations/intent_script/) component. The `GetTemperature` example for `intent_script` has the following configuration:

{% raw %}
```yaml
intent_script:
  GetTemperature:  # Intent type
    speech:
      text: We have {{ states.sensor.temperature }} degrees
    action:
      service: notify.notify
      data_template:
        message: Hello from an intent!
```
{% endraw %}

To have Rhasspy trigger `GetTemperature`, just add a section to your `sentences.ini`:

```ini
[GetTemperature]
what is the temperature
how (cold | hot) is it
```

After training, Rhasspy will now respond to commands like "what is the temperature" and "how cold is it". These voice commands will cause Home Assistant to speak the templated text and produce a notification.

## Speech to Text Platform

The `rhasspy` integration works with Home Assistant's native [speech to text component](https://www.home-assistant.io/components/stt). It simply forwards the recorded WAV data to Rhasspy's `/api/speech-to-text` endpoint and returns the transcription.

```yaml
# Example configuration.yaml entry
# This will use the api_url from the rhasspy provider.
stt:
  - platform: rhasspy
```

## Supported Languages

Rhasspy supports the following languages/locales. Support comes from publically available models and datasets. If you'd like a new language to be supported, consider [donating your voice](https://voice.mozilla.org)!

* U.S. English (`en-US`)
* Dutch (`nl-NL`)
* French (`fr-FR`)
* German (`de-DE`)
* Greek (`el-GR`)
* Italian (`it-IT`)
* Brazilian Portuguese (`pt-BR`)
* Russian (`ru-RU`)
* Spanish (`es-ES`)
* Swedish (`sv-SV`)
* Vietnamese (`vi-VI`)

## The Zen of Rhasspy

* Built on free/open source software
* No online account required
* No data ever leaves your device
* Works for the majority, customizable by the minority
