# TDS Learning Trace: GA4 — All 20 Questions

**Student:** Vansh Rathee | **Roll No:** 23f2004420 | **Date:** 2026-05-04

---

## 📊 Assignment Overview

- **Total Time Invested:** ~22 hours (spread across 6 days)
- **Overall Difficulty:** 4/5 — A few questions were straightforward once I knew the right tool, but several required real trial-and-error with scripts and deployments.
- **Primary Discovery Method:** LLM-first (Gemini Pro + Claude), falling back to docs and manual testing when outputs were wrong or incomplete.

---

## 🧩 Q1: Excel — Operational Margin Consolidation (Orbit Commerce)

### 1. The Starting Point
- **Initial Confidence:** 2/5
- **Pre-existing Knowledge:** I knew basic Excel (SUM, VLOOKUP), but had never dealt with mixed date formats or currency strings programmatically in Excel.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | Pasted the full question, asked for Excel formulas to clean the data | Got a decent starting point — TRIM, SUBSTITUTE, and TEXT functions suggested. Formulas looked right in theory. |
| 2 | Excel (manual) | Tried applying the date conversion formula for "2024 Q3" type strings | Formula returned an error. The fiscal quarter logic needed a nested IF with DATEVALUE — Gemini's first suggestion was incomplete. |
| 3 | Gemini Pro | "Your formula for fiscal quarter conversion didn't handle the YYYY QN format, fix it" | Got a corrected formula using MID + IF that extracted quarter number and mapped it to the last day of the quarter. |
| 4 | Claude | Pasted cleaned data and asked to verify the FILTER + SUMPRODUCT logic for Europe/Billing/≤10 Jun 2024 | Claude caught that I was using `<=` on a text date column, not a real date — told me to use DATEVALUE() first. |
| 5 | Excel | Final variance = Revenue − Expense on filtered rows | Got my answer. |

### 3. Critical Verification
- Cross-checked a few rows manually: confirmed the 37% expense fill-in was applying only to blank/TBD rows and not overwriting real values.
- **AI Error:** Gemini initially forgot to handle the `USD TBD` string — its ISNUMBER check failed because the cell was text, not truly blank. Had to explicitly prompt it to also catch the literal string.

### 4. Technical Synthesis
- **"Aha!" Moment:** Realising that TRIM alone doesn't fix non-breaking spaces — needed CLEAN+TRIM together.
- **Key Takeaway:** Always convert text-dates to real serial dates before filtering by date in Excel.

---

## 🧩 Q2: Excel — Z-Score Outlier Surveillance (PulseCare Clinics)

### 1. The Starting Point
- **Initial Confidence:** 3/5
- **Pre-existing Knowledge:** Had heard of z-scores in a stats class but never used STANDARDIZE in Excel.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | "How to compute z-scores for a column in Excel and count outliers with abs(z) >= 2.5" | Got exact formula: `=STANDARDIZE(B2, AVERAGE($B$2:$B$N), STDEV.S($B$2:$B$N))` and COUNTIF on the helper column. |
| 2 | Excel | Applied formula, dragged down | Worked first time. Counted with `=COUNTIF(C2:CN, ">2.5") + COUNTIF(C2:CN, "<-2.5")` |
| 3 | — | Verified with manual check on max/min values | Matched. |

### 3. Critical Verification
- Used STDEV.S (sample) not STDEV.P (population) as specified.
- **No AI hallucination observed** here — this was a textbook formula application.

### 4. Technical Synthesis
- **Key Takeaway:** STDEV.S vs STDEV.P distinction actually matters — using the wrong one shifts your z-scores slightly and could change the count.

---

## 🧩 Q3: dbt — Customer Analytics Model (Daily MRR)

### 1. The Starting Point
- **Initial Confidence:** 1/5
- **Pre-existing Knowledge:** Had seen dbt mentioned in lectures but never written an actual model file.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | "Write a dbt intermediate model for daily MRR analysis using CTEs and Jinja templating, following dbt conventions" | Got a solid starting model with `{{ ref() }}`, CTEs, and `current_date - interval '30 days'` filter. |
| 2 | Claude | "Review this dbt model — does it follow naming conventions, are the CTEs structured correctly?" | Claude pointed out the model name should start with `int_` for intermediate models, and that I was missing a `{{ config() }}` block. |
| 3 | Gemini Pro | "Add materialization config and comments explaining the MRR calculation logic" | Final version looked clean. |

### 3. Critical Verification
- Checked the dbt docs (docs.getdbt.com) to confirm `{{ ref() }}` syntax and materialization options.
- **AI Error:** Gemini used `DATE_SUB(current_date, INTERVAL 30 DAY)` which is MySQL syntax — not valid in Snowflake/BigQuery. Caught it while reviewing; replaced with `current_date - 30`.

### 4. Technical Synthesis
- **Key Takeaway:** dbt Jinja is not just templating — the `{{ config() }}` block is how you control warehouse-level behavior. Easy to miss if you're only looking at the SQL logic.

---

## 🧩 Q4: dbt — Operations Performance Mart (Orbit Ops)

### 1. The Starting Point
- **Initial Confidence:** 1/5
- **Pre-existing Knowledge:** Same as Q3 — very little.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | "Write a dbt mart model for weekly percent refunded over last 14 days using Snowflake dialect, reference stg_shipments and stg_returns, handle NULLs with coalesce" | Got a working model with `date_trunc('week', shipped_date)`, COALESCE, and RMA logic. |
| 2 | Claude | "Does this model correctly compute percent_refunded and handle edge cases like zero shipments in a week?" | Claude flagged a divide-by-zero risk — added `NULLIF(shipment_count, 0)` in the denominator. |

### 3. Critical Verification
- Checked that `{{ config(materialized='table', ...) }}` was present.
- **AI Error:** First draft had `source()` and `ref()` mixed up for staging models — Gemini called `{{ source('raw', 'stg_returns') }}` which is wrong; `stg_` models are referenced with `ref()`.

### 4. Technical Synthesis
- **Key Takeaway:** `source()` is for raw tables, `ref()` is for everything you've already modeled. This is a very common beginner mistake with dbt.

---

## 🧩 Q5: OpenRefine — Supplier Spend Consolidation

### 1. The Starting Point
- **Initial Confidence:** 1/5
- **Pre-existing Knowledge:** Had never opened OpenRefine before.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | "Step-by-step guide to clean supplier names in OpenRefine using clustering, remove duplicate invoice_ids, strip currency from amount_usd using GREL" | Got a full walkthrough. Used key-collision clustering first, then nearest-neighbour with radius 2. |
| 2 | OpenRefine (actual) | Ran key-collision cluster on supplier_name | Merged most variants. Some "Astra Supplies" variants with trailing punctuation needed a second pass. |
| 3 | OpenRefine | Applied GREL: `value.replace(/[^0-9.]/g, "").toNumber()` | Worked for most rows. Two rows had double periods (e.g., `USD..450.00`) — regex caught that too on second thought. |
| 4 | OpenRefine | Faceted on supplier_name = Astra Supplies, category = Software, status = Approved | Filtered down to final rows, exported, summed in Excel. |

### 3. Critical Verification
- Sorted by invoice_id in a text facet to spot duplicates — manually verified that the "first clean entry" was being kept.
- **AI Error:** The GREL Gemini gave used `/[^0-9.]/` without the global flag — some strings only had the first symbol stripped. Had to add the correct GREL syntax.

### 4. Technical Synthesis
- **Key Takeaway:** Always do nearest-neighbour clustering *after* key-collision — the order matters because key-collision is lossless and nearest-neighbour is more aggressive.

---

## 🧩 Q6: JSON — Sensor Roll-up Analytics (ThermalWatch)

### 1. The Starting Point
- **Initial Confidence:** 2/5
- **Pre-existing Knowledge:** Comfortable with Python and basic JSON, but streaming JSONL and handling mixed temperature units was new.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Claude | "Write a Python script to stream a JSONL file, filter to site=Lab-East, device id starting with pump, time window 2024-08-21 to 2024-08-31, exclude maintenance/offline, convert Fahrenheit to Celsius, compute average temp" | Got a clean ijson-based script. |
| 2 | Terminal | Ran the script | `KeyError: 'temperature_unit'` — some records had no unit field. |
| 3 | Claude | "Handle missing temperature_unit field — assume Celsius if absent" | Fixed with `.get('temperature_unit', 'C')`. |
| 4 | Terminal | Reran — got avg temp to 2 decimal places | Correct. |

### 3. Critical Verification
- Spot-checked 5 records manually — confirmed F→C conversion: `(F - 32) * 5/9`.
- **AI Error:** Claude initially used `json.load()` on the full file instead of streaming line by line. Explicitly told it to use `ijson` or `for line in f`.

### 4. Technical Synthesis
- **Key Takeaway:** For large JSONL files, streaming line-by-line with `for line in open(file)` + `json.loads(line)` is simpler and safer than ijson for most cases.

---

## 🧩 Q7: JSON — Flatten Nested Customer Orders (Slingshot Cloud)

### 1. The Starting Point
- **Initial Confidence:** 2/5
- **Pre-existing Knowledge:** Had done basic JSON parsing in Python.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Claude | "Write a Python script to read JSONL, explode orders array, filter by region=Europe, channel=Reseller, category=Security, date between 2024-08-08 and 2024-09-03, sum quantity" | Clean script, worked on the first run after I fixed a date comparison bug. |
| 2 | Terminal | Ran script | Date filter was off — I had forgotten to parse date strings with `datetime.strptime`. |
| 3 | Claude | Fixed the date parsing | Correct total on second run. |

### 3. Critical Verification
- Reran with a smaller filtered sample printed to console to verify each row matched all four conditions before summing.
- **No hallucinations noted.**

### 4. Technical Synthesis
- **Key Takeaway:** When filtering by date in JSON, string comparison `>=` happens to work for ISO dates (YYYY-MM-DD) but is fragile — always parse properly.

---

## 🧩 Q8: Parse Partial / Corrupted JSON (ReceiptRevive)

### 1. The Starting Point
- **Initial Confidence:** 2/5
- **Pre-existing Knowledge:** Knew `json.loads()` fails on truncated JSON.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | "How to parse a JSON array where entries are truncated mid-way — extract sales values even from incomplete records" | Suggested reading the file as text, splitting on `},{`, trying `json.loads()` on each chunk, and falling back to regex `"sales":\s*(\d+\.?\d*)` on failures. |
| 2 | Python | Implemented the hybrid approach | Worked — regex caught the truncated tail records. |
| 3 | Python | Summed all sales values | Got total. |

### 3. Critical Verification
- Printed count of successfully parsed vs regex-parsed records to confirm total = 100.
- **Key check:** Regex `"sales":\s*([\d.]+)` — made sure it wasn't double-counting from valid records.

### 4. Technical Synthesis
- **Key Takeaway:** `json.JSONDecodeError` + regex fallback is a practical pattern for OCR-damaged data. Don't try to "fix" the JSON — just extract what you need.

---

## 🧩 Q9: GitHub Copilot — Data Transformation

### 1. The Starting Point
- **Initial Confidence:** 3/5
- **Pre-existing Knowledge:** Comfortable with JavaScript and recursive functions.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | VS Code + Copilot | Wrote comment: `// Function that extracts all 'id' fields from deeply nested object structure` | Copilot suggested a clean recursive function immediately. |
| 2 | Node.js | Tested with provided data structure `{id:1, children:[{id:2, children:[{id:3}]}, {id:4}]}` | Returned `[1, 2, 3, 4]` — correct. |

### 3. Critical Verification
- Manually traced the recursion to verify all nodes were visited regardless of depth.
- **No issues.** Copilot handled this well.

### 4. Technical Synthesis
- **Key Takeaway:** Copilot is excellent for "extract field from nested structure" type tasks — the comment quality directly affects suggestion quality.

---

## 🧩 Q10: Google Sheets — AI Formula to Extract Zip Codes

### 1. The Starting Point
- **Initial Confidence:** 3/5
- **Pre-existing Knowledge:** Had used Google Sheets formulas but never the `=AI()` function.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Google Sheets | Used `=AI("Extract the zip code... " & A2)` as instructed | Worked but was slow — 100 rows took a few minutes to process. |
| 2 | Google Sheets | Some cells returned "N/A" — verified those addresses genuinely had no zip code | Correct behavior. |
| 3 | Google Sheets | `=TEXTJOIN(",", TRUE, B2:B101)` | Got the full concatenated string. |

### 3. Critical Verification
- Spot-checked ~10 addresses manually — confirmed zip codes matched.
- **One issue:** The `=AI()` formula occasionally returned the zip with extra spaces. Used `TRIM()` inside the TEXTJOIN range to clean.

### 4. Technical Synthesis
- **Key Takeaway:** `=AI()` is genuinely useful for messy extraction tasks, but you can't trust it blindly — spot verification is essential.

---

## 🧩 Q11: FastAPI — Batch Sentiment Analysis

### 1. The Starting Point
- **Initial Confidence:** 2/5
- **Pre-existing Knowledge:** Had used Flask before, not FastAPI. Understood REST APIs conceptually.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Claude | "Write a FastAPI app with a POST /sentiment endpoint that classifies sentences as happy/sad/neutral using a rule-based approach" | Got a working app with keyword lists for happy/sad and neutral as default. |
| 2 | Terminal (`uvicorn main:app`) | Started the server | Worked. Tested with curl. |
| 3 | curl | `curl -X POST http://localhost:8000/sentiment -H "Content-Type: application/json" -d '{"sentences":["I love this!","This is terrible.","Meeting at 3 PM"]}'` | Returned correct classifications. |
| 4 | — | Submitted URL — passed 8/10 test cases | Two edge cases ("I can't complain" → should be neutral, classified as sad). Acceptable. |

### 3. Critical Verification
- Tested about 15 sentences manually to tune keyword lists.
- **AI Error:** Claude's first version returned a list instead of a dict with `results` key — didn't match the required schema. Had to prompt it to fix the response model.

### 4. Technical Synthesis
- **Key Takeaway:** FastAPI's Pydantic models enforce the response shape — define them explicitly and test with real curl calls before submitting.

---

## 🧩 Q12: Shell — Parse and Aggregate Messy CSV Transaction Logs

### 1. The Starting Point
- **Initial Confidence:** 1/5
- **Pre-existing Knowledge:** Basic `grep` and `cat`, not comfortable with `awk`.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | "Shell one-liner to handle pipe and comma separators in a CSV, strip whitespace, filter out missing category rows, sum amounts per category, output as Category:Amount sorted alphabetically" | Got a long awk command. |
| 2 | Terminal | Ran it | Some categories had amounts in scientific notation — needed `printf "%.2f"` fix. |
| 3 | Gemini Pro | "Fix the awk command to output amounts with exactly 2 decimal places and no scientific notation" | Corrected. Final output format matched. |

### 3. Critical Verification
- Ran a secondary Python script to double-check totals for two categories.
- **AI Error:** First awk script treated `|` as a field separator globally but the CSV also had `|` in note fields — had to refine the approach to only split on the first occurrence.

### 4. Technical Synthesis
- **Key Takeaway:** `awk -F'[|,]'` splits on all occurrences — if your data has delimiters in free-text fields, you need a smarter split strategy.

---

## 🧩 Q13: Shell — Extract and Flatten Nested JSON from Multiple Files

### 1. The Starting Point
- **Initial Confidence:** 2/5
- **Pre-existing Knowledge:** Had used `jq` once before but not for nested extraction.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Claude | "Shell commands to unzip archive, use jq to extract metrics.level from all JSON files, count occurrences of each level" | Got: `unzip archive.zip && find . -name "*.json" | xargs jq -r '.[] .metrics.level' | sort | uniq -c` |
| 2 | Terminal | Ran it | Some files had arrays at root, others had single objects — jq path error on some. |
| 3 | Claude | "Handle both array and single object at root level in jq" | Used `jq -r 'if type == "array" then .[] else . end | .metrics.level'` |
| 4 | Terminal | Reformatted output to `level1:count|level2:count` with awk | Done. |

### 3. Critical Verification
- Summed all counts — matched total record count in the dataset.
- **AI Hallucination:** Claude initially suggested `.records[].metrics.level` — the field was not under a `records` key. Had to inspect one file manually first.

### 4. Technical Synthesis
- **Key Takeaway:** Always `cat one_file.json | jq '.'` before writing your jq path — don't trust AI to know the exact structure of your file.

---

## 🧩 Q14: Shell — Deduplicate and Aggregate Semi-Structured Address Data

### 1. The Starting Point
- **Initial Confidence:** 1/5
- **Pre-existing Knowledge:** Knew `sort | uniq` basics.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | "Shell pipeline to normalize addresses (lowercase, trim extra spaces, remove trailing punctuation), deduplicate, count unique" | Got: `cat addresses.txt | tr 'A-Z' 'a-z' | sed 's/  */ /g' | sed 's/[.,]*$//' | sort | uniq | wc -l` |
| 2 | Terminal | Ran it | Count seemed high — realized some lines had extra trailing text like "apt 2B (ground floor)" that varied. |
| 3 | Gemini Pro | "How to extract only the core address before any parenthetical or extra annotation" | Added `sed 's/(.*//'` before the normalization step. |
| 4 | Terminal | Final unique count obtained. | Done. |

### 3. Critical Verification
- Manually inspected 20 lines pre- and post-normalization.
- **No hallucinations** — pipeline was logically sound.

### 4. Technical Synthesis
- **Key Takeaway:** Address normalization is an iterative process. You can't write the perfect regex first — look at actual data anomalies first.

---

## 🧩 Q15: The Recursive Corrupted JSON Fixer

### 1. The Starting Point
- **Initial Confidence:** 1/5 — This one intimidated me the most upfront.
- **Pre-existing Knowledge:** Basic Python file I/O. No experience with 40MB+ files or streaming parsers.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Claude | "Write a Python streaming script to read a large corrupted JSON log line by line, attempt json.loads on each line, extract metric_1935 from valid records only, sum values, compute SHA-256 of the integer sum" | Got a clean try/except streaming script. |
| 2 | Terminal | Ran it on the ~45MB file | Ran fine — took about 40 seconds. Got integer sum. |
| 3 | Python | `import hashlib; print(hashlib.sha256(str(total).encode()).hexdigest())` | Got the SHA-256 hash. |
| 4 | — | Double-checked: no trailing newline in the hash input | Confirmed with `echo -n` equivalent in Python (no `\n` in the string). |

### 3. Critical Verification
- Reran the script twice to confirm the sum was deterministic.
- **AI Error:** Claude's first version used `hashlib.sha256(str(total) + "\n")` — the newline would have given the wrong hash. Caught this by reading the question carefully ("without any trailing newlines").

### 4. Technical Synthesis
- **"Aha!" Moment:** The hash of the *integer* sum, not a float. If `metric_1935` was stored as `1935.0` in some records and I summed as floats, the string representation would differ. Forced `int(total)` before hashing.
- **Key Takeaway:** Cryptographic output questions require you to be exact about data types and string encoding. Floats and ints hash differently.

---

## 🧩 Q16: Cross-Lingual Entity Disambiguation

### 1. The Starting Point
- **Initial Confidence:** 1/5 — Most complex-sounding question in the set.
- **Pre-existing Knowledge:** Familiar with LLM APIs but hadn't done multi-language NLP before.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Claude | "How to build a pipeline that reads documents in 15 languages, identifies a historical figure, and maps to an entity_id from a reference list" | Claude outlined the approach: load reference list, for each doc send to LLM with the reference list as context, ask it to return the entity_id. |
| 2 | Python + Claude API | Wrote a script that processed docs in batches of 20, passing reference entities as context | Worked but slow. Rate limits hit after ~200 docs. |
| 3 | Gemini Pro | Switched to Gemini for remaining docs — larger context window helped pass more reference entities at once | Faster. |
| 4 | Python | Merged outputs into CSV `doc_id,entity_id` | Done. |

### 3. Critical Verification
- Spot-checked 20 outputs manually — a few "Ivan" disambiguations looked suspicious. Re-prompted with more explicit era/region context for those.
- **AI Error:** LLM occasionally confused "Ivan III" with "Ivan IV" when the doc only mentioned "Ivan the Tsar." Had to add a second pass for low-confidence cases.

### 4. Technical Synthesis
- **Key Takeaway:** When using LLMs for disambiguation, passing the *full* reference list as context dramatically improves accuracy compared to zero-shot guessing.

---

## 🧩 Q17: LLM Hallucination Trap Matrix

### 1. The Starting Point
- **Initial Confidence:** 2/5
- **Pre-existing Knowledge:** Knew common Python libraries well enough to spot fake method names.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Claude | "Write a Python script using AST to parse 1000 .py files and detect calls to non-existent methods on standard library objects (pandas, json, os, etc.)" | Got an AST-based approach that flagged method calls on inferred object types. |
| 2 | Python | Ran AST scanner on all 1000 files | Flagged ~995 files with suspicious calls. Narrowed down to the one with zero flags. |
| 3 | Manual review | Opened the candidate script — reviewed it line by line | Confirmed all method calls exist in the actual libraries. |

### 3. Critical Verification
- Independently Googled 3 of the flagged "fake" methods from other scripts (e.g., `df.drop_nulls()`, `json.parse()`) — confirmed they don't exist.
- **AI Error:** The AST approach had false positives for third-party libraries (e.g., `polars` which *does* have `drop_nulls`). Had to filter those out manually.

### 4. Technical Synthesis
- **Key Takeaway:** AST analysis alone is enough to narrow down candidates, but you still need to know which library methods are real — maintain a curated list of "valid" methods per library.

---

## 🧩 Q18: DuckDB — Data Preparation for RetailCo Analytics

### 1. The Starting Point
- **Initial Confidence:** 3/5
- **Pre-existing Knowledge:** Comfortable with SQL, new to DuckDB specifically.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Gemini Pro | "Write a DuckDB SQL query that filters LATAM orders, fills NULL customers with 'Unknown', assigns price bands with CASE, returns count and total for the medium band only" | Got a clean query. |
| 2 | DuckDB CLI | Ran it | Correct. Had to add `ROUND(..., 2)` explicitly — Gemini's version omitted it. |

### 3. Critical Verification
- Ran without the band filter first to confirm all rows were being processed, then added `WHERE price_band = 'medium'`.
- **AI Error:** Gemini used `IFNULL` — DuckDB prefers `COALESCE` which is ANSI SQL standard and works fine. Minor but worth noting.

### 4. Technical Synthesis
- **Key Takeaway:** DuckDB syntax is very close to standard SQL. Most Postgres/BigQuery queries work with minimal changes.

---

## 🧩 Q19: Reconstruct and Desaturate an Image (PixelGuard)

### 1. The Starting Point
- **Initial Confidence:** 2/5
- **Pre-existing Knowledge:** Basic PIL/Pillow usage. Had never done grid-based image reassembly.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Claude | "Write Python (Pillow) to split a 5x5 image grid, reassemble using a mapping table, convert to grayscale using luminance weights 0.2126R + 0.7152G + 0.0722B, save as PNG" | Got clean script. Paste the mapping as a list of tuples. |
| 2 | Python | Downloaded `jigsaw.webp`, ran the script | First attempt had rows/columns inverted — the mapping is `(scrambled_row, scrambled_col) → (original_row, original_col)` and I placed pieces in the wrong direction initially. |
| 3 | Claude | "Clarify: for each (scrambled_r, scrambled_c) → (orig_r, orig_c), I should take the tile at scrambled position and place it at original position in the output, right?" | Confirmed. Fixed the loop. |
| 4 | Python | Reran — reconstructed image looked correct. Grayscale conversion applied via numpy dot product. | Saved as PNG. |

### 3. Critical Verification
- Visually inspected the reassembled image before grayscale — edges lined up correctly.
- Verified grayscale formula was applied per-pixel, not using PIL's built-in `.convert('L')` which uses a slightly different formula.
- **Key issue:** PIL's `convert('L')` uses different coefficients. Had to use numpy: `gray = 0.2126*r + 0.7152*g + 0.0722*b`.

### 4. Technical Synthesis
- **Key Takeaway:** When a question specifies exact luminance coefficients, never use a library's built-in grayscale — compute it manually with numpy to be exact.

---

## 🧩 Q20: Audio Extraction and Transcription (MediaIndex)

### 1. The Starting Point
- **Initial Confidence:** 3/5
- **Pre-existing Knowledge:** Had used ffmpeg before. New to yt-dlp and Whisper.

### 2. The Interaction Loop

| Attempt # | Tool/Resource | Action | Result/Friction Point |
| :--- | :--- | :--- | :--- |
| 1 | Terminal | Ran the provided `yt-dlp` command | Worked. Got `audio.mp3`. |
| 2 | Terminal | Ran `ffmpeg -ss 00:00:20 -to 00:00:50 -i audio.mp3 -c copy segment.mp3` | Got 30-second segment. |
| 3 | ElevenLabs (speech-to-text) | Uploaded `segment.mp3` | Got a clean transcript. Cross-checked with faster-whisper output to verify. |
| 4 | Manual review | Read through transcript, corrected one word that sounded garbled | Final transcript pasted. |

### 3. Critical Verification
- Listened to the audio segment myself while reading the transcript to confirm accuracy.
- **No hallucinations** — transcription tools are generally reliable for clean English audio.

### 4. Technical Synthesis
- **Key Takeaway:** For short, clean audio segments, transcription tools are highly accurate. The effort here is mostly in the download/trim pipeline, not the transcription itself.

---

## 🧠 Meta-Reflection

### Content Gap
A 5-minute walkthrough showing how to combine `awk` + `jq` for semi-structured data would have saved me 3–4 hours across Q12–Q14. Similarly, a worked example of dbt model naming conventions and `config()` blocks would have shortened Q3 and Q4 significantly.

### Learning Verdict
**No** — I don't think a traditional lecture would have taught me more. Working through real, broken data forced me to understand *why* things fail, not just what the syntax looks like. That said, I think a 15-minute "common shell data tools" video would have given me a much better starting foundation for the shell questions.

### Honest Note on AI Use
I relied heavily on Gemini Pro and Claude throughout. The key skill I developed wasn't memorizing syntax — it was learning to **verify AI outputs**, catch where they made wrong assumptions about data structure or dialect, and prompt iteratively rather than accepting the first answer. Almost every question required at least one correction cycle.
