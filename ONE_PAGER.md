# Elevance Health: fixing ghost networks with Exa

Elevance can check every provider listing against the live web each quarter, for under a cent of search per listing, and send outreach calls only to the listings that are wrong.

## The problem

Provider directories go stale faster than plans can check them, and members pay for it. Doctors move, change numbers, retire or stop taking patients; the listing stays. In a 2023 Senate Finance Committee secret-shopper study, callers booked an appointment with only 18% of listed mental health providers ([Senate Finance](https://www.finance.senate.gov/chairmans-news/wyden-calls-for-action-to-get-rid-of-ghost-networks-releases-secret-shopper-study)).

The exposure is legal as well as reputational:

- **Litigation.** Anthem plans are defendants in ghost-network class actions ([Mazzola v. Anthem Health Plans](https://litigationtracker.law.georgetown.edu/litigation/mazzola-et-al-v-anthem-health-plans-inc-et-al/), [Doe v. Anthem HealthChoice](https://litigationtracker.law.georgetown.edu/litigation/doe-et-al-v-anthem-healthchoice-assurance-inc-et-al/)). NBC News counted at least seven such suits in two years ([NBC News](https://www.nbcnews.com/health/health-news/patients-hit-dead-ends-insurance-ghost-networks-now-are-suing-rcna255976)).
- **Enforcement.** San Diego won a $5.25M judgment and an injunction against Molina over inaccurate directories ([City of San Diego](https://www.sandiego.gov/sites/default/files/2026-07/nr260713a.pdf)).
- **Regulation.** The No Surprises Act requires plans to verify directory data at least every 90 days and to remove providers they cannot verify ([KFF/AHLA summary](https://modern.kffhealthnews.org/wp-content/uploads/sites/8/2022/07/Provider-Directories-Federal-rules-article.pdf)).

Today that verification runs on provider attestations and outreach calls, one listing at a time. The data it starts from, such as the federal NPI registry, is self-reported and often years out of date.

## The solution

Use the web as the verification source, and call only the offices where the web says the listing is wrong. Each listing goes through three steps:

1. **Search.** One Exa search per provider finds the pages that describe where they practise today: the practice's own site, health-system provider profiles and dated professional profiles. Exa's query-focused highlights return just the passages about address, phone and new patients.
2. **Compare.** A model compares the listing with that evidence. First-party and health-system pages outrank aggregators, which are often stale copies. Sites that only republish the NPI registry are excluded from the search, because they would echo the listing back and falsely confirm it.
3. **Route.** Each listing comes back *confirmed*, *needs update* or *unverified*. Every field shows the stale value, the current value and the page it came from. Confirmed listings close with evidence on file. The rest go to the outreach queue with the evidence already attached.

The same search runs on a schedule, so the directory is re-checked continuously rather than in a 90-day scramble.

## What the prototype found

On 12 real psychiatrist listings in Indianapolis, where Elevance is headquartered, the prototype flagged 3 for update, confirmed 5 and left 4 unverified. The batch took 34 seconds and $0.084 of Exa search, $0.007 per listing. The listings came from the public NPI registry, standing in for an Elevance directory extract.

| Listing | Directory says | Web says | Evidence |
| --- | --- | --- | --- |
| Psychiatrist A | Indianapolis hospital address | Practising in Anchorage, Alaska | 2025 professional profiles; flagged for a confirming call |
| Psychiatrist B | Corporate campus, Indianapolis | Retired from that role in Dec 2021 | Dated professional profile |
| Psychiatrist C | Indianapolis university address | Practising in Boston | Health-system and aggregator profiles |
| Neurologist D (separate test) | Indianapolis hospital address and phone | A different clinic and phone | The health system's own provider page |

The 4 unverified listings matter as much as the corrections. Only aggregators agreed with them, so the prototype declined to call them confirmed. Results are leads for an analyst, not automatic edits to the directory.

## The business case

A web check costs about 5 cents a listing, so it pays for itself whenever an avoided verification call costs more than about 12 cents.

| Cost item | Per listing | Per 100,000 listings, each 90-day cycle |
| --- | --- | --- |
| Exa search, at $7 per 1,000 ([Exa pricing](https://exa.ai/docs/pricing)) | $0.007 | $700 |
| Model comparison (estimate, to be measured in the pilot) | about $0.04 | about $4,000 |
| **Total** | **about $0.05** | **about $4,700** |

The prototype confirmed 42% of listings (5 of 12) with web evidence. Each of those is a call not made, so the break-even is $0.05 / 0.42, or about $0.12 per call. The other gains are harder to price: corrections arrive weeks sooner, every decision has a source on file for regulators and litigation, and the outreach team starts each call knowing what changed.

## Proposed pilot

Run it for 30 days on behavioral health listings in one state, scored against outcomes Elevance already has.

|  |  |
| --- | --- |
| Scope | 5,000 behavioral health listings in one state; the area with the most litigation and member complaints |
| Elevance provides | A directory extract, plus listings its outreach team has already verified by phone, as the answer key |
| Exa provides | Search access, solutions-engineer support, and tuning of the query and source rules |
| Measured | Precision of *needs update*; share of listings confirmed without a call; cost per listing; time to flag |
| Success | 90% or better precision on *needs update*, and at least 30% of listings confirmed without a call |

Open questions for Elevance: what one verification call costs today, and whether directory corrections can be fed back automatically or need an analyst to sign off.
