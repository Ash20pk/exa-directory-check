# Directory Check

**Prospect:** Elevance Health (Anthem), on the Fortune 500.
**Problem:** stale provider directories, known as "ghost networks."
**Built on:** the Exa Search API.

## The problem

Health-plan directories list doctors who have moved, changed numbers, retired or stopped
taking patients. Members call and hit dead ends; in mental health this is the subject of
a wave of class actions. Anthem plans are defendants in
[Mazzola v. Anthem Health Plans](https://litigationtracker.law.georgetown.edu/litigation/mazzola-et-al-v-anthem-health-plans-inc-et-al/)
and [Doe v. Anthem HealthChoice](https://litigationtracker.law.georgetown.edu/litigation/doe-et-al-v-anthem-healthchoice-assurance-inc-et-al/).
NBC News counted [at least seven such suits in two years](https://www.nbcnews.com/health/health-news/patients-hit-dead-ends-insurance-ghost-networks-now-are-suing-rcna255976),
and San Diego won a [$5.25M judgment against Molina](https://www.sandiego.gov/sites/default/files/2026-07/nr260713a.pdf)
over the same failure. Plans must re-verify their directories regularly, and in practice
that means outreach calls and provider attestations, one listing at a time.

## What it does

1. **Load listings.** Real providers from the CMS NPI Registry, the self-reported data
   that directories are built from. In production this would be Elevance's own directory,
   via its public Provider Directory API.
2. **Check each listing.** One Exa search per provider finds the pages that describe
   where they practise today: the practice's own site, health-system profiles and dated
   professional profiles. Query-focused highlights pull just the address, phone and
   new-patient passages.
3. **Compare and cite.** An LLM compares the listing with that evidence. It weights
   first-party pages over aggregators and returns *confirmed*, *needs update* or
   *unverified*. For each field it shows the stale value struck out and the current
   value beside it, with the sources behind it.

A first real run on 12 Indianapolis psychiatrists flagged 3 for update. One had moved
to Anchorage, one had retired from a corporate role in 2021, and one now practises in
Boston. It cost $0.007 of Exa search per listing and took about 35 s for the batch.

## Design decisions an Exa customer would care about

* **Circular evidence is excluded.** Sites that republish the NPI registry would simply
  echo the listing back and "confirm" it. They are passed to Exa as `excludeDomains`.
* **Source hierarchy.** Aggregators (Healthgrades, WebMD and the like) are often stale
  copies. The check trusts the provider's own and health-system pages first, and says
  "unverified" rather than "confirmed" when only aggregators agree.
* **Unit economics.** At $7 per 1,000 searches, re-checking a 100,000-listing directory
  every quarter costs about $700 in search, against outreach calls priced per listing.

## Run it

```sh
cp .env.example .env      # EXA_API_KEY, OPENROUTER_API_KEY
set -a; source .env; set +a
uv run uvicorn app:app --port 8000
open http://localhost:8000
```

The LLM step goes through OpenRouter (default `anthropic/claude-opus-5.5`; set
`LLM_MODEL` to change it).

## Next steps for a pilot

* Run against an Elevance directory extract and score the results against outcomes
  already known from its call-centre verification, to measure precision.
* Monitor the in-network roster continuously, rather than checking it in quarterly batches.
* Add new-patient availability and telehealth checks, the fields members complain
  about most in mental health.
