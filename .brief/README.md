# Protocol catalog sync

A hospital customer's document system exports its full clinical-protocol catalog every night. Build the sync that keeps our searchable copy correct however often or in whatever order it runs, and tells our indexer what to re-embed or remove. A clinician must never get a retracted protocol.

## What's in this repo

The `exports/` folder holds six nightly exports, from `2026-10-01.jsonl` to `2026-10-06.jsonl`. Each nightly export is a complete snapshot of the catalog. Each line is one protocol document with these fields.

| Field | Meaning |
| --- | --- |
| `doc_id` | The protocol's ID in the hospital's system |
| `title` | The protocol's title |
| `text` | The protocol's body text |
| `version` | An integer that goes up with each revision |
| `updated_at` | When the hospital's system last changed the record |
| `status` | `active` or `retracted` |
| `owner_email` | The person who owns the protocol |
| `notes` | Free-text notes from the hospital's system |

## Requirements

These are listed roughly in the order we'd build them. There's more here than fits in the time, so choose what to build first and tell us why.

1. Load the exports into a local store of your choice, such as SQLite, DuckDB or plain files. Keep a current table with one row per `doc_id` that holds its latest state.
2. Loading the same exports again, or loading them in a different order, must leave the store in the same state.
3. When a protocol is retracted or is no longer in the catalog, it must stop being searchable.
4. Protect the catalog from bad exports. If loading a file would remove more than 20% of the current protocols, stop without changing anything, exit with a nonzero code and say why. `--allow-mass-delete` loads the file anyway.
5. Produce the output the indexer consumes. For each load, that's a list of upserts and deletes, with enough of each protocol to search and display it.
6. Provide `changes(since, limit)`, which returns every change after a cursor, one page at a time, so the indexer can catch up. A consumer paging through must see every change exactly once, even if a new export is loaded between pages.
7. Produce the re-embed set, which is the `doc_id`s whose searchable content changed, so that we only pay to re-embed those. Embedding costs about $0.0001 per document.

## Interface

Please expose these commands so we can run your work. Any language is fine, and a small wrapper script called `sync` is the easiest way to do it. Keep all state under `./state/`, so that deleting that folder resets everything.

| Command | What it does |
| --- | --- |
| `sync load <file>... [--allow-mass-delete]` | Loads one or more exports, in the order given |
| `sync current` | Prints the current table as JSONL, one protocol per line |
| `sync index-out` | Prints the indexer output from the most recent load as JSONL |
| `sync changes --since <cursor> --limit <n>` | Prints `{"changes": [...], "next_cursor": "..."}` as JSON. An empty `--since` starts from the beginning |
| `sync reembed` | Prints the `doc_id`s to re-embed after the most recent load, one per line |

## Ground rules

Use any language, libraries and coding agent you like. Ask us anything. If this README doesn't answer a question, make a call, tell us the assumption, and keep going.
