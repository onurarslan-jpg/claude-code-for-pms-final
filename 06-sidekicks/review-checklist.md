# Review checklist — briefs

A reusable check for any brief before it goes further. Point Claude at this
file plus the brief and ask it to run the check — same four questions,
every time, no need to re-explain.

## The checklist

1. **Owner named** — does the brief say who owns it? Not "the team," a
   person.
2. **Success defined** — does it say how we'll know it worked? A metric,
   a signal, a threshold — not just a description of the fix.
3. **Scope holds** — does the scope stated at the end match the scope
   stated at the start? Flag anything that crept in or got quietly
   dropped between the opening framing and the closing ask.
4. **Problem before fix** — does it explain the problem before it
   proposes the solution? Flag any brief that opens with the fix and
   backfills the problem afterward.

## How to use it

Give Claude the brief and say: "Run this against the review checklist in
`06-sidekicks/review-checklist.md`." Expect a pass/fail on each of the
four items, with a one-line reason for any fail, not a rewrite of the
brief.
