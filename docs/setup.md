# Pool AI Safety Alarm Setup

This package adds an AI-assisted pool monitoring blueprint for Home Assistant. It is a supplemental alerting system only. Do not use it as a replacement for supervision, locked barriers, pool covers, certified alarms, rescue equipment, or emergency procedures.

## Requirements

- Home Assistant with YAML mode access.
- A camera entity that can see the pool area.
- Trigger entities that turn `on` for events such as UniFi Protect person detection, motion, door open, or gate open.
- A configured Home Assistant AI Task entity that supports image attachments.
- Optional Home Assistant Companion App devices, speakers, a TTS engine, and siren entities.

## Import the Blueprint

Import this blueprint in Home Assistant:

```text
blueprints/automation/pool_ai_safety_alarm.yaml
```

Raw import URL:

```text
https://raw.githubusercontent.com/magaman/pool-ai-safety-alarm/main/blueprints/automation/pool_ai_safety_alarm.yaml
```

Home Assistant automation blueprints cannot create helpers during blueprint import. Importing the blueprint creates only the reusable automation definition. Create the helpers below before enabling the automation.

## Configure AI Task

1. Configure an AI provider in Home Assistant that exposes AI Task support.
2. Add an AI Task sub-entry/entity for that provider.
3. Confirm the AI Task entity can process camera/image attachments.
4. Select that `ai_task.*` entity in the blueprint's `AI Task entity` field.

The blueprint calls `ai_task.generate_data` with a structured response and expects one of these statuses:

```text
SAFE, WATCH, EDGE_RISK, WATER_ENTRY, DISTRESS, DROWNING_RISK, ERROR
```

## Create Helpers in the UI

In Home Assistant, go to Settings > Devices & services > Helpers, then create these helpers. After each helper is created, open its settings and confirm the entity ID exactly matches the value shown here.

### Toggle Helper

Create a Toggle helper:

| Field | Value |
| --- | --- |
| Name | Pool AI Safety Enabled |
| Entity ID | `input_boolean.pool_ai_safety_enabled` |
| Icon | `mdi:pool` |
| Initial state | On |

### Dropdown Helper

Create a Dropdown helper:

| Field | Value |
| --- | --- |
| Name | Pool AI Safety Status |
| Entity ID | `input_select.pool_ai_safety_status` |
| Icon | `mdi:pool-thermometer` |
| Options | `SAFE`, `WATCH`, `EDGE_RISK`, `WATER_ENTRY`, `DISTRESS`, `DROWNING_RISK`, `ERROR` |
| Initial option | `SAFE` |

### Text Helpers

Create two Text helpers:

| Name | Entity ID | Max length | Initial value |
| --- | --- | --- | --- |
| Pool AI Safety Summary | `input_text.pool_ai_safety_summary` | `255` | `No pool AI analysis has run yet.` |
| Pool AI Safety Raw Response | `input_text.pool_ai_safety_raw_response` | `255` | `{}` |

### Timer Helpers

Create these Timer helpers:

| Name | Entity ID | Duration |
| --- | --- | --- |
| Pool AI WATCH Suppression | `timer.pool_ai_watch_suppression` | `00:15:00` |
| Pool AI EDGE_RISK Suppression | `timer.pool_ai_edge_risk_suppression` | `00:10:00` |
| Pool AI WATER_ENTRY Suppression | `timer.pool_ai_water_entry_suppression` | `00:05:00` |
| Pool AI DISTRESS Suppression | `timer.pool_ai_distress_suppression` | `00:02:00` |
| Pool AI DROWNING_RISK Suppression | `timer.pool_ai_drowning_risk_suppression` | `00:01:00` |
| Pool AI ERROR Suppression | `timer.pool_ai_error_suppression` | `00:30:00` |

YAML users can alternatively copy `packages/pool_ai_helpers.yaml` into a packages directory and restart Home Assistant.

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

## Suppression Handling

The blueprint includes mobile notification buttons for these statuses only:

- `WATCH`
- `EDGE_RISK`
- `WATER_ENTRY`
- `ERROR`

The blueprint automation handles those button taps itself and starts the matching suppression timer. Do not add suppression actions for `DISTRESS` or `DROWNING_RISK`; those critical statuses intentionally have no notification suppression buttons.

Critical suppression timers exist for local admin automations, maintenance windows, or manual overrides. The included notification flow does not expose critical suppression buttons.

## Testing

Test during a supervised, safe scenario before relying on any alert:

1. Confirm trigger entities turn `on`.
2. Confirm the selected AI Task entity can analyze the selected camera.
3. Confirm each selected Companion App device receives notifications.
4. Confirm speaker volume and TTS output are appropriate.
5. Confirm sirens are selected only where immediate audible alerts are desired.
6. Watch Home Assistant logs for AI Task errors, unavailable entities, or notification service-name mismatches.
