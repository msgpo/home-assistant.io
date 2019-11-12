---
title: "rhasspy"
description: "Instructions on how to integrate Rhasspy within Home Assistant."
logo: rhasspy.png
ha_category:
  - Voice
ha_release: 0.101.3
---

[Rhasspy](https://rhasspy.readthedocs.io) is a highly customizable, truly private voice assistant that [runs completely offline](#the-zen-of-rhasspy) and [supports many languages](https://rhasspy.readthedocs.io/en/latest/#supported-languages). It generates a personalized speech/intent recognizer based on voice commands that [you can create and customize](#customizing-voice-commands) on your device.

For this integration to function, you must [install a Rhasspy server](https://rhasspy.readthedocs.io/en/latest/installation/) and make sure it's running before you start Home Assistant (or [re-train](#re-training) after Rhasspy starts).

```yaml
# Example configuration.yaml entry
rhasspy:
    api_url: http://YOUR_RHASSPY_SERVER:12101/api
    register_conversation: true
    make_intent_commands: true
    
# Rhasspy will register as a conversation agent
conversation:

stt:
    - platform: rhasspy
```

By default, voice commands are [automatically generated](#built-in-voice-commands) for all of Home Assistant's (and Rhasspy's) [built-in intents](#built-in-intents) using [your friendly entity names](https://www.home-assistant.io/docs/configuration/customizing-devices/#friendly_name). Out of the box, you can turn devices on and off, ask about their states, set a timer, and run automations via voice.

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
make_intent_commands:
  description: Controls how voice comands are auto-generated. True or false enables or disables voice command generation. You can [customize this more](#controlling-voice-command-generation) by include or excluding specific intents.
  required: false
  type: boolean
  default: true
intent_commands:
  description: Map of intents and [voice command templates](https://rhasspy.readthedocs.io/en/latest/training/#sentencesini). This is where you specify [custom voice commands](#custom-voice-commands).
  required: false
  type: map
response_templates:
  description: Map of intents and [jinja2 speech templates](https://www.home-assistant.io/docs/configuration/templating/). This controls how Rhasspy generates speech in response to the [built-in intents](#built-in-intents)
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
shopping_list_items:
  description: List of possible items that can be added to your shopping list
  required: false
  type: list
train_timeout:
  description: Number of seconds after the last `EVENT_COMPONENT_LOADED` has been received to [re-train Rhasspy](#re-training)
  required: false
  type: float
  default: 1.0
{% endconfiguration %}

## Speech to Text Platform

The `rhasspy` integration works with Home Assistant's native [speech to text component](https://www.home-assistant.io/components/stt).

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
  default: Use rhasspy setting or http://localhost:12101/api/speech-to-text
{% endconfiguration %}

## Customizing Voice Commands

Like the [conversation](/integrations/conversation/) integration, `rhasspy` works together with [`intent_script`](/integrations/intent_script) to trigger actions in your Home Assistant. For example, the following configuration snippet creates a voice command that will make Home Assistant tell you the temperature:

```yaml
rhasspy:
  intent_commands:
    LivingRoomTemperature:
      - command: what is the temperature in the living room

intent_script:
  LivingRoomTemperature:
    speech:
      text: It is currently {{ states.sensor.temperature }} degrees in the living room.
```

After [training Rhasspy](#re-training)

## Built-in Voice Commands

The `rhasspy` integration can generate voice commands for all of Home Assistant's (and some of Rhasspy's) [built-in intents](#built-in-intents). This is done by catching the `EVENT_COMPONENT_LOADED` event from Home Assistant, and generating commands for specific domains and entities. Rhasspy uses *your friendly entity names* in these commands, so you're in control!

By default, voice commands are [automatically generated](#built-in-voice-commands) for all lights, switches, covers, binary_sensors, and sensors. You can say things like "turn on the living room lamp", "is the garage door open?", and "run automation light show in ten minutes". Additionally, you can say "set a timer for three and a half minutes", and perform a [scripted action](https://www.home-assistant.io/integrations/intent_script/) with Rhasspy's [built-in `TimerReady` intent](#built-in-intents).

### Turn On/Off and Toggle

| Intent           | Example                   |
|------------------|---------------------------|
| `HassTurnOn`     | turn on the FRIENDLY_NAME |
| `HassTurnOff`    | FRIENDLY_NAME off         |
| `HassTurnToggle` | toggle all lights         |
| `HassOpenCover`  | open FRIENDLY_NAME        |
| `HassCloseCover` | close all covers          |


### Get Device State

| Intent           | Example                 |
|------------------|-------------------------|
| `IsDeviceOn`     | is FRIENDLY_NAME on     |
| `IsDeviceOff`    | is FRIENDLY_NAME off    |
| `IsDeviceOpen`   | is FRIENDLY_NAME open   |
| `IsDeviceClosed` | is FRIENDLY_NAME closed |
| `IsDeviceState`  | is the sun set          |

### Set a Timer

| Intent     | Example                            |
|------------|------------------------------------|
| `SetTimer` | set a timer for five minutes       |
| `SetTimer` | set a timer for an hour and a half |

When the timer elapses, a `TimerReady` intent in generated.

### Run Automations

| Intent                   | Example                                 |
|--------------------------|-----------------------------------------|
| `TriggerAutomation`      | run automation FRIENDLY_NAME            |
| `TriggerAutomation`      | execute FRIENDLY_NAME                   |
| `TriggerAutomationLater` | in five minutes run FRIENDLY_NAME       |
| `TriggerAutomationLater` | run automation FRIENDLY_NAME in an hour |

### Controlling Voice Command Generation

Include only turn on/off commands:

```yaml
rhasspy:
  make_intent_commands:
    include:
      - HassTurnOn
      - HassTurnOff
```

Just exclude timer commands:

```yaml
rhasspy:
  make_intent_commands:
    exclude:
      - SetTimer
      - TriggerAutomationLater
```

Don't generate *any* voice commands (only those in `intent_commands`):

```yaml
rhasspy:
  make_intent_commands: false
```

### Entity Names

Rhasspy uses the friendly names of your entities in voice commands. These names may not be compatible with its speech recognition system, so the `rhasspy` integration attempts to "clean" the names up before sending them to your Rhasspy server. The cleaning steps are:

1. Numbers are replaced with words ("65" becomes "sixty-five")
2. Dashes (`-`) and underscores (`_`) are replaced with a space

Before cleaning a name, the `rhasspy` integration checks `name_replace.entities` for a matching entity id and manually cleaned name. For example:

```yaml
rhasspy:
  name_replace:
    light.light_1: "living room lamp"
```

You can override the cleaning behavior using `name_replace.regex`. This is a list, where each item is a map with [regular expression](https://docs.python.org/3/library/re.html) keys and replacement values. The default behavior looks like this:

```yaml
rhasspy:
  name_replace:
    regex:
      - "[-_]": " "
```

Each item in the `name_replace.regex` is run in order on the name, so you can build on previous substitutions.

## Built-in Intents

Rhasspy handles all of [Home Assistant's built-in intents](https://developers.home-assistant.io/docs/en/intent_builtin.html), including:

* `HassTurnOn`
    * Turns a device or group on
    * intent slots
        * `name` - name of device
* `HassTurnOff`
    * Turns a device or group off
    * intent slots
        * `name` - name of device
* `HassToggle`
    * Toggles a device or group
    * intent slots
        * `name` - name of device
* `HassOpenCover`
    * Opens a cover or cover group
    * intent slots
        * `name` - name of device
* `HassCloseCover`
    * Closes a cover or cover group
    * intent slots
        * `name` - name of device
* `HassLightSet`
    * Sets the color or brightness of a light or light group
    * intent slots
        * `name` - name of device
        * `color` - CSS color name
        * `brightness` - number from 0 to 100
* `HassShoppingListAddItem`
    * Adds an item to your shopping list (see `shopping_list_items` configuration option)
    * intent slots
        * `item` - name of item
* `HassShoppingListLastItems`
    * Lists the last five items from your shopping list

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

The Rhasspy integration will automatially replace numbers in entity names with words using the [num2words library](https://pypi.org/project/num2words/). Unfortunately, Greek and Swedish are not supported. For Swedish, Danish number words are used instead. For Greek, English number words substituted.

## Voice Command Examples

## Re-Training

Rhasspy needs to be trained before it can recognize your voice commands. This usually only takes a few seconds, but may take a minute or more if you have a large number of possible commands.

When Home Assistant starts the `rhasspy` integration, it will automatically contact your Rhasspy server and train your profile. If you need to re-train Rhasspy without restarting Home Assistant, use the `rhasspy.train` service in Home Assistant or visit Rhasspy's web interface at [http://RHASSPY_SERVER:12101](http://RHASSPY_SERVER:12101) where `RHASSPY_SERVER` is the hostname of your Rhasspy server (e.g., `localhost`).

## The Zen of Rhasspy

* Built on free/open source software
* No online account required
* No data leaves your device
