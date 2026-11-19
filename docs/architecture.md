# Architecture

`bbsearch.cpp` is a single-file implementation (about 1856 lines). This document describes its internal structure in the order "data → spec → engine → CLI".

## Contents

- [1. Core table (from the core file)](#1-core-table-from-the-core-file)
- [2. Landmark spec](#2-landmark-spec)
- [3. Counter slots](#3-counter-slots)
- [4. The three landmark kinds](#4-the-three-landmark-kinds)
- [5. Search engine](#5-search-engine)
- [6. Cycle detection: Zobrist + open-addressing table](#6-cycle-detection-zobrist--open-addressing-table)
- [7. Landmark matching and ranking](#7-landmark-matching-and-ranking)
- [8. The two start modes](#8-the-two-start-modes)
- [9. CLI layer](#9-cli-layer)
- [10. Overall data flow](#10-overall-data-flow)

---

## 1. Core table (from the core file)

The core table has **no built-in copy**: the program reads it at start-up from [bbcores_new.txt](../bbcores_new.txt) (`--cores-file` changes the path) into the run-time table `gCores`; each `CoreEntry` holds the name, fixed state count, symbol count, transition rows and that core's landmark spec:

```cpp
struct CoreEntry {
    std::string name;
    int  nfixed;                     // number of fixed states (A..)
    int  nsym;                       // number of symbols covered by the core rows
    std::vector<std::string> rows;   // rows[s], each row is 3*nsym characters
    CoreSpec spec;
};
```

There is one row per state, each row listing symbols 0..nsym-1, **3 characters per cell**:

```
write symbol  move  next state
  1             L       E
```

| Encoding | Meaning |
|---|---|
| `---` | FREE cell: the DFS branches here and fills the table |
| `wRZ` | write `w`, move `R`, then halt |
| `wLZ` | write `w`, move `L`, then halt |
| `1LE` | write `1`, move `L`, enter state E |

Each core in the core file is a four-line block:

```
name <core-name>
cfg  <landmark description>
slot <counter importance order>
core <transition rows, states separated by _>
```

The `cfg` line supports these spellings:

| Spelling | kind | Meaning |
|---|---|---|
| `[5^1\|5^r] BCFG(A\|B) <seg>...` | `KIND_BCFG` | The head is the first cell of the `1^n` run; the right side is the listed segments |
| `<state letter>> <seg>...` | `KIND_THREE` | The head is the first cell of the first segment; the right side is the listed segments |
| `1^<counter> <state letter>> <seg>...` | `KIND_ONES` | The head is the first cell of the first segment, with `<counter>` cells of `1` immediately to its left |

- A segment `<symbol>^<count>` means exactly that many cells; `<symbol>^<counter>` means the length is determined by that counter; `BLANK` means everything after it must be blank.
- The `slot` line lists the counters from highest to lowest importance, optionally with minimums: `p>=1 k n m`.
- The `r` of `5^r` is the left-hand 5-run (slot `r5`), and is **disambiguated by context** from the `r` of the right-hand `1^r` (slot `r`): when the cfg has a `5^r`, the `r` in the `slot` line means `r5`.

## 2. Landmark spec

```cpp
struct CoreSpec {
    int      kind;         // KIND_BCFG / KIND_THREE / KIND_ONES
    int      left5;        // L5_NONE / L5_ONE / L5_MANY
    unsigned headMask;     // bit mask of the allowed head states
    Seg      right[6];     // right-hand segments (at most 6)
    int      nright;
    bool     termBlank;    // require a blank cell after the last segment
    int      slot[8];      // counter slots, from highest to lowest importance
    int      nslot;
    int      leftSlot;     // KIND_ONES: which slot receives the count of the 1^m run left of the head
};
```

The spec is parsed from the `cfg` / `slot` lines and stored in that core's `CoreEntry::spec`; `gSpec` is an **overridable copy of the current core's spec** (`--left5` / `--right` / `--slots` / `--noblank` modify it, not the core file).

Right-hand segments:

```cpp
struct Seg { uint8_t sym; int minCnt; int maxCnt; int slot; };
```

- `sym`: the segment's symbol
- `minCnt`: the minimum number of cells (a counter level must be non-empty, otherwise the landmark degenerates)
- `maxCnt`: `-1` = unbounded
- `slot`: which counter slot receives the length, `-1` = not a counter (e.g. a trailing fixed symbol `2`)

## 3. Counter slots

```cpp
enum { SLOT_K = 0, SLOT_N = 1, SLOT_M = 2, SLOT_P = 3, SLOT_Q = 4,
       SLOT_R = 5, SLOT_R5 = 6, SLOT_S = 7, SLOT_NUM = 8 };
static const char *SLOT_NAME[SLOT_NUM] = { "k", "n", "m", "p", "q", "r", "r5", "s" };
```

| Slot | Meaning |
|---|---|
| `k` | Length of the `2^k` run left of the head |
| `n` | Length of the counter segment right of the head (`1^n` for `KIND_BCFG`, `3^n` for `KIND_THREE`, `0^n` / `2^n` for `KIND_ONES`) |
| `m` | `(length-1)/2` of the `3^(2m+1)` run on the left |
| `p` `q` `r` `s` | Lengths of the right-hand counter segments (`2^p`, `4^q`, `1^r`, `2^s`, etc.; the exact symbols are decided by each core's `cfg` line) |
| `r5` | Length of the `5^r` run on the left |

**Note that `r` and `r5` are two different slots.** In `bbcores_new.txt` both are written as `r` and are disambiguated by context: the `r` in the `slot` line of `fwp2_6` / `fwp21_6` / `fwp21_7` / `fwp22_7` means `r5`.

**The importance order** (`spec.slot[]`) comes from the `slot` line of `bbcores_new.txt`. Comparison is **lexicographic**, not by numeric magnitude. For example `fw3_4` is `p > k > n > m`, meaning: compare `p` first, then `k` if `p` is equal, and so on.

## 4. The three landmark kinds

### `KIND_BCFG` — the classic shape

```
0^inf [5^r] 3^(2m+1) 2^k <head> 1^n <right>
```

- `[5^r]`: `L5_NONE` (none) / `L5_ONE` (exactly one 5) / `L5_MANY` (`5^r`, `r>=1`)
- `<head>`: the head states allowed by `headMask`, decided by the `BCFG(A)` / `BCFG(B)` / `BCFG(D)` of the `cfg` line (`fwm2_5` is the only core with the head in state `D`)
- `<right>`: described by `spec.right[]`

Right-hand segments may carry no slot (fixed-length symbols) or carry one (counters). For example `fwpw_5`'s landmark `0^inf 3^(2m+1) 2^k A> 1^n 4 2` is two fixed segments `{4,1,1,-1}` and `{2,1,-1,-1}`, with `termBlank` false — no blank is required after the `2`. `fwpw1_5a` is `0^inf 3^(2m+1) 2^k A> 1^n 4 2^p 0`: `{4,1,1,-1}` and `{2,1,-1,SLOT_P}`, with `termBlank` true — everything after `2^p` must be blank.

**Last-segment rule**: when the cfg line does not end with `BLANK`, the last fixed segment without a slot is not locked to exactly one cell but means "≥1 cell" (`maxCnt = -1`) — a spelling like `2^1` only says "the `2` segment has at least one cell", not that it has exactly one. `BLANK` (and `KIND_THREE`, whose tail is always blank) are unaffected; a trailing blank cell `0` is treated as a sentinel (only that one cell is checked).

### `KIND_THREE` — the counter segment sits right under the head

```
0^inf <head> 3^n 0^inf
```

Used by `fwp2_4` and `fwp2_5`: in `fwp2_4` the head state is `A` / `C` / `G` (all three have the transition `3LG` on symbol `3`), in `fwp2_5` it is `C` / `G` only. In both cases the head cell is the first cell of the `3^n` run and the rest of the tape is blank. The counter `n` is the length of that `3` run (`n >= 3`); there is no other counter.

Matching is done by `noteCfgThree()`: scan left to the end confirming everything is blank, count `3^n` to the right according to `spec.right[]`, then confirm everything after it is blank. `right[]` holds a single segment `{3, 3, -1, SLOT_N}` (symbol 3, at least 3 cells, counter slot `n`).

`KIND_THREE` can also be multi-segment (`fwp21_4` / `fwp21_5` / `fwp22_4` / `fwp22_5` / `fwp23_4`): `right[]` is `{3,1,-1,SLOT_M}`, `{0,<n minimum>,-1,SLOT_N}`, several counter segments, and finally a sentinel segment (a fixed single cell), after which the tape must still be blank. The minimum of `n` comes from the `slot` line (`fwp21_4` is `n>=2` → `{0,2,...}`, `fwp21_5` is `n>=1` → `{0,1,...}`). `fwp22_4` is three counters plus a fixed `2`: `{3,1,-1,SLOT_M}`, `{0,2,-1,SLOT_N}`, `{1,1,-1,SLOT_K}`, `{2,1,1,-1}`; `fwp22_5` replaces the `0^n` run with a `4^n` run and ends in a fixed `4`: `{3,1,-1,SLOT_M}`, `{4,1,-1,SLOT_N}`, `{0,1,-1,SLOT_K}`, `{4,1,1,-1}`; `fwp23_4` adds a `2^p` counter and a fixed `3`: `{3,1,-1,SLOT_M}`, `{0,2,-1,SLOT_N}`, `{1,1,-1,SLOT_K}`, `{2,1,-1,SLOT_P}`, `{3,1,1,-1}`.

### `KIND_ONES` — a run of `1^m` on the left, head in the first cell of the right-hand first segment

```
0^inf 1^m <head> <right config>
```

The uniform "there is a run of `1`s left of the head" shape. Notation convention: the segment after the `<head>` marker starts at the head cell, i.e. **the head cell is the first cell of the `right[0]` segment**.

- The head has `m` cells of `1` immediately to its left (`m >= 1`), and everything further left must be blank; that run of `m` cells is recorded in `leftSlot`
- `right[0]` is the segment the head cell belongs to, followed by `right[1]`, `right[2]`, …
- When `termBlank` is true, the whole tape after the last segment must be blank

Typical shapes:

| Core | Landmark | `right[]` | `leftSlot` |
|---|---|---|---|
| `fw1_3` | `1^m F> 1^n 2` | `{1,1,-1,SLOT_N}`, `{2,1,-1,-1}` | `SLOT_M` |
| `fw2_3` | `1^m F> 0^n 1^k 2` | `{0,1,-1,SLOT_N}`, `{1,1,-1,SLOT_K}`, `{2,1,-1,-1}` | `SLOT_M` |
| `fw3_3` | `1^m F> 0^n 1^k 2^p 0` | `{0,1,-1,SLOT_N}`, `{1,1,-1,SLOT_K}`, `{2,1,-1,SLOT_P}`, `{0,1,1,-1}` | `SLOT_M` |
| `fw3_4` | `1^m F> 0^n 1^k 2^p 3` | the above plus `{3,1,-1,-1}` | `SLOT_M` |
| `fwm2_3` | `1^m B> 0 2^k 1^n 0` | `{0,1,1,-1}`, `{2,1,-1,SLOT_K}`, `{1,1,-1,SLOT_N}`, `{0,1,1,-1}` | `SLOT_M` |
| `fwm21_3` | `1^m B> 0 2^k 1^n 0 1^p 2` | `{0,1,1,-1}`, `{2,1,-1,SLOT_K}`, `{1,1,-1,SLOT_N}`, `{0,1,1,-1}`, `{1,1,-1,SLOT_P}`, `{2,1,-1,-1}` | `SLOT_M` |
| `fwm21_4` | `1^m B> 0 2^k 1^n 0^p 3` | `{0,1,1,-1}`, `{2,1,-1,SLOT_K}`, `{1,1,-1,SLOT_N}`, `{0,1,-1,SLOT_P}`, `{3,1,-1,-1}` | `SLOT_M` |
| `fwm22_3` | `1^m B> 0 2^k 1^n 0 1^p 2^q 0` | `{0,1,1,-1}`, `{2,1,-1,SLOT_K}`, `{1,1,-1,SLOT_N}`, `{0,1,1,-1}`, `{1,1,-1,SLOT_P}`, `{2,1,-1,SLOT_Q}` (then blank) | `SLOT_M` |
| `fw_3` | `1^n A> 1 0` | `{1,1,1,-1}`, `{0,1,1,-1}` | `SLOT_N` |
| `fwp21fw_4` / `fwp22fw_4` | `1^n G> 1^m 0` | `{1,1,-1,SLOT_M}`, `{0,1,1,-1}` | `SLOT_N` |
| `fe0_5` | `1^m B> 2^n 0^inf` | `{2,3,-1,SLOT_N}` | `SLOT_M` |

Two special cases:

- **The head segment is blank and exactly one cell long** (`{0,1,1,-1}` in `fwm2_3`): the zero segment is not a counter; all counters live in the right-hand segments
- **The last segment is a blank cell without a slot** (`{0,1,1,-1}` in `fw_3`): treated as a **sentinel** — only that one cell is checked for being blank, and the blank region after it does not count towards the length

The decision order of `noteCfgOnes()`: the head cell must be `right[0].sym` → count the consecutive `1`s to the left to get `m` (written into `leftSlot`) → everything further left must be blank → walk right segment by segment according to `spec.right[]` (each segment's length goes into its own slot; a trailing blank cell without a slot is only a sentinel check) → when `termBlank` is true the right side must be blank. In `--start cfg` (loose) mode only the left half is checked; the right side is free.

## 5. Search engine

```cpp
static void search(int pos, int st, long long steps, uint64_t h, int lo, int hi);
```

The main loop (infinite loop + recursion):

1. Boundary checks: `gAbort` / `steps >= CAP` / head out of range → return
2. **Landmark matching**: `noteCfg(st, pos, lo, hi, steps)` — every step checks whether the current configuration is a candidate landmark
3. **Cycle detection**: `htTry(key)` fails → return (this path has already been explored)
4. Read the transition `trans_[st][sym]`
   - `UNASSIGNED` (a FREE cell) → **branch**: recurse over all `(write symbol, move, next state)` combinations; `--fill K` limits how many free cells may be filled
   - `HALT_UNDEF` (a cell pinned to halt by `--fix`) → return
   - otherwise → write the symbol, move, step, and continue the loop
5. `HALT_Z` (`wRZ`, write then halt) → return

When branching, the new state must satisfy `allowNst()`: states are normalised **by order of introduction** (states 0, 1, 2 come before 3, 4, …), so that different spellings of the same machine are not explored twice.

**Search space**: the blank tape starts at `0^inf A> 0^inf`, with `CELLS = 1<<16` cells centred.

## 6. Cycle detection: Zobrist + open-addressing table

```cpp
static uint64_t zob_[MAXSYM * CELLS];   // one random number per (symbol, position)
static uint64_t zpos_[CELLS];           // position
static uint64_t zst_[MAXST];            // state
```

- **Zobrist hash**: `h` = the hash of all non-blank cells XOR the state hash XOR the head-position hash. Writing a symbol only requires XORing the old and new hashes, an O(1) incremental update.
- **Open-addressing table**: `HB = 22`, `HS = 1<<22` slots (4M), open addressing with linear probing holds `hkey_` / `hocc_`; a failed `htry()` insert means a cycle. `htUndo()` supports backtracking.

## 7. Landmark matching and ranking

```cpp
static void noteCfg(int st, int pos, int lo, int hi, long long step);
```

Dispatched on `gSpec.kind`:

- `KIND_BCFG` → match the left half (`0^inf [5^r] 3^(2m+1) 2^k`) + the head state + `1^n` + the right-hand segments
- `KIND_THREE` → `noteCfgThree()` (the head cell must be non-blank; the counter segment is counted from the head cell)
- `KIND_ONES` → `noteCfgOnes()` (the head cell must be `right[0].sym`; the run of `1^m` on the left is a counter)

The decision order of `noteCfgOnes()`:

1. The head cell must be `right[0].sym`
2. `m >= 1` cells of `1` immediately to the left (with `m` written into `leftSlot`)
3. Everything further left must be blank
4. Match `spec.right[]` segment by segment starting at the head cell, writing each segment's length into the corresponding `slot` (a trailing blank cell without a slot is only a sentinel check)
5. When `termBlank` is true, everything after the last segment must be blank

On a successful match, `considerCfg()` is called: the counter values are compared against the current best in the lexicographic order given by `spec.slot[]`; if larger, `bestCfg` is replaced and `[maxcfg] NEW BEST` is printed.

**Loose mode** (`gLoose`): with `--start cfg` only the **left half** of the landmark is validated and the right side is free. This is because the machine transforms the input configuration into a larger configuration, so the right half necessarily differs from the input.

## 8. The two start modes

### `--start blank` (default)

```cpp
static void startBlank();   // 0^inf A> 0^inf, head at CELLS/2, state A
```

`--startstate X` changes the start state. Some cores (`dfw_4` / `tfw_5`) halt within a few steps when started in state A and must be started elsewhere.

### `--start cfg`

```cpp
static int buildStartConfig(uint64_t &h);   // builds the input configuration on the tape per gSpec
static void startAtConfig();
```

The documented **input configuration** of the core is laid out on the tape using the values of `--k --n --m --p --q --r --r5`, the head is placed at `firstHeadState()` (the lowest state in `headMask`), and the search then proceeds in loose mode.

`buildStartConfig()` handles the three kinds separately:

- `KIND_BCFG`: place `2^k`, `3^(2m+1)` and `5^r` (when `L5_MANY`) on the left; place `1^n` to the right of the head; then place the right-hand segments per `spec.right[]`. All the left-hand segments are written towards further left, so they must be placed **from right to left**: `2^k` adjacent to the head, `3^(2m+1)` to its left, `5^r` leftmost — matching the landmark `0^inf [5^r] 3^(2m+1) 2^k <head>`
- `KIND_THREE`: place the segments one by one starting at `head` per `spec.right[]` (`fwp2_4` is a single `3^n` run; `fwp21_4` / `fwp21_5` are `3^m`, `0^n` and a sentinel), and the head cell is the first cell of the first segment
- `KIND_ONES`: place `m` cells of `1` to the left of the head (`m` takes the input value of `leftSlot`, padded to at least 1); place the segments one by one **starting at `head`** per `spec.right[]` — a segment's length comes from the input value of its slot, and a segment without a slot uses `maxCnt` (a fixed-length segment), still padded up to `minCnt` when short. The head cell is the first cell of the first segment

## 9. CLI layer

```cpp
int main(int argc, char **argv);
```

Argument parsing → validation (`--symbols` 2..10, `--states` 3..25) → `initZobrist()` → read the core file (`--cores-file`, default `bbcores_new.txt`; failing to read it is a fatal error) → dispatch:

- `--cores` → `listCores()`: print the core table + landmark templates + importance
- `--all` → call `runCore()` for every core, each with `max(nStates, core.nfixed)` states
- otherwise → a single core via `runCore()`

The `runCore()` flow: `setupCore()` loads the core rows → `checkSpecSymbols()` / `checkCoreWrites()` validate the landmark symbols and the core's written symbols → parse the landmark overrides (`--left5` / `--right` / `--slots` / `--noblank`) → start the search → print the best configuration.

## 10. Overall data flow

```
bbcores_new.txt (data source)
        │
        │  parsed at run time: name/cfg/slot/core
        ▼
   gCores (run-time core table)────► gSpec ──► fmtLeft / fmtTemplate / noteCfg / buildStartConfig
                                  │
                                  ▼
   CLI options ──► gSpec override ──► search() ──► noteCfg() ──► considerCfg()
   (--left5/--right/--slots/--noblank)      │                │
                                    ▼                ▼
                            trans_[st][sym]      bestCfg
                            Zobrist cycle detection      (lexicographic ranking)
```
