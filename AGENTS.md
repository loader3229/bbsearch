# AGENTS.md — bbsearch AI Rulebook

> This file is the **onboarding handbook + code of conduct for AI agents** working in this project. It is not the project introduction for ordinary readers (that is [README.md](README.md)).
> Detailed mechanism notes live in [docs/architecture.md](docs/architecture.md); this file does not repeat them.

## 1. Project overview and core goal

`bbsearch` is a single-file C++11 command-line tool: it runs a DFS over the **core** of a Busy Beaver champion machine to find the **maximum landmark configuration** that core can produce (compared by each core's own importance order, lexicographically).

**Core goal**: within a finite search budget, report the maximum landmark configuration a given core can produce. **It is a simulation search, not a proof** — the output is always "the maximum found within the budget" and must never be presented as a mathematical conclusion.

## 2. Tech stack and key dependencies

| Item | Value |
|---|---|
| Language | C++11 (`-std=c++11`) |
| Dependencies | None — standard library only (`<vector> <map> <string> <algorithm>` etc.) |
| Compiler | Any C++11 compiler (`g++`); the project was developed on Windows with TDM-GCC-64, but the source is portable |
| Build | Single command, no build system (no Makefile / CMake / package.json) |
| Platform | Linux / macOS / Windows — the source is platform-independent |
| Size | `bbsearch.cpp` is about 1856 lines, everything in one file |

## 3. Project structure and architecture

```
bbsearch/
├── bbsearch.cpp        # the only source file: core file parsing + search engine + CLI
├── bbcores_new.txt     # data source: the core list (56 cores; name/cfg/slot/core four lines per block)
├── bbsearch            # build artifact (bbsearch.exe on Windows; not committed, a gitignore candidate)
├── README.md           # project description for humans
├── AGENTS.md           # this file (AI entry point)
├── CLAUDE.md / GEMINI.md # pointer files to AGENTS.md
└── docs/               # detailed documentation
```

Logical layers inside `bbsearch.cpp` (top to bottom):

1. **Core file parsing** — `parseCfg()` / `parseSlotLine()` / `parseCoreRows()` read the cores from `bbcores_new.txt` into the run-time table `gCores` (each `CoreEntry` = name / fixed state count / symbol count / transition rows + that core's landmark spec `CoreSpec`). **There is no built-in core table in the code**; failing to read the file is a fatal error.
2. **Global state** — the `trans_[MAXST][MAXSYM]` transition table, Zobrist hashes, open-addressing hash table
3. **Search engine** — `search()` recursive DFS + `noteCfg()` landmark matching + `considerCfg()` ranking
4. **CLI** — `main()` parses arguments → `runCore()` runs a single core → `listCores()` lists cores

**Key constraint: there is exactly one definition of the cores — `bbcores_new.txt`.** To add a core or change a landmark or a transition row, edit the txt; no recompilation is needed. `bbsearch.cpp` only parses and searches.

## 4. Development environment and common commands

```bash
cd /path/to/bbsearch

# Build
g++ -O3 -march=native -std=c++11 -o bbsearch bbsearch.cpp

# List the 56 cores (mandatory after changing the core table)
./bbsearch --cores

# Single core search
./bbsearch --core fw3_3 --states 9 --cap 50000 --budget 3e9

# All cores
./bbsearch --all --cap 200000 --budget 3e9 --log all.log

# Input-configuration mode
./bbsearch --core fwm2_3 --states 8 --start cfg --m 3 --k 2 --n 4
```

> The commands above are the Linux/macOS form. On Windows the binary is `bbsearch.exe` — write `.\bbsearch.exe` instead of `./bbsearch` (the build command is identical; `g++` appends `.exe` itself).

**Success criterion for a build**: g++ prints nothing, exit code 0, and the binary has a new timestamp.

## 5. Code style and conventions

Follow the style already in the file; do not introduce a new system:

- **Naming**: `g` prefix = global variables (`gNSym` `gSpec` `gSlots`…); `camelCase` = locals/functions; `SCREAMING_SNAKE` = constants/enums; `CoreSpec` `CoreEntry` `Seg` `Tr` = aggregate structs
- **Constants**: `static const int`, grouped in the "constants" region at the top of the file
- **Enums**: unnamed `enum { ... }` plus a comment, in the style of `SLOT_*` / `L5_*` / `KIND_*`
- **Strings**: core transition rows use compact literals such as `"---1LE2RA"`, 3 characters per cell (write symbol / move L,R / next state A..Y, or `Z` meaning "write then halt")
- **Indentation**: 4 spaces, no tabs
- **Comments**: explain *why*, do not restate *what*; the overview comment block at the top of the file must be updated when a new core type is added
- **C++11 only**: do not introduce C++14/17 features (`std::make_unique`, structured bindings, etc.); the compiler is invoked with `-std=c++11`

## 6. Testing requirements

**This project has no automated test framework and no test directory.** Verification is manual, and the following checks must be run after a change:

```bash
# 1) Core list self-check (core count, each core's state/symbol counts and row lengths)
./bbsearch --cores        # expect 56 rows + 1 header row; landmark and importance columns correct
# 2) Actual search
./bbsearch --core <changed core> --cap 20000 --budget 2e7
# 3) Full smoke run (confirm no crash)
./bbsearch --all --cap 3000 --budget 3e6
```

(On Windows: `.\bbsearch.exe` instead of `./bbsearch`.)

When you change a core in `bbcores_new.txt` (the `name`/`cfg`/`slot`/`core` four lines), the program reads it directly at run time, so **no recompilation is needed**; after the change run at least `--cores` and the `--all` smoke test.

The transition rows of every block in the core list must be checked **character by character** (each row length must be `3 × nsym`, the number of rows must equal the state count) — checking 56 multi-row strings by hand is extremely error-prone. An automated self-check script is in [docs/getting-started.md](docs/getting-started.md) under "Core list self-check script".

## 7. Git workflow and commit conventions

**This directory is a git repository** (initialised 2026-11-16, branch `main`).

- The first commit contains `bbsearch.cpp`, `bbcores_new.txt`, `README.md`, `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `LICENSE.md`, `.gitignore`, `docs/`
- The build artifact **must not be committed**: `bbsearch` on Linux/macOS, `bbsearch.exe` on Windows
- **Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/)**:

  ```
  <type>(<scope>): <subject>

  <body>

  <footer>
  ```

  - `<type>`: `feat` (new capability) · `fix` (bug fix) · `docs` (documentation only) · `refactor` (no behaviour change) · `perf` · `test` · `build` · `chore` (tooling, formatting, ignore rules)
  - `<scope>`: the area touched — `core` (the core list / landmark specs) · `search` (the DFS engine) · `cli` · `docs` · `build`; omit it when no single area applies
  - `<subject>`: imperative mood, lower case, no trailing period, ≤ 72 characters
  - `<body>`: separated by a blank line, wrapped at ~72 characters, explaining **why** the change was made; never write uninformative descriptions such as "update files"
  - `<footer>`: optional — `BREAKING CHANGE:` notes and issue references
  - Examples:
    - `feat(core): add the fwp22_5 core to bbcores_new.txt`
    - `docs: rewrite README and docs/ in English and make the build platform-neutral`
    - `chore(build): ignore the compiled binary and the log files`
- Unless the user explicitly asks, **do not commit or push on your own**

## 8. Key design principles and constraints

### Hard constraints (violating them breaks correctness)

1. **There is exactly one core definition — `bbcores_new.txt`** — the code has no built-in table; `gCores` is read from the file at run time and failing to read it must be a fatal error (never silently degrade to an empty table)
2. **Core transition row length = 3 × symbol count**, row count = state count; `---` is a FREE cell (the DFS fills it in on demand), `wRZ` means "write w, move R, then halt"
3. **There are only three landmark kinds** (`KIND_BCFG` / `KIND_THREE` / `KIND_ONES`); adding a fourth structure requires changing all of: the `kind` enum, `fmtLeft()`, `fmtTemplate()`, the `noteCfg()` dispatch, `buildStartConfig()`, `checkSpecSymbols()`, and the core file parser `parseCfg()` / `buildSpec()`
4. **There are exactly 8 counter slots** (`k n m p q r r5 s`); adding a slot requires syncing `SLOT_NAME[]`, the matching CLI option and the default-value array
5. **The landmark importance order must come verbatim from the `slot` line of `bbcores_new.txt`** — it is a lexicographic comparison, not a numeric magnitude comparison

### Common failure modes (avoid these)

- ❌ **Treating "budget exhausted, nothing found" as "the core does not exist"** — landmarks often appear deep in the DFS tree; the "no landmark configuration was reached" message after a `--budget` cutoff only means the budget was too small
- ❌ **Writing core rows from memory** — they must be copied character by character from `bbcores_new.txt`
- ❌ **Treating the `--cores` output of `bbsearch.cpp` as authoritative** — it reflects the code's own state, which may be exactly what is wrong; the authority is `bbcores_new.txt`
- ❌ **Writing change history in `AGENTS.md` / `README.md`** — that belongs in git log / CHANGELOG

### Search semantics

- `--start blank`: blank tape `0^inf A> 0^inf`, head in state A
- `--start cfg`: start from the documented **input configuration** (the `B(k,n,m) ...` family), head at the lowest state allowed by the landmark; this mode **only validates the left half** of the landmark (loose), because the machine transforms the input configuration into a larger one whose right half necessarily differs
- `--startstate` is needed for `dfw_4` / `tfw_5` — starting in state A halts them within a few steps

## 9. Rules for AI agents

- **Change only what was asked** — no opportunistic refactoring, no extra features, no "optimising" unrelated code. The one exception is new Turing machine cores, see the last bullet
- **Do not create files unless necessary** — prefer editing existing files; create documentation files only when explicitly requested
- **Do not proactively create README / CHANGELOG / documentation** — the project already has `README.md` and `docs/`; when syncing, **edit them in place** rather than appending narrative paragraphs
- **Deletion over retention** — completed temporary plans and superseded approaches are deleted outright
- **Absolute dates** — write concrete dates such as `2026-11-15` in documentation, never relative wording like "today" or "recently"
- **The code is the source of truth** — but when the code's behaviour is obviously a bug, do not rewrite the documentation to match the bug; mark it as "⚠️ suspected code bug" in your summary and let the documentation keep describing the intended behaviour
- **Background search processes** — use `run_in_background` for long runs; to terminate, use `Stop-Process -Id <pid> -Force`
- **New Turing machine cores are yours to add** — when you come across a core that `bbcores_new.txt` does not contain yet, **add it yourself** so that `bbsearch` can search its landmark configuration. This needs **no code change and no recompilation**: the core table is read from the txt at run time. A core block is four lines — `name` / `cfg` / `slot` / `core` — where `cfg` describes the landmark shape and the head state, `slot` gives the counter importance order and the counter minima, and `core` holds the transition rows (each row exactly `3 × nsym` characters, `---` = a FREE cell). Copy the transition rows **character by character** from the source; never retype them from memory. After adding a core, run the self-check script and `./bbsearch --cores` (the row count must grow by one and the landmark template / importance columns must be correct), then update the tables in `docs/cores.md`

## 10. Common pitfalls and known issues

| Pitfall | Symptom | Handling |
|---|---|---|
| Budget too small | `no landmark configuration was reached` | Increase `--budget` (on the order of `3e9`); do not jump to conclusions |
| `--symbols` smaller than the core's symbol count | `skipped (--symbols N < M symbols of the core)` | Use the core's own symbol count, or more |
| `KIND_BCFG` landmark needs symbols 1/2/3 | error exit | `--symbols` must be at least 4 (at least 6 when a 5-run is needed) |
| `dfw_4` / `tfw_5` die immediately from a blank tape | halts within 3–17 steps | Add `--startstate B` (or `--start cfg`) |
| `fw3_4` was replaced | the old 8-state version was removed from the txt | `fw3_4` is now the 7-state, 4-symbol version with landmark `1^m F> 0^n 1^k 2^p 3` |
| Core rows rewritten to a different length | compiles fine but behaves strangely | Every row length must be `3 × nsym`, row count must equal the state count |
| Running from another directory | `error: cannot open 'bbcores_new.txt'` (exit code 1) | Pass `--cores-file` with the list path, or run inside the project directory |
| A core in the list was not read | the core is not found / `--cores` is short by one row | A core must be written as the four lines `name` / `cfg` / `slot` / `core`; blocks in the old format (bare transition rows) are ignored |

## 11. Miscellaneous

- **The data source is `bbcores_new.txt`**: the program reads it at run time (`name` / `cfg` / `slot` / `core` four lines per block), and **the code contains no built-in copy**. The current txt holds **56** cores. Cores added to the list are appended to the end of the run-time table; when two blocks share a name, the later one in the file wins.
- **Semantics of the new format** (implemented by the parser `parseCfg()` / `buildSpec()`):
  - `<symbol>^<count>` = exactly that many cells (`4^1` = one 4); `<symbol>^<counter>` = length determined by that counter (`2^p`); `BLANK` = everything after the last segment must be blank (`termBlank`).
  - **Last-segment rule**: when the `cfg` line does **not** end with `BLANK`, the last fixed segment **without a slot** (`1^1` / `2^1` / `3^1` / `5^1`) is not locked to exactly one cell but means "≥1 cell" (`{sym,1,-1,-1}`, i.e. `maxCnt = -1`). `BLANK` and `KIND_THREE` (whose tail is always blank) are unaffected; a trailing `0` (blank cell) is still treated as a sentinel.
  - `KIND_THREE` (`<state letter>> ...`) is always `termBlank` (the head lands on the first cell of the counter segment, the rest is blank).
  - `5^1` = exactly one 5 on the left; `5^r` = a run of 5 on the left, counted into slot `r5`.
  - The `slot` line gives the importance order (lexicographic) and the counter minimums (`p>=1`); the minimum is written into the corresponding segment's `minCnt`.
  - The `r` in the `slot` line: when the cfg has a `5^r` prefix it means `r5`, otherwise it means the `r` of the right-hand `1^r`.
  - The trailing `0^1` in `fw_3`'s `cfg 1^n A> 1^1 0^1` is a blank **sentinel** segment (only that one cell is checked for being blank); the blank region after it is not required.
- **The landmark shape, slot order and special spellings of each core** (`fw1_4` / `fw2_4` / `fw2_5`'s `B'` family, `fwm2_5`'s `D` head state, `fw1fw_4`'s start state, the left-hand `5^r` of `fwp2_6` / `fwp21_6` / `fwp21_7` / `fwp22_7`, etc.) are documented in [docs/cores.md](docs/cores.md) under "Core table" and "Special core notes"; the `cfg` / `slot` lines in the txt are authoritative.
- **`dfwpw_5` is the double `f_(ω^ω)` core**: landmark `0^inf 3^(2m+1) 2^k A> 1^n 4 3 0`, importance order `k > n > m`, `right[]` = `{4,1,1,-1}`, `{3,1,1,-1}`, `termBlank` is true. The txt's `cfg` reads `BCFG(A) 4^1 3^1 BLANK` — the trailing blank is expressed by `BLANK`, and you **must not** add an explicit `0^1` segment: the `KIND_BCFG` matching branch counts the number of consecutive blank cells (far more than 1), so it would never match. `H0` / `K0` in the core are FREE cells; `--fix H0=3RJ,K0=3LI` completes them into the full machine corresponding to the txt.
- **`KIND_BCFG`'s `termBlank` checks the entire right side**: after the last segment, the scan to the right must find only blanks, so `4 3 0 1` (writing `1` one cell after the `3`) is not `4 3 0 0`. Different FREE completions of the same core produce different right-hand shapes.
- **`SLOT_R5` (`r5`) and `SLOT_R` (`r`) are two different slots**: `r` counts the right-hand `1^r` segment, `r5` counts the left-hand `5^r` segment. In the txt both are written as `r` and are disambiguated by context — the `r` in the `slot` line of `fwp2_6` / `fwp21_6` / `fwp21_7` / `fwp22_7` means `r5`.
- **`KIND_ONES` is the only "left-hand run of `1`" shape**: the landmark looks like `0^inf 1^m <head> <right config>`, the head cell is the **first cell** of the `right[0]` segment (notation convention: the segment after the `<head>` marker starts at the head cell), the head has `m` cells of `1` immediately to its left (`m >= 1`) and everything further left must be blank; `m` is recorded in `leftSlot`, and `right[0]` and the following segments are recorded in `sp.right[]`.
- **The head segment of `KIND_ONES` (`right[0]`) has three forms**: a counter segment (`1^n` in `fw1_3`, `2^n` in `fe0_5`, `maxCnt = -1`, unbounded), a fixed single blank cell (the lone `0` in `fwm2_3`), or a fixed single symbol (the `1` in `fw_3`).
- **When the last segment of `KIND_ONES` is a blank cell without a slot** (`fw_3`'s `1^n A> 1 0`), it is treated as a **sentinel**: only that one cell is checked for being blank, and the blank region after it does not count towards the length.
- **`KIND_THREE` (`fwp2_4`) has its counter segment right under the head**: `right[]` holds a single segment `{3, 3, -1, SLOT_N}`, the head cell is the first cell of that `3^n` run and the rest of the tape must be blank; `n >= 3`.
- **`KIND_THREE` is also multi-segment** (`fwp21_4` / `fwp21_5` / `fwp22_4` / `fwp22_5` / `fwp23_4`): `right[]` = `{3,1,-1,SLOT_M}`, `{0,<n minimum>,-1,SLOT_N}`, several counter segments, and finally a sentinel segment (a fixed single cell), after which the tape must still be blank. The per-core `right[]` values are in the txt and in docs/cores.md.
- **The left-hand `1^n` of `KIND_ONES` can be contiguous with the right-hand `1^m`** (`fwp21fw_4` / `fwp22fw_4` / `fw1_3`): in the landmark `1^n G> 1^m 0`, `1^n` and `1^m` are **the same run of `1`s**, with the head cell inside it — `n` cells of `1` to the left of the head, and `m` cells of `1` from the head cell rightwards (the `m` count includes the head cell); the trailing `0` is the blank sentinel segment `{0,1,1,-1}` of `right[]`. `fw1_3` uses the same convention with the roles of the two counters swapped (`1^m F> 1^n 2`).
- **The maximum state count is `MAXST = 25`** (A..Y): `'Z'` in a transition row is the "write then halt" marker and **is not an ordinary state**; so `'Y'` is the 25th ordinary state, and the `'Z' → HALT_Z` encoding in `setRow()` is compatible with this by construction. `--startstate` accepts A..Y.
- **Measured reachability** (which cores reach their landmark under the default budget, the steps / nodes / abort numbers, and suggestions for the next round) is in [docs/cores.md](docs/cores.md) under "Measured reachability" and "Cores with no landmark found" — those are dated measurement results and do not belong here.
- **`buildStartConfig()`'s `KIND_BCFG` branch must write from right to left**: first `2^k`, then `3^(2m+1)`, finally `5^r` — the landmark requires `0^inf 3^(2m+1) 2^k A>` (the `3` segment leftmost, the `2` segment adjacent to the head); writing them the other way round makes `--start cfg` build the wrong input-configuration shape. This only affects `--start cfg` mode; blank-tape searches are unaffected.
