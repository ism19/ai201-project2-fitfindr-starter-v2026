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

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr helps someone thrifting to decide whether a thrift find works for them. The user types what they're looking like "vintage graphic tee under $30 size M". FitFindr searches the listings for the best match, then suggests one or two outfits that pair it with pieces from the user's wardrobe. Then it writes a short post caption for the find.


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

- **What it does:** 
Searches the listings for items matching a description, size, and price limit
- **Inputs:** <!-- name and type each: `max_price` (float), not "a price" -->
description(str)
size (str)
max_price (float)
- **Returns:**
A list of matching items as dicts in order of greatest match
- **When it has nothing:**
Returns an empty list 

### `suggest_outfit`

- **What it does:** 
Suggests outfits for the user given the user's thrift find by matching pieces from their wardrobe or suggests general styling tips if wardrobe isn't shared
- **Inputs:** 
new_item (dict)
wardrobe (dict)
- **Returns:**
Returns a string that describes the outfit suggestion for the new thrift find
- **When it has nothing:**
If no outfit ideas could be generated, a string explaining that no outfit ideas were generated for the new item is returned

### `create_fit_card`

- **What it does:** Generates a social media caption for the new thrift find using the generated outfit description
- **Inputs:** 
outfit (str)
new_item (dict)
- **Returns:**
Returns a 2-3 sentence caption describing the find in a natural way such as mood/style.
- **When it has nothing:**
If the caption comes as an empty response it returns a string that says no caption could be generated for the find. If the outfit is passed as an empty param, it tells the user to run suggest_outfit.

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

**Branch rule:** If `search_listings` returns an empty list, put a message in `session["error"]` that names what was searched and what to loosen (raise the max price, drop the size, or use broader words), and stop without calling `suggest_outfit`. Otherwise, take the first result as `session["selected_item"]` and go to `suggest_outfit`, then `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex, in `agent.py::parse_query`. The price comes from phrases like "under $30", "max 40" or a bare "$25". The size is whatever follows the word "size" ("size M", "size S/M", "size US 8.5", "size W30 L30"). The description is what's left after removing those and filler like "looking for" or "I need". A size not preceded by "size" (e.g. "one size bucket hat") is not picked up and stays in the description.

**What moves through the session:** `query` → `parsed` (description, size, max_price) → `search_results` → `selected_item` → `outfit_suggestion` → `fit_card`. Each tool reads its input from the session and writes its result back. `error` is set only when the run stops early, and the later fields stay `None`.

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

`search_listings`: returns tees first and nothing over $30. Weaker matches that only mention "tee" in their description rank last.

```
$ python -c "from tools import search_listings; print([(x['title'], x['size'], x['price']) for x in search_listings('graphic tee', max_price=30)])"

[('Graphic Tee — 2003 Tour Bootleg Style', 'L', 24.0), ('Y2K Baby Tee — Butterfly Print', 'S/M', 18.0), ('Vintage Band Tee — Faded Grey', 'L', 19.0), ('Vintage Graphic Hoodie — Faded Black', 'L', 26.0), ('Mesh Long-Sleeve Top — Black', 'S/M', 15.0), ('Low-Rise Cargo Pants — Khaki', 'W29', 27.0)]
```

`suggest_outfit`: with the example wardrobe, every piece it names is one the user owns.

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

**Outfit 1: Casual Streetwear**
- **New Item:** Vintage Levi's 501 Jeans
- **Owned Pieces:** White ribbed tank top, Vintage black denim jacket, Chunky white sneakers, Black crossbody bag

**Why it works:** The white ribbed tank tucked into the medium-wash 501s creates a classic, effortless base. Layering the slightly cropped vintage black denim jacket adds edge, while chunky white sneakers and the black crossbody bag tie the vintage streetwear aesthetic together seamlessly.

***

**Outfit 2: Cozy & Relaxed**
- **New Item:** Vintage Levi's 501 Jeans
- **Owned Pieces:** Oversized grey crewneck sweatshirt, Brown leather belt, Black combat boots

**Why it works:** Tucking the front of the oversized grey crewneck into the 501s balances the volume of the top with the straight-leg fit of the denim. The brown leather belt adds a polished detail, and black combat boots ground the relaxed look with a touch of toughness.
```

`create_fit_card`: run three times on the same item with the cache off (`AI201_CACHE=0`). The three captions are different, so TEMPERATURE (0.9) is doing its job. With the cache on, all three came back word-for-word identical.

```
$ AI201_CACHE=0 python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Nothing beats finding the holy grail of denim—these vintage Levi's 501s have the absolute best fading at the knees. I kept it classic for the fit today, just throwing them on with some fresh white sneakers for that effortless 90s streetwear vibe. Snagged this exact pair on depop for $38 and I honestly might never take them off.

$ (run 2)
Scored these vintage Levi's 501s on depop for just $38 and I'm honestly obsessed with the knee fading. Keeping it super effortless today by pairing them with crisp white sneakers for that ultimate 90s running-errands vibe.

$ (run 3)
Living in these vintage Levi's 501s lately—that perfect medium wash and knee fading give off the ultimate effortless streetwear vibe. Just kept it classic today with a crisp white tee and my go-to beat-up sneakers. Snagged them on Depop for just $38 and I honestly don't think I'll ever take them off.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I wrote the helper functions for `search_listings` (`_keywords`, `_size_tokens`, `_size_matches`) and asked Claude for help getting the search working.
- *What came back:* Claude found two bugs in my `_size_tokens`. My regex `r"\n([^)]*\)"` matched a newline instead of `(`, so parentheticals like "(oversized)" were never removed. And `.upper` was missing its `()`. It also suggested scoring keyword matches in the title twice as high as matches elsewhere.
- *What I changed:* I fixed the bugs Claude pointed out.

**Moment 2**

- *What I asked for:* Help writing the prompts for `suggest_outfit`with the case where the user hasn't shared a wardrobe.
- *What came back:* Claude suggested two prompts. The empty-wardrobe one asks for general styling advice and tells the model not to pretend it knows what the user owns. The other lists the user's pieces and tells the model to use only those. It also suggested small helper functions to turn the item and wardrobe dicts into readable text for the prompt. Both system prompts started with "You are a thrift stylist."
- *What I changed:* I changed "thrift stylist" to "clothing stylist". I also moved the helper functions above `suggest_outfit`.

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
| 1.  Full three-tool run returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET 5/5 |
| 2. Empty search stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET 5/5 |
| 3. Item in session matches item passed on | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET 5/5 |
| 4. Fit card 2-4 sentences, includes price and platform | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET 5/5 |
| 5. Every result matches size and is at or below price | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET 5/5 |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

**Criterion 1 — matching query completes** (`vintage graphic tee under $30`, Try 1)

Produced by `tools.py::search_listings` (via MCP), `tools.py::suggest_outfit` and `tools.py::create_fit_card`.

```
- stopped early: no
- selected_item: Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
- search_results: 10

[1] parse_query
      in:  vintage graphic tee under $30
      out: {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
[2] search_listings (VIA MCP)
      in:  {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
      out: 10 items: Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey, Vintage Graphic Hoodie — Faded Black … +7 more
      →    10 match(es)
[3] select_item
      out: Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
[4] suggest_outfit
      in:  Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
      out: **Outfit 1: 90s Grunge Streetwear** * Graphic Tee * Baggy straight-leg jeans, dark wash * Black combat boots *…
[5] create_fit_card
      in:  Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
      out: Obsessed with how perfectly worn-in this 2003 tour bootleg tee is, which I actually scored on Depop for just $…
```

**Criterion 2 — impossible query stops early** (`designer ballgown size XXS under $5`, Try 1)

Produced by `tools.py::search_listings` (via MCP), which returned `[]`.

```
- stopped early: yes — No listings matched 'designer ballgown' in size XXS under $5. Try to raise your max price, or drop the size filter, or use broader words (e.g. 'dress' instead of 'ballgown').
- selected_item: (none)
- search_results: 0

[1] parse_query
      in:  designer ballgown size XXS under $5
      out: {'description': 'designer ballgown', 'size': 'XXS', 'max_price': 5.0}
[2] search_listings (VIA MCP)
      in:  {'description': 'designer ballgown', 'size': 'XXS', 'max_price': 5.0}
      out: [] (empty)
      →    0 match(es)
[3] branch
      →    search returned []: stopping here before suggest outfit
```

**Criterion 3 — selected item passes through unchanged** (`denim jacket size M under $50`, Try 1)

Produced by `tools.py::search_listings` (via MCP), whose first result went into `tools.py::suggest_outfit` and `tools.py::create_fit_card`.

```
- stopped early: no
- selected_item: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
- search_results: 3

[1] parse_query
      in:  denim jacket size M under $50
      out: {'description': 'denim jacket', 'size': 'M', 'max_price': 50.0}
[2] search_listings (VIA MCP)
      in:  {'description': 'denim jacket', 'size': 'M', 'max_price': 50.0}
      out: 3 items: 90s Track Jacket — Navy/White Stripe, Denim Vest — Medium Wash, Studded, Shacket — Olive Canvas
      →    3 match(es)
[3] select_item
      out: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
[4] suggest_outfit
      in:  90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
      out: **Outfit 1: 90s Streetwear** *   **Top:** White ribbed tank top *   **Outerwear:** 90s Track Jacket — Navy/Whi…
[5] create_fit_card
      in:  90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
      out: Scored this vintage Champion track jacket for $45 on Poshmark and I’m obsessed with how lightweight it is. I p…
```

**Criterion 4 — fit card has price and platform** (`vintage tee size M under $25`, Try 1)

Produced by `tools.py::create_fit_card`.

```
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 7

Fit card:
Obsessed with this little butterfly tee I just scored on Depop for $18! Paired it with some baggy dark wash denim and my go-to zip hoodie for the ultimate Y2K street fit, but it’s also so cute dressed down with wide-leg khakis.
```

**Criterion 5 — search respects price and size** (`vintage tee size M under $25`, Try 1)

Produced by `tools.py::search_listings` (via MCP).

```
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 7

[1] parse_query
      in:  vintage tee size M under $25
      out: {'description': 'vintage tee', 'size': 'M', 'max_price': 25.0}
[2] search_listings (VIA MCP)
      in:  {'description': 'vintage tee', 'size': 'M', 'max_price': 25.0}
      out: 7 items: Y2K Baby Tee — Butterfly Print, Vintage Polo Shirt — Forest Green, Vintage Knit Vest — Argyle Brown/Cream … +4 more
      →    7 match(es)
```

The run log only names the first 3 results, so I listed all 7 by calling the
same search through `mcp_client.py::call_tool` with the same inputs:

```
Y2K Baby Tee — Butterfly Print                size='S/M'                        $18.0
Vintage Polo Shirt — Forest Green             size='M'                          $18.0
Vintage Knit Vest — Argyle Brown/Cream        size='M'                          $25.0
Leather Belt — Brown, Braided                 size='One Size (adjustable)'      $12.0
Mesh Long-Sleeve Top — Black                  size='S/M'                        $15.0
Henley Long Sleeve — Washed Burgundy          size='M'                          $16.0
Bucket Hat — Reversible, Brown Plaid          size='One Size'                   $14.0
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
| 1 | Full three-tool run returns a fit card | 4 of 5 | MET (5/5) | All 5 tries ran search_listings, suggest_outfit and createda fit card. |
| 2 | Empty search stops before tool 2 | 5 of 5 | MET (5/5) | All 5 tries stopped after search_listings. |
| 3 | Selected item passes through unchanged | 5 of 5 | MET (5/5) | In all 5 tries, the first title from search_listings, the select_item title, and the suggest_outfit input title are the same: 90s Track Jacket |
| 4 | Fit card is 2–4 sentences with price and platform | 4 of 5 | MET (5/5) | All 5 cards are 2–3 sentences, and mention "$18" and "Depop". |
| 5 | Every result matches the size and is at or below the price | 5 of 5 | MET (5/5) | All are size M, S/M or One Size and $12–$25 with a $25 limit. |

**Diagnoses**

I missed nothing: all five criteria met their targets in 5 of 5 tries.

Some of the targets were low though.

- **Criteria 2, 3 and 5 could only fail through a code bug.** `parse_query` and `search_listings` don't call the model, so the same query gives the same result every try. Five tries of one fixed query is really one test repeated five times.
- **Criteria 1 and 4 allowed a miss (4 of 5) that never came close to happening.** Every fit card across all 25 completed tries had 2–3 sentences, the price and the platform. Criterion 4 should be 5 of 5.
- **Numeric sizes are dropped.** In `black boots size 8 under $60`, `parse_query`'s size regex only recognises letter sizes, `US 8`, `W30` and "one size". "size 8" stays in the description as keywords, `size` is `None`, and the search returns tops and hoodies.
- **One keyword is enough for a match.** `unicorn hair jacket` returned a track jacket instead of stopping, because "jacket" matched. `vintage tee` returned a belt and a bucket hat because of "vintage". `denim jacket` picked the track jacket over the denim vest.

**Criterion I'd tighten: 5.** It says "every result matches the size", but I only tested a letter size. A tighter version would test a numeric size too (`size 8`).


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
$ python app.py ask 'vintage graphic tee under $30' --trace
[1] parse_query
      in:  vintage graphic tee under $30
      out: {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
[2] search_listings (VIA MCP)
      in:  {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
      out: 10 items: Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey, Vintage Graphic Hoodie — Faded Black … +7 more
      →    10 match(es)
[3] select_item
      out: Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
[4] suggest_outfit
      in:  Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
      out: **Outfit 1: 90s Grunge Streetwear** *   **New item:** Graphic Tee *   **Owned pieces:** Baggy straight-leg jea…
[5] create_fit_card
      in:  Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
      out: Finally scored this 2003 tour bootleg tee for $24 on Depop and I’m obsessed with how soft and perfectly faded …
```

**Empty search**

```
$ python app.py ask 'invisible cloak' --trace
[1] parse_query
      in:  invisible cloak
      out: {'description': 'invisible cloak', 'size': None, 'max_price': None}
[2] search_listings (VIA MCP)
      in:  {'description': 'invisible cloak', 'size': None, 'max_price': None}
      out: [] (empty)
      →    0 match(es)
[3] branch
      →    search returned []: stopping here before suggest outfit

  No listings matched 'invisible cloak'. Try to use broader words (e.g. 'dress' instead of 'ballgown').
```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



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
