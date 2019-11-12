---
title: "rhasspy"
description: "Instructions on how to integrate Rhasspy within Home Assistant."
logo: rhasspy.png
ha_category:
  - Voice
ha_release: 0.101.3
---

[Rhasspy](https://rhasspy.readthedocs.io) is a highly customizable, truly private voice assistant that [runs completely offline](#the-zen-of-rhasspy) and [supports many languages](https://rhasspy.readthedocs.io/en/latest/#supported-languages). It generates a personalized speech/intent recognizer based on voice commands that [you can create and customize](#customizing-voice-commands) on your device.

For this integration to function, you *must* do the following:

1. [Install a Rhasspy server](https://rhasspy.readthedocs.io/en/latest/installation/)
2. Visit the Rhasspy web interface at [http://YOUR_RHASSPY_SERVER:12101](http://YOUR_RHASSPY_SERVER:12101) to download a language-specific profile

Make sure your Rhasspy server is running *before* you start Home Assistant (or you will need to [re-train from Home Assistant](#re-training) after Rhasspy starts).

```yaml
# Example configuration.yaml entry
rhasspy:
  api_url: http://YOUR_RHASSPY_SERVER:12101/api
  language: en-US
  register_conversation: true
  make_intent_commands: true
    
# Rhasspy will register as a conversation agent
conversation:

stt:
  - platform: rhasspy
```

By default, voice commands are [automatically generated](#built-in-voice-commands) for all [built-in intents](#built-in-intents) using [your friendly entity names](https://www.home-assistant.io/docs/configuration/customizing-devices/#friendly_name). Out of the box, you can turn devices on and off, ask about their states, set a timer, and run automations via voice.

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
  description: Map of intents and [jinja2 speech templates](https://www.home-assistant.io/docs/configuration/templating/). This controls how the `rhasspy` integration generates speech in response to the [built-in intents](#built-in-intents).
  required: false
  type: map
handle_intents:
  description: List of intents that the `rhasspy` integration should handle (choose from [built-in intents](#built-in-intents)).
  required: false
  type: list
intent_states:
  description: Map of [intents and state names](#intent-state-names) for deciding when devices are "on", "open", etc.
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
  description: List of possible items that can be added to your shopping list. If this list is empty (the default), you will not be able to add anything to your shopping list via voice.
  required: false
  type: list
train_timeout:
  description: Number of seconds after the last `EVENT_COMPONENT_LOADED` has been received to [re-train Rhasspy](#re-training).
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
  default: Use `rhasspy` setting or http://localhost:12101/api/speech-to-text
{% endconfiguration %}

## Built-in Voice Commands

The `rhasspy` integration generates voice commands for all of Home Assistant's [built-in intents](#built-in-intents). This is done by catching the `EVENT_COMPONENT_LOADED` event for specific domains and entities. `rhasspy` uses *your friendly entity names* in these commands, so they're personalized to your Home Assistant!

Voice commands are automatically generated for:

* the `light` domain and `group.all_lights`
* the `switch` domain and `group.all_switches`
* the `cover` domain and `group.all_covers`
* the `binary_sensor` domain
* the `sensor` domain
* the `automation` domain

You can [change these domains/entities](#controlling-voice-command-generation) or create entirely [new voice commands/intents](#customizing-voice-commands). Examples of the built-in voice commands are listed below by intent.

### Turn On/Off and Toggle

| Intent           | Example                   |
|------------------|---------------------------|
| `HassTurnOn`     | turn on the FRIENDLY_NAME |
| `HassTurnOff`    | FRIENDLY_NAME off         |
| `HassTurnToggle` | toggle all lights         |
| `HassOpenCover`  | open FRIENDLY_NAME        |
| `HassCloseCover` | close all covers          |


### Get Device State

| Intent           | Example                     |
|------------------|-----------------------------|
| `IsDeviceOn`     | is FRIENDLY_NAME on         |
| `IsDeviceOff`    | are all lights off          |
| `IsDeviceOpen`   | is FRIENDLY_NAME open       |
| `IsDeviceClosed` | is the FRIENDLY_NAME closed |
| `IsDeviceState`  | is the sun set              |

Example responses:

* "Yes. FRIENDLY_NAME is on."
* "No. FRIENDLY_NAME is closed."
* "Yes. The sun is below horizon."

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
| `TriggerAutomationLater** | run automation FRIENDLY_NAME in an hour |

### Manipulate Shopping List

You must have at least one item in `shopping_list_items` to be able to add items:

```yaml
rhasspy:
  shopping_list_items:
    - apples
    - bananas
```

| Intent                      | Example                      |
|-----------------------------|------------------------------|
| `HassShoppingListAddItem`   | add ITEM to my shopping list |
| `HassShoppingListLastItems` | what is on my shopping list  |

### Controlling Voice Command Generation

Automatic voice command generation can be finely tuned by including/excluding specific intents, or even disabled entirely, with the `make_intent_commands` parameter.

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

Don't automatically generate *any* voice commands (only those in `intent_commands`):

```yaml
rhasspy:
  make_intent_commands: false
```

### Entity Names

`rhasspy` uses the friendly names of your entities in voice commands. These names may not be compatible with Rhasspy's speech recognition system, so the integration attempts to "clean" names before training your Rhasspy server. These cleaning steps are:

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

`rhasspy` handles all of [Home Assistant's built-in intents](https://developers.home-assistant.io/docs/en/intent_builtin.html), including:

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

### Rhasspy Specific Intents

The `rhasspy` integration defines several *new* intents to help you ask questions about your devices, trigger automations, and set timers:

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
* `IsDeviceState`
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

### Intent State Names

`rhasspy`'s query intents need to determine whether a device is on/off or open/closed in order to answer yes or no. By default, a device's `state` is compared with the following values per intent:

* `IsDeviceOn` - "on"
* `IsDeviceOff` - "off"
* `IsCoverOpen` - "open"
* `IsCoverClosed` - "closed"

If you have a device that you want to be considered "on" in a particular state (for example, "active"), use the `intent_states` parameter:

```yaml
rhasspy:
  intent_states:
    IsDeviceOn:
      - "on"
      - "active"
```

## Customizing Voice Commands

You can add brand new voice commands and intents to `rhasspy` entirely on your device! (Hint: you can even [add your own words](#custom-words))

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

After [training Rhasspy](#re-training), you can now say "what is the temperature in the living room" and have Home Assistant response with speech while handing the `LivingRoomTemperature` intent. You can also send this same sentence to your Home Assistant's `conversation/process` service.

Each intent in the `intent_commands` configuration must contain exactly one of the following parameters:

{% configuration %}
command:
  description: Single [voice command](#voice-command-language)
  required: false
  type: string
commands:
  description: List of [voice commands](#voice-command-language)
  required: false
  type: list
command_template:
  description: Single [jinja2 template](https://www.home-assistant.io/docs/configuration/templating/) that generates one or more [voice commands](#voice-command-language)
  required: false
  type: string
command_templates:
  description: List of [jinja2 template](https://www.home-assistant.io/docs/configuration/templating/) that generates one or more [voice commands](#voice-command-language)
  required: false
  type: list
data:
  description: Map of slot names and values for the recognized intent.
  required: false
  type: map
data_template:
  description: Map of slot names and [jinja2 templates](https://www.home-assistant.io/docs/configuration/templating/). Slot values in the recognized intent will have the rendered values.
  required: false
  type: map
{% endconfiguration %}

`command` and `commands` are literal voice commands in Rhasspy's [voice command language](#voice-command-language). `command_template` and `command_templates`, however, are [jinja2 templates](https://www.home-assistant.io/docs/configuration/templating/) that are rendered for each entity in a specific domain or from a list of entity ids. For example:

```yaml
rhasspy:
  intent_commands:
    LockDoor:
      - command_template: "lock [the] ({% raw %}{{ speech_name }}{% endraw %}){name}"
        data_template:
          entity_id: "{% raw %}{{ entity.entity_id }}{% endraw %}"
        include:
          domains:
            - lock
lock:
  - platform: ...
    name: door_1
  - platform: ...
    name: door_2
```

This `command_template` will be rendered for each entity in the `lock` domain in your `configuration.yaml`. The template will receive three parameters:

* `speech_name` - the [cleaned name](#entity-names) of the entity
* `friendly_name` - the friendly name of the entity
* `entity` - the entity's [state object](https://www.home-assistant.io/docs/configuration/state_object/) when it was loaded

For the lock example above, the `command_template` will be rendered twice to produce the equivalent `commands`:

```yaml
- "lock [the] (door one){name} (:){entity_id:lock.door_1}"
- "lock [the] (door two){name} (:){entity_id:lock.door_2}"
```

(The `(:)` syntax is Rhasspy's way of attaching "unspoken" metadata to a voice command)

If you say "lock door one", Rhasspy will generate a `LockDoor` intent with `name` and `entity_id` slots set to "door one" and "lock.door_1" respectively. You can use Home Assistant's [`intent_script`](/integrations/intent_script) component to do something with this intent:

```yaml
intent_script:
  LockDoor:
    speech:
      text: "Locking {% raw %}{{ name }}{% endraw %}."
    action:
      service: lock.lock
      data_template:
        entity_id: {% raw %}{{ entity_id }}{% endraw %}
    
```

### Voice Command Language

Rhasspy's [voice command language](https://rhasspy.readthedocs.io/en/latest/training/#sentencesini) allows you to capture many different ways of expressing a voice command in a compact but readable form. Besides describing word choices, you can also annotate your voice command with [tags](https://rhasspy.readthedocs.io/en/latest/training/#tags) that determine what ends up in the recognized intent's slots.

The following syntax is available:

* `[optional words]` - words may or may not be spoken
* `(alternative | choice | set)` - exactly one of the words will be spoken
* `$slot_name` - name of a [slot](#slots) whose values should included here as an alternative
* `(...){tag_name}` - tags the value in parentheses. This will show up in the recognized intent with `tag_name` as the slot name

Note that you can combine syntax, so `[the | an | a]` will optionally be either "the", "an", or "a". Use parentheses to group words to avoid ambiguity, e.g. `(the dog) | (that cat)`.

You can use `:` in words and tags to *substitute* what was literally spoken (left side of `:`) with something you want in the recognized intent (right side of `:`). This is usually done for numbers, ids, etc. For example, if you want to say "set kitchen fan speed to medium", you might write a voice command like this:

```yaml
rhasspy:
  intent_commands:
    SetFanSpeed:
      -command: "set (kitchen fan){entity_id:fan.kitchen} speed to (medium){speed:50}"
```

When the `SetFanSpeed` intent is fired, it will have `entity_id` and `speed` slots set to "fan.kitchen" and "50" (both strings), respectively. You can convert `speed` to an integer in a Home Assistant template with something like `{% raw %}{{ speed|int }}{% endraw %}`

### Slots

You can list slot names and values in your `rhasspy` configuration and then reference them in voice commands as `$slot_name`. This is equivalent to putting the slot values into an `(alternative | choice | set)`, but is faster and more convenient for large sets of items. For example:

```yaml
rhasspy:
  slots:
    colors:
      - red
      - green
      - blue
```

You can now use `$colors` in a voice command like "set the light to ($colors){color}".

### Custom Words

Need to say a word that Rhasspy doesn't know or that you want pronounce differently? No problem! Just edit the `custom_words` parameter:

```yaml
rhasspy:
  custom_words:
    moogles: "M UW G AH L Z"
```

The pronunciation of moogles (M UW G AH L Z) was created by visiting the "Words" tab in the Rhasspy web interface at [http://YOUR_RHASSPY_SERVER:12101](http://YOUR_RHASSPY_SERVER:12101) where `YOUR_RHASSPY_SERVER` is the hostname of your Rhasspy server. You can use this interface to look up existing word pronunciations and guess new ones. See  the [custom words documentation](https://rhasspy.readthedocs.io/en/latest/training/#custom-words) for more details.

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

## Re-Training

Rhasspy needs to be trained before it can recognize your voice commands. This usually only takes a few seconds, but may take a minute or more if you have a large number of possible commands.

When Home Assistant starts the `rhasspy` integration, it will automatically contact your Rhasspy server and train your profile. If you need to re-train Rhasspy without restarting Home Assistant, use the `rhasspy.train` service in Home Assistant or visit Rhasspy's web interface at [http://YOUR_RHASSPY_SERVER:12101](http://YOUR_RHASSPY_SERVER:12101) where `YOUR_RHASSPY_SERVER` is the hostname of your Rhasspy server (e.g., `localhost`).

## The Zen of Rhasspy

* Built on free/open source software
* No online account required
* No data leaves your device
* Works for the majority, customizable by the minority
