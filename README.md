# Pool AI Safety Alarm for Home Assistant

AI-assisted pool monitoring for Home Assistant using UniFi Protect-style triggers, a camera snapshot, and Home Assistant AI Task image analysis.

This project is a supplemental alerting aid only. It is not a certified drowning detection system and must not replace direct supervision, locked barriers, pool covers, certified alarms, rescue equipment, or emergency services. AI vision can miss events, misclassify images, or fail during network, camera, provider, or Home Assistant outages.

## Blueprint Import

Blueprint file:

```text
https://github.com/magaman/pool-ai-safety-alarm/blob/main/blueprints/automation/pool_ai_safety_alarm.yaml
```

Raw blueprint URL:

```text
https://raw.githubusercontent.com/magaman/pool-ai-safety-alarm/main/blueprints/automation/pool_ai_safety_alarm.yaml
```

In Home Assistant, go to Settings > Automations & scenes > Blueprints > Import Blueprint, then paste the raw blueprint URL.

## What It Does

```text
Trigger entity turns on
  -> Pool camera image is sent to an AI Task entity
  -> AI returns structured data
  -> Helper entities store status and summary
  -> Suppression timers are checked
  -> Built-in notifications, speakers, and sirens run
  -> Notification suppression button taps are handled by the same automation
  -> Optional per-severity custom actions run
```

Supported AI statuses:

- `SAFE`
- `WATCH`
- `EDGE_RISK`
- `WATER_ENTRY`
- `DISTRESS`
- `DROWNING_RISK`
- `ERROR`

Built-in alert routing:

- Mobile notifications: all non-`SAFE` statuses.
- Suppression notification buttons: `WATCH`, `EDGE_RISK`, `WATER_ENTRY`, and `ERROR` only.
- Voice messages to speakers: `WATER_ENTRY`, `DISTRESS`, and `DROWNING_RISK`.
- Sirens: `DISTRESS` and `DROWNING_RISK` only.

## Repository Structure

```text
blueprints/
  automation/
    pool_ai_safety_alarm.yaml
packages/
  pool_ai_helpers.yaml
docs/
  setup.md
```

## Requirements

- Home Assistant with YAML access.
- A configured Home Assistant AI Task entity that supports image attachments.
- A camera entity pointed at the pool area.
- Trigger entities such as UniFi Protect person detection, motion sensors, gate sensors, or door sensors.
- Optional Home Assistant Companion App devices, speakers, a TTS engine, and siren entities.

## Setup

1. Import the blueprint from the raw URL above.
2. Copy `packages/pool_ai_helpers.yaml` into your Home Assistant `packages/` directory.
3. If packages are not enabled, add this to `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

4. Restart Home Assistant so the helpers and timers are created.
5. Configure an automation from the blueprint.

See [docs/setup.md](docs/setup.md) for detailed setup and testing steps.

Blueprint import does not create Home Assistant helpers. The helper package must still be installed through YAML packages or recreated manually in the UI before the automation runs.

## Notification Targets

Mobile notification targets are selected directly in the blueprint from Home Assistant Companion App devices. The blueprint derives the matching `notify.mobile_app_*` service from each selected device name.

Critical `DISTRESS` and `DROWNING_RISK` notifications do not include suppression buttons.

## Helpers

The helper package creates:

- `input_boolean.pool_ai_safety_enabled`
- `input_select.pool_ai_safety_status`
- `input_text.pool_ai_safety_summary`
- `input_text.pool_ai_safety_raw_response`
- suppression timers for `WATCH`, `EDGE_RISK`, `WATER_ENTRY`, `DISTRESS`, `DROWNING_RISK`, and `ERROR`

Critical timers exist for local admin automations, maintenance windows, or manual overrides. The included notification flow does not expose critical suppression buttons.
