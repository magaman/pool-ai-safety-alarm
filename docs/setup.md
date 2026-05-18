# Pool AI Safety Alarm Setup

This package adds an AI-assisted pool monitoring blueprint for Home Assistant. It is a supplemental alerting system only. Do not use it as a replacement for supervision, locked barriers, pool covers, certified alarms, rescue equipment, or emergency procedures.

## Requirements

- Home Assistant with YAML mode access.
- A camera entity that can see the pool area.
- Trigger entities that turn `on` for events such as UniFi Protect person detection, motion, door open, or gate open.
- The LLM Vision custom integration installed and configured with at least one provider.
- Optional mobile app notification services, speakers, a TTS engine, and siren entities.

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

## Configure LLM Vision

1. Install LLM Vision through HACS or the integration's documented install path.
2. Add a provider in LLM Vision.
3. Open the LLM Vision image analyzer UI and select your provider.
4. Copy the provider/config entry ID and paste it into the blueprint's `LLM Vision provider ID` field.
5. Leave the model field blank unless you need a specific model override.

The blueprint calls `llmvision.image_analyzer` with structured JSON and expects one of these statuses:

```text
SAFE, WATCH, EDGE_RISK, WATER_ENTRY, DISTRESS, DROWNING_RISK, ERROR
```

## Create the Blueprint Automation

1. In Home Assistant, go to Settings > Automations & scenes > Blueprints.
2. Import or select `Pool AI Safety Alarm`.
3. Choose the pool camera entity.
4. Add trigger entities. UniFi Protect person detection, motion sensors, gate sensors, and door sensors should work when their alert state is `on`.
5. Add mobile app notify services, one per line or comma-separated:

```text
notify.mobile_app_janes_iphone
notify.mobile_app_pool_ipad
```

6. Optionally choose speaker entities and enter a TTS engine such as `tts.home_assistant_cloud`.
7. Optionally choose siren entities. Sirens are triggered only for `DISTRESS` and `DROWNING_RISK`.
8. Configure any custom action selectors for each severity.

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
2. Confirm LLM Vision can analyze the selected camera.
3. Confirm each mobile app notify service works.
4. Confirm speaker volume and TTS output are appropriate.
5. Confirm sirens are selected only where immediate audible alerts are desired.
6. Watch Home Assistant logs for LLM Vision errors, unavailable entities, or invalid service names.
