---
name: home-security-engineering
description: >-
  Domain expertise for building smart home security products and systems — covers threat modeling
  for connected security devices, sensor/alarm logic (multi-sensor fusion, false-alarm reduction,
  arm/disarm state machines), emergency dispatch integration (911/NG911, monitoring station
  protocols, CSAA/SIA verification standards), access control (smart locks, keypads, credential
  management, camera-based/video verification), and neighborhood crime data integration (sourcing
  and responsibly using local/historical crime data for risk context). Use whenever the user is
  designing, building, evaluating, or writing about a home security product, smart home security
  feature, DIY security system, alarm panel, sensor network, Home Assistant security automation,
  emergency-response integration, or a neighborhood safety/crime-data feature — even without the
  words "security" or "threat model." Trigger on mentions of intrusion detection, motion/door/window
  sensors, false alarms, alarm verification, 911 dispatch, NG911, RapidSOS, central station
  monitoring, panic buttons, glass-break detection, camera-based security, access control, smart
  locks, local/recent crime data, crime mapping, neighborhood safety score, or named crime-data
  sources (CrimeMapping.com, LexisNexis, CityProtect, Citizen App) in a security-product context.
---

# Home Security Engineering

Domain knowledge for people building products, features, or systems in smart home security — from DIY Home Assistant setups to commercial security products (alarm panels, professional monitoring, dispatch-integrated devices).

This skill is a router. It covers five areas that come up constantly in this domain and that generic knowledge tends to get wrong or oversimplify. Read the relevant reference file(s) before producing substantive output — don't just answer from general instincts, because this field has hard-won conventions (verified alarm standards, dispatch protocols, crime-data sourcing constraints) that aren't obvious from first principles.

## When to use which reference

| The user is... | Read |
|---|---|
| Designing a system, evaluating what could go wrong, doing a security review, thinking about adversaries (burglars, tamperers, network attackers, insiders) | `references/threat-modeling.md` |
| Working on sensor logic, deciding when to trigger an alarm, reducing false alarms, combining signals from multiple sensors, arm/disarm states | `references/sensor-alarm-logic.md` |
| Connecting a system to 911/emergency services, working with a monitoring center, thinking about alarm verification or response time | `references/dispatch-integration.md` |
| Designing locks, keypads, credentials, camera-based detection, or video verification | `references/access-control-and-locks.md` |
| Incorporating local/recent or historical crime data (CrimeMapping.com, LexisNexis, CityProtect, Citizen, or similar) into a risk score, onboarding flow, or safety feature | `references/crime-data-integration.md` |

Many real requests touch two or more of these at once (e.g., "should my system call 911 directly or go through a monitoring center" is dispatch + threat modeling; "should the front door unlock on camera person-detection" is access control + sensor logic; "use neighborhood crime data to set default sensor sensitivity" is crime data + threat modeling). Read all that apply.

## How to use this skill well

1. **Identify what's actually being asked.** A request like "help me design the sensor logic for my security system" is an engineering task — read `sensor-alarm-logic.md` and produce concrete logic (state machines, thresholds, pseudocode), not a generic essay about sensors.
2. **Default to specificity over caution-washing.** This is a legitimate, common engineering domain (this is what companies like Ring, SimpliSafe, ADT, and Vivint do for a living). Don't hedge requests for technical detail on security *products* as if they were requests to *defeat* security — those are different things. Helping someone build a good alarm system is not the same as helping someone bypass one.
3. **Distinguish "detecting/responding to threats" from "creating attack tools."** This skill is about building defensive systems: sensor fusion, verified dispatch, hardened architecture, legitimate access control, responsibly-sourced risk data. If a request shifts toward specific techniques for defeating physical security (lock-picking mechanics, relay-attack tooling, jamming specific commercial alarm frequencies, disabling a *named* real system) or toward scraping/circumventing ToS on a named crime-data platform instead of using an open or licensed source, treat that as outside this skill's scope and reason about it as you normally would.
4. **Ground recommendations in real standards where they exist.** This field has real industry bodies and standards — cite them by name (CSAA, SIA, UL 827/1023, NFPA 72, RapidSOS/NG911) rather than inventing plausible-sounding but fake standards.
5. **For Home Assistant-specific work**, apply the relevant reference's thinking but express it in HA's actual constructs (automations, template sensors, `binary_sensor`/`lock` domains, input helpers for arm state) — write working YAML, not pseudo-YAML. See `examples/home-assistant/` for a worked reference implementation.
6. **Ask about scale and context when it changes the answer.** DIY single-home vs. multi-unit commercial product vs. venture-backed startup changes what's proportionate (e.g., UL listing matters a lot for a company selling monitored hardware, much less for a hobbyist HA automation).

## Quick reference: core vocabulary

- **False alarm rate (FAR)**: the single most important metric in this industry — most jurisdictions fine or deprioritize repeat false-alarm addresses, and it's the top reason people abandon security products.
- **Alarm verification**: confirming a real event before dispatching a costly response (police/fire). Levels range from unverified (raw sensor trip) to audio/video verified to third-party-witnessed.
- **Sensor fusion**: combining signals from multiple sensor types (motion + door + glass-break + camera) to raise confidence and cut false positives, vs. any single sensor firing.
- **Dispatch path**: the route an alarm takes from device → (monitoring center, optional) → emergency services. Not all systems go through a human monitoring center; some go device-direct.
- **NG911**: Next Generation 911 — IP-based 911 infrastructure that can carry richer data (location, sensor context, video) than legacy analog 911, and is the basis for direct-to-911 dispatch products.
- **Credential revocation**: removing a specific credential's (code, fob, app account) access without affecting others — the design property that makes lost-phone, terminated-employee, and insider-abuse scenarios recoverable rather than requiring a full re-key.
- **NIBRS**: the FBI's National Incident-Based Reporting System, the current federal crime-data standard (replacing the old UCR Summary Reporting System) — coverage still has real per-agency gaps from an incomplete nationwide transition, which matters for anyone treating federal crime data as uniformly complete.

See the reference files for depth on each of these.
