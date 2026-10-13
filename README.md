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

A user types what they want, like `vintage graphic tee under $30, size M`.
FitFindr pulls a description, a size and a price ceiling out of it, then
searches 40 secondhand listings from depop, thredUp and poshmark. It takes the
best match and asks the model for two outfits that pair it with clothes the
user already owns. The user gets back the listing, the outfit ideas and a
short caption ready to post. If nothing matches, it stops and says what to
change.


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

- **What it does:** Searches the listings for items matching the requested description, optional size, and optional maximum price.
- **Inputs:** `description` (str), `size` (str | None), `max_price` (float | None). Size matching is case-insensitive and uses complete size values/tokens, so `M` can match `S/M` but should not match `XL` or `US 9`.
- **Returns:** A list of matching listing dictionaries, best match first, with fields including `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list `[]`.

### `suggest_outfit`

- **What it does:** Suggests one or two outfits using the selected listing and the user's wardrobe.
- **Inputs:** `new_item` (dict), `wardrobe` (dict).
- **Returns:** A non-empty string containing outfit suggestions.
- **When it has nothing:** If the wardrobe has no items, returns general styling advice for the new item instead of failing.

### `create_fit_card`

- **What it does:** Creates a short social-media-style caption for the selected item and outfit.
- **Inputs:** `outfit` (str), `new_item` (dict).
- **Returns:** A two-to-four sentence caption that mentions the item, price, platform, and outfit vibe.
- **When it has nothing:** If `outfit` is empty or only whitespace, returns a descriptive message instead of raising an error.

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

**Branch rule:** If `search_listings` returns an empty list, put a message in the session telling the user to change the description, size, or maximum price, then stop. Otherwise, select the first result and continue to `suggest_outfit`, then `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which --> String splitting. The query is split into words. `under $30`, `below $30`, `max $30` or a bare `$30` sets `max_price`. `size M` sets `size`. The remaining words, minus filler like "looking for a", become `description`.

**What moves through the session:** <!-- which fields, in what order --> `query` → `parsed` (`description`, `size`, `max_price`) → `search_results` → `selected_item` (`search_results[0]`) → `outfit_suggestion` → `fit_card`. Each tool reads its inputs back out of the session. `session["steps"]` records every tool call in order, including the `item_id` that `suggest_outfit` and `create_fit_card` each received.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30, size M'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Outfit 1: Pair the Y2K Baby Tee — Butterfly Print with the Baggy straight-leg jeans, dark wash for a classic noughties contrast. Complete the look with the Chunky white sneakers and the Black crossbody bag for an effortless everyday ensemble.

Outfit 2: Layer the Y2K Baby Tee — Butterfly Print underneath the Vintage black denim jacket for an edgy vintage vibe. Pair them with the Wide-leg khaki trousers and the Black combat boots to anchor the pastel graphic top with structured pieces.

  Fit card: Found the ultimate Y2K baby tee with the cutest butterfly print for just $18 on Depop. I'm obsessed with pairing it with baggy dark-wash jeans and chunky sneakers for that effortless noughties look. Grab it before I change my mind!
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Outfit 1: Pair the Vintage Levi's 501 Jeans — Medium Wash with the White ribbed tank top and the Chunky white sneakers for a classic, effortless look. Finish the outfit by adding the Brown leather belt to tie the ensemble together.

Outfit 2: Combine the Vintage Levi's 501 Jeans — Medium Wash with the Oversized grey crewneck sweatshirt and the Black combat boots for an edgy, streetwear-inspired vibe. Accessorize with the Black crossbody bag to complete the look.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Scored these vintage Levi’s 501 jeans on Depop for only $38 and they fit like an absolute dream. Nothing beats the medium wash for everyday wear. I’m living in these this fall with crisp white sneakers and a chunky oversized sweater.
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

I kept the five criteria and targets from Unit 3. I ran `python run_eval.py --label before` with cache off and temperature 0.9. The log recorded 80 real model calls. I used the same data, model, and key.

I ran 45 full agent attempts: five each for criteria 1–4, plus five for each of the five queries in criterion 5. In row 5, each Try column passes only when all five queries keep the typed filters and every returned listing respects them. That gives me 25 checks for criterion 5.

For criterion 4, I counted sentences at `.`, `!`, or `?` followed by whitespace, and counted words by whitespace. I checked the dollar price and platform without caring about capitalization. I marked every card sharing an opening sentence FAIL. The original rule also requires no repeated opening across all five cards; four valid cards alone cannot excuse a duplicate.

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before suggest_outfit | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Same item reaches both model tools | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card facts, length, and different openings | 4 of 5 | PASS | FAIL | FAIL | FAIL | FAIL | MISSED (1/5) |
| 5. All five queries keep the typed size and budget | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

I saved every session and trace in [run_2026-10-13_1952_before.json](results/run_2026-10-13_1952_before.json) and the readable output in [run_2026-10-13_1952_before.md](results/run_2026-10-13_1952_before.md). The values below come from those sessions. The captions come from `tools.py::create_fit_card`, and the search results come from `tools.py::search_listings` through MCP.

**Criterion 1, try 1** — `agent.py::run_agent`

Query: `vintage graphic tee under $30`

```text
Tool order: ['search_listings', 'suggest_outfit', 'create_fit_card']
Obsessed with this Y2K butterfly baby tee I just scored on Depop for only $18. It looks so good for casual street style with baggy dark wash jeans and chunky white sneakers. Grab it before I change my mind and keep it for myself!
```

**Criterion 2, try 1** — `agent.py::run_agent`

Query: `designer ballgown size XXS under $5`

```text
No listings matched 'designer ballgown' in size XXS under $5. To find something: use fewer or different words for the item (like 'graphic tee' or 'denim jacket'); or try a size other than XXS, or leave the size out; or raise your max price above $5.
Tool order: ['search_listings']
fit_card: None
```

**Criterion 3, try 1** — `agent.py::run_agent`

Query: `vintage graphic tee under $30`

```text
search_results[0].id: lst_002
selected_item.id: lst_002
suggest_outfit item_id: lst_002
create_fit_card item_id: lst_002
```

**Criterion 4, try 1** — `agent.py::run_agent`

Query: `vintage graphic tee under $30`

```text
Found this absolute gem of a Y2K baby tee with the cutest butterfly print for just $18 on Depop. I’m obsessed! Honestly can't wait to style it with baggy dark-wash straight-leg jeans and chunky white sneakers for the ultimate noughties casual silhouette. Check out my shop before I change my mind and keep it.
```

**Criterion 5, try 1** — `agent.py::run_agent`

Query: `vintage graphic tee under $30`

```text
vintage graphic tee under $30
parsed: {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
search_results (id, size, price): [('lst_002', 'S/M', 18.0), ('lst_033', 'L', 19.0), ('lst_006', 'L', 24.0), ('lst_015', 'L', 26.0), ('lst_012', 'XL (fits oversized)', 20.0), ('lst_017', 'S/M', 15.0)]
90s track jacket in size M
parsed: {'description': '90s track jacket', 'size': 'M', 'max_price': None}
search_results (id, size, price): [('lst_004', 'M', 45.0), ('lst_022', 'M', 75.0)]
platform sneakers size 8
parsed: {'description': 'platform sneakers', 'size': '8', 'max_price': None}
search_results (id, size, price): [('lst_019', 'US 8', 48.0)]
denim jacket under $50
parsed: {'description': 'denim jacket', 'size': None, 'max_price': 50.0}
search_results (id, size, price): [('lst_007', 'S', 42.0)]
vintage tee size S under $20
parsed: {'description': 'vintage tee', 'size': 'S', 'max_price': 20.0}
search_results (id, size, price): [('lst_002', 'S/M', 18.0)]
```

---

## Verdicts and Diagnoses

| # | Target | Verdict | How I decided |
|---|---|---|---|
| 1 | 4 of 5 | MET (5/5) | I checked the three tool names in order and a non-empty fit card. |
| 2 | 5 of 5 | MET (5/5) | I checked that only search ran and the message named what to change. |
| 3 | 5 of 5 | MET (5/5) | I compared the first result id, selected item id, and both recorded tool input ids. |
| 4 | 4 of 5 | MISSED (1/5) | I checked the facts and length, then compared all five opening sentences. |
| 5 | 5 of 5 | MET (5/5) | I checked the parsed size and price and every returned listing for all five queries in each try. |

**Diagnoses**

I missed criterion 4. The problem is the model output from `tools.py::create_fit_card`, at trace step 5 (the third tool). The price, platform, word count, and sentence count passed in all five cards, but tries 2 and 4 had the same first sentence, and tries 3 and 5 had another shared first sentence. Only try 1 had a unique opening. My prompt asked for a real post but gave no guidance about the opening. The model kept using the same “Found this absolute dream” wording. Search, the branch, and the session did not cause this miss.

I saw one pattern, not four separate bugs: repeated openings from the same model tool. I kept the original target.

| Try | Sentences | Words | Price | Platform | Unique opening |
|---|---|---|---|---|---|
| 1 | 4 | 54 | True | True | True |
| 2 | 2 | 38 | True | True | False |
| 3 | 2 | 38 | True | True | False |
| 4 | 2 | 44 | True | True | False |
| 5 | 3 | 49 | True | True | False |

I compared these actual opening sentences:

```text
Try 1: Found this absolute gem of a Y2K baby tee with the cutest butterfly print for just $18 on Depop.
Try 2: Found this absolute dream of a Y2K baby tee with the cutest butterfly print on Depop for just $18!
Try 3: Found this absolute dream of a butterfly print Y2K baby tee on Depop for just $18!
Try 4: Found this absolute dream of a Y2K baby tee with the cutest butterfly print on Depop for just $18!
Try 5: Found this absolute dream of a butterfly print Y2K baby tee on Depop for just $18!
```

---

## Loop Trace

I recorded each step in `agent.py::run_agent`. I ran both paths with `python app.py ask '...' --trace` and saved the terminal output in [loop_trace.txt](results/loop_trace.txt). Each model call includes its full inputs and response. Search shows a short list summary; the JSON log keeps the full returned listings.

**Happy path**

```text
$ python app.py ask 'vintage graphic tee under $30' --trace

[1] parse query
      in:  vintage graphic tee under $30
      out: {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
      →    Use the parsed filters for search.
[2] search_listings (via MCP)
      in:  {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
      out: 6 items: Y2K Baby Tee — Butterfly Print, Vintage Band Tee — Faded Grey, Graphic Tee — 2003 Tour Bootleg Style … +3 more
[3] select first result
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
      →    Keep lst_002 in the session for both model tools.
[4] suggest_outfit
      in:  {'new_item': {'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, 'wardrobe': {'items': [{'id': 'w_001', 'name': 'Baggy straight-leg jeans, dark wash', 'category': 'bottoms', 'colors': ['dark blue', 'indigo'], 'style_tags': ['denim', 'streetwear', 'baggy'], 'notes': 'High-waisted, sits above the hip'}, {'id': 'w_002', 'name': 'Wide-leg khaki trousers', 'category': 'bottoms', 'colors': ['khaki', 'tan'], 'style_tags': ['earth tones', 'minimal', 'wide-leg'], 'notes': None}, {'id': 'w_003', 'name': 'White ribbed tank top', 'category': 'tops', 'colors': ['white'], 'style_tags': ['basics', 'minimal', 'fitted'], 'notes': 'Goes with everything'}, {'id': 'w_004', 'name': 'Oversized grey crewneck sweatshirt', 'category': 'tops', 'colors': ['grey', 'charcoal'], 'style_tags': ['oversized', 'basics', 'cozy'], 'notes': 'Really oversized — drops below the hip'}, {'id': 'w_005', 'name': 'Black cropped zip hoodie', 'category': 'tops', 'colors': ['black'], 'style_tags': ['athletic', 'streetwear', 'cropped'], 'notes': None}, {'id': 'w_006', 'name': 'Vintage black denim jacket', 'category': 'outerwear', 'colors': ['black'], 'style_tags': ['denim', 'vintage', 'classic'], 'notes': 'Slightly cropped'}, {'id': 'w_007', 'name': 'Chunky white sneakers', 'category': 'shoes', 'colors': ['white'], 'style_tags': ['sneakers', 'chunky', 'streetwear'], 'notes': None}, {'id': 'w_008', 'name': 'Black combat boots', 'category': 'shoes', 'colors': ['black'], 'style_tags': ['boots', 'grunge', 'classic'], 'notes': 'Lace-up, mid-ankle height'}, {'id': 'w_009', 'name': 'Brown leather belt', 'category': 'accessories', 'colors': ['brown'], 'style_tags': ['classic', 'earth tones', 'accessories'], 'notes': None}, {'id': 'w_010', 'name': 'Black crossbody bag', 'category': 'accessories', 'colors': ['black'], 'style_tags': ['minimal', 'accessories', 'everyday'], 'notes': None}]}}
      out: Outfit 1: Pair the Y2K Baby Tee — Butterfly Print with the Baggy straight-leg jeans, dark wash for a classic noughties contrast. Complete the look with the Chunky white sneakers and the Black crossbody bag for an effortless everyday ensemble.

Outfit 2: Layer the Y2K Baby Tee — Butterfly Print underneath the Vintage black denim jacket for an edgy vintage vibe. Pair them with the Wide-leg khaki trousers and the Black combat boots to anchor the pastel graphic top with structured pieces.
      →    Pass the outfit and the same item to create_fit_card.
[5] create_fit_card
      in:  {'outfit': 'Outfit 1: Pair the Y2K Baby Tee — Butterfly Print with the Baggy straight-leg jeans, dark wash for a classic noughties contrast. Complete the look with the Chunky white sneakers and the Black crossbody bag for an effortless everyday ensemble.\n\nOutfit 2: Layer the Y2K Baby Tee — Butterfly Print underneath the Vintage black denim jacket for an edgy vintage vibe. Pair them with the Wide-leg khaki trousers and the Black combat boots to anchor the pastel graphic top with structured pieces.', 'new_item': {'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}}
      out: Tuck this Y2K baby tee into baggy dark-wash denim for that classic noughties contrast. I’m listing this butterfly print piece on Depop for $18. Grab it before someone else does!
      →    All three tools finished; return the session.

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Outfit 1: Pair the Y2K Baby Tee — Butterfly Print with the Baggy straight-leg jeans, dark wash for a classic noughties contrast. Complete the look with the Chunky white sneakers and the Black crossbody bag for an effortless everyday ensemble.

Outfit 2: Layer the Y2K Baby Tee — Butterfly Print underneath the Vintage black denim jacket for an edgy vintage vibe. Pair them with the Wide-leg khaki trousers and the Black combat boots to anchor the pastel graphic top with structured pieces.

  Fit card: Tuck this Y2K baby tee into baggy dark-wash denim for that classic noughties contrast. I’m listing this butterfly print piece on Depop for $18. Grab it before someone else does!

1 model calls this session, 1 served from cache, 234 prompt + 44 output tokens
```

**Empty search**

```text
$ python app.py ask 'designer ballgown size XXS under $5' --trace

[1] parse query
      in:  designer ballgown size XXS under $5
      out: {'description': 'designer ballgown', 'size': 'XXS', 'max_price': 5.0}
      →    Use the parsed filters for search.
[2] search_listings (via MCP)
      in:  {'description': 'designer ballgown', 'size': 'XXS', 'max_price': 5.0}
      out: [] (empty)
[3] empty-search branch
      out: No listings matched 'designer ballgown' in size XXS under $5. To find something: use fewer or different words for the item (like 'graphic tee' or 'denim jacket'); or try a size other than XXS, or leave the size out; or raise your max price above $5.
      →    No item to pass on; stop before suggest_outfit.

  No listings matched 'designer ballgown' in size XXS under $5. To find something: use fewer or different words for the item (like 'graphic tee' or 'denim jacket'); or try a size other than XXS, or leave the size out; or raise your max price above $5.

0 model calls this session
```

**On the MCP move:** I registered only `search_listings` in `mcp_server.py` and replaced the direct search call with `mcp_client.call_tool` in `run_agent`. I ran `python mcp_client.py` as a separate client and saw the tool, its description, and its typed inputs. I compared an MCP search for `graphic tee`, size `M`, max price `30` with the direct function: both returned the same two listing dictionaries. The other two tools still call the model directly through the existing adapter.

**Deliberate failures**

I saved the full output in [failure_modes.txt](results/failure_modes.txt).

I triggered empty search with `designer ballgown size XXS under $5`. It stopped before the second tool and told me to change the words, size, or price.

```text
No listings matched 'designer ballgown' in size XXS under $5. To find something: use fewer or different words for the item (like 'graphic tee' or 'denim jacket'); or try a size other than XXS, or leave the size out; or raise your max price above $5.
```

I triggered empty wardrobe with `denim jacket under $50` and an empty wardrobe, the same input as `--empty-wardrobe`. The existing empty-wardrobe handler returned advice instead of crashing. I could try the suggested basics or add my own clothes to the wardrobe.

```text
Outfit 1: Pair the cropped denim jacket with high-waisted black skinny jeans and a tucked-in white crewneck t-shirt. Finish the look with classic white canvas sneakers for a casual, everyday vibe.

Outfit 2: Layer the light wash jacket over a fitted black midi dress for a balanced mix of casual and chic. Complete the outfit with comfortable black leather ankle boots.
```

I triggered model unavailable by changing one character of the key in a child process environment. I used the new query `vintage graphic tee under $29.81` and turned cache off. I kept `.env` intact. It made one real model call, stopped without a stack trace, and told me to check the key and connection. I added a `ModelUnavailable` handler around each model tool so the message names which step failed.

```text
The outfit model is unavailable. The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com. Check your API key and connection, then try again.
```

---

## The Improvement

**What I changed:** I changed only the caption prompt in `tools.py::create_fit_card`. I asked it to begin with a concrete styling detail from the outfit and avoid the repeated shopping phrases. I kept the same item facts, length request, model, temperature, data, and scenarios.

**Which failure it was meant to fix:** I wanted the model to stop repeating the first sentence in criterion 4. I did not add retries, change the search, or change the targets.

### Run Log — After

I ran `python run_eval.py --label after` with the same 45 attempts and cache off. The log recorded another 80 real model calls.

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before suggest_outfit | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Same item reaches both model tools | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card facts, length, and different openings | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. All five queries keep the typed size and budget | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

I saved the complete after sessions in [run_2026-10-13_1958_after.json](results/run_2026-10-13_1958_after.json) and the readable output in [run_2026-10-13_1958_after.md](results/run_2026-10-13_1958_after.md).

**Criterion 1, try 1** — `agent.py::run_agent`

Query: `vintage graphic tee under $30`

```text
Tool order: ['search_listings', 'suggest_outfit', 'create_fit_card']
Tucked into baggy straight-leg jeans with chunky white sneakers, this butterfly print Y2K baby tee brings major early-2000s mallrat energy to the street. Snagged this piece for eighteen bucks on Depop and I am never taking it off.
```

**Criterion 2, try 1** — `agent.py::run_agent`

Query: `designer ballgown size XXS under $5`

```text
No listings matched 'designer ballgown' in size XXS under $5. To find something: use fewer or different words for the item (like 'graphic tee' or 'denim jacket'); or try a size other than XXS, or leave the size out; or raise your max price above $5.
Tool order: ['search_listings']
fit_card: None
```

**Criterion 3, try 1** — `agent.py::run_agent`

Query: `vintage graphic tee under $30`

```text
search_results[0].id: lst_002
selected_item.id: lst_002
suggest_outfit item_id: lst_002
create_fit_card item_id: lst_002
```

**Criterion 4, try 1** — `agent.py::run_agent`

Query: `vintage graphic tee under $30`

```text
Cinch your waist with a brown leather belt over wide-leg khaki trousers, then layer a black cropped zip hoodie for an edgy touch. This butterfly print Y2K baby tee ties the whole vibe together. Grab it on Depop for just $18 before someone else does!
```

**Criterion 5, try 1** — `agent.py::run_agent`

Query: `vintage graphic tee under $30`

```text
vintage graphic tee under $30
parsed: {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
search_results (id, size, price): [('lst_002', 'S/M', 18.0), ('lst_033', 'L', 19.0), ('lst_006', 'L', 24.0), ('lst_015', 'L', 26.0), ('lst_012', 'XL (fits oversized)', 20.0), ('lst_017', 'S/M', 15.0)]
90s track jacket in size M
parsed: {'description': '90s track jacket', 'size': 'M', 'max_price': None}
search_results (id, size, price): [('lst_004', 'M', 45.0), ('lst_022', 'M', 75.0)]
platform sneakers size 8
parsed: {'description': 'platform sneakers', 'size': '8', 'max_price': None}
search_results (id, size, price): [('lst_019', 'US 8', 48.0)]
denim jacket under $50
parsed: {'description': 'denim jacket', 'size': None, 'max_price': 50.0}
search_results (id, size, price): [('lst_007', 'S', 42.0)]
vintage tee size S under $20
parsed: {'description': 'vintage tee', 'size': 'S', 'max_price': 20.0}
search_results (id, size, price): [('lst_002', 'S/M', 18.0)]
```

**Before and after**

| Criterion | Before | After |
|---|---|---|
| 1 | 5/5 | 5/5 |
| 2 | 5/5 | 5/5 |
| 3 | 5/5 | 5/5 |
| 4 | 1/5 | 5/5 |
| 5 | 5/5 | 5/5 |

**After-run opening sentences**

```text
Try 1: Cinch your waist with a brown leather belt over wide-leg khaki trousers, then layer a black cropped zip hoodie for an edgy touch.
Try 2: Throwing on baggy dark-wash jeans instantly nails that classic 2000s silhouette with this Y2K Baby Tee — Butterfly Print.
Try 3: Layering this butterfly print Y2K baby tee under a black cropped zip hoodie gives off such an edgy contrast.
Try 4: Cinch this Y2K Baby Tee with baggy dark-wash denim and chunky white sneakers for the ultimate noughties mall-rat aesthetic.
Try 5: Tuck this Y2K Baby Tee with the butterfly print into wide-leg khaki trousers and pull on black combat boots for an edgy, textured contrast.
```

**Did it help, and how do I know:** It helped in this test. Criterion 4 went from MISSED (1/5 unique openings) to MET (5/5), with the price, platform, sentence count, and word count still passing. The other four criteria stayed at 5/5. I made one prompt change between the two runs. Five tries show improvement here, but they do not prove the model will never repeat itself.

---

## What's Still Broken

I have no remaining missed criteria in the after run. I still saw a problem my criteria do not measure: after-run caption 2 says “I'm listing this butterfly tee,” and caption 4 says “DM to claim yours.” The input never said I was selling the item. Next I would make the buyer voice clear in the prompt and test that claim. I stopped here because this unit asks for one measured improvement. I would also test more items and more runs before trusting the opening variation on new inputs.



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
