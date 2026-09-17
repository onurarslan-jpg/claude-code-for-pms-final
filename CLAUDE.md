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

### Root-cause mechanism, confirmed in code (session 3 — "Rewind")

Read the actual routing code (`00-rook/code/dispatch-routing/`). Ranking
score = 0.60·proximity + 0.25·recent-acceptance + 0.15·capability
(proximity was 0.45, recent-acceptance was 0.40, before 4.2). Two things
compound into a one-way trap for anyone pushed low: (1) a timeout is scored
identically to an active decline (`history.py: record_declined` fires on
`NO_ANSWER` too); (2) the recent-acceptance score never decays back toward
neutral on its own — open TODO since 2019, still unresolved — it only moves
via `record_accepted`, which requires actually landing an offer inside the
60s window. Combined: fewer offers → fewer chances to earn the score back →
any rare offer is more likely to time out anyway → score drops further,
permanently. The only ways a "gone quiet" responder gets pinged again: a
very close incident (proximity can override a bad score), the whole
higher-ranked candidate list declining first, a handler's manual routing
override (exists since 4.0, logged — see `06-sidekicks/briefs/
routing-override-audit-log.txt`), or an engineering/config change. Nothing
in the system recovers this on its own.

Same mechanism explains the aggregate recovery: self-sorting pulls
reliable/fast responders to the top over time, cutting wasted cascade
hops — why acceptance rate climbed 54%→66%→67%→73% with zero code changes
shipped after 4.2 (checked the changelog, nothing post-4.2). Behavioral
adaptation helped too — Aunt Dot's interview: she now tells Vesper's
partner to keep the phone in his pocket rather than upstairs, to beat the
60s window.

### Season vs. release (session 3)

Total weekly callout volume was flat-to-peak right through release week
(177 sent, the highest in the file) and only dropped starting the week
after — timing argues against Priya's "mostly seasonal" read being the main
driver. Also can't be pure seasonal demand softening, since that wouldn't
create winners — it's redistribution (winner cluster rising while loser
cluster collapses), not a uniform dip. The CSV only covers one season/one
year, so it can't confirm or refute "August is always soft" as a general
claim either way.

### New data-integrity anomaly: Nightwell (session 3)

Same pattern as the Ironvale anomaly, but starker. Three tickets (T-004,
T-009, T-011 — both Nightwell and her handler Marjorie Sung) say she's
getting almost nothing, filed during the exact weeks her CSV row shows her
as the single highest-volume, highest-accepting responder in the whole
dataset (16→18→20→21 sent/week, rising every week). Neither she nor her
handler has any way to see the backend record themselves. Likely a
device/notification-delivery issue (e.g. a stale second session answering
silently), not a routing problem — a Marcus/engineering question, separate
from the Wen Li ranking question.

### Routing mechanics, spelled out further (session 4 — "X-Ray Vision")

Proximity scoring is a linear falloff, not a flat pass/fail: `1.0 − (travel_minutes /
45)`, zero at 45 minutes or beyond. Two responders both "within range" can score very
differently — 5 minutes out and 40 minutes out are not treated the same. Capability
match is the unaccounted-for 15% in the 0.60/0.25 proximity/acceptance split
(`WEIGHT_CAPABILITY_MATCH = 0.15`) — unchanged since 4.0, not touched by 4.2.

Confirmed mechanically (not just inferred): there is no reset or migration logic
anywhere in `history.py`/`config.py`. A responder's recent-acceptance score carries
straight across a release boundary with no adjustment, so the 4.2 reweight applied
immediately to whatever score everyone already had — including responders who were
already low going in. Marcus asked exactly this on 14 Aug ("was that meant to apply
to responders who've been turning jobs down too, or only everyone else?") and never
got an answer — Wen Li said she'd look after PTO (back the 24th) and the thread
never picks it back up. Still open; drafted a message to send her, not yet sent.

Also asked Wen Li (not yet sent): how responder location/travel-time actually gets
retrieved for proximity scoring, which device/data source, how often it updates.
`availability.py`'s `available_for()`, `travel_time_minutes()`, and `current_record()`
are all unimplemented stubs — nothing in this repo documents the location pipeline,
so I can't independently verify whether the four "gone quiet" responders (Farlight,
Meteor Mite, Undertow, Vesper) were also just geographically unlucky versus it being
pure score collapse. Leaning toward score collapse, not geography: the four moved in
a synchronized, monotonically-worsening pattern starting the week after 4.2 (not the
noisy up-and-down you'd expect from geographic luck), and Kip's interview has Meteor
Mite and The Gale — same handler, same city, same weeks — moving in opposite
directions, which argues against "no nearby incidents" as the explanation.

### First-month framing (my own call, not something to re-litigate each session)

Decided not to make the 4.2 aftermath my sole focus. Running roughly in
parallel: (1) bounded 4.2 investigation — segmented acceptance data by
region, ticket-theme trend, resolve the decline-penalty question, land a
real recommendation at the September regroup; (2) relationship-building
with Helen/Marcus/Wen Li/Nadia/Ravi; (3) closing the missing
routing-logic documentation gap; (4) getting oriented on what's next —
confirm slipped Q3 items with Helen, start shaping the Q4 mutual-aid
exploration.
