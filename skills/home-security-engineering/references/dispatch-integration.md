# Emergency Dispatch Integration

Use this when the user is connecting a security system to emergency services — 911, monitoring centers, or thinking about how an alarm becomes a real-world response.

## The three dispatch architectures

It's important to get the user to be explicit about which model they're building, because the tradeoffs (cost, liability, speed, false-dispatch risk) differ a lot:

1. **Self-response only (no dispatch)**: system notifies the homeowner/app only; homeowner decides whether to call 911 themselves. Lowest cost, no false-dispatch liability, but slowest and fully dependent on the homeowner seeing and acting on the alert.
2. **Central (third-party) monitoring station**: alarm signal routes to a UL-listed / CSAA-member monitoring center staffed by human operators, who attempt to verify (often by calling the home, sometimes via video/audio) before contacting 911 dispatch on the homeowner's behalf. This is the traditional ADT/Vivint-style model. Adds a human verification step, which reduces false dispatch but adds latency (industry response-time targets are commonly discussed in the tens-of-seconds-to-low-minutes range for signal receipt to operator action, though real-world timing varies by provider and event type).
3. **Direct-to-911 / NG911-integrated dispatch**: system or its backend sends an alert directly into the 911 system rather than routing through a human monitoring center first. This is only possible where the receiving 911 center has **Next Generation 911 (NG911)** infrastructure — IP-based call handling that can accept richer inputs (structured data, location, sensor context) rather than only analog voice calls. Companies like RapidSOS operate as data platforms that let connected devices and apps push emergency data into NG911-capable dispatch centers' existing workflows. This model can cut the human-relay step out of the latency chain, but raises the bar on **false-dispatch prevention** since there's no human monitoring-center operator as a verification backstop before real emergency resources are engaged.

## Alarm verification levels

Independent of which architecture is used, "verification" is the key concept determining how confidently a system can justify dispatch:

- **Unverified/unconfirmed**: raw sensor trip only, no corroboration. Many jurisdictions have "verified response" policies where police deprioritize or in some cases won't dispatch to unverified residential alarms at all, because historically the overwhelming majority of raw alarm trips are false.
- **Enhanced call verification (ECV)**: monitoring center attempts to call the premises (and sometimes a backup contact) before dispatching, to rule out user error (forgot to disarm, tripped their own sensor).
- **Audio verification**: two-way audio at the panel/hub lets a monitoring operator (or in some designs, an algorithm) listen for corroborating sound (breaking glass, voices, distress) before escalating.
- **Video verification**: camera clip or live feed reviewed by a human (or AI pre-filter flagging likely-real events for human review) before dispatch — increasingly the industry direction because it gives dispatchers actionable context (number of intruders, weapons visible, etc.) that pure audio/data can't.
- **Sensor-fusion-based verification**: see `sensor-alarm-logic.md` — using multiple corroborating sensors as an automated substitute/supplement for human verification, common in modern AI-driven security products aiming to reduce the monitoring-center bottleneck without simply lowering the bar for dispatch.

The industry framing worth knowing: **CSAA (Central Station Alarm Association)**, together with SIA, maintains alarm-response-priority standards used in many jurisdictions (frequently referenced as the **ANSI/SIA CS-V-01** verified-response categories), which grade an alarm signal by verification level and this grading affects how law enforcement prioritizes the response. A product claiming "faster dispatch via AI verification" is implicitly making a claim about where it sits on this verification ladder — worth making explicit when helping someone position or design such a feature.

## What real dispatch integration requires (beyond the tech)

If a user is actually building (not just designing) a direct-to-911 or monitoring-relationship feature, flag that this is not a pure software integration — it typically involves:

- **Relationships/agreements with PSAPs (Public Safety Answering Points)** or with an NG911 data platform that already has those relationships, since individual 911 centers can't be integrated with one-by-one from scratch by most companies.
- **UL/FM listing or CSAA/SIA compliance** if operating as or contracting a monitoring center — this is a regulated space in most US states (alarm monitoring company licensing).
- **False-alarm ordinance awareness**: many cities require permits for monitored alarm systems and fine repeat false-alarm addresses; some have adopted verified-response policies (won't dispatch police to an unverified alarm at all). A product's value proposition and legal obligations both hinge on this local-jurisdiction variability.
- **Liability framing**: a system that dispatches (or fails to dispatch) carries different liability exposure than one that merely notifies. This is worth flagging as a "loop in legal/insurance" item rather than something to solve purely at the engineering layer — this skill can help think through the technical design, not provide legal advice.

## Latency budget thinking

When discussing "speed" as a differentiator (a common pitch in this space), it's useful to break down where time actually goes, since marketing claims often compress a multi-stage pipeline into one number:

1. Sensor detection → local processing/fusion decision (device/edge latency)
2. Local hub/device → cloud or dispatch-platform transmission (network latency)
3. Verification step, if any (human monitoring center call/video review, or automated fusion-based verification) — historically the largest and most variable chunk
4. Platform → PSAP/dispatch handoff (data platform or phone-relay latency)
5. PSAP → responding units dispatched (outside the product's control entirely)

A product that removes or automates step 3 (verification) is making the strongest speed claim, but also carries the most false-dispatch risk — tie any "speed" positioning back to how the corresponding verification/false-alarm-reduction problem (see `sensor-alarm-logic.md`) is being solved, since the two are directly in tension.

## Out of scope

This reference is about designing legitimate emergency-dispatch integrations. It's not a guide to interfacing with or spoofing real 911/PSAP systems outside an authorized commercial/data-platform relationship — treat requests that drift toward "how do I send signals into the 911 system without going through an authorized provider" as outside scope.
