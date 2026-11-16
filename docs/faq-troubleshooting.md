# FAQ and Troubleshooting

## Contents

- [The search found no landmark](#the-search-found-no-landmark)
- [The core halts immediately](#the-core-halts-immediately)
- [Option and validation errors](#option-and-validation-errors)
- [The result looks wrong](#the-result-looks-wrong)
- [Performance and runtime](#performance-and-runtime)
- [Build problems](#build-problems)
- [Concept clarifications](#concept-clarifications)

---

## The search found no landmark

### Symptom

```
---- fw2_6 done: steps=16777216 nodes=156 abort=1 ----
no landmark configuration was reached
```

### Cause and handling

**`abort=1` means the budget ran out, not that the core cannot produce this landmark.** Landmarks often appear deep in the DFS tree, and with too small a budget they are never reached.

Try these in order:

1. **Increase the budget**: `--budget 3e9` (a practical magnitude)
2. **Increase the per-run limit**: `--cap 200000`
3. **Increase the state count**: make `--states` 1–2 larger than the core's fixed state count; the extra rows are FREE cells and give the DFS more branching room
4. **Change the start mode**: `--start cfg` (see below)
5. **Change the start state**: `--startstate B`
6. **Loosen the landmark guess**: `--right` / `--slots` / `--noblank` / `--left5`

### `abort=0` can also mean "not found"

```
---- dfw_4 done: steps=3 nodes=0 abort=0 ----
no landmark configuration was reached
```

`abort=0` with very few steps means the machine halted almost immediately and never really ran. This is not "not found", it is "never started". See [The core halts immediately](#the-core-halts-immediately).

## The core halts immediately

### Symptom

The run ends after a handful of steps or a few dozen, with `nodes=0`.

### Cause

`dfw_4` and `tfw_5` halt within a few steps when started in state `A` — their A row leads straight into the halt.

### Handling

```bash
# Change the start state
./bbsearch --core dfw_4 --startstate B --states 4

# Or start from the documented input configuration (closer to real usage)
./bbsearch --core dfw_4 --start cfg --states 4
```

In `--start cfg` mode only the left half of the landmark is validated (loose), because the machine transforms the input configuration into a larger configuration whose right half necessarily differs.

## Option and validation errors

| Message | Cause | Handling |
|---|---|---|
| `bad --symbols N (want 2..10)` | The alphabet size is out of range | Use 2..10 |
| `bad --states N (want 3..25)` | The state count is out of range | Use 3..25 |
| `the landmark needs symbols 1, 2 and 3 but --symbols N has only symbols 0..M` | A `KIND_BCFG` landmark needs at least 4 symbols | Increase `--symbols` |
| `the landmark needs symbol 5 (the 5-run left of the 3-run) but --symbols N has only symbols 0..M` | The landmark contains a `5^r` segment, so the alphabet needs at least 6 symbols | Increase `--symbols` |
| `the landmark's right side needs symbol X but --symbols N has only symbols 0..M` | A right-hand segment uses a symbol that is not in the alphabet | Increase `--symbols` |
| `A0 writes symbol 4 but --symbols 3 has only symbols 0..2` | The core's own transition writes a symbol outside the alphabet | Increase `--symbols` to at least the core's symbol count |
| `bad --core 'xxx' (use one of: ...)` | The `--core` name does not exist | Use a name from `--cores`, or an index |
| `--core fw1_6 needs --states >= 3 (A..C are fixed)` | The state count is smaller than the core's fixed state count | Increase `--states` |
| `skipped (--symbols N < M symbols of the core)` | The alphabet is smaller than the core's | Drop `--symbols` or increase it |
| `skipped (--startstate X is outside the machine's N states)` | The start state is beyond the state count | Increase `--states` or pick another state |
| `fix '...' conflicts with the fixed core` | The `--fix` conflicts with a cell the core already defines | Drop that fix or change its value |
| `cell 'A9' uses a state outside --states` | The `--fix` refers to a state out of range | Increase `--states` |
| `unknown option xxx` | The option name is misspelled | See [cli-reference.md](cli-reference.md) |

## The result looks wrong

### The configuration found looks "too small"

Normal. The search reports the **maximum found within the budget**, not the mathematical maximum. Increase the budget.

### The printed configuration does not match the `--cores` template

`--cores` prints the **template** (the counters written symbolically as `k`, `n`, `m`), while the actual output contains **concrete values** (`2^16`, `1^3`). Substitute the numbers for the symbols when comparing.

### Different cores give the same configuration

Normal. Several cores share the same family of landmark structure (e.g. `fw_4` / `dfw_5` / `tfw_6` all use `0^inf 3^(2m+1) 2^k A> 1^n 2`), and in the same machine they may all hold.

### `--cores` prints only the header row

The core list was not read: the program reads the cores from `bbcores_new.txt` at start-up (`--cores-file` changes the path) and exits with an error if it cannot read the file; blocks in the list that are still in the old format (bare transition rows, no `name` line) are ignored.

### The landmark of `--core fw3_4` is not what you expected

`fw3_4` is the 7-state / 4-symbol version (landmark `1^m F> 0^n 1^k 2^p 3`). The old 8-state version (once named `fw3_4_old`, landmark `0^inf 3^(2m+1) 2^k A|E> 1^n 2 1^p 0^q 1^r 2`) has been removed from `bbcores_new.txt` and is no longer built into the program.

### `--start cfg` has no effect for `fwm2_3`

`fwm2_3` is a `KIND_ONES` core: the head state is `B`, the head cell is **exactly one** blank cell (not a counter segment), and the counters `k` and `n` are both on the right. `--start cfg` builds the input configuration according to that structure:

```bash
./bbsearch --core fwm2_3 --states 8 --start cfg --m 3 --k 2 --n 4
# the configuration built: 1^3 B> 0 2^2 1^4 0
```

## Performance and runtime

### The search is too slow

| Means | Effect |
|---|---|
| Increase `--budget` | Coverage ↑, time ↑ |
| Increase `--cap` | Deeper single paths, time ↑ |
| **Decrease `--cap`** | Cuts deep paths, **may actually produce results faster** |
| `--fill K` | Limits the total number of FREE cells filled, branching ↓ |
| Decrease `--states` | Fastest when close to the core's fixed state count |
| Decrease `--symbols` | Must not go below the core's symbol count |

### Progress output

The program prints progress to stderr every `2^24` steps:

```
  ... steps=16777216 nodes=145
```

`nodes` is the number of DFS branch nodes, `steps` is the number of TM steps.

### Running in the background

```bash
./bbsearch --all --cap 200000 --budget 3e9 --log all.log
```

To terminate: `Stop-Process -Id <pid> -Force` (Windows) or `kill <pid>` (Linux/macOS).

## Build problems

### `g++ is not recognized as an internal or external command` / `g++: command not found`

`g++` is not on `PATH`. Install a C++11 compiler and make sure it is reachable:

| Platform | How to get `g++` |
|---|---|
| Debian / Ubuntu | `sudo apt install g++` |
| Fedora / RHEL | `sudo dnf install gcc-c++` |
| macOS | `xcode-select --install` (Xcode command line tools) |
| Windows | Install MinGW-w64 or MSYS2 and add its `bin` directory to `PATH` |

### `cc1plus.exe: fatal error: ... No such file or directory`

The compiler installation is incomplete or has been moved. Reinstall the compiler package (for example the MinGW-w64 / MSYS2 toolchain), or switch to another GCC.

### The build works but the binary is not found afterwards

On Linux/macOS the output is `bbsearch` — run it as `./bbsearch`. On Windows the output is `bbsearch.exe` — run it as `.\bbsearch.exe`.

### `illegal instruction`

The binary built with `-march=native` fails on another machine. Rebuild without `-march=native`.

### The build reports C++14/17-related errors

The code is only guaranteed to be C++11 compatible. Check for accidental use of `std::make_unique`, structured bindings and similar features.

## Concept clarifications

### What is a "core"?

The fixed part of the transition table of a Busy Beaver champion machine. Free cells are written as `---` and are filled in by the DFS during the search.

### What is a "landmark"?

A special configuration shape. The landmark of a core is described in `bbcores_new.txt`, for example:

```
0^inf 3^(2m+1) 2^k A> 1^n 2
```

Meaning: an infinite blank region on the left, then `2m+1` cells of `3`, then `k` cells of `2`, the head in state `A` at the end of the `2^k` run, then `n` cells of `1` to the right, and one more `2` beyond that.

### Why do cores have three landmark kinds

Different families of champion machines write their "construction phase" in different places:

- `KIND_BCFG`: `0^inf 3^(2m+1) 2^k <head> 1^n <right>` — the classic shape
- `KIND_THREE`: `0^inf <head> 3^n 0^inf` — the head lands on the first cell of the `3^n` run and everything else is blank; the counter segment is right under the head (`fwp21_4` / `fwp21_5` are its multi-segment variants: `0^inf <head> 3^m 0^n <sentinel>`)
- `KIND_ONES`: `0^inf 1^m <head> <right config>` — there is a run of `1^m` left of the head and the head cell is the **first cell** of the right-hand first segment; the head segment can be blank (`0^n` in `fw1_3a`, the lone `0` in `fwm2_3`), a `1` (`fw_3`) or a `2` (`2^n` in `fe0_5`)

### How is the importance order compared?

**Lexicographically**, not by numeric magnitude. The value of `slot[0]` is compared first, then `slot[1]` if they are equal, and so on.

Example: `fw3_4`'s `p > k > n > m` means that `(p,k,n,m) = (2,1,5,0)` beats `(1,9,9,9)` — `p` already decides it, so the rest is not compared.

### The loose mode of `--start cfg`

In `cfg` mode only the **left half** of the landmark is validated. The whole point of the machine is to transform the input configuration into a larger one, so the right half necessarily differs from the input — what is interesting is the configuration after the transformation.

### What can this tool prove?

**It cannot prove any mathematical conclusion.** It is a simulation search: it reports "the maximum landmark configuration found on this search path within the given budget". Drawing conclusions requires exhausting the search space or an independent argument.
