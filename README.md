# Draft Board Reranker

A single-file tool for reordering fantasy football ranking boards by typing ranks, then exporting a CSV that [FantasySage](https://fantasy-sage-football.vercel.app/) will actually accept in its upload slot.

Type a player's new rank, press <kbd>Enter</kbd>, and they move there — everyone in between shifts one slot and the board renumbers. Moving a player from 40 to 1 pushes ranks 1–39 down one each; nothing else reorders.

## Use it

Open `index.html` in a browser. No build step, no dependencies, no server — one file, everything inline. The eight source boards are embedded, so it opens in a working state.

It also runs as a claude.ai Artifact, where the browser sandbox blocks page-initiated downloads; the export routes through the host's `downloads` capability when that's present and falls back to a normal Blob download otherwise.

## Controls

| | |
|---|---|
| Rank box | Type a number, <kbd>Enter</kbd> to commit, <kbd>Esc</kbd> to cancel |
| <kbd>↑</kbd> / <kbd>↓</kbd> | Nudge one slot, keeping focus for repeats |
| Find player | Filter by name, team, or position — ranks stay global while filtered |
| Teams | League size, for the round dividers (default 12) |
| Undo / Redo | Also <kbd>Cmd/Ctrl+Z</kbd> and <kbd>Cmd/Ctrl+Shift+Z</kbd> |
| Upload CSV | Load your own board — or drop a CSV anywhere on the page |
| Copy CSV / Download CSV | Export the current order |

Edits persist per board in `localStorage`, so you can close the tab mid-rerank. They only reach FantasySage when you export and upload.

## Import format

Upload reads the same two column shapes FantasySage does, plus this tool's own export:

- **Underdog** — `firstName` + `lastName` + `slotName`, optionally `teamName` and `adp`
- **FantasyPros** — a `Player` column (or `name`), optionally `Pos`/`Position`, `Team`, and `Rank`/`ADP`/`ADP on <date>`

Rows are ordered by the rank column when there is one, otherwise file order is kept. Quoted fields, CRLF endings, and a UTF-8 BOM are all handled; duplicate player names are dropped, keeping the first. Digits are stripped from positions, so Underdog's `WR1` reads as `WR`.

Uploaded boards join the dropdown under **Your uploads** and are remembered in `localStorage`; **Remove upload** discards one. Parsing happens entirely in the page — no file is ever uploaded anywhere.

## Export format

```csv
Rank,Player,Pos,Team,ADP
1,Bijan Robinson,RB,ATL,1
2,Jahmyr Gibbs,RB,DET,2
```

FantasySage's uploader accepts exactly two column shapes, and silently loads zero rows for anything else — its parser logs to the console and the drop zone just sits there. It needs either:

- **Underdog:** all three of `firstName`, `lastName`, `slotName`
- **FantasyPros:** `Player` **and** `Pos`/`Position`

This tool emits the second shape. `Team` and `ADP` are optional to the parser but carried anyway; row order in the file doesn't matter, since the app sorts on the ADP column (falling back to `Rank`).

Upload **one board at a time**. The upload path does no de-duplication, so a file with several boards stacked together produces a board with every player repeated.

## `data/`

The eight boards as standalone upload-ready CSVs, same format as the export:

| File | Analyst / source | Format | Updated | Players |
|---|---|---|---|---|
| `fantasypros-top-five-full-ppr.csv` | FantasyPros Top 5 Analysts | Full PPR | 8/27 | 368 |
| `miller-boone-full-ppr.csv` | Seth Miller + Justin Boone | Full PPR | 7/14 | 364 |
| `boone-del-don-full-ppr.csv` | Justin Boone + Dalton Del Don | Full PPR | 7/14 | 364 |
| `miller-willan-full-ppr.csv` | Seth Miller + Jason Willan | Full PPR | 7/13 | 364 |
| `hybrid-half-ppr.csv` | Sage Rankings (best analyst per position) | Half PPR | 6/22 | 305 |
| `boone-half-ppr.csv` | Justin Boone (Yahoo) | Half PPR | 6/3 | 301 |
| `del-don-half-ppr.csv` | Dalton Del Don (The Deep Shot) | Half PPR | 6/22 | 301 |
| `underdog-best-ball-june-24.csv` | Underdog ADP | Best Ball | 6/24 | 240 |

The rankings are hardcoded into FantasySage's client bundle rather than served from an API, and were read out of it — a snapshot as of the dates above, not a live feed. A redeploy of that site can change them, and refreshing means re-extracting. The Underdog board carries `FA` for every team because the source data never had teams; that's cosmetic.

Rankings belong to the analysts and outlets credited above.
