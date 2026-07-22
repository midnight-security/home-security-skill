# Access Control, Smart Locks & Camera-Based Security

Use this when the user is designing physical access control (locks, keypads, credentials), video/camera security, or how either integrates with alarm and dispatch systems.

## Credential types and their threat profiles

Every access-control decision is really a choice about which credential class to trust, and each has a distinct failure mode:

- **PIN/keypad codes**: vulnerable to shoulder-surfing and informal sharing (a code given to a dog-walker doesn't stay with the dog-walker). Needs rate-limiting/lockout after repeated failed attempts, and per-user codes (not one shared code) so access can be revoked individually.
- **Physical keys/fobs (RFID/NFC)**: threat profile depends heavily on the underlying tech — legacy 125kHz proximity cards are trivially cloneable with cheap hardware; encrypted credential systems (MIFARE DESFire EV*, HID Seos) resist cloning. Don't assume "RFID" implies security; ask which generation.
- **Mobile/Bluetooth credentials (phone-as-key)**: exposed to BLE relay attacks (an attacker relays the phone's signal from far away to make the lock think the phone is nearby) — the same class of attack known from keyless-entry cars. Mitigate with proximity-verification (RSSI thresholds, or UWB for precise ranging) rather than simple "is the phone connected" logic.
- **Biometric (fingerprint, face)**: can't be rotated or revoked the way a code or fob can if compromised. Requires liveness detection (to resist photo/mask spoofing) and a documented fallback for false rejects — a biometric-only exterior door with no fallback is a lockout risk, not just a security one.
- **Multi-factor entry**: combining two different categories (something you have + something you are, or + a PIN) is the standard way to raise assurance on a single high-value door without over-relying on any one credential's weaknesses.

## Smart lock engineering concerns

- **Fail-safe vs. fail-secure behavior on power loss**: fail-safe (unlocks) is often required by fire/egress code on designated egress doors; fail-secure (stays locked) is typical for exterior perimeter doors where power loss shouldn't create a bypass. Get this backwards and you've either created a fire-code violation or a free entry point.
- **Auto-relock and jamming**: a lock that free-swings or is mechanically obstructed shouldn't self-report "locked" — verify actuator position, not just "command sent," the same supervision principle from `threat-modeling.md` applied to actuators instead of sensors.
- **The cylinder is still a lock**: a "smart" lock is frequently a standard pin-tumbler cylinder with an electronic actuator added. Bump-key and pick resistance of the underlying mechanical hardware still matters and shouldn't be assumed solved because the product is marketed as smart.
- **Remote-unlock audit logging**: every remote unlock should be logged with who/when/how (app, code, key), because for a lock — unlike most other IoT categories — an account-takeover translates directly into physical entry. This is the sharpest instance of the credential/account-takeover risk in `threat-modeling.md`.
- **Offline fallback**: mechanical key override or local Bluetooth-only unlock when Wi-Fi/cloud is down, so a connectivity outage doesn't become a lockout (or, worse, a fail-open).

## Camera-based security specifics

- **On-device vs. cloud AI detection**: on-device person/vehicle/package classification cuts latency and keeps footage from leaving the premises by default (privacy win); cloud processing is often more accurate and easier to keep current, but expands the breach surface described in `threat-modeling.md`. Many products use on-device as a pre-filter and only ship clips to the cloud for confirmed events.
- **Ties to verification**: camera-based detection is frequently the "video verification" step in `dispatch-integration.md`'s alarm-verification ladder. Be explicit about what a false "person detected" costs — informing the homeowner is low-stakes, auto-escalating toward a 911 dispatch is not, so keep the confidence bar proportional to what the detection triggers.
- **Local vs. cloud storage**: local (SD card/NVR) footage can be destroyed by an attacker who disables the camera itself; cloud storage survives that but centralizes footage in a way that raises breach/subpoena exposure. Local-first with selective cloud backup of flagged events is a common middle ground.
- **Interior camera placement is a distinct privacy question**, not just a security one — cameras covering bedrooms, bathrooms, or shared living spaces raise consent concerns for household members, guests, and domestic workers that exterior/perimeter cameras don't. Always-on-microphone doorbell cameras also intersect with two-party-consent recording laws in some jurisdictions.
- **Private-zone masking**: masking out a neighbor's window or shared property line in the field of view is good practice and, in some jurisdictions, an explicit requirement — worth raising proactively rather than only when asked.

## Access control system architecture

- **Centralized vs. distributed credential management**: a central panel that provisions every door controller allows instant revoke-everywhere (critical for a lost phone, a terminated household member, or the ex-partner/insider-abuse scenario in `threat-modeling.md`); independently-programmed locks are more resilient to a single point of compromise but require touching every lock to revoke access.
- **Guest/temporary credentials**: time-boxed codes and one-time codes for delivery or service access, with automatic expiry, are the standard mitigation for the same insider/abuse pattern — don't rely on manually remembering to delete a code later.

## Home Assistant implementation notes

- Use the `lock` domain (`lock.lock` / `lock.unlock`) and, for keypad locks, the `code_format` attribute rather than modeling codes as an ad hoc text helper.
- Camera-based person detection is typically wired through an add-on (Frigate, DeepStack, or similar) that exposes a `binary_sensor` (e.g., `person_detected`) — feed that sensor into the fusion automations described in `sensor-alarm-logic.md` rather than treating camera and door/motion sensors as separate systems.
- Treat a valid keypad code as a legitimate disarm signal, not just an RFID/NFC fob tap — an automation that only disarms on fob presence will false-alarm on every keypad-only entry.

## Out of scope

Same boundary as the other references: this is about designing legitimate access-control and camera systems, not a guide to defeating a *named* real lock, panel, or camera product — relay-attack tooling, cylinder-specific pick/bump techniques, or bypasses for a specific commercial system are out of scope regardless of framing.
