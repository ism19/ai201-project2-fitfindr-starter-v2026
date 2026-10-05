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

**Branch rule:**

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->

**What moves through the session:** <!-- which fields, in what order -->

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

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

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
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

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
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



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

```

**Empty search**

```

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
