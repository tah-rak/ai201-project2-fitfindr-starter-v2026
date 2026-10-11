# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

FitFindr takes a plain-language request for a secondhand clothing item, such as
"a vintage graphic tee under $30," and searches the local listings data. It
chooses a matching listing, suggests ways to wear it with the user's wardrobe,
and writes a short caption for the find. If the search is empty, it explains
what the user could change and stops instead of sending an empty item to the
next tool.

---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Searches the listings data for clothing that matches the user's description, with optional size and price filters.
- **Inputs:** `description` (`str`), `size` (`str | None`), and `max_price` (`float | None`, in dollars).
- **Returns:** A list of matching listing dictionaries, each containing fields such as `id`, `title`, `description`, `size`, `price`, `colors`, `brand`, and `platform`, ordered from the best match to the weakest.
- **When it has nothing:** Returns an empty list (`[]`).

### `suggest_outfit`

- **What it does:** Suggests one or two outfits that work with the selected listing and the user's existing wardrobe.
- **Inputs:** `new_item` (`dict`) and `wardrobe` (`dict` with an `items` list).
- **Returns:** A non-empty outfit suggestion string with specific combinations from the wardrobe when wardrobe items are available.
- **When it has nothing:** If the wardrobe has no items, returns general styling advice for the listing instead of failing.

### `create_fit_card`

- **What it does:** Turns the selected item and outfit suggestion into a short caption suitable for sharing as a thrift find.
- **Inputs:** `outfit` (`str`) and `new_item` (`dict` containing the item's `title`, `price`, and `platform`).
- **Returns:** A two-to-four sentence caption that mentions the item, price, platform, and overall style or vibe.
- **When it has nothing:** If `outfit` is empty or only whitespace, returns a descriptive message instead of trying to create a caption.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If `search_listings` returns an empty list, the agent puts a message in the session explaining what the user could change and stops before calling `suggest_outfit`. Otherwise, it stores the first result as the selected item and passes it to `suggest_outfit`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regular expressions extract an optional `size` and `max_price`; the remaining words become the description used for searching.

**What moves through the session:** The query is stored first, followed by the parsed description, size, and price. Search results are stored next, then the selected listing moves into `suggest_outfit` with the wardrobe. The outfit suggestion and selected listing finally move into `create_fit_card`, and all results remain in the session.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask '...'

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([(item['title'], item['price']) for item in search_listings('graphic tee', max_price=30)])"
[('Mesh Long-Sleeve Top — Black', 15.0), ('Y2K Baby Tee — Butterfly Print', 18.0), ('Vintage Band Tee — Faded Grey', 19.0), ('Graphic Tee — 2003 Tour Bootleg Style', 24.0), ('Vintage Graphic Hoodie — Faded Black', 26.0), ('Low-Rise Cargo Pants — Khaki', 27.0)]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_empty_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_empty_wardrobe()))"
Here are two easy ways to style these classic vintage jeans: pair them with a white tee and canvas sneakers, or layer them with an open button-down and add simple accessories.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; item = load_listings()[0]; print(create_fit_card('Pair it with a white tee and vintage sneakers.', item))"
Nothing beats the broken-in feel of these Vintage Levi's 501 Jeans. I just scored them for $38 on Depop and paired them with a crisp white tee and retro sneakers for an effortless weekend uniform.
```

The first fit-card test returned the same cached answer on all three runs. I
then set `AI201_CACHE=0` for the process and ran the same command three more
times; the captions used different wording while still mentioning the item,
price, and platform.

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked AI to help me turn the tool specifications into
     the three standalone functions in `tools.py`, especially the keyword search,
     the empty-wardrobe case, and the empty-outfit case.
- *What came back:* It helped me break the work into a data-only search tool
     and two tools that call the existing `generate()` adapter. It also pointed
     out that a size filter should match complete labels, so `M` can match `S/M`
     without accidentally matching `XL`.
- *What I changed:* I used `load_listings()` instead of reading the JSON file
     directly, made no matches return `[]`, and added prompts that return general
     styling advice when the wardrobe is empty. I tested each tool separately
     before connecting them.

**Moment 2**

- *What I asked for:* I asked AI to explain how to build the planning loop in
     `agent.py` so the selected listing would move through session state rather
     than being passed directly from one function call to the next.
- *What came back:* It showed that the loop needed two different paths: an
     empty search should set an error and return, while a non-empty search should
     continue to the outfit and fit-card tools. It also helped me use regular
     expressions to extract the optional size and maximum price from the query.
- *What I changed:* I implemented the three stages in `run_agent()`, stored
     every result in the session, read those stored values for the next call, and
     checked both a successful query and an impossible query. The impossible
     query now leaves `fit_card` as `None` and tells the user what to change.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item state is preserved | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card contains item details | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Price ceiling is respected | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```
Run file: results/run_2026-10-11_1730_before.md
Produced by: run_eval.py::main

Criterion 1 — matching query completes:
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10
Fit card:
Channeling major early 2000s mall rat energy with this Y2K Baby Tee — Butterfly Print. I love balancing the fitted crop with baggy denim and chunky sneakers for an effortless weekend look. Snagged this gem for just $18 over on Depop!

Criterion 2 — impossible query stops early:
- stopped early: yes — No matching listings found. Try changing the description, size, or maximum price.
- selected_item: (none)
- search_results: 0

Criterion 3 — selected item state:
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10
Trace from agent.py::run_agent:
[1] search_listings (via MCP)
[2] suggest_outfit
[3] create_fit_card

Criterion 4 — fit card contains item details:
Channeling peak 2000s energy with this dreamy Y2K Baby Tee — Butterfly Print, paired with baggy dark-wash jeans and chunky sneakers for the ultimate casual street look. I just snagged it on Depop for $18, and it’s giving total early-2000s mall-rat nostalgia. Run, don't walk!

Criterion 5 — price ceiling:
Output from mcp_client.call_tool("search_listings", ...):
prices: [42.0, 24.0, 27.0, 30.0, 33.0, 38.0, 45.0]
over_budget: []
```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Matching query completes all three tools | 4 of 5 | MET (5/5) | All five matching runs returned search results, an outfit suggestion, and a fit card. |
| 2 | Impossible query stops before the second tool | 5 of 5 | MET (5/5) | All five empty searches stopped after `search_listings`, returned a useful change message, and left `fit_card` as `None`. |
| 3 | Selected item state is preserved | 5 of 5 | MET (5/5) | All five runs selected the same listing, and `run_agent()` reads that selected item from the session before calling `suggest_outfit`. |
| 4 | Fit card contains item details | 4 of 5 | MET (5/5) | Every fit card in the five runs stayed within the required length and mentioned the item, price, and platform. |
| 5 | Price ceiling is respected | 5 of 5 | MET (5/5) | All returned prices were at or below the requested $50 limit; the direct check produced an empty `over_budget` list. |

**Diagnoses**

No criteria were missed, so there is no repeated failure pattern to diagnose in
the tools, loop branch, session state, or model output. The two 4-of-5 targets
were conservative: criterion 1 allows for keyword-search phrasing to miss a
listing, and criterion 4 allows for normal variation in model-generated
captions. Both passed 5 of 5 in this run. Criterion 1 could reasonably be
tightened to 5 of 5 in a future revision, but the current target remains the
original target used for this evaluation.


---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```
[1] search_listings (via MCP)
     in:  dict with keys: description, size, max_price
     out: 10 items: Y2K Baby Tee — Butterfly Print, Vintage Band Tee — Faded Grey, Graphic Tee — 2003 Tour Bootleg Style … +7 more
     →    branch: results found, continuing
[2] suggest_outfit
     in:  dict with keys: new_item, wardrobe
     out: Here are two realistic, easy-to-pull-off outfits using the Y2K butterfly baby tee: **1. Y2K Streetwear Contra…
[3] create_fit_card
     in:  dict with keys: outfit, new_item
     out: Channeling peak 2000s energy with this sweet Y2K Baby Tee — Butterfly Print, paired here with baggy dark denim…
```

**Empty search**

```
[1] search_listings (via MCP)
     in:  dict with keys: description, size, max_price
     out: [] (empty)
     →    branch: empty, stopping
```

**On the MCP move:** `search_listings` is registered in `mcp_server.py` with
the same `description`, `size`, and `max_price` inputs documented in the Tool
Inventory. `agent.py` now calls it through `mcp_client.call_tool`; the server
returned the same list-of-dictionaries shape, so the existing empty-search
branch continued to work. The trace labels the call `search_listings (via MCP)`
so the transport change is visible.
---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:** I tightened the prompt in `tools.py::create_fit_card` so
the model must return only a natural two-to-four sentence caption with the
exact item title, price, platform, outfit, and style vibe. It must not add a
heading or bullet points.

**Which failure it was meant to fix:** The before run already met the fit-card
criterion, but the diagnosis identified model-output variation as the main
risk. This change was meant to make the required details and caption shape more
consistent without changing the loop or search behavior.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item state is preserved | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card contains item details | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Price ceiling is respected | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Did it help, and how do I know:**

It helped qualitatively, but it did not increase the numerical verdicts: the
before run was already 5 of 5 for every criterion, and the after run was also 5
of 5. The after fit cards were consistently caption-only two-sentence outputs
that included the required details, so the prompt became clearer without
changing search, branching, or session behavior.

After-run evidence from `results/run_2026-10-11_1853_after.md`, produced by
`run_eval.py::main`:

```
I just scored this Y2K Baby Tee — Butterfly Print on depop for $18.0. It gives off the ultimate early 2000s streetwear vibe when you style it with baggy straight-leg jeans, chunky white sneakers, and a black crossbody bag.
```



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
