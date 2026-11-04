---
name: check
---

Please run the steps below in this repo and collect the outputs. This is a fixed script, so follow it exactly.

**Rules**

- Don't change any code or any file outside `./state/` and `./results/`. If a step fails, record it and move on. Don't try to fix anything.
- Use the `sync` command from this repo. If it's run some other way (for example `python sync.py` or `npm run sync --`), ask me once how to run it, then use that everywhere below.
- For every command, save stdout to the file named in the step, or to `results/<set>/load-NN.out` for a load that names no file (numbering the loads within the set). Save stderr next to it with `.err` added, and append a line to `results/log.tsv` with the set, the command exactly as run (quotes included) and its exit code.
- If a command takes longer than 60 seconds, stop it, record exit code `timeout` and move on.
- Start each set, and each run in set C, from a clean, empty state.

**Set A**

1. `sync load exports/2026-10-01.jsonl`
2. `sync current` → `results/A/current.jsonl`

**Set B**

1. `sync load exports/2026-10-01.jsonl exports/2026-10-02.jsonl exports/2026-10-03.jsonl exports/2026-10-04.jsonl`
2. `sync current` → `results/B/current.jsonl`

**Set C**

Load the files one at a time, in the order given. After the last load of each run, save `sync current`.

1. Run 1: 01, 02, 03, 04, 06, then 01, 02, 03, 04, 06 again → `results/C/run1.jsonl`
2. Run 2: 06, 04, 03, 02, 01 → `results/C/run2.jsonl`
3. Run 3: 03, 06, 01, 04, 02 → `results/C/run3.jsonl`

Here 01 means `exports/2026-10-01.jsonl`, and so on.

**Set D**

1. Load 01, 02, 03 and 04, one at a time.
2. `sync current` → `results/D/before.jsonl`
3. `sync load exports/2026-10-05.jsonl` → `results/D/load1.out`
4. `sync current` → `results/D/after1.jsonl`
5. `sync load exports/2026-10-05.jsonl --allow-mass-delete` → `results/D/load2.out`
6. `sync current` → `results/D/after2.jsonl`

**Set E**

1. Load 01, 02, 03, 04 and 06, one at a time.
2. `sync index-out` → `results/E/index.jsonl`
3. Copy the whole `state/` folder to `results/E/state/`.

**Set F**

1. Load 01, 02, 03 and 04, one at a time.
2. `sync changes --since "" --limit 2` → `results/F/page-01.json`
3. Read `next_cursor` from that page and run `sync changes --since "<next_cursor>" --limit 2` → `results/F/page-02.json`. Keep going the same way, numbering the pages.
4. Right after saving `page-03.json`, run `sync load exports/2026-10-06.jsonl`, then carry on paging from the last `next_cursor`.
5. Stop when a page has an empty `changes` list, when `next_cursor` is missing or the same as the one before, or after 60 pages.

**Set G**

1. Load 01, 02 and 03, one at a time.
2. `sync load exports/2026-10-04.jsonl`
3. `sync reembed` → `results/G/after-04.txt`
4. `sync load exports/2026-10-06.jsonl`
5. `sync reembed` → `results/G/after-06.txt`

**Finish**

1. Copy your source code into `results/source/`: everything in the repo except `state/`, `results/`, `exports/`, `.git/` and dependency folders such as `node_modules/` or `.venv/`.
2. Zip `results/` as `results.zip` and tell me where it is.
3. Print the contents of `results/log.tsv`.
