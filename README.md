# bbsearch

A Busy Beaver **core** landmark searcher — a depth-first search over a given Turing machine core (core), looking for the **maximum landmark configuration** that core can produce.

This is a **single-file C++ command-line tool**. Core data comes from loader3229's Busy Beaver core list ([bbcores_new.txt](bbcores_new.txt)); the program reads the cores from that list at run time (currently 80), and can search them one at a time or all at once.

## Features

- **80 cores**: covering the `f_ω`, `f_(ω+1)` … `f_(ω*2+2)`, `f_(ω^2)`, `f_(ω^2+1)`, `f_(ω^2+2)`, `f_(ω^2+3)`, `f_(ω^ω)`, `f_(ω^ω+1)`, `f_(ω^ω+2)`, `f_(ε0)`, `f_(ε0+1)`, `f_(ε1)` families, with 3–21 states and 3–8 symbols
- **Core table comes from the list file**: the program reads [bbcores_new.txt](bbcores_new.txt) at run time (each core is a `name` / `cfg` / `slot` / `core` four-line block); `--cores-file` changes the path; there is no built-in copy in the code
- **New cores can be added by hand**: discovering a Turing machine core that is not in the list yet is enough — append its four-line block to `bbcores_new.txt` and `bbsearch` searches that core's landmark configuration on the next run, with no recompilation
- **Three landmark structures**: the tool recognises `0^inf 3^(2m+1) 2^k <head> 1^n <right>` (`KIND_BCFG`), `0^inf <head> 3^n 0^inf` (`KIND_THREE`) and `0^inf 1^m <head> <right config>` (`KIND_ONES`, the head cell is the first cell of the right-hand first segment) automatically
- **Per-core importance order**: every core has its own counter precedence order (compared lexicographically), e.g. `fw3_4` is `p > k > n > m`
- **DFS + Zobrist hash cycle detection**: expands every machine containing that core over a blank tape, branching on FREE cells to fill the table
- **Two start modes**: `--start blank` (default, blank tape `0^inf A> 0^inf`) and `--start cfg` (start from the documented input configuration, for cores without a construction phase)
- **Landmark spec overridable at run time**: `--left5 / --right / --slots / --noblank` let you try another landmark guess without recompiling

> This is a **simulation search, not a proof**. The tool reports "the maximum landmark configuration found within the given search budget"; it does not establish a mathematical conclusion.

## Quick start

> **Platform note**: the commands below are written in the Linux/macOS form (`./bbsearch`). On Windows the compiler produces `bbsearch.exe`, so use `.\bbsearch.exe` (or `./bbsearch.exe`) in PowerShell. The build command itself is identical on both platforms.

### Build

Requires any C++11 compiler (`g++` is used below); there are no third-party dependencies.

```bash
cd /path/to/bbsearch
g++ -O3 -march=native -std=c++11 -o bbsearch bbsearch.cpp
```

`-march=native` is an optimisation flag and can be dropped on machines where it is unsupported.

### List all cores

```bash
cd /path/to/bbsearch
./bbsearch --cores
```

Each output line lists, in order: core name, number of states, number of symbols, landmark template, counter importance order.

### Search a single core

```bash
cd /path/to/bbsearch
./bbsearch --core fw1_6 --states 3 --cap 50000 --budget 3e9
```

### Search all cores

```bash
cd /path/to/bbsearch
./bbsearch --all --cap 200000 --budget 3e9 --log all.log
```

### Start from the input configuration for cores without a construction phase

```bash
cd /path/to/bbsearch
./bbsearch --core dfw_4 --start cfg --states 4 --startstate B --k 16 --n 3
```

## Configuration

The tool has **no configuration file**; everything is controlled by command-line options. The full option reference is in [docs/cli-reference.md](docs/cli-reference.md).

The most commonly used options:

| Option | Default | Meaning |
|---|---|---|
| `--core NAME` | `fw1_6` | Select a core by name, or by index |
| `--states N` | The core's own fixed state count | Total machine states (3–25) |
| `--symbols N` | The core's own symbol count | Alphabet size (2–10); extra columns beyond the core's symbols are filled by the DFS |
| `--cap N` | `50000` | Step limit for a single run |
| `--budget N` | `0` (unlimited) | Total step budget for this run |
| `--start blank\|cfg` | `blank` | Start mode |
| `--startstate A..Y` | Depends on the mode | The state the machine starts in |
| `--k --n --m --p --q --r --r5` | `16 3 0 1 1 1 1` | Counter values for the input configuration in `cfg` mode |

## Adding a new core

The core table is **data, not code**: if you come across a Turing machine core that the list does not contain yet, you can add it yourself. No recompilation is needed — `bbsearch` reads the list at run time and picks the new core up immediately.

A core is a four-line block in [bbcores_new.txt](bbcores_new.txt):

```
name fwp22_5
cfg  ACG> 3^m 4^n 0^k 4^1
slot k>=1 n>=2 m>=1
core  <one transition row per state, separated by '_'>
```

| Line | Meaning |
|---|---|
| `name` | The core's name, as passed to `--core` |
| `cfg` | The landmark shape; the letters before `>` are the states the head may be in (`ACG>` = `A`, `C` or `G`) |
| `slot` | The counters in importance order (left = most important), with optional minima such as `n>=2` |
| `core` | The transition rows, one per state, separated by `_`; every row is exactly `3 × nsym` characters (write, move `L`/`R`, next state) and `---` marks a FREE cell |

Afterwards run `./bbsearch --cores` and check that the row count grew by one and that the landmark template and the importance column are correct, then search the new core:

```bash
./bbsearch --core <the-new-core> --cap 20000 --budget 2e7
```

A landmark that lies deep in the construction phase needs a larger `--budget` / `--cap`; `no landmark configuration was reached` only means the search stopped too early, not that the core cannot produce the shape. How to write each of the three landmark shapes, and the self-check script for the transition rows, are described in [docs/getting-started.md](docs/getting-started.md).

## Documentation

- [docs/index.md](docs/index.md) — documentation index
- [docs/architecture.md](docs/architecture.md) — internal structure and algorithms
- [docs/getting-started.md](docs/getting-started.md) — local development guide
- [docs/cli-reference.md](docs/cli-reference.md) — command-line option reference
- [docs/cores.md](docs/cores.md) — landmarks and measured reachability of the 80 cores
- [docs/faq-troubleshooting.md](docs/faq-troubleshooting.md) — common problems and troubleshooting

The rulebook for AI collaborators is in [AGENTS.md](AGENTS.md).

## Contributing

This project has no automated tests. After changing the core table or a landmark spec, run at least:

```bash
./bbsearch --cores
```

(on Windows: `.\bbsearch.exe --cores`)

and confirm that the number of output lines matches the number of cores and that the landmark templates and importance orders are correct; then run an actual search on the core you changed.

## License

Released under the GNU General Public License v3.0 — see [LICENSE.md](LICENSE.md).

## Acknowledgements

The core data and landmark descriptions come from loader3229's Busy Beaver core list (`bbcores_new.txt`), whose core table is based on work such as Pavel Kropitz's `f_ω` core and Racheline's `double f_ω` core. For mathematical background, see Michel's analysis of "cores in Busy Beaver champion machines".
