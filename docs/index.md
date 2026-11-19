# bbsearch Documentation Index

`bbsearch` — a Busy Beaver core landmark searcher. A single-file C++11 command-line tool.

## Documentation map

| Document | Audience | Contents |
|---|---|---|
| [README.md](../README.md) | Human users | Project intro, build, 5-minute quick start |
| [AGENTS.md](../AGENTS.md) | AI collaborators | Code of conduct, hard constraints, standard workflow for changing cores |
| [architecture.md](architecture.md) | People changing the code | Internal structure, algorithms, the three landmark kinds |
| [getting-started.md](getting-started.md) | Developers | Environment setup, build, verification workflow |
| [cli-reference.md](cli-reference.md) | Users | Every command-line option explained |
| [cores.md](cores.md) | Users / researchers | Landmarks, importance and measured reachability of the 56 cores |
| [faq-troubleshooting.md](faq-troubleshooting.md) | Anyone stuck | Common blockers and troubleshooting |

## The project in one sentence

Input: a **core** (a fixed fragment of the transition table) of a Busy Beaver champion machine, and a **landmark** (a special configuration shape).
Output: the **maximum landmark configuration** that core can produce, compared by that core's importance order, within the search budget.

## The three-minute route

```bash
cd /path/to/bbsearch
g++ -O3 -march=native -std=c++11 -o bbsearch bbsearch.cpp
./bbsearch --cores
./bbsearch --core fw3_3 --states 9 --cap 50000 --budget 3e9
```

> Linux/macOS form. On Windows the binary is `bbsearch.exe` — use `.\bbsearch.exe` instead of `./bbsearch`.

## When you want to change a core

Read the "Key design principles and constraints" and "Common pitfalls and known issues" sections of [AGENTS.md](../AGENTS.md), then follow the verification workflow in [getting-started.md](getting-started.md). **The authoritative data source is [bbcores_new.txt](../bbcores_new.txt)** — the program reads it at run time (`name` / `cfg` / `slot` / `core` four lines per block); the code contains no built-in core table.
