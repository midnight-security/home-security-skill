# Contributing

This is a Claude Code skill: `SKILL.md` routes to reference files, and each reference file is the actual domain content Claude reads before answering. The bar for a change is whether it makes that domain content more accurate, more specific, or better organized — not longer.

## Adding or editing a reference

- Ground claims in real, named standards or conventions (CSAA, SIA, UL, NFPA, NG911, etc.) rather than plausible-sounding invented ones.
- Prefer concrete, actionable content (state machines, thresholds, worked examples, working YAML) over generic prose a model could already produce unprompted — the whole point of a reference file is the field-specific knowledge that generic instincts get wrong.
- Keep the "Out of scope" boundary in each file: this skill covers designing/building defensive systems, not techniques for defeating a *named* real product.
- If you add a new topic area, add a row to the routing table in `skills/home-security-engineering/SKILL.md` and, if relevant, a trigger phrase to its frontmatter `description`.

## Adding an example

Examples under `examples/` should be runnable/adaptable, not pseudocode, and should reference the specific reference-file concept they implement.

## Opening a PR

Small, focused PRs preferred. Explain what gap or inaccuracy the change addresses.
