---
title: "voice2json"
description: "Instructions on how to integrate voice2json within Home Assistant."
logo: voice2json.png
ha_category:
  - Voice
ha_release: 0.101.3
---

[voice2json](https://voice2json) is an offline, private speech/intent recognition system that [supports many languages](https://voice2json.org/#supported-languages). It generates custom speech and intent recognizers based on [templates](https://voice2json.org/sentences.html) that *you* specify.

More documentation can be found at [voice2json.org](https://voice2json.org).

## Home Assistant configuration

{% configuration %}
profile_name:
  description: Name of voice2json profile to use.
  required: false
  type: string
  default: "en-us_pocketsphinx-cmu"
detect_silence:
  description: True if voice2json should wait for silence before processing voice command.
  required: false
  default: true
  type: boolean
default_utterances:
  description: When true, "turn on/off and toggle" utterances are generated for every entity with a friendly name
  required: false
  default: true
  type: boolean
transcription_mode:
  description: Either "closed" (only specific voice commands are recognized) or "open" (anything is recognized)
  required: false
  default: closed
  type: string
{% endconfiguration %}

### Triggering actions

Actions are triggered based on intents using the [`intent_script`](/integrations/intent_script) component. For instance, the following block handles a `ActivateLightColor` intent to change light colors:

Note: If your Snips action is prefixed with a username (e.g., `john:playmusic` or `john__playmusic`), the Snips integration in Home Assistant will try and strip off the username. Bear this in mind if you get the error `Received unknown intent` even when what you see on the MQTT bus looks correct. Internally the Snips integration is trying to match the non-username version of the intent (i.e., just `playmusic`).

{% raw %}
```yaml
snips:

intent_script:
  ActivateLightColor:
    action:
      - service: light.turn_on
        data_template:
          entity_id: light.{{ objectLocation | replace(" ","_") }}
          color_name: {{ objectColor }}
```
{% endraw %}

In the `data_template` block, we have access to special variables, corresponding to the slot names for the intent. In the present case, the `ActivateLightColor` has two slots, `objectLocation` and `objectColor`.

### Slots

Several special values for slots are populated with the `siteId` the intent originated from and the probability value for the intent, the `sessionId` generate by the dialogue manager, and `slote_name` raw which will contain the raw, uninterpreted text of the slot value.

In the above example, the slots are plain strings. However, Snips has a duration builtin value used for setting timers and this will be parsed to a seconds value.

In this example if we had an intent triggered with 'Set a timer for five minutes', `duration:` would equal 300 and `duration_raw:` would be set to 'five minutes'. The duration can be easily used to trigger Home Assistant events and the `duration_raw:` could be used to send a human readable response or alert.

{% raw %}
```yaml
SetTimer:
  speech:
    type: plain
    text: 'Set a timer'
  action:
    service: script.set_timer
    data_template:
      name: "{{ timer_name }}"
      duration: "{{ timer_duration }}"
      siteId: "{{ site_id }}"
      sessionId: "{{ session_id }}"
      duration_raw: "{{ raw_value }}"
      probability: "{{ probability }}"
```
{% endraw %}

### Configuration Examples

#### Turn on a light

```yaml
intent_script:
  turn_on_light:
    speech:
      type: plain
      text: 'OK, turning on the light'
    action:
      service: light.turn_on
```

##### Open a Garage Door

```yaml
intent_script:
  OpenGarageDoor:
    speech:
      type: plain
      text: 'OK, opening the garage door'
    action:
      - service: cover.open_cover
        data:
          entity_id: garage_door
```
