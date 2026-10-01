# Standing — what we'd build instead of changing the number

**Draft one-pager for Helen · 22 Sept 2026 · Onur, PM Dispatch**
Companion prototype: `prototype.html` (open in a browser)

---

## The thing we'd be papering over

A responder's recent-acceptance score is the only part of routing that
remembers. It goes down when we ask and don't get a yes, it goes up only
when they take a job, and it never moves on its own. `history.py` has
carried Wen's open question about that since 2019.

4.2 didn't create the trap, it just narrowed the escape route: proximity
went 0.45 → 0.60, acceptance 0.40 → 0.25, and the answer window went 90s →
60s. Fewer offers, less time to answer each one, and a score that only
recovers by answering one.

**Ship the small fix.** Stop scoring a missed phone the same as a refusal —
it's a real defect, it's correct on its own merits, and it doesn't depend on
anything below. My first draft said hold it and ship it alongside this in
4.3. That was wrong, and it was me protecting the proposal rather than the
product. Let it go this afternoon.

What it doesn't do is change the experience. Nobody in this story ever finds
out what happened to them. That's the separate, slower thing worth building,
and it isn't ready — see the open questions at the end, which are real.

## Who it happens to

**Kip**, handler for Meteor Mite and The Gale. Same city, same weeks, two
cards side by side on his center monitor:

| Week | Meteor Mite | The Gale |
|---|---|---|
| 3 Aug | 11 offers, 8 taken | 13 offers, 10 taken |
| 10 Aug (4.2 ships 12th) | 10 offers, 4 taken | 15 offers, 9 taken |
| 17 Aug | 4 offers, 1 taken | 17 offers, 12 taken |
| 24 Aug | 2 offers, 0 taken | 20 offers, 14 taken |
| 31 Aug | 1 offer, 0 taken | 21 offers, 16 taken |

In his own words: *"two cards on the same screen that might as well be two
different products… I don't have anything better than 'hang in there, it'll
pick up.'"*

**Meteor Mite** is texting Kip asking if something's broken. Nothing is
broken. Mite is being told, silently and correctly, that Mite is ranked
low — and there is no realistic path back, because the path back is
answering an offer that isn't coming.

Neither of them can see the score. Neither can Nadia's support team.

## What we'd build

Three pieces. None of them is a config value.

**1 · Standing, visible, in words.** The coverage card stops being a ping
count and starts being an explanation. *"Meteor Mite is being ranked below
most nearby responders. Six missed offers since 10 Aug. Standing is low and
isn't recovering on its own."* Kip's ask was to make it "look less like a
coincidence at midnight." This is that, and it's most of the value.

**2 · A guaranteed look.** When a responder drops below standing and stays
there with almost no offers for two weeks, Dispatch deliberately sends them
the next suitable callout regardless of rank — and gives that one 90
seconds, not 60, because they're cold. Take it and normal credit applies
and they climb out. This is the honest answer to Wen's 2019 question: not
automatic decay (which would quietly re-rank people who really have
stopped working, exactly what she was worried about), and not permanence
either. You earn it back, but we make sure you get the chance to.

**3 · The responder is told.** Today Meteor Mite's phone says nothing, so
Mite texts Kip to find out whether the product is broken. Instead: a
standing line in the app, what it means, when the next guaranteed look
lands. A quiet week you understand is a completely different experience
from a quiet week you don't. Dot wants the same alert on her laptop — she's
currently listening for a buzz through a ceiling.

Piece 1 is the one I'd ship first. I'd argued it was only worth doing
alongside piece 2; Dot's "I don't need to do anything about it, I'd just
like to know" says knowing has value on its own, and she's the user.

## Why this and not decay

A decay constant fixes Mite's number and teaches us nothing. We would ship
it, the aggregate would tick up, and the next time routing re-ranks someone
into silence — mutual aid in Q4 will do exactly this — we'd be back here
with no visibility and no recovery path, just a different constant to argue
about. Standing is the durable thing: it survives the next reweight.

## The one piece of direct evidence

Kip never asked for this. Asked what would help, he said *"I don't even know
what I'd want it to tell me… I'm not the person to ask about that part."*
Standing is my inference from his problem. What he asked for, four times, is
dark mode.

Aunt Dot, who handles Vesper — another responder who went quiet — did ask
for it, unprompted, three weeks ago: *"maybe something that tells me too, not
just him. **I don't need to do anything about it. I'd just like to know.**"*
She also said *"I hadn't really put those two things next to each other
before, the quiet weeks and the fast-vanishing ones"* — she's been living
both halves of 4.2 and couldn't see they were one story. That's the case for
this, and it's hers, not mine.

She changes the design too: she works standing at a kitchen counter and
can't read the console without leaning in over wet hands. And she never
filed a ticket. Vesper went from 14 callouts a week to 1 and she just
adapted. The ticket pile would never have found her.

## Open questions — the honest list

These are in the prototype under "What we don't know." Shortest version:

- **Blocker.** `push_to_device()` returns nothing; Dispatch has no delivery
  confirmation. Every "11 offers reached you" here is a claim the system
  can't make — and if Nightwell and Ironvale are delivery faults, standing
  would show them a well-designed lie. Marcus, and it's a prerequisite.
- **Unpriced.** A guaranteed look is taken from the incident. Mite at 34
  minutes instead of someone at 6, plus 90 seconds before the cascade. That's
  a safety trade and I've costed it at zero.
- **Scale.** My own threshold flags Farlight, Meteor Mite, The Undertow and
  Vesper at once — 4 of 16. That's a second routing policy, not a valve.
- **Undefined.** "The next callout Mite can do" isn't a concept routing has:
  capability is partial credit, ranking never filters, callouts carry no
  severity.
- **Cost.** This lowers acceptance rate on purpose — Ravi's weekly number.
  Needs a segmented metric or an explicit agreement.
- **Scope.** The 90s partially reverses a committed 4.2 decision. Your call,
  not an implementation detail.
- **Untested.** Neither Kip nor any gone-quiet responder has seen this. Mite's
  phone screen is my invention of how that week feels.
- **Not asked.** Wen hasn't seen it, and it answers her 2019 note by name.

Recovery arithmetic is illustrative; thresholds are placeholders. Dark mode
in the mock is only because that's how Kip works at 11pm — the real ask is
Sofia's.

## Ask

Ship the arithmetic fix now, decoupled from all of this.

Then, before anything else: Marcus on delivery confirmation — it's the
prerequisite here and the answer to Nightwell and Ironvale either way. One
consolidated message to Wen. Ravi to check the 4-of-16 number against real
scores. Sofia on the card, with Dot's setup as a constraint.

Real recommendation at the September regroup, with the investigation and
this as one story.
