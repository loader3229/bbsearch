# Local Development Guide

For developers who want to modify `bbsearch.cpp`.

## Contents

- [1. Requirements](#1-requirements)
- [2. Building](#2-building)
- [3. First run](#3-first-run)
- [4. Standard workflow for changing a core](#4-standard-workflow-for-changing-a-core)
- [5. Standard workflow for changing a landmark spec](#5-standard-workflow-for-changing-a-landmark-spec)
- [6. Verification checklist](#6-verification-checklist)
- [7. Performance tips](#7-performance-tips)
- [8. Debugging](#8-debugging)

---

## 1. Requirements

| Item | Requirement | Verification command |
|---|---|---|
| OS | Linux / macOS / Windows | — |
| Compiler | Any C++11 compiler (`g++` recommended) | `g++ --version` |
| Third-party dependencies | **None** | — |
| Build system | **None** (single-command build) | — |

`g++` must be on `PATH`. On Debian/Ubuntu: `sudo apt install g++`; on macOS: `xcode-select --install`; on Windows: install MinGW-w64 or MSYS2 and put `g++` on `PATH`. The project was developed on Windows with TDM-GCC-64, but the source itself is platform-independent — it uses only the standard library and no platform-specific calls.

## 2. Building

Run the command in the repository root:

```bash
cd /path/to/bbsearch
g++ -O3 -march=native -std=c++11 -o bbsearch bbsearch.cpp
```

**Success criterion for a build**: g++ prints nothing, the exit code is 0, and the binary has an updated timestamp.

On Windows the compiler appends `.exe` itself, so the produced binary is `bbsearch.exe` even though `-o bbsearch` was given.

### Build option notes

| Option | Effect | Can it be removed |
|---|---|---|
| `-O3` | Optimisation | Not recommended; the DFS is the hot spot |
| `-march=native` | Native instruction set | Remove it if another machine reports illegal instruction |
| `-std=c++11` | Language standard | **Do not change to c++14/17**; the code is only guaranteed to be C++11 compatible |

## 3. First run

```bash
cd /path/to/bbsearch

# List the 56 cores: name / states / symbols / landmark template / importance order
./bbsearch --cores

# Run a fast small core to confirm it produces a result
./bbsearch --core fw_4 --states 3 --cap 50000 --budget 1e8
```

> The commands in this document use the Linux/macOS form. On Windows the binary is `bbsearch.exe` — write `.\bbsearch.exe` instead of `./bbsearch`.

Expect output of the form:

```
[maxcfg] NEW BEST  0^inf 3^(1) 2^16 A> 1^3 2
---- fw_4 done: steps=... nodes=... abort=0 ----
best 0^inf 3^(1) 2^16 A> 1^3 2
```

## 4. Standard workflow for changing a core

**The authoritative data source is [bbcores_new.txt](../bbcores_new.txt)**, read by the program at run time (`--cores-file` changes the path). `bbsearch.cpp` has **no built-in core table** — failing to read the file is a fatal error, so every core in the list must use the four-line format below.

### The new format

A core is a four-line block:

```
name fw_4
cfg  BCFG(A) 2^1
slot k n m
core ---3LB1RZ2RA_---3RB1LC2RA_3RB1LB3LC2RC
```

| Line | Contents |
|---|---|
| `name` | The core name, matching the `--core` name |
| `cfg` | The landmark description (see below) |
| `slot` | The counter importance order, optionally with minimums: `p>=1 k n m` |
| `core` | The transition rows, states separated by `_`; each row is 3×symbol-count characters, `---` is a FREE cell |

The three spellings of `cfg`:

| Spelling | Meaning |
|---|---|
| `[5^1\|5^r] BCFG(<states>) <seg>...` (e.g. `BCFG(A)`, `BCFG(B)`) | The classic shape: the head is the first cell of the `1^n` run, the right side is the listed segments |
| `<state letter>> <seg>...` (e.g. `ACG>`) | The head is the first cell of the first segment, the right side is the listed segments |
| `1^<counter> <state letter>> <seg>...` (e.g. `1^n A>`) | The head is the first cell of the first segment, with `<counter>` cells of `1` immediately to its left |

The segment list (`<seg>...`) spells segments like this:

| Spelling | Meaning |
|---|---|
| `<symbol>^<count>` (e.g. `4^1`) | Exactly that many cells |
| `<symbol>^<counter>` (e.g. `2^p`) | The length is determined by that counter (the same-named counter in the `slot` line) |
| `BLANK` | Everything after the last segment must be blank |

The `5^r` prefix means there is a run of 5s on the left (`5^1` = exactly one, `5^r` = the length is determined by counter `r`, corresponding to slot `r5`). The `r` in the `slot` line means `r5` when a `5^r` prefix is present, and otherwise the `r` of the right-hand `1^r`.

### Steps

1. **Edit the txt**: write the core in the four-line format above. If the file still contains blocks in the old format (a comment block plus a bare transition row), they are **ignored** (they produce no core at all).
2. **Update the header comment**: the top of `bbsearch.cpp` carries a full overview comment (data source, the landmark not being unique, the importance order being per core). It must be updated whenever a new landmark structure or a new kind of head state is added.
3. **Build and verify** (see section 6).

### Checking row lengths

```bash
# Per core: the row count == the state count, each row length == 3 * symbol count
./bbsearch --cores
```

### Core list self-check script

There is exactly one definition of the cores, in the txt, with no second copy to compare against, so the list itself must be **self-checked**: walk every block and verify the transition rows' lengths and characters. With Python (run from the repository root):

```python
import re
txt = open('bbcores_new.txt', encoding='utf-8', errors='replace').read()

cores, cur = {}, None
for ln in txt.split('\n'):
    s = ln.strip()
    if s.startswith('name '):
        cur = s[5:].strip()
        cores[cur] = []
    elif s.startswith('core ') and cur:
        cores[cur] = s[5:].strip().split('_')

bad = 0
for name, rows in cores.items():
    if not rows:
        print(f'{name}: no core rows'); bad += 1; continue
    lens = {len(r) for r in rows}
    if len(lens) != 1:
        print(f'{name}: row lengths differ {sorted(lens)}'); bad += 1; continue
    if next(iter(lens)) % 3:
        print(f'{name}: row length {next(iter(lens))} is not a multiple of 3'); bad += 1
    for r in rows:
        for cell in (r[i:i+3] for i in range(0, len(r), 3)):
            if cell != '---' and not re.fullmatch(r'\d[LR][A-Z]', cell):
                print(f'{name}: bad cell {cell!r}'); bad += 1
print('ALL OK' if not bad else f'{bad} PROBLEM(S)')
print('cores:', len(cores))
```

## 5. Standard workflow for changing a landmark spec

The meaning of every field of the landmark spec (`CoreSpec`) is in [architecture.md](architecture.md#2-landmark-spec).

**The landmark spec is edited directly in the txt's `cfg` / `slot` lines; no recompilation is needed afterwards.**

### Common changes

| Change | What to touch |
|---|---|
| Change the landmark shape | Edit the segment list and `BLANK` on the txt's `cfg` line |
| Change the importance order | Edit the txt's `slot` line |
| Allow a new head state | Add `#define HEAD_X (1u << n)`, set the bit in `headMask`; add a fallback in `firstHeadState()` if needed |
| Add a new landmark kind | `enum` gains `KIND_*` → `fmtLeft()` → `fmtTemplate()` → the `noteCfg()` dispatch → the matching `noteCfg*()` → `buildStartConfig()` → `checkSpecSymbols()` |
| Add a counter slot | `SLOT_*` enum → `SLOT_NAME[]` → the `gCfgVal[]` / `gSlotKnown[]` defaults → the matching CLI option |
| Left-hand 5-run | Set `left5` to `L5_NONE` / `L5_ONE` / `L5_MANY` |

### Trying things out without recompiling

`--left5` / `--right` / `--slots` / `--noblank` override the spec from the command line:

```bash
# Try a guess for the right-hand segments: symbol:min:max (-1 = unbounded)
./bbsearch --core fw3_5 --right "4:1:-1,3:1:-1,1:1:-1"

# Try another importance order
./bbsearch --core fw3_5 --slots R,Q,P,K,N,M

# Do not require a blank after the last segment
./bbsearch --core fw1_5 --noblank
```

Once you know which guess works, bake it into the txt's `cfg` / `slot` lines.

## 6. Verification checklist

After a change, execute **each** of these:

```bash
cd /path/to/bbsearch

# 1) Build
g++ -O3 -march=native -std=c++11 -o bbsearch bbsearch.cpp

# 2) Core list self-check (script in section 4): every block's row lengths and cells

# 3) The core list output is correct: row count == core count, landmark templates and importance correct
./bbsearch --cores

# 4) Actual search on the changed core
./bbsearch --core <core-name> --states <N> --cap 20000 --budget 2e7

# 5) Full smoke run, confirming no crash
./bbsearch --all --cap 3000 --budget 3e6
```

(On Windows the binary is `bbsearch.exe` — write `.\bbsearch.exe` instead of `./bbsearch`.)

Expectation for step 3: `--cores` prints **56 rows** (plus one header row).

## 7. Performance tips

The search is exponential; the budget decides the coverage:

| Option | Effect | Recommendation |
|---|---|---|
| `--cap N` | Step limit for a single run (a single DFS path) | Too small cuts off deep paths; start at `50000` |
| `--budget N` | Total step budget for the run | `3e9` is a practical magnitude; `0` = unlimited |
| `--fill K` | At most K FREE cells may be filled | Limits the branching explosion |
| `--states N` | Number of machine states | Rows beyond the core's fixed states are entirely FREE, so more branching |
| `--symbols N` | Alphabet size | Columns beyond the core's symbols are also FREE, so more branching |

**Do not treat "budget exhausted" as "the core does not exist"** — landmarks often appear deep in the DFS tree. `abort=1` means the budget ran out; only `abort=0` together with "no landmark" means it truly was not found.

For long runs, use the background:

```bash
./bbsearch --all --cap 200000 --budget 3e9 --log all.log
```

To terminate: `Stop-Process -Id <pid> -Force` (Windows) or `kill <pid>` (Linux/macOS).

## 8. Debugging

### Printing the transition table of the best configuration

After finding the best configuration, `runCore()` prints the transition table (`printTabF`):

```
best 0^inf 3^(1) 2^16 A> 1^3 2
  step=1055139 states_used=3
  A0 --- A1 1LB A2 3LB
  ...
```

### Surveying the observed right-hand shapes

`--census` does not match the landmark; it counts the distribution of actually observed right-hand shapes. `--censusL` keeps only the shapes whose left half already matched. Use these to "guess what the landmark looks like".

### Pinning down transition cells

`--fix` pins a cell so the DFS no longer branches there:

```bash
./bbsearch --core fw1_6 --fix "A0=4LC,B0=1RZ"
```

The format is `CELL=WRITE-MOVE-NEXT`, where `CELL` is a state letter plus a symbol digit (e.g. `A0`). It errors out when it conflicts with the core's existing definition.

### Common error messages

| Message | Cause |
|---|---|
| `bad --symbols N (want 2..10)` | The alphabet size is out of range |
| `bad --states N (want 3..25)` | The state count is out of range |
| `the landmark needs symbols 1, 2 and 3 but --symbols N has only symbols 0..` | A `KIND_BCFG` landmark needs at least 4 symbols |
| `the landmark's right side needs symbol X but --symbols N has only symbols 0..` | A right-hand segment uses a symbol that is not in the alphabet |
| `skipped (--symbols N < M symbols of the core)` | The alphabet is smaller than the core's; the extra columns are dropped |
| `bad --core 'xxx' (use one of: ...)` | The `--core` name does not exist (an index also works) |
