# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Read and write only within this directory.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

Role: PM for **Rook Dispatch**, starting 31 August 2026. Predecessor Priya
Raghunathan left 21 August after 14 months; there was no handover overlap.

### The company

Rook Industries (founded 2014, ~241 people) builds coordination and
provisioning software for the protective-response sector — sold to
independently-operating "responders" and the handlers/quartermasters who
support them. Two products, both on a monthly 4.x release train, current
release **4.2**:

- **Rook Dispatch** (mine) — routes incidents to responders: ranks
  available responders, sends a callout offer, responder accepts/declines,
  offer times out and moves on if unanswered. Web console for handlers,
  native mobile for responders.
- **Rook Supply** — equipment provisioning: requisitions → quartermaster
  approval → fulfillment → maintenance schedules, plus field failure
  reports. Supply *reads* Dispatch's Responder Availability Record (to
  avoid scheduling maintenance during likely callout windows) but never
  writes to it.

Confidentiality: responder cover identities are never stored or derivable
in production — only capability tags, availability, and callout history.
Don't design anything that assumes we can map a cover identity to a legal
identity.

### Vocabulary

- **Responder** — accepts callouts, not a Rook employee.
- **Handler** — manages a responder's (or small group's) availability,
  gear, readiness; the day-to-day console user.
- **Quartermaster** — owns equipment stock/approvals (Supply side).
- **Callout / callout offer** — a request for a responder to attend an
  incident, presented to one responder at a time.
- **Callout timeout** — how long an offer stays live before moving to the
  next responder (cut from 90s → 60s in 4.2).
- **Routing priority** — the ranking score for a callout: proximity
  (travel-time), availability, capability match, and recent acceptance
  history. Declining/timing out lowers a responder's recent-acceptance
  component, which lowers their future priority.
- **Acceptance rate** — Dispatch's headline metric: share of offers
  accepted vs. declined/timed out, reported weekly in aggregate.
- **Time-to-accept**, **Coverage gap** (secondary metrics) — see
  `company/dispatch-one-pager.pdf` and `company/glossary.docx` for full
  definitions.
- **Capability tag** — flight, structural-entry, hazmat-tolerant,
  cold-weather, aquatic, crowd-management, de-escalation.
- **Responder Availability Record** — shared record written by Dispatch,
  read-only for Supply.
- Routing config ships as part of a release, not a runtime setting.

### People

- **Helen Achebe** — Director of Product (Dispatch & Supply), Chicago.
  Owns roadmap/commitments; changes to committed items go through her.
- **Marcus Oyelaran** — Engineering Manager, Dispatch, Chicago. First stop
  for anything unclear; can usually pull data.
- **Wen Li** — Staff engineer, built routing (Berlin). The only real
  source of truth on how ranking works — no adequate doc exists (was on
  PTO 14–24 Aug during the 4.2 aftermath).
- **Sofia Marino** — Product Designer, Dispatch (console + phone app),
  Chicago.
- **Nadia Hoffmann** — Support Lead, Dispatch & Supply, Berlin. Sees
  complaint/ticket volume first; worth a standing check-in.
- **Ravi Menon** — Data Analyst, Dispatch & Supply, Singapore. Weekly
  acceptance-rate numbers; requests go through #data.
- **Priya Raghunathan** — predecessor, departed 21 Aug 2026.

### Where things stand (as of 8 Sept 2026)

4.2 shipped 12 August with three bundled changes: (1) routing reweighted
toward proximity over recent-acceptance history (a long-standing,
deliberate ask — do not treat as a bug), (2) callout timeout 90s→60s, (3)
console filter persistence (cosmetic, generating noise tickets).

Since release, acceptance rate is down and support tickets are up ~3x,
split roughly 2:1 between "phone never rings" (unexplained) and "offer
gone before I could answer" (expected, caused by the timeout cut). Priya's
read: mostly seasonal (August is always soft) plus the timeout change;
she was wary of over-reading two weeks of tickets and did **not** want 4.2
reverted — the routing change traded one complaint for another
intentionally. Team agreed to regroup properly in September once my first
week is done. Nadia has been tracking the ticket-theme split; Marcus can
pull rough acceptance-rate numbers faster than Ravi's official weekly
report.

Open items Priya flagged: no written spec exists for how routing/ranking
actually decides who gets pinged (needs to be written); check with Helen
on which Q3-roadmap items got squeezed out of 4.2 and quietly slipped.

**Q3 roadmap** (committed items are locked; changes go through Helen):
- Dispatch 4.2 (shipped): change to who gets pinged, availability
  confidence score, ping timeout tuning — all committed.
- Supply 4.3: requisition approval chains — committed.
- Q4 (exploring, not committed): handler phone app (Supply), shared
  cover between responders / mutual aid (Dispatch).

Source docs: `00-rook/company/` (one-pagers, roadmap, release history,
glossary, team directory) and `00-rook/company/notes/` (Slack thread,
Priya's handover doc).

### Where to find things

- Support tickets: `00-rook/feedback/tickets/` (t-001 through t-025).
- Customer interviews: `00-rook/feedback/interviews/` (ambrose, aunt-dot,
  halloran, kip).
- Acceptance/callout data: `00-rook/data/callout-history.csv`.
- Routing code (Wen Li's): `00-rook/code/dispatch-routing/` — includes
  `routing.py`, `offer.py`, `history.py`, `availability.py`, `config.py`,
  plus a `CHANGELOG.md` and `README.md`.

### Working hypothesis on the "phone never rings" theme (revised — see data below)

Original hypothesis, not confirmed by ticket evidence: Marcus's unanswered
Slack question (does the recent-acceptance penalty apply to responders who
*decline* callouts, or does routing config not distinguish that from anyone
else?). No ticket actually shows a responder declining before going quiet —
plain proximity-reweighting redistribution looks sufficient on its own to
explain the theme (see callout-history findings below). Still worth
confirming the config detail with Wen Li, but don't lead with the
decline-penalty framing.

### Ticket + interview findings (analyzed 15 Sept)

- 25 tickets in `00-rook/feedback/tickets/` span 13 Aug–5 Sept, entirely
  post-release: 16 pure "quiet stretch," 4 pure "offer timed out," 5
  compound (quiet stretch, then lost the one offer that finally arrived —
  responders: Nightwell, The Undertow, Ironvale, The Drift, Cindermark).
  About a third of the quiet-stretch onsets pre-date the 12 Aug release.
- `00-rook/data/callout-history.csv` (weekly pings_sent/pings_taken, 29
  Jun–31 Aug) gives real pre/post numbers. Aggregate acceptance rate was
  stable ~76–78% for 6 pre-release weeks, crashed to 54% the week 4.2
  shipped, then has been recovering weekly (66% → 67% → 73%) — a
  shock-and-recovery, not a sustained decline.
- The non-recovering issue is volume concentration: callouts are piling
  onto a "winner" cluster (Vantage, Nightwell, The Gale, Stormwrack,
  Falkirk — pings rising every week) while a "loser" cluster collapses
  toward zero (Farlight, The Undertow, Vesper, Meteor Mite). Matches Kip's
  interview (The Gale vs. Meteor Mite) exactly, and is getting worse, not
  better, week over week.
- Anomaly to check with Wen Li/engineering: Ironvale's tickets say "no
  callouts since the 12th" three times, but her CSV row shows steadily
  rising pings_sent with a normal accept rate post-release — possibly a
  console display bug, not a routing issue.
- Ticket-filers and Sofia's four UX-interview subjects are almost entirely
  disjoint populations (only Captain Vantage/Ambrose overlaps). Tickets
  skew toward whoever's angry enough to complain; the interviews (UX
  research, not incident reports) undersold the severity even for
  directly-affected responders (Vesper, Meteor Mite). The CSV is the
  least-filtered source for "how bad is it, really."

### First-month framing (my own call, not something to re-litigate each session)

Decided not to make the 4.2 aftermath my sole focus. Running roughly in
parallel: (1) bounded 4.2 investigation — segmented acceptance data by
region, ticket-theme trend, resolve the decline-penalty question, land a
real recommendation at the September regroup; (2) relationship-building
with Helen/Marcus/Wen Li/Nadia/Ravi; (3) closing the missing
routing-logic documentation gap; (4) getting oriented on what's next —
confirm slipped Q3 items with Helen, start shaping the Q4 mutual-aid
exploration.
