# home-security-skill

A [Claude Code](https://claude.com/claude-code) skill that gives Claude domain expertise in smart home security engineering — the kind of hard-won, field-specific knowledge that generic instincts tend to get wrong or oversimplify.

It covers six areas:

- **Threat modeling** — separating the physical intruder, adversaries against the system itself (jamming, tampering, spoofing), and adversaries *via* the system (account takeover, insider abuse, data exposure).
- **Sensor & alarm logic** — false-alarm reduction, multi-sensor fusion, arm/disarm state machines, and Home Assistant implementation patterns.
- **Emergency dispatch integration** — self-response vs. central monitoring vs. direct-to-911/NG911 architectures, alarm verification levels, and the real (non-technical) requirements behind dispatch integration.
- **Access control & camera security** — credential types (PIN, RFID/NFC, mobile/BLE, biometric) and their threat profiles, smart lock engineering concerns, and camera/video verification.
- **Neighborhood crime data integration** — the real access/licensing landscape behind sources like CrimeMapping.com, LexisNexis, CityProtect, and Citizen; legitimate open-data alternatives (FBI CDE, municipal open-data portals); and the bias/fairness risk of crime-score features.
- **Home Assistant integration patterns** — Alarmo and similar community integrations (envisalink, AlarmDecoder, Konnected) as prior art, and how to architect a dispatch integration against the `alarm_control_panel` domain contract instead of coupling to one backend.

Grounded in real industry standards (CSAA, SIA, UL 827/1023, NFPA 72, RapidSOS/NG911, FBI NIBRS) rather than invented ones, and includes worked Home Assistant examples.

## Structure

```
.claude-plugin/plugin.json                          — plugin manifest
skills/home-security-engineering/
  SKILL.md                                           — router: what to read for which kind of request
  references/
    threat-modeling.md
    sensor-alarm-logic.md
    dispatch-integration.md
    access-control-and-locks.md
    crime-data-integration.md
    home-assistant-integration-patterns.md
examples/home-assistant/
  fused-intrusion-alarm.yaml                         — worked automation implementing the sensor-fusion escalation ladder
  dispatch-handoff-pattern.yaml                       — domain-not-backend pattern for handing off to a dispatch integration
.github/workflows/validate.yml                       — CI: validates plugin.json, SKILL.md frontmatter, and example YAML
```

## CI

Every push and PR runs `.github/scripts/validate.py`, which checks: `plugin.json` is valid JSON with required fields; every `skills/*/SKILL.md` has valid frontmatter (`name`, `description`) and its routing table doesn't point at a missing reference file; and every file under `examples/` is valid YAML (Home Assistant tags like `!secret`/`!include` are accepted).

## Install

**As a Claude Code plugin** (recommended — Claude loads `SKILL.md` automatically and only reads a reference file when a request actually calls for it):

```
claude plugin marketplace add midnightsecurity/home-security-skill
claude plugin install home-security-skill
```

**Manually**, for any tool that reads Anthropic-style skill folders: copy `skills/home-security-engineering/` into your tool's skills directory (for Claude Code, `~/.claude/skills/` for personal use, or a project's `.claude/skills/`).

## Scope

This skill is about designing and building *defensive* systems — sensor fusion, verified dispatch, access control, hardened architecture, responsibly-sourced risk data. It explicitly treats specific techniques for defeating a *named* real commercial product (lock-picking a specific cylinder, jamming a specific panel's RF frequency, relay-attack tooling for a specific lock) or for scraping/circumventing ToS on a named crime-data platform as out of scope; see the "Out of scope" section in each reference file.

## License

Apache-2.0 — see [LICENSE](LICENSE).
