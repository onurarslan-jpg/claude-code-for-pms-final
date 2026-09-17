# 04 · X-Ray Vision — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly. Last session the numbers showed you what happened: a
ping used to wait ninety seconds and now it waits sixty, people
missed pings they used to catch, and missing one counts the same as
turning one down — so four responders stopped hearing from us
altogether.

At the end of the session, ask Claude Code to save the prompts you
wrote yourself below — not the starter prompt. The closing slide has
the exact prompt to paste. By Module 6 this file is a prompt library
built from your own questions.

---

### 1.

45% distance vs. 40% reliability and 60% vs. 25%.
When I add both up, it makes 85%. Which variable is the remaining 15%?

### 2.

how do you explain while the distance gained more weight, some responders started to receive no calls? Are they never close to the incident location?

### 3.

Did the change to who gets asked first apply to responders who'd already been turning jobs down, or just new ones? Can't tell from the code and don't want to guess on this. Anyone know?

### 4.

can you share a chart of all respondents reliability scores log day by day since the beginning of August?

### 5.

Find me the part of this code that takes points off somebody when they miss a ping or turn one down. Show it to me and explain it in plain English. Then find me every single thing in this code that puts points back on.

### 6.

Somebody has been quiet for a month. Walk me through, step by step, exactly what would have to happen for them to start getting work again.

### 7.

does the proximity scoring work in a way that it gives full score to every responder with 45 minutes of reach?

### 8.

how does the reliability scoring work?

### 9.

can refetch the weekls chart and amount of received calls per respondent?

### 10.

can you also add their successfull answer ratio next to pings sent values?

### 11.

How do you explain why farlight and meteor mite and undertow and vesper started to receive this few calls?
Have they never been in reach of the incident so that by chance at least due to their high proximity scores by chance?

### 12.

how does the location data gets retrieved? by which devices and how frequent?

### 13.

let's ask Wen Li

### 14.

Before we wrap up, three things. First: look back through this session and find the prompts I wrote myself, not the starter I pasted. Save them into 04-x-ray-vision/prompts.md, one per numbered slot, exactly as I typed them. Don't tidy them up. Second: add a few lines to the Working context in CLAUDE.md, anything we figured out today that isn't in there yet and that I'd want you to already know next session. Third: commit everything that's changed with a short message describing what this session did, then push. Tell me when it's done and give me the link to my repository on GitHub.
