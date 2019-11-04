---
title: "rhasspy"
description: "Instructions on how to integrate Rhasspy within Home Assistant."
logo: rhasspy.png
ha_category:
  - Voice
ha_release: 0.101.3
---

[Rhasspy](https://rhasspy.readthedocs.io) is an offline, private voice assistant that [supports many languages](https://rhasspy.readthedocs.io/en/latest/#supported-languages). It generates a personalized speech/intent recognizer based on [custom voice commands](#custom-voice-commands) that *you* specify.

For this integration to function, you must first [install a Rhasspy server](https://rhasspy.readthedocs.io/en/latest/installation/).

```yaml
# Example configuration.yaml entry
rhasspy:
    web_url: http://localhost:12101
```

{% configuration %}
web_url:
  description: URL of your Rhasspy server.
  required: false
  type: string
  default: http://localhost:12101
intents:
  description: Map of intents and [sentence templates](https://rhasspy.readthedocs.io/en/latest/training/#sentencesini). This is where you specify [custom voice commands](#custom-voice-commands).
  required: false
  type: map
slots:
  description: Map of slot names and their values (see [slot lists](https://rhasspy.readthedocs.io/en/latest/training/#slots-lists)). Referenced as `$name` in your sentence templates.
  required: false
  type: map
stream:
  description: True if audio should be streamed directly to Rhasspy (see `stream_url`). When false, audio is buffered locally and then POST-ed to Rhasspy's [speech-to-text endpoint](https://rhasspy.readthedocs.io/en/latest/usage/#http-api).
  required: false
  default: true
  type: boolean
stream_url:
  description: When `stream` is true, audio sent to this integration is forwarded to Rhasspy's [HTTP Stream Audio Recorder](https://rhasspy.readthedocs.io/en/latest/audio-input/#http-stream).
  required: false
  default: http://localhost:12333
  type: string
detect_silence:
  description: When true, local audio is segmented by silence before being POST-ed to Rhasspy (only when `stream` is false).
  required: false
  default: true
  type: boolean
default_utterances:
  description: When true, "turn on/off and toggle" utterances are generated for every entity with a friendly name
  required: false
  default: true
  type: boolean
{% endconfiguration %}

## Custom Voice Commands

Like the [conversation](/integrations/conversation/) integration, `rhasspy` works together with [`intent_script`](/integrations/intent_script) to trigger actions in your Home Assistant.

```yaml
rhasspy:
  intents:
    LivingRoomTemperature:
     - what is the temperature in the living room

conversation:
  intents:
    LivingRoomTemperature:
     - What is the temperature in the living room

intent_script:
  LivingRoomTemperature:
    speech:
      text: It is currently {{ states.sensor.temperature }} degrees in the living room.
```
