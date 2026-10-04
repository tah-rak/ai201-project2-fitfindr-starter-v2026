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
The search uses keyword matching, so a reasonable request can be phrased in a
way that misses the listing data. Four successful runs still shows that the
full path usually works without pretending the search is more flexible than it
is.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
An empty result is deterministic once the filters have been applied. Stopping
before the next tool is also the safety rule for this branch, so it should work
every time rather than only most of the time.

---

## 3. Something about state

For 5 of 5 matching runs, the `id` and `title` in
`session["selected_item"]` must be identical to the `id` and `title` of the
item passed to `suggest_outfit`.

**Why this target:**
The selected item is a single listing that should move through the session
unchanged. Five out of five is appropriate because this is local state copying,
not model-generated behavior, so there is no reason to accept occasional loss
or substitution.



---

## 4. Something about the fit card

In at least 4 of 5 successful runs, the fit card must be 2 to 4 sentences and
must mention the selected item's title, price, and platform.

**Why this target:**
The model may choose different wording on different runs, so exact text is not
a fair requirement. These length and content checks keep the result useful as a
caption while allowing normal variation in model output.



---

## 5. Your choice

For 5 of 5 searches with a `max_price`, every returned listing must have a
`price` less than or equal to that limit.

**Why this target:**
The price ceiling is a direct numeric filter, so its behavior should be
consistent every time. A 5-of-5 target makes sure the agent never recommends an
item that violates the user's stated budget.



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
