# Neighborhood Crime Data Integration

Use this when the user wants to incorporate local/neighborhood crime data — recent incidents, historical trends, or a "neighborhood safety score" — into a security product, onboarding flow, or risk assessment. This is a data-sourcing and responsible-use problem as much as an engineering one; get the sourcing wrong and the feature carries real legal and fairness risk that generic instincts won't flag.

## The source landscape: three very different categories

Not all "crime data" sources are the same kind of thing, and conflating them is the most common mistake:

1. **Official agency-published data** — data originated and verified by law enforcement, published either directly by the agency or through a vendor platform. Examples: **CrimeMapping.com** and **CityProtect** (both are vendor platforms — CentralSquare Technologies and Motorola Solutions respectively — that participating police departments opt into, not independent data collectors), **LexisNexis Community Crime Map** (LexisNexis Risk Solutions' free public-facing crime map, same agency-opt-in model), and the **FBI's UCR/NIBRS data** via the **Crime Data Explorer (CDE) API** (`api.usa.gov/crime/fbi/cde`, keyed through api.data.gov) — the national authoritative dataset these vendor platforms are often built on top of, and the only source in this group with an open, public, terms-friendly API.
2. **Crowdsourced/scanner-derived data** — **Citizen App** is the main example: built substantially from scanner-traffic parsing and user-submitted reports, not verified police reports. This data arrives faster and covers more granular "in progress" events, but has materially different accuracy and verification characteristics than agency-published data — the same unverified-vs-verified distinction that matters in `dispatch-integration.md` applies here.
3. **Commercial risk-scoring aggregators** — **LexisNexis Risk Solutions'** *commercial* products (distinct from its free public Community Crime Map) and comparable vendors (CAP Index, ATTOM, Verisk) compile crime and other risk signals into proprietary scores sold B2B, typically for insurance underwriting, tenant screening, or security consulting. This tier requires a data-licensing relationship and contract, not an API key you sign up for — budget for a sales cycle and a legal review of the license terms, not an integration sprint.

## Access reality: most of the named platforms don't have a public API for you

This is the part that trips up engineering estimates the most: **CrimeMapping.com, CityProtect, LexisNexis Community Crime Map, and Citizen do not offer public/free APIs for automated or bulk access**, and their terms of service generally prohibit scraping. Building a product feature by scraping these sites creates real exposure — ToS breach-of-contract claims and, in the US, potential CFAA exposure for circumventing access controls — and the underlying data typically isn't yours to redistribute even if you could extract it (it's the agency's data, licensed to the platform).

The legitimate paths, roughly in order of how ready-to-use they are:

- **FBI Crime Data Explorer (CDE) API**: free, public, official, has an actual API contract (key via api.data.gov). Good national/state baseline, but reporting participation varies by agency and coverage gaps are real (see below) — confirm current endpoint paths and response schema against FBI's own CDE documentation before building against it, since the API has changed shape across versions.
- **Municipal/state open-data portals**: many cities and states publish incident-level crime data directly with a real public API (frequently Socrata-based, e.g. NYC Open Data, City of Chicago Data Portal, LA's open data portal). Usually the best combination of granularity, freshness, and terms-friendliness — but it's per-jurisdiction, so multi-city coverage means integrating N different portals with N different schemas.
- **SpotCrime**: a genuine third-party aggregator with an actual commercial API product — worth evaluating as a paid alternative to hand-rolling municipal portal integrations, though it's still a proprietary feed you license rather than an open dataset.
- **Direct relationships with LexisNexis / CityProtect / CrimeMapping.com / their agencies**: possible, but goes through a business-development or data-licensing conversation with the vendor or the agencies, not a signup form — treat it the same way `dispatch-integration.md` treats PSAP relationships: a partnerships problem, not a pure engineering one.

## Known data-quality gaps to design around

- **NIBRS transition gap**: the FBI required agencies to move from the old UCR Summary Reporting System to NIBRS by 2021; a meaningful number of agencies (including some large ones) missed the transition and have incomplete or absent federal-level data for those years. Don't assume federal data has uniform national coverage without checking.
- **Reported crime undercounts actual crime substantially** — most crime, especially property crime and many violent-crime categories, goes unreported. Any "safety score" built from reported incidents is a score of *reporting*, not of *actual risk*, and should be framed to the end user that way.
- **Crowdsourced data has the opposite bias** — Citizen-style scanner/user-report data can over-represent incidents in areas with more active app users or denser scanner-traffic coverage, independent of actual crime rates.

## The bias risk this feature carries

Crime data is geographically concentrated by *policing patterns*, not just by crime itself — historically over-policed neighborhoods generate more recorded incidents, which a naive "neighborhood crime score" reflects as literally higher risk, reproducing and amplifying that bias. This is the same failure mode that's drawn well-documented criticism of predictive-policing tools and of crime-data-based insurance/lending risk scores functioning as a proxy for redlining. If a security product's onboarding, pricing, sensitivity defaults, or marketing copy key off a crime score, that's a fairness-review item, not just a data-engineering one — worth raising explicitly rather than treating "more data = better product" as self-evidently true here.

## How to actually use this in a security product

- **Feed it into risk *context*, not into alarm logic.** A "your neighborhood had elevated burglary reports last month" panel is a legitimate, useful onboarding/engagement feature. Treating an external crime feed as an input to actual sensor fusion or alarm escalation (see `sensor-alarm-logic.md`) is a category error — it's not a signal *from this household's own sensors*, and mixing it into trigger logic would mean a nearby-but-unrelated incident could raise this household's alarm posture without anything actually happening on their property.
- **Tie to proportionality, same as `threat-modeling.md`**: elevated local crime data is a reasonable input into recommending professional monitoring, additional sensors, or higher-sensitivity defaults at install time — a genuinely useful, defensible use of the data.
- **Prefer normalized rates over raw counts** for anything user-facing (incidents per 1,000 residents, or a trend delta) — raw incident counts read scarier than they are and are the same engagement-via-fear pattern that's drawn criticism of apps like Citizen; a security product doesn't need to borrow that dynamic to be useful.
- **Don't expose incident-level detail that could identify a specific household** (e.g., a domestic-violence call at a specific address) — aggregate to block/neighborhood level, not address level, for anything surfaced to other users.

## Out of scope

This reference is about sourcing and using crime data responsibly for a legitimate product feature. It's not a guide to circumventing ToS or access controls on a specific named platform (Citizen, CityProtect, CrimeMapping.com, LexisNexis, or any other) to scrape data you're not licensed to use — treat that the same as any other request to bypass a system's access restrictions.
