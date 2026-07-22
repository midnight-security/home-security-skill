# Sensor & Alarm Logic

Use this when the user is designing sensor behavior, alarm state machines, false-alarm reduction, or multi-sensor fusion — including Home Assistant automations.

## Why false alarms are the central design problem

This is the most important framing in the whole domain: **the hardest part of a home security system is not detecting intruders, it's not crying wolf.** Raw PIR motion sensors alone routinely produce false-trigger rates driven by pets, HVAC airflow, sunlight/heat sources, insects on the lens, and foliage movement seen through windows. Every design decision below exists primarily to manage this tradeoff between sensitivity (catch real events) and specificity (don't alert on non-events), because:

- High false-alarm rates cause **alert fatigue** — users stop responding to or trusting notifications, or disable the system entirely.
- Many municipalities impose **false-alarm fines** or will **deprioritize/refuse dispatch** to addresses with a history of false alarms — so for anything touching real dispatch, false-alarm rate isn't just a UX metric, it has direct real-world consequences.

## Core alarm system states

A standard arm/disarm state machine, useful as a starting point for both product design and Home Assistant automations:

- **Disarmed** — no monitoring active (or monitoring for non-security purposes only, e.g. presence-based automations).
- **Armed Away** — full monitoring, all zones (interior + perimeter) active, no expected occupants.
- **Armed Home / Armed Stay** — perimeter zones (doors, windows) active; interior motion zones typically excluded or on a separate, less-sensitive profile, since occupants are moving around inside.
- **Entry/Exit Delay** — a timed grace window after arming (to leave) and after a perimeter trip while armed (to disarm before alarm), typically 30–60 seconds, configurable per zone.
- **Alarm/Triggered** — a zone tripped past the required confirmation logic (see below) with no valid disarm in the delay window.
- **Trouble/Tamper** — a fault state distinct from Alarm: sensor lost supervision, tamper switch tripped, low battery, etc. Should notify but not necessarily dispatch emergency response.

## Sensor fusion patterns

Rather than any single sensor trip = alarm, fusion approaches raise confidence before escalating:

- **AND logic across sensor types**: e.g., require a door/window contact trip *and* a motion or glass-break event within a short window to escalate to a verified alarm, vs. either alone triggering only a low-priority notification.
- **Sequential/spatial logic**: motion detected in sequence across zones consistent with someone moving through the house (e.g., entry zone → hallway → interior) is higher-confidence than an isolated single-zone trip.
- **Cross-modal confirmation**: pairing a binary sensor trip with a camera snapshot/clip or audio classifier (glass-break acoustic signature, aggressive dog bark vs. ambient noise) for human or AI-assisted verification before an unverified-to-verified escalation.
- **Environmental/contextual gating**: barometric pressure sensors can detect the pressure change from a door or window opening even without a direct contact sensor, and can help distinguish "door opened" from "HVAC cycled" when correlated with other signals — useful as a redundant/independent channel in fusion logic, since it's a different failure mode than optical or magnetic sensors.
- **Pet immunity**: PIR sensors with pet-immune lensing (typically rated for pets under ~40–80 lbs) reduce nuisance trips; combining with mass-discrimination or dual-tech (PIR + microwave, requiring both to agree) sensors further cuts false positives.

## A worked example: escalation ladder

A reasonable default escalation model for a fused system:

1. **Single low-confidence sensor trip** (e.g., one PIR motion zone) → log event, no notification, or silent low-priority push.
2. **Single high-confidence sensor trip** (e.g., door contact while armed away) → immediate push notification + begin entry delay countdown.
3. **Two+ corroborating signals within N seconds** (door + motion, or motion + glass-break) → escalate to "likely intrusion," start any monitoring-center/dispatch-relevant timer, request video verification if available.
4. **No disarm/cancel within the delay window** → alarm state; proceed per the dispatch integration reference for what happens next (siren, monitoring center, direct dispatch).

Tune "N seconds" and which sensor combinations count as corroborating based on the home's layout — this is a real per-install tuning problem, not a fixed universal constant.

## Home Assistant implementation notes

When the user wants this expressed in actual Home Assistant config:

- Use the `alarm_control_panel` domain rather than reinventing arm-state tracking from scratch — it already models `armed_away`, `armed_home`, `pending`, `triggered`, etc. The built-in **`manual`** platform implements it at a basic level; **Alarmo** (HACS) implements it with per-zone arm-mode membership, per-zone delays, and per-user codes, and is the better starting point for anything beyond a single shared code — see `home-assistant-integration-patterns.md` for the fuller landscape (including bridged-panel options for legacy wired systems) before deciding which to build on.
- Fusion logic is typically implemented as an **automation** that watches multiple `binary_sensor` entities (`device_class: motion`, `door`, `window`, `vibration`, `moisture` etc.) and only calls `alarm_control_panel.alarm_trigger` when the combined condition is met — write this as an explicit `and`/`or` condition block, not a chain of independent single-sensor automations that each call trigger. If arm-state is already owned by Alarmo or a bridged panel, this becomes "watch for its `triggered` state" rather than calling `alarm_trigger` directly — don't fight another integration for ownership of the state machine.
- `input_boolean` or `input_select` helpers are commonly used for arm-state nuances beyond the built-in states (e.g., a custom "armed_night" profile) when not using something like Alarmo that already provides them.
- For sensor supervision (see threat-modeling.md), HA can surface a `binary_sensor` or `sensor` entity's `unavailable`/stale-`last_changed` state — an automation that alerts on a sensor not reporting within X hours approximates RF supervision even for consumer hardware that doesn't natively support it.
- Template sensors are the right tool for combining raw entities into a derived "confidence" or "fused" sensor state that other automations key off of, keeping the fusion logic in one place rather than scattered across automations.

## Metrics worth tracking

If the user is building (not just designing) a system, suggest instrumenting:

- False-alarm rate per install and in aggregate (the industry's north-star metric)
- Time-to-acknowledge (how long between trigger and user/monitoring response)
- Sensor uptime/supervision compliance
- Escalation-ladder drop-off (how many events stop at each stage vs. proceed to full alarm)
