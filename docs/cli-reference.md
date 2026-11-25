# Command-Line Option Reference

All options of `bbsearch`. Every command is run from the repository root.

> The examples use the Linux/macOS form `./bbsearch`. On Windows the binary is `bbsearch.exe` — write `.\bbsearch.exe` instead.

## Contents

- [Basic forms](#basic-forms)
- [Modes](#modes)
- [Machine options](#machine-options)
- [Search control](#search-control)
- [Start configuration](#start-configuration)
- [Landmark spec overrides](#landmark-spec-overrides)
- [Diagnostic tools](#diagnostic-tools)
- [Quick reference table](#quick-reference-table)

---

## Basic forms

```
bbsearch --core CORE [options...]
bbsearch --all [options...]
bbsearch --cores
```

Running without any option prints the usage message and exits with code 1.

`CORE` can be a core name or the index of the core in the core list (starting from 0). When a name is given that does not exist, the program errors out and lists all valid names.

## Modes

### `--core NAME`

Selects the core to search. Default `fw1_6`.

The 80 core names:

```
fw_4      fw_3      dfw_5     dfw_4     tfw_5     tfw_6
fw1_3     fw1_4     fw1_5     fw1_6
fw1fw_4
fw2_3     fw2_4     fw2_5     fw2_6     fw2_7
fw3_3     fw3_4     fw3_5     fw3_6     fw3_7
fw4_5     fw4_8
fwm2_3    fwm2_5    fwm2_6    fwm2_7
fwm2fw_5
dfwm2_5
fwm21_3   fwm21_4   fwm21_5   fwm21_6   fwm21_7
fwm21_7i  fwm21_8   fwm21fw1_7 fwm21dfw1_7 fwm21tfw1_7
dfwm21_8
fwm22_3   fwm22_8
fwp2_3    fwp2_4    fwp2_5    fwp2_6    fwp21_3
fwp21_4   fwp21_5   fwp21_6   fwp21_7   fwp21fw_4
fwp21fw_6 fwp22_4   fwp22_5   fwp22_7   fwp22fw_4
fwp23_4   fwpw_4    fwpw_5    fwpw_7    fwpw_8
dfwpw_5   fwpw1_4   fwpw1_5a  fwpw1_5b  fwpw1_6
fwpw1_7   fwpw1_8   fwpw1fwpw_4 fwpw2_6
fe0_5_old fe0_5     fe01_5    fe01_6    fe01_7
fe01_soc_7 fe1_6    fe1_7     fe1_8
```

### `--all`

Searches all 80 cores in turn.

- The state count used per core = `max(--states, the core's fixed state count)`
- The symbol count used per core = `--symbols` (if given) or the core's own symbol count
- A core is skipped with a printed reason when the alphabet is smaller than the core's symbol count
- A core is likewise skipped when `--startstate` lies outside its state count

### `--cores`

Prints the core list and exits without searching. The output columns are:

```
core       states   syms  landmark                                             importance
fw_4            3      4  0^inf 3^(2m+1) 2^k A> 1^n 2                          k > n > m
```

`landmark` is the **template** of the landmark (the counters written symbolically), `importance` is that core's counter importance order.

## Machine options

### `--states N`

The total number of machine states, in the range **3..25** (`A`..`Y`).

The default is the core's own fixed state count. It must be ≥ the core's fixed state count (states `A`..`fixed states-1` are hard-wired by the core), otherwise it errors out:

```
--core fw1_6 needs --states >= 3 (A..C are fixed)
```

When it is larger than the core's state count, the extra rows are entirely FREE cells and the DFS fills them in.

### `--symbols N`

The alphabet size, in the range **2..10**. The default is the core's own symbol count.

- **Larger** than the core's symbol count: the core only describes symbols `0..coreSym-1`, the extra columns stay FREE and are filled by the DFS — the search then ranges over "N-symbol machines that contain the core"
- **Smaller** than the core's symbol count: the core's extra columns are dropped and their transitions can never fire — which may make the landmark unreachable

### `--startstate A..Y`

The state the machine starts in.

- `--start blank` mode: default `A`
- `--start cfg` mode: the default is the lowest head state allowed by the landmark (`firstHeadState()`, usually `A`; `fwm2_3` / `fwm21_3` / `fwm21_4` use `B`, `fe0_5` also uses `B`, `fwp2_5` uses `C`, the other `KIND_THREE` cores use `A`)

`dfw_4` and `tfw_5` halt within a few steps when started in state `A` and need a different state.

## Search control

### `--cap N`

The TM step limit for a single run (one DFS path). Default `50000`.

Exceeding it returns to the previous level. Too small a value cuts off the deep paths that produce the landmark.

### `--budget N`

The total step budget for the run. Default `0` = unlimited.

The program checks the budget every `2^24` steps and stops with `gAbort = true` when it is exceeded. `abort=1` in the output means **the budget ran out**; `abort=0` means the run finished.

**Important**: `abort=1` together with "no landmark configuration was reached" only means the budget was too small; it does not imply that the core does not exist.

### `--fill K`

At most K FREE cells may be filled (default `0` = unlimited).

After K cells have been filled the next one stays undefined — semantically equivalent to halting there. Use it to limit the branching explosion.

### `--log FILE`

Appends the output to a file (by default the output goes to stdout only). The file is created if it does not exist.

```bash
./bbsearch --all --cap 200000 --budget 3e9 --log all.log
```

## Start configuration

### `--start blank|cfg`

| Value | Meaning |
|---|---|
| `blank` (default) | Blank tape `0^inf A> 0^inf`, head in the middle cell, state `A` |
| `cfg` | Start from the core's documented **input configuration** |

`cfg` mode exists for cores **without a construction phase** — their blank-tape run dies immediately (`dfw_4` halts within 3 steps).

In `cfg` mode only the **left half** of the landmark is validated (loose mode), because the machine transforms the input configuration into a larger configuration whose right half necessarily differs — the interesting thing is the configuration after the transformation.

### `--k --n --m --p --q --r --r5`

The counter values of the input configuration in `cfg` mode.

| Option | Slot | Default | Meaning |
|---|---|---|---|
| `--k N` | `k` | 16 | Length of the `2^k` run left of the head |
| `--n N` | `n` | 3 | Length of the `1^n` run right of the head |
| `--m N` | `m` | 0 | `(length-1)/2` of the `3^(2m+1)` run on the left |
| `--p N` | `p` | 1 | Length of the right-hand `1^p` segment |
| `--q N` | `q` | 1 | Length of the right-hand `0^q` segment |
| `--r N` | `r` | 1 | Length of the right-hand `1^r` segment |
| `--r5 N` | `r5` | 1 | Length of the left-hand `5^r` run |

The meaning of the slots of `KIND_ONES` cores is decided by each core's own `cfg` / `slot` lines:

- `fwm2_3` / `fwm21_3` / `fwm21_4`: the zero segment is fixed at one cell and is not controlled by `--n`; `--k` / `--n` correspond to the right-hand `2^k` and `1^n` segments, and `--m` is the length of the `1^m` run left of the head (minimum 1)
- `fw_3`: `--n` is the length of the `1^n` run left of the head, minimum 1
- `fe0_5`: `--n` is the length of the `2^n` run under the head, minimum 3; `--m` is the length of the `1^m` run left of the head, minimum 1

For the `KIND_THREE` cores (`fwp2_4`, `fwp2_5`, `fwp21_4`, `fwp21_5`, `fwp22_4`, `fwp22_5`, `fwp23_4`), `--n` is the length of the `3^n` run under the head (minimum 3 for `fwp2_4` / `fwp2_5`; for the multi-segment variants `--n` is the length of the second segment, `0^n` in `fwp21_4` / `fwp21_5` / `fwp22_4` / `fwp23_4` and `4^n` in `fwp22_5`).

Example:

```bash
./bbsearch --core fwm2_3 --states 8 --start cfg --m 3 --k 2 --n 4
```

## Landmark spec overrides

These four options override at run time the spec parsed from the current core's `cfg` / `slot` lines, so you can **try another landmark guess without recompiling**.

### `--left5 none|one|many`

The form of the left-hand `5` run:

| Value | Meaning |
|---|---|
| `none` | No `5` between the 3-run and the blank region |
| `one` | Exactly one `5` |
| `many` | `5^r` (`r >= 1`), with `r` taken from `--r5` |

### `--right SYM:MIN:MAX[,SYM:MIN:MAX...]`

The list of right-hand segments. Each segment is `symbol:min cells:max cells`, where `-1` means the maximum is unbounded.

```bash
./bbsearch --core fw3_5 --right "4:1:-1,3:1:-1,1:1:-1"
```

Meaning: on the right there is first a `4` segment (at least 1 cell, unbounded), then a `3` segment (at least 1 cell, unbounded), then a `1` segment (at least 1 cell, unbounded).

### `--slots P,R5,K,N,M`

The importance order of the counter slots (from highest to lowest). Available names: `k` `n` `m` `p` `q` `r` `r5` `s`.

```bash
./bbsearch --core fw3_5 --slots R,Q,P,K,N,M
```

### `--noblank`

Do not require a blank cell after the last right-hand segment (i.e. `termBlank = false`).

## Diagnostic tools

### `--census`

Does not match the landmark; instead it counts the distribution of **actually observed** right-hand shapes. This is the exploration tool for when "what the landmark looks like" is still unknown.

### `--censusL`

Same as `--census`, but keeps only the configurations whose left half already matches `0^inf 5^r 3^(2m+1) 2^k`, and prints the shape together with `r`.

### `--fix CELL=WRITE-MOVE-NEXT[,...]`

Pins certain transition cells so the DFS no longer branches there.

- `CELL` = state letter + symbol digit, e.g. `A0`, `B3`
- `WRITE-MOVE-NEXT` = 3 characters, e.g. `4LC` (write 4, move left, enter state C)
- `---` pins the cell to "halt before writing"

```bash
./bbsearch --core fw1_6 --fix "A0=4LC,B0=1RZ"
```

It errors out when it conflicts with the core's existing definition (`fix '...' conflicts with the fixed core`). Referring to a state beyond `--states` also errors out.

## Quick reference table

| Option | Default | Range | Description |
|---|---|---|---|
| `--core NAME` | `fw1_6` | One of the 80 core names, or an index | Selects the core |
| `--all` | — | — | Searches all cores |
| `--cores` | — | — | Lists the cores and exits |
| `--cores-file PATH` | `bbcores_new.txt` | Path | The file the core table and landmark specs are read from |
| `--states N` | The core's fixed state count | 3..25 | Number of machine states |
| `--symbols N` | The core's own symbol count | 2..10 | Alphabet size |
| `--cap N` | `50000` | ≥0 | Step limit for a single run |
| `--budget N` | `0` | ≥0 | Total step budget, 0 = unlimited |
| `--fill K` | `0` | ≥0 | At most how many FREE cells to fill, 0 = unlimited |
| `--log FILE` | — | Path | Appends the output to a file |
| `--start blank\|cfg` | `blank` | One of the two | Start mode |
| `--startstate A..Y` | Depends on the mode | A..Y | Start state |
| `--k N` | `16` | ≥0 | Counter k |
| `--n N` | `3` | ≥0 | Counter n |
| `--m N` | `0` | ≥0 | Counter m |
| `--p N` | `1` | ≥0 | Counter p |
| `--q N` | `1` | ≥0 | Counter q |
| `--r N` | `1` | ≥0 | Counter r |
| `--r5 N` | `1` | ≥0 | Counter r5 (the left-hand `5^r`) |
| `--left5 none\|one\|many` | From the list | One of the three | Form of the left-hand 5-run |
| `--right SEG[,SEG...]` | From the list | `SYM:MIN:MAX` | List of right-hand segments |
| `--slots LIST` | From the list | Comma-separated slot names | Importance order |
| `--noblank` | Off | — | Do not require a blank after the last segment |
| `--census` | Off | — | Count the observed shapes |
| `--censusL` | Off | — | Same, but keeps only the ones whose left half matches |
| `--fix SPEC` | — | `CELL=WMN` | Pin transition cells |
