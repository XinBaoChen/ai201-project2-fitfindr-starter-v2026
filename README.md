# FitFindr
By XinBao Chen
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

FitFindr helps a user find secondhand clothing by description, size, and price.
It searches the listings and chooses the best match. It then suggests outfits
using the user's wardrobe and writes a short fit card for the item. If nothing
matches, it explains what the user can change.



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

- **What it does:** Searches the listings data for description keywords, an optional size, and an optional inclusive price ceiling.
- **Inputs:** `description` (str), `size` (str | None), `max_price` (float | None); size matching is case-insensitive and token-aware so `M` does not match `XL`.
- **Returns:** A best-match-first list of listing dicts, each containing `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list (`[]`).

### `suggest_outfit`

- **What it does:** Suggests one or two outfits that style the selected listing with pieces from the user's wardrobe.
- **Inputs:** `new_item` (dict containing one listing), `wardrobe` (dict with an `items` list of wardrobe-item dicts).
- **Returns:** A non-empty string containing outfit ideas; with an empty wardrobe, it contains general styling advice for the new item.
- **When it has nothing:** An empty wardrobe is handled with general styling advice rather than an empty result or an error.

### `create_fit_card`

- **What it does:** Writes a short, post-ready caption for the selected item and its suggested outfit.
- **Inputs:** `outfit` (str), `new_item` (dict containing one listing).
- **Returns:** A two-to-four sentence string that mentions the item, price, platform, and outfit vibe.
- **When it has nothing:** If `outfit` is empty or whitespace, returns a descriptive message instead of raising an exception.

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

**Branch rule:** If `search_listings` returns an empty list, put a message in the session and stop. Otherwise, take the first result and go to `suggest_outfit`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex extracts the price and size. The remaining
words become the description.

**What moves through the session:** The query becomes `parsed`, then search
results go into `search_results`. The first result becomes `selected_item`,
then the outfit goes into `outfit_suggestion`, and the caption goes into
`fit_card`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'

Found: Y2K Baby Tee — Butterfly Print — $18.0 on depop
Outfit: Y2K streetwear with baggy jeans, chunky white sneakers, and a black crossbody bag.
Fit card: Channeling peak 2000s energy with this dreamy little graphic find.
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([item['id'] for item in search_listings('graphic tee', max_price=30)])"

['lst_002', 'lst_006', 'lst_017', 'lst_033', 'lst_011', 'lst_012', 'lst_015']
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

Outfit 1: Casual Streetwear with a white tank top, chunky white sneakers, and a black crossbody bag.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Nothing beats the effortless feel of a truly broken in pair of denim. I just scored these jeans for $38 on depop.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked AI to fact check the listing fields and look for search edge cases.
- *What came back:* It pointed out that a simple size search could match `M` inside `XL`, and that many brands are `None`.
- *What I changed:* I used token aware size matching and treated a missing brand as empty search text.

**Moment 2**

- *What I asked for:* I asked AI to review the planning loop for empty results and state problems.
- *What came back:* It said the loop must stop before the model tools when search returns an empty list, and must keep the selected item in the session.
- *What I changed:* I added the empty search message and made each later tool read its input from the session.

**Moment 3 — Unit 4**

- *What I asked for:* I asked AI to help check whether the five acceptance criteria
  were measurable, then reviewed the five uncached tries in each saved run log.
- *What came back:* It confirmed that the original criteria all passed, but pointed
  out that criterion 4 did not measure whether cards had fresh openings. It also
  helped me verify that moving `search_listings` behind
  `mcp_client.call_tool` did not change the list or the empty-list branch.
- *What I changed:* I added a fresh-hook instruction to the `create_fit_card`
  prompt, ran the same five scenarios again, and recorded both results in
  `results/run_2026-10-09_2107_before.md` and
  `results/run_2026-10-09_2158_after.md`. I kept the criterion unchanged
  because it was measurable as originally written; the fresh-opening check is
  documented as a future improvement rather than pretending it was part of the
  scored target.

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
| 1. Full three tool run returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before tool two | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item stays in session state | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Five different fit cards contain item, price, and platform | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Price limit is respected | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```
results/run_2026-10-09_2107_before.md, produced by agent.py::run_agent

Criterion 1 output:
Channeling major early 2000s energy with this dreamy little butterfly graphic. I love styling fitted crops like this Y2K Baby Tee — Butterfly Print with baggy denim for that effortless downtown look. Grab this sweet little piece over on my Depop for just $18 before it’s gone!

Criterion 2 output:
No listings matched. Try broader keywords, another size, or a higher maximum price.

Criterion 3 output:
selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[2] suggest_outfit
     in:  dict with keys: new_item, wardrobe

Criterion 4 output:
Channeling major retro off-duty energy with this pristine 90s Track Jacket — Navy/White Stripe. It's the ultimate throw-on layer for breezy days, and it’s up for grabs for just $45. Snag this little piece of nostalgia over on my Poshmark closet before someone else beats you to it!

Criterion 5 output:
Channeling major Y2K pop star energy with these chunky Velcro kicks. I just dropped these Platform Sneakers — White Chunky Sole over on Poshmark for $48. They are begging to be styled with baggy denim and a little white tank.
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
| 1 | Full three tool run returns a fit card | 4 of 5 | MET (5/5) | Every matching run returned search results, outfit advice, and a fit card. The loop completed all three stages five times. |
| 2 | Empty search stops before tool two | 5 of 5 | MET (5/5) | Every impossible query returned an empty search list, stored a helpful message, and stopped before `suggest_outfit`. |
| 3 | Selected item stays in session state | 5 of 5 | MET (5/5) | The listing id in `session["selected_item"]` matched the id received by `suggest_outfit` in every run. |
| 4 | Five different fit cards contain item, price, and platform | 5 of 5 | MET (5/5) | Five different listings produced captions that named the item, price, and platform. The model changed the wording without losing these details. |
| 5 | Price limit is respected | 5 of 5 | MET (5/5) | Every result from the five price limited searches was at or below the requested maximum price. |

**Diagnoses**

No criteria were missed, so there is no failed step or mechanism to diagnose.
The search tool returned the right results, the MCP branch stopped correctly,
the session carried the selected item, and the model returned usable cards.
The targets were not automatic because criterion 1 allowed only one miss and
the other criteria required perfect results. I would tighten criterion 4 next
by checking that the five cards also have different opening sentences. The
current criterion checks useful content, but it does not catch a model that
repeats the same opening like it is stuck on one fashionable sentence.


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
     out: 10 items: Y2K Baby Tee, Graphic Tee, Vintage Band Tee
[2] suggest_outfit
     in:  dict with keys: new_item, wardrobe
     out: Outfit 1: Y2K Streetwear
[3] create_fit_card
     in:  dict with keys: outfit, new_item
     out: Channeling peak 2000s energy with this dreamy little graphic find.
```

**Empty search**

```
[1] search_listings (via MCP)
     in:  dict with keys: description, size, max_price
     out: [] (empty)
[2] branch
     out: No listings matched. Try broader keywords, another size, or a higher maximum price.
          empty search, stopping before model tools
```

**Failure messages**

The empty search message says that no listings matched and suggests broader
keywords, another size, or a higher maximum price.

The empty wardrobe test returned general styling advice instead of crashing.

The model failure message says that the model could not be reached while
creating outfit advice. It suggests checking the API key or internet connection
and trying again.

Search now runs through `mcp_client.call_tool`. It still returns a list of
listing dictionaries, so the branch behaves the same and does not get lost in
the protocol forest.


---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**
I added one instruction to the `create_fit_card` prompt. It now asks for a
fresh hook based on the item's title or visual details and tells the model not
to repeat a generic template.

**Which failure it was meant to fix:**
The diagnosis pointed to model output quality. The original criteria all
passed, but criterion 4 did not check whether different items received fresh
openings.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Full three tool run returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before tool two | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item stays in session state | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Five different fit cards contain item, price, and platform | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Price limit is respected | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from the after run**

The output came from `results/run_2026-10-09_2158_after.md`, produced by
`run_eval.py::main` and `agent.py::run_agent`.

Criterion 1 returned this fit card:

```
Nostalgic flutter graphics and pink-tinted nostalgia just landed in the shop. This Y2K Baby Tee with a butterfly print is serving major early 2000s energy and is ready for your next low-rise denim moment. Grab it now for just $18 over on my Depop before it's gone!
```

Criterion 2 stopped with this message:

```
No listings matched. Try broader keywords, another size, or a higher maximum price.
```

Criterion 3 recorded this selected item:

```
selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
```

Criterion 4 produced these five item specific openings:

```
That crisp white sleeve piping hits different on a true vintage piece.
These chunky velcro soles carry so much 2000s mall-rat nostalgia.
That warm, worn-in patina on a Western buckle just hits different.
Channeling total 90s indie sleaze by throwing an oversized grey crewneck right over this muted floral midi.
That 80s shoulder structure on this Wrangler Denim Jacket is everything.
```

Criterion 5 returned this item under the requested limit:

```
selected_item: Platform Sneakers — White Chunky Sole ($48.0, poshmark)
search_results: 2
```

**Did it help, and how do I know:**
Yes. The score stayed at five out of five, and the five different fit card
scenarios produced fresh openings while keeping the item, price, and platform.
The change improved the model output without breaking the existing behavior.

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

No scored criterion is currently missed: all five criteria met their targets in
both the before and after runs. The remaining limitation is that criterion 4
checks item, price, platform, and sentence count, but does not formally require
five distinct opening sentences. To address that, I would save the first
sentence of each card and compare the five normalized openings, then add that
as a revised, separately scored criterion. I stopped there because the original
criterion was already written and measurable, and the Unit 4 improvement was
intended to preserve its target rather than change the grading rule after
seeing the results. The prompt change produced visibly different openings, but
I did not claim a stricter pass that I did not formally measure.

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
