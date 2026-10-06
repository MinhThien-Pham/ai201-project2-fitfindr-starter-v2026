# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
<!-- Why 4 of 5 and not 5 of 5? Something about your search, probably —
     "my search is a plain keyword match and some phrasings will miss" is a
     real answer. -->
My search is a plain keyword match, and the query is split up word by word, so
some phrasings will miss a listing that is really there. Two of the three
tools also call the model, so a rate limit or a failed call can stop an
otherwise good run.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->
This path never reaches the model. Parsing, the search and the empty-list check
are plain code, so the same query gives the same result every time. Any miss is
a bug in the branch, not randomness.

---

## 3. Something about state

<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->

Given a query that matches at least one listing, the finished session shows
the same listing `id` in three places: `session["search_results"][0]`,
`session["selected_item"]`, and the `item_id` recorded in `session["steps"]`
for both the `suggest_outfit` and the `create_fit_card` calls. The user types
the query once. This holds in 5 of 5 tries.

**Why this target:**
Passing the item along is plain dictionary code with no model involved. If
the ids differ even once, something overwrote the session or a tool got the
wrong value. That is a bug, so the target is 5 of 5.

---

## 4. Something about the fit card

<!-- YOU WRITE THIS ONE.

     The fit card calls a model, so the same input can produce different words
     each time. That's not a bug — it's the nature of the tool. So what would
     make it acceptable?

     Think about what you'd actually be unhappy to see. A caption that never
     mentions the price? Two different items producing the same opening
     sentence? A card longer than a caption anyone would post? Any of those can
     be turned into a number. -->

For 5 runs of `vintage graphic tee under $30` with the cache off, at least 4
of the 5 fit cards meet all of these:

- 2 to 4 sentences and at most 70 words.
- Contains the selected item's price as `$` and the number (e.g. `$18`).
- Contains the selected item's platform name.

Across the 5 cards, no two have the same first sentence.

**Why this target:**
The words are supposed to change (TEMPERATURE is 0.9), so I check facts and
length, not wording. The prompt asks for the price and platform, but the model
sometimes writes "18 bucks" or drops the platform, so I allow one miss in 5. A
repeated first sentence means the caption is acting like a template.

---

## 5. Your choice

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. Speed, the empty
     wardrobe path, what happens when the model can't be reached, whether the
     search respects a price ceiling — anything, as long as it names a number
     or an observable outcome. -->

For each of these 5 queries — `vintage graphic tee under $30`,
`90s track jacket in size M`, `platform sneakers size 8`,
`denim jacket under $50`, `vintage tee size S under $20` —
`session["parsed"]` holds the size and max price that were typed (None when
none was typed). Every listing in `session["search_results"]` has
`price <= max_price`, and its size contains the requested size as a whole
token (`S` matches `S/M` but never `US 9` or `XS`). Target: 5 of 5 queries.

**Why this target:**
A result outside the user's budget or size is worse than no result, because
the agent builds an outfit around something they can't buy. Parsing and
filtering are deterministic, so anything below 5 of 5 is a bug. Query 5
checks the `"s" in "us 9"` trap the starter warns about.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
