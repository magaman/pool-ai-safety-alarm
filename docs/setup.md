# Pool AI Safety Alarm Setup

This package adds an AI-assisted pool monitoring blueprint for Home Assistant. It is a supplemental alerting system only. Do not use it as a replacement for supervision, locked barriers, pool covers, certified alarms, rescue equipment, or emergency procedures.

## Requirements

- Home Assistant with YAML mode access.
- A camera entity that can see the pool area.
- Trigger entities that turn `on` for events such as UniFi Protect person detection, motion, door open, or gate open.
- A configured Home Assistant AI Task entity that supports image attachments.
- Optional Home Assistant Companion App devices, speakers, a TTS engine, and siren entities.

## Install Files

Copy these folders into your Home Assistant config directory:

```text
blueprints/automation/pool_ai_safety_alarm.yaml
packages/pool_ai_helpers.yaml
examples/suppression_handler.yaml
docs/setup.md
```

If you do not already use packages, add this to `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Restart Home Assistant after adding the package so the helper entities are created.

## Configure AI Task

1. Configure an AI provider in Home Assistant that exposes AI Task support.
2. Add an AI Task sub-entry/entity for that provider.
3. Confirm the AI Task entity can process camera/image attachments.
4. Select that `ai_task.*` entity in the blueprint's `AI Task entity` field.

The blueprint calls `ai_task.generate_data` with a structured response and expects one of these statuses:

```text
SAFE, WATCH, EDGE_RISK, WATER_ENTRY, DISTRESS, DROWNING_RISK, ERROR
```

## Create the Blueprint Automation

1. In Home Assistant, go to Settings > Automations & scenes > Blueprints.
2. Import or select `Pool AI Safety Alarm`.
3. Choose the pool camera entity.
4. Choose the AI Task entity.
5. Add trigger entities. UniFi Protect person detection, motion sensors, gate sensors, and door sensors should work when their alert state is `on`.
6. Select one or more Home Assistant Companion App devices for mobile notifications.
7. Optionally choose speaker entities and enter a TTS engine such as `tts.home_assistant_cloud`.
8. Optionally choose siren entities. Sirens are triggered only for `DISTRESS` and `DROWNING_RISK`.
9. Configure any custom action selectors for each severity.

## Suppression Handler

The blueprint includes mobile notification buttons for these statuses only:

- `WATCH`
- `EDGE_RISK`
- `WATER_ENTRY`
- `ERROR`

Use `examples/suppression_handler.yaml` as an automation to start suppression timers when those actions are tapped. Do not add suppression actions for `DISTRESS` or `DROWNING_RISK`; those critical statuses intentionally have no notification suppression buttons.

## Helper Entities

The package creates these state helpers:

- `input_boolean.pool_ai_safety_enabled`
- `input_select.pool_ai_safety_status`
- `input_text.pool_ai_safety_summary`
- `input_text.pool_ai_safety_raw_response`

It also creates suppression timers for:

- `timer.pool_ai_watch_suppression`
- `timer.pool_ai_edge_risk_suppression`
- `timer.pool_ai_water_entry_suppression`
- `timer.pool_ai_distress_suppression`
- `timer.pool_ai_drowning_risk_suppression`
- `timer.pool_ai_error_suppression`

Critical timers exist for local admin automations, maintenance windows, or manual overrides. The included notification flow does not expose critical suppression buttons.

## Testing

Test during a supervised, safe scenario before relying on any alert:

1. Confirm trigger entities turn `on`.
2. Confirm the selected AI Task entity can analyze the selected camera.
3. Confirm each selected Companion App device receives notifications.
4. Confirm speaker volume and TTS output are appropriate.
5. Confirm sirens are selected only where immediate audible alerts are desired.
6. Watch Home Assistant logs for AI Task errors, unavailable entities, or notification service-name mismatches.
