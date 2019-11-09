---
title: "rhasspy"
description: "Instructions on how to integrate Rhasspy within Home Assistant."
logo: rhasspy.png
ha_category:
  - Voice
ha_release: 0.101.3
---

[Rhasspy](https://rhasspy.readthedocs.io) is a highly customizable, truly private voice assistant that [runs completely offline](#the-zen-of-rhasspy) and [supports many languages](https://rhasspy.readthedocs.io/en/latest/#supported-languages). It generates a personalized speech/intent recognizer based on voice commands that [you can create and customize](#customizing-voice-commands).

For this integration to function, you must [install a Rhasspy server](https://rhasspy.readthedocs.io/en/latest/installation/) and make sure it's running before you start Home Assistant (or [re-train](#re-training) after Rhasspy starts).

By default, voice commands are automatically generated for all lights, switches, covers, binary_sensors, and sensors. You can say things like "turn on the living room lamp", "is the garage door open?", and "run automation light show in ten minutes". Additionally, you can say "set a timer for three and a half minutes", and perform a [scripted action](https://www.home-assistant.io/integrations/intent_script/) with Rhasspy's [built-in `TimerReady` intent](#built-in-intents).


```yaml
# Example configuration.yaml entry
rhasspy:
    api_url: http://YOUR_RHASSPY_SERVER:12101/api
    register_conversation: true
    
# Rhasspy will register as a conversation agent
conversation:

stt:
    - platform: rhasspy
```

## Rhasspy Configuration

{% configuration %}
api_url:
  description: URL of your Rhasspy server's web API.
  required: false
  type: string
  default: http://localhost:12101/api
language:
  description: Language/locale of your Rhasspy profile. See [supported languages](#supported-languages).
  required: false
  type: string
  default: en-US
register_conversation:
  description: When true, Rhasspy will register a [conversation](https://www.home-assistant.io/integrations/conversation/) agent and handle intent recognition.
  required: false
  type: boolean
  default: true
intent_commands:
  description: Map of intents and [sentence templates](https://rhasspy.readthedocs.io/en/latest/training/#sentencesini). This is where you specify [custom voice commands](#custom-voice-commands).
  required: false
  type: map
response_templates:
  description: Map of intents and [jinja2 templates](https://www.home-assistant.io/docs/configuration/templating/). This controls how Rhasspy generates speech in response to the [built-in intents](#built-in-intents)
  required: false
  type: map
handle_intents:
  description: List of intents that the Rhasspy integration should handle (choose from [built-in intents](#built-in-intents))
  required: false
  type: list
intent_states:
  description: Map of intents and state names for deciding when devices are "on", "open", etc. (see [intent states](#intent-states))
  required: false
  type: map
slots:
  description: Map of slot names and their values (see [slot lists](https://rhasspy.readthedocs.io/en/latest/training/#slots-lists)). Referenced as `$name` in your sentence templates.
  required: false
  type: map
custom_words:
  description: Map of words and how you will pronounce them (see [custom words](https://rhasspy.readthedocs.io/en/latest/training/#custom-words)).
  required: false
  type: map
name_replace:
  description: 
  required: false
  type: map
train_timeout:
  description: Number of seconds after the last `EVENT_COMPONENT_LOADED` has been received to [re-train Rhasspy](#re-training)
  required: false
  type: float
  default: 1.0
{% endconfiguration %}

## Speech to Text Platform

```yaml
# Example configuration.yaml entry
# This will use the api_url from the rhasspy provider.
stt:
    - platform: rhasspy
```

{% configuration %}
speech_url:
  description: URL of your Rhasspy server's speech-to-text endpoint.
  required: false
  type: string
  default: Use provider setting or http://localhost:12101/api/speech-to-text
{% endconfiguration %}

## Customizing Voice Commands

Like the [conversation](/integrations/conversation/) integration, `rhasspy` works together with [`intent_script`](/integrations/intent_script) to trigger actions in your Home Assistant.

```yaml
rhasspy:
  intents:
    LivingRoomTemperature:
     - what is the temperature in the living room

intent_script:
  LivingRoomTemperature:
    speech:
      text: It is currently {{ states.sensor.temperature }} degrees in the living room.
```

## Built-in Voice Commands

### Entity Names


## Built-in Intents

Rhasspy handles most of [Home Assistant's built-in intents](https://developers.home-assistant.io/docs/en/intent_builtin.html), specifically:

* `HassTurnOn`
* `HassTurnOff`
* `HassToggle`
* `HassOpenCover`
* `HassCloseCover`
* `HassShoppingListAddItem`
* `HassShoppingListLastItems`

The following Home Assistant intents are

* `IsDeviceOn`
    * Reports whether a device is [currently on or not](#intent-states) via speech
    * intent slots
        * `name` - name of device
    * response slots
        * `entity` - [state object](https://www.home-assistant.io/docs/configuration/state_object/) of device
* `IsDeviceOff`
    * Reports whether a device is [currently off or not](#intent-states) via speech
    * response slots
        * `entity` - [state object](https://www.home-assistant.io/docs/configuration/state_object/) of device
* `IsCoverOpen`
    * Reports whether a cover is [currently open or not](#intent-states) via speech
    * response slots
        * `entity` - [state object](https://www.home-assistant.io/docs/configuration/state_object/) of cover
* `IsCoverClosed`
    * Reports whether a cover is [currently closed or not](#intent-states) via speech
    * response slots
        * `entity` - [state object](https://www.home-assistant.io/docs/configuration/state_object/) of cover
* `DeviceState`
    * Reports a device's current state via speech
    * slots
        * `name` - name of device
* `TriggerAutomation`
    * Triggers an automation by name and responds via speech
    * intent slots
        * `name` - name of automation to trigger
    * response slots
        * `automation` - [state object](https://www.home-assistant.io/docs/configuration/state_object/) of automation that was triggered
* `TriggerAutomationLater`
    * Sets a timer and generates an `TriggerAutomation` intent when it elapses
    * intent slots
        * `name` - name of automation to trigger
        * `seconds` - number of seconds to wait (in addition to `minutes`, `hours`)
        * `minutes` - number of minutes to wait (in addition to `seconds`, `hours`)
        * `hours` - number of hours to wait (in addition to `seconds`, `minutes`)
* `SetTimer`
    * Sets a timer and generates an `TimerReady` intent when it elapses
    * intent slots
        * `seconds` - number of seconds to wait (in addition to `minutes`, `hours`)
        * `minutes` - number of minutes to wait (in addition to `seconds`, `hours`)
        * `hours` - number of hours to wait (in addition to `seconds`, `minutes`)
* `TimeReady`
    * Generated from `SetTimer` intent after timer has elapsed

### Intent States

* `IsDeviceOn` - "on"
* `IsDeviceOff` - "off"
* `IsCoverOpen` - "open"
* `IsCoverClosed` - "closed"

## Supported Languages

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

### Numbers in Entity Names

The Rhasspy integration will automatially replace numbers in entity names with words using the [num2words library](https://pypi.org/project/num2words/). Unfortunately, Greek and Swedish. For Swedish, Danish number words are used instead. For all other cases, English number words are used.

## Re-Training

## The Zen of Rhasspy

* Built on free/open source software
* No online account required
* No data leaves your device
