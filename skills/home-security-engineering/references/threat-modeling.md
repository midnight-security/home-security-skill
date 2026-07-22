# Threat Modeling for Home Security Systems

Use this when the user is designing a system, doing a security review, or reasoning about "what could go wrong" for a smart home security product.

## Frame: three separate adversary classes

Home security threat models fail when they treat "security" as one thing. Separate it into three adversary classes, because the mitigations barely overlap:

1. **Physical intruder** — the person the system is meant to detect (burglar, trespasser). Threat model question: *will the system reliably notice this person and respond in time?*
2. **Adversary against the system itself** — someone trying to defeat, disable, or spoof the security system so the physical intrusion goes undetected. Threat model question: *can this device be jammed, powered down, network-isolated, or fooled without triggering a tamper alert?*
3. **Adversary via the system** — someone attacking the homeowner *through* the security product (network intrusion, camera feed access, data breach, stalkerware-style misuse by an abusive partner/ex who has app access). Threat model question: *does this product become a new attack surface or surveillance tool?*

A good product threat model addresses all three explicitly rather than assuming "detects intruders" implies "is itself secure."

## A practical structure (STRIDE-adjacent, adapted for physical+IoT)

For each component (sensor, hub, app, cloud backend, dispatch integration), walk through:

- **Spoofing** — can a fake signal impersonate a real sensor (e.g., replay a "door closed" RF signal while the door is open)?
- **Tampering** — can the device be physically opened, disabled, or have its firmware altered without alerting anyone? (This is where **tamper switches** and **line/RF supervision** — see below — matter.)
- **Repudiation** — if something goes wrong, is there an audit trail (event log, video clip) to establish what happened?
- **Information disclosure** — does the system leak occupancy/schedule data (e.g., "armed away" state broadcast unencrypted) that itself creates risk?
- **Denial of service** — can the system be jammed (RF jamming of wireless sensors is a well-known real technique), power-cut, or network-isolated (cutting Wi-Fi/ethernet before Wi-Fi/cellular backup kicks in)?
- **Elevation of privilege** — can a guest-level app user or a compromised IoT device on the same network reach admin functions?

## Supervision: the concept generic engineering knowledge tends to miss

Professional alarm systems don't just wait for a sensor to say "triggered" — they continuously supervise sensors so that the *absence* of communication is itself an alert:

- **RF supervision**: wireless sensors send periodic "I'm still here" heartbeat signals (commonly every ~1–4 hours in commercial panels). Missing heartbeats past a threshold raises a trouble/tamper condition, catching a sensor that's been destroyed, removed, or its battery pulled — none of which would otherwise fire a "detected" signal.
- **Line supervision** (for wired systems): end-of-line resistors detect a cut or shorted wire as a distinct fault state, not just "no signal."
- **Tamper switches**: physical microswitches that trip if a sensor enclosure is opened or pulled off the wall, independent of the sensor's main detection function.

If a user is designing a DIY or product-grade sensor and hasn't thought about supervision, that's usually the single highest-value thing to raise — it's the difference between "detects intrusion" and "detects intrusion, assuming the attacker doesn't touch the sensor first."

## Network/cloud-specific risks for connected security products

- **Cloud dependency as single point of failure**: if arm/disarm, video, and alerts all route through a cloud backend, an outage or DDoS against the vendor disables every customer's system simultaneously. Ask whether local/offline fallback modes exist (local siren trigger, local network alert) when cloud is unreachable.
- **Credential/account takeover as physical risk**: unlike most SaaS breaches, an account takeover on a security app can translate directly into physical burglary (attacker disarms system remotely, or simply learns the home is empty from occupancy patterns). This raises the bar for auth (MFA, anomaly detection on remote disarm) above typical consumer-app norms.
- **Local network trust assumptions**: many smart-home protocols (older Zigbee/Z-Wave implementations, some local APIs) assume the local network is trusted. A compromised smart bulb or unrelated IoT device on the same LAN can sometimes reach the security hub. Segmentation (separate VLAN/SSID for security devices) is a real, recommended mitigation — worth raising for anyone with a mixed IoT + security setup, including Home Assistant users running everything on one flat network.
- **Insider/abuse risk**: shared-access features (family app accounts, guest codes) can be misused for surveillance or control by someone with legitimate former access (ex-partner, estranged family member). This is a known, documented harm pattern in the connected-home industry and worth designing for explicitly — e.g., a clear "remove all other users and re-pair devices" reset flow, and activity logs visible to the primary account holder.

## Proportionality

Not every system needs commercial-grade mitigations. Calibrate to context:

- **DIY/hobbyist HA setup**: prioritize local fallback (don't depend entirely on cloud), tamper awareness is nice-to-have, network segmentation is a good practice to suggest.
- **Product sold to consumers**: supervision and tamper detection become close to expected-baseline; account security (MFA, remote-disarm anomaly detection) is important; consider UL 2 (line security / burglar alarm) relevant listings if going through professional monitoring.
- **Product integrating with real 911/emergency dispatch**: threat modeling must also cover *false-dispatch* risk (see `dispatch-integration.md`) — a spoofed or malicious trigger doesn't just fail to help the homeowner, it consumes real emergency-response resources and can carry liability.

## What's out of scope for this skill

Reasoning about *defending* a system is in scope. Specific step-by-step techniques for defeating a *named, real* commercial security product or lock (frequencies to jam a specific model, bypass codes for a specific panel) is not something to produce, regardless of framing — treat it the same as any other request for security-bypass instructions.
