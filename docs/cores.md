# Core List and Measured Reachability

This document lists all 56 cores the program supports: **landmark shape, counter importance order, head state, and measured reachability under the default search budget**. The core definitions come from [bbcores_new.txt](../bbcores_new.txt) (read at run time; the code contains no built-in copy).

The "Landmark" column is the normalised spelling of the txt's `cfg` line: `KIND_ONES` omits the leading `0^inf`, and trailing blanks are written uniformly as `0` (the program prints `0^inf`). When the cfg does not end with `BLANK`, the last fixed segment without a slot is interpreted by the **last-segment rule** as "≥1 cell" rather than "exactly one cell" (for example the trailing `1` of `1^m F> 0^n 1`).

## Contents

- [Core table](#core-table)
- [Grouped by family](#grouped-by-family)
- [Measured reachability (2026-11-19)](#measured-reachability-2026-11-19)
- [Cores with no landmark found](#cores-with-no-landmark-found)
- [Special core notes](#special-core-notes)

---

## Core table

`states` = number of fixed states (A..); `syms` = number of symbols covered by the core.

| # | Core | states | syms | Landmark | importance |
|---|---|---|---|---|---|
| 0 | `fw_4` | 3 | 4 | `0^inf 3^(2m+1) 2^k A> 1^n 2` | k > n > m |
| 1 | `fw_3` | 5 | 3 | `1^n A> 1 0` | n |
| 2 | `dfw_5` | 3 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 2` | k > n > m |
| 3 | `dfw_4` | 4 | 4 | `0^inf 3^(2m+1) 2^k A> 1^n 0` | k > n > m |
| 4 | `tfw_5` | 4 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 0` | k > n > m |
| 5 | `tfw_6` | 3 | 6 | `0^inf 3^(2m+1) 2^k A> 1^n 2` | k > n > m |
| 6 | `fw1_3` | 7 | 3 | `1^m F> 1^n 2` | n > m |
| 7 | `fw1_4` | 4 | 4 | `0^inf 3^(2m+1) 2^k B> 1^n 2^p 0` | p > k > n > m |
| 8 | `fw1_5` | 5 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 4^p 0` | p > k > n > m |
| 9 | `fw1_6` | 3 | 6 | `0^inf 3^(2m+1) 2^k A> 1^n 4 5^p 0` | p > k > n > m |
| 10 | `fw1fw_4` | 5 | 4 | `0^inf 3^(2m+1) 2^k A> 1^n 0` | k > n > m |
| 11 | `fw2_3` | 7 | 3 | `1^m F> 0^n 1^k 2` | k > n > m |
| 12 | `fw2_4` | 6 | 4 | `0^inf 3^(2m+1) 2^k B> 1^n 2^p 0^q 1` | q > p > k > n > m |
| 13 | `fw2_5` | 5 | 5 | `0^inf 3^(2m+1) 2^k B> 1^n 2^p 4^q 0` | q > p > k > n > m |
| 14 | `fw2_6` | 4 | 6 | `0^inf 3^(2m+1) 2^k A> 1^n 4^p 5^q 0` | q > p > k > n > m |
| 15 | `fw2_7` | 3 | 7 | `0^inf 3^(2m+1) 2^k A> 1^n 4 5^p 6^q 0` | q > p > k > n > m |
| 16 | `fw3_3` | 9 | 3 | `1^m F> 0^n 1^k 2^p 0` | p > k > n > m |
| 17 | `fw3_4` | 7 | 4 | `1^m F> 0^n 1^k 2^p 3` | p > k > n > m |
| 18 | `fw3_5` | 6 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 4^p 3^q 1^r 0` | r > q > p > k > n > m |
| 19 | `fw3_6` | 5 | 6 | `0^inf 3^(2m+1) 2^k A> 1^n 4^p 5^q 2^r 0` | r > q > p > k > n > m |
| 20 | `fw3_7` | 4 | 7 | `0^inf 3^(2m+1) 2^k A> 1^n 4^p 5^q 6^r 0` | r > q > p > k > n > m |
| 21 | `fw4_5` | 7 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 4^p 3^q 1^r 2^s 0` | s > r > q > p > k > n > m |
| 22 | `fw4_8` | 4 | 8 | `0^inf 3^(2m+1) 2^k A> 1^n 4^p 5^q 6^r 7^s 0` | s > r > q > p > k > n > m |
| 23 | `fwm2_3` | 8 | 3 | `1^m B> 0 2^k 1^n 0` | k > n > m |
| 24 | `fwm2_5` | 6 | 5 | `0^inf 3^(2m+1) 2^k D> 1^n 0` | k > n > m |
| 25 | `fwm2_6` | 6 | 6 | `0^inf 5 3^(2m+1) 2^k A> 1^n 2` | k > n > m |
| 26 | `fwm2_7` | 5 | 7 | `0^inf 3^(2m+1) 2^k A> 1^n 2` | k > n > m |
| 27 | `fwm2fw_5` | 6 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 0` | k > n > m |
| 28 | `dfwm2_5` | 8 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 2` | k > n > m |
| 29 | `fwm21_3` | 10 | 3 | `1^m B> 0 2^k 1^n 0 1^p 2` | p > k > n > m |
| 30 | `fwm21_4` | 10 | 4 | `1^m B> 0 2^k 1^n 0^p 3` | p > k > n > m |
| 31 | `fwm21_5` | 8 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 4^p 0` | p > k > n > m |
| 32 | `fwm21_6` | 7 | 6 | `0^inf 5 3^(2m+1) 2^k A> 1^n 2^p 5` | p > k > n > m |
| 33 | `fwm21_7` | 6 | 7 | `0^inf 5 3^(2m+1) 2^k A> 1^n 2^p 5 0` | p > k > n > m |
| 34 | `fwm22_3` | 12 | 3 | `1^m B> 0 2^k 1^n 0 1^p 2^q 0` | q > p > k > n > m |
| 35 | `fwm22_8` | 6 | 8 | `0^inf 5 3^(2m+1) 2^k A> 1^n 2^p 7^q 5 0` | q > p > k > n > m |
| 36 | `fwp2_4` | 8 | 4 | `0^inf A\|C\|G> 3^n 0` | n |
| 37 | `fwp2_5` | 8 | 5 | `0^inf C\|G> 3^n 0` | n |
| 38 | `fwp2_6` | 7 | 6 | `0^inf 5^r 3^(2m+1) 2^k A> 1^n 2` | r5 > k > n > m |
| 39 | `fwp21_4` | 10 | 4 | `0^inf A\|C\|G> 3^m 0^n 1 0` | n > m |
| 40 | `fwp21_5` | 9 | 5 | `0^inf A\|C\|G> 3^m 0^n 4 0` | n > m |
| 41 | `fwp21_6` | 8 | 6 | `0^inf 5^r 3^(2m+1) 2^k A> 1^n 2^p 5` | p > r5 > k > n > m |
| 42 | `fwp21_7` | 7 | 7 | `0^inf 5^r 3^(2m+1) 2^k A> 1^n 2^p 5 0` | p > r5 > k > n > m |
| 43 | `fwp21fw_4` | 11 | 4 | `1^n G> 1^m 0` | n > m |
| 44 | `fwp21fw_6` | 8 | 6 | `0^inf 3^(2m+1) 2^k A> 1^n 4 0` | k > n > m |
| 45 | `fwp22_4` | 11 | 4 | `0^inf A\|C\|G> 3^m 0^n 1^k 2 0` | k > n > m |
| 46 | `fwp22_5` | 10 | 5 | `0^inf A\|C\|G> 3^m 4^n 0^k 4 0` | k > n > m |
| 47 | `fwp22_7` | 8 | 7 | `0^inf 5^r 3^(2m+1) 2^k A> 1^n 2^p 5^q 6` | q > p > r5 > k > n > m |
| 48 | `fwp22fw_4` | 12 | 4 | `1^n G> 1^m 0` | n > m |
| 49 | `fwp23_4` | 12 | 4 | `0^inf A\|C\|G> 3^m 0^n 1^k 2^p 3 0` | p > k > n > m |
| 50 | `fwpw_5` | 12 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 4 2` | k > n > m |
| 51 | `dfwpw_5` | 12 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 4 3 0` | k > n > m |
| 52 | `fwpw1_5a` | 12 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 4 2^p 0` | p > k > n > m |
| 53 | `fwpw1_5b` | 12 | 5 | `0^inf 3^(2m+1) 2^k A> 1^n 4 0^p 2` | p > k > n > m |
| 54 | `fwpw1_6` | 12 | 6 | `0^inf 3^(2m+1) 2^k A> 1^n 4 2^p 5` | p > k > n > m |
| 55 | `fe0_5` | 17 | 5 | `1^m B> 2^n 0` | n > m |

> The order of the table matches `bbcores_new.txt` exactly — that file is the only source of core definitions.

## Grouped by family

| Family | Cores | Shared feature |
|---|---|---|
| `f_ω` | `fw_4`, `fw_3` | The original cores (Pavel Kropitz / Jacobzheng) |
| `double f_ω` | `dfw_5`, `dfw_4` | Racheline's double f_ω |
| `triple f_ω` | `tfw_5`, `tfw_6` | Triple f_ω |
| `f_(ω+1)` | `fw1_3`, `fw1_4`, `fw1_5`, `fw1_6` | The head is in state `F`, `A` or `B` (`fw1_4`'s head is in `B`) |
| `f_(ω+1)+f_ω` | `fw1fw_4` | Landmark `0^inf 3^(2m+1) 2^k A> 1^n 0`, the right side is a blank cell |
| `f_(ω+2)` | `fw2_3`, `fw2_4`, `fw2_5`, `fw2_6`, `fw2_7` | — |
| `f_(ω+3)` | `fw3_3`, `fw3_4`, `fw3_5`, `fw3_6`, `fw3_7` | — |
| `f_(ω+4)` | `fw4_5`, `fw4_8` | `fw4_5` has 7 counters, `fw4_8` has 8 symbols on the right side |
| `f_(ω*2)` | `fwm2_3`, `fwm2_5`, `fwm2_6`, `fwm2_7` | — |
| `f_(ω*2)+f_ω` | `fwm2fw_5` | Landmark `0^inf 3^(2m+1) 2^k A> 1^n 0`, the right side is a blank cell |
| `double f_(ω*2)` | `dfwm2_5` | — |
| `f_(ω*2+1)` | `fwm21_3`, `fwm21_4`, `fwm21_5`, `fwm21_6`, `fwm21_7` | — |
| `f_(ω*2+2)` | `fwm22_3`, `fwm22_8` | `fwm22_3` is a `KIND_ONES` core (`1^m B> 0 2^k 1^n 0 1^p 2^q 0`), `fwm22_8` a `KIND_BCFG` one |
| `f_(ω^2)` | `fwp2_4`, `fwp2_5` | The counter segment is the `3^n` run itself; `fwp2_4`'s head may be in `A`/`C`/`G`, `fwp2_5`'s only in `C`/`G` |
| `f_(ω^2+1)` | `fwp21_4`, `fwp21_5`, `fwp21_6`, `fwp21_7` | `fwp21_4` / `fwp21_5` have the landmark `0^inf A\|C\|G> 3^m 0^n <sentinel>` with two counters, `fwp21_6` / `fwp21_7` the `f_(ω^2+1)` shape with a `5^r` prefix |
| `f_(ω^2+1)+f_ω` | `fwp21fw_4`, `fwp21fw_6` | `fwp21fw_4`'s landmark is `1^n G> 1^m 0` (head in state `G`); `fwp21fw_6` has the `f_(ω^2+1)` shape |
| `f_(ω^2+2)` | `fwp22_4`, `fwp22_5`, `fwp22_7` | `fwp22_4` is `0^inf A\|C\|G> 3^m 0^n 1^k 2` (three counters); `fwp22_5` is `0^inf A\|C\|G> 3^m 4^n 0^k 4` (the `4^n` run replaces the `0^n` run); `fwp22_7` is the `5^r`-prefixed variant |
| `f_(ω^2+2)+f_ω` | `fwp22fw_4` | Landmark `1^n G> 1^m 0`, head in state `G`, one extra blank cell after `1^m` |
| `f_(ω^2+3)` | `fwp23_4` | Landmark `0^inf A\|C\|G> 3^m 0^n 1^k 2^p 3`, four counters |
| `f_(ω^ω)` | `fwpw_5` | Landmark `0^inf 3^(2m+1) 2^k A> 1^n 4 2`, two fixed segments after `1^n` |
| `double f_(ω^ω)` | `dfwpw_5` | `0^inf 3^(2m+1) 2^k A> 1^n 4 3 0` |
| `f_(ω^ω+1)` | `fwpw1_5a`, `fwpw1_5b`, `fwpw1_6` | Three variants of `0^inf 3^(2m+1) 2^k A> 1^n 4 <segment>`: `2^p 0`, `0^p 2` and `2^p 5`; `2^p` is the fourth counter |
| left-hand `5^r` variants | `fwp2_6`, `fwp21_6`, `fwp21_7`, `fwp22_7` | A `5^r` segment on the left of the `3` run |
| `f_(ε0)` | `fe0_5` | Landmark `1^m B> 2^n 0`, the counter segment `2^n` is right under the head |

## Measured reachability (2026-11-19)

Test conditions: `--all --cap 3000 --budget 3e6`, blank-tape start, state count = the core's fixed state count, symbol count = the core's own symbol count.

The "Result" column: `✅` = landmark found, `❌` = not found. The `abort` column: 0 = the search ended naturally, 1 = the budget ran out. `steps` / `nodes` are what was actually consumed.

| Core | Result | abort | steps | nodes | Maximum configuration found |
|---|---|---|---|---|---|
| `fw_4` | ✅ | 0 | 1 055 139 | 17 | `0^inf 3^(1) 2^16 A> 1^3 2` |
| `fw_3` | ❌ | 0 | 60 007 | 1 | Reached only 1 node; the txt notes that the state / symbol count must be increased (with `--states 6`, `1^3 A> 1` was found) |
| `dfw_5` | ✅ | 0 | 1 395 667 | 20 | `0^inf 3^(1) 2^16 A> 1^3 2` |
| `dfw_4` | ❌ | 0 | 3 | 0 | Halted after 3 steps (needs `--startstate` or `--start cfg`) |
| `tfw_5` | ❌ | 0 | 17 | 0 | Halted after 17 steps (same) |
| `tfw_6` | ✅ | 0 | 1 838 748 | 23 | `0^inf 3^(1) 2^16 A> 1^3 2` |
| `fw1_3` | ✅ | 1 | 16 777 216 | 159 | `1^22 F> 1^3 2` (the new 7-state core; the old `fw1_3a` / `fw1_3b` found `1^2 F> 0^1 1` / `1^2 F> 0^2 1`) |
| `fw1_4` | ❌ | 0 | 54 206 | 1 | Reached only 1 node (the newer 4-state core) |
| `fw1_5` | ✅ | 1 | 16 777 216 | 181 | `0^inf 3^(1) 2^30 A> 1^2 4^2 0^inf` |
| `fw1_6` | ❌ | 0 | 36 055 | 1 | Reached only 1 node |
| `fw1fw_4` | ✅ | 0 | 3 000 | 0 | `0^inf 3^(1) 2^5 A> 1^1 0^inf` |
| `fw2_3` | ❌ | 0 | 233 856 | 2 | Reached only 2 nodes |
| `fw2_4` | ❌ | 0 | 6 398 877 | 54 | Reached 54 nodes (the newer 6-state core) |
| `fw2_5` | ✅ | 1 | 16 777 216 | 292 | `0^inf 3^(1) 2^1 B> 1^3 2^2 4^3 0^inf` |
| `fw2_6` | ❌ | 1 | 16 777 216 | 156 | Budget exhausted, no match |
| `fw2_7` | ❌ | 0 | 45 056 | 1 | Reached only 1 node; the 3-state / 7-symbol core has no construction phase of its own |
| `fw3_3` | ✅ | 0 | 6 995 640 | 58 | `1^1 F> 0^1 1^3 2^4 0` |
| `fw3_4` | ✅ | 1 | 16 777 216 | 112 | `1^1 F> 0^4 1^2 2^1 3` |
| `fw3_5` | ✅ | 1 | 16 777 216 | 132 | `0^inf 3^(1) 2^1 A> 1^2 4^2 3^5 1^2 0^inf` |
| `fw3_6` | ✅ | 1 | 16 777 216 | 190 | `0^inf 3^(1) 2^1 A> 1^2 4^2 5^1 2^2 0^inf` |
| `fw3_7` | ✅ | 1 | 16 777 216 | 149 | `0^inf 3^(1) 2^1 A> 1^2 4^4 5^3 6^2 0^inf` |
| `fw4_5` | ❌ | 1 | 16 777 216 | 145 | Budget exhausted, no match |
| `fw4_8` | ✅ | 1 | 16 777 216 | 144 | `0^inf 3^(1) 2^1 A> 1^2 4^4 5^3 6^3 7^4 0^inf` |
| `fwm2_3` | ❌ | 0 | 195 174 | 3 | Reached only 3 nodes |
| `fwm2_5` | ✅ | 1 | 16 777 216 | 160 | `0^inf 3^(1) 2^10 D> 1^2 0^inf` (the new 6-state core with the head in `D`; the old 8-state `A`-head core found `0^inf 3^(1) 2^1 A> 1^1 2`) |
| `fwm2_6` | ❌ | 1 | 16 777 216 | 96 | Budget exhausted, no match |
| `fwm2_7` | ✅ | 1 | 16 777 216 | 93 | `0^inf 3^(1) 2^2 A> 1^3 2` |
| `fwm2fw_5` | ✅ | 0 | 2 232 425 | 18 | `0^inf 3^(1) 2^3 A> 1^3 0^inf` |
| `dfwm2_5` | ✅ | 1 | 16 777 216 | 93 | `0^inf 3^(1) 2^1 A> 1^5 2` |
| `fwm21_3` | ❌ | 1 | 16 777 216 | 115 | Budget exhausted, no match (the txt notes that the state / symbol count must be increased) |
| `fwm21_4` | ❌ | 1 | 16 777 216 | 88 | Budget exhausted, no match |
| `fwm21_5` | ✅ | 1 | 16 777 216 | 133 | `0^inf 3^(1) 2^3 A> 1^2 4^2 0^inf` |
| `fwm21_6` | ❌ | 1 | 16 777 216 | 81 | Budget exhausted, no match |
| `fwm21_7` | ❌ | 1 | 16 777 216 | 159 | Budget exhausted, no match |
| `fwm22_3` | ❌ | 1 | 16 777 216 | 171 | Budget exhausted, no match |
| `fwm22_8` | ❌ | 1 | 16 777 216 | 118 | Budget exhausted, no match |
| `fwp2_4` | ❌ | 0 | 2 653 673 | 18 | 18 nodes, ended naturally; the txt notes that the state / symbol count must be increased |
| `fwp2_5` | ❌ | 1 | 16 777 216 | 101 | Budget exhausted, no match |
| `fwp2_6` | ✅ | 1 | 16 777 216 | 94 | `0^inf 5^5 3^(1) 2^1 A> 1^1 2` |
| `fwp21_4` | ✅ | 1 | 16 777 216 | 85 | `0^inf A\|C\|G> 3^4 0^3 1 0^inf` |
| `fwp21_5` | ❌ | 1 | 16 777 216 | 79 | Budget exhausted, no match |
| `fwp21_6` | ✅ | 1 | 16 777 216 | 75 | `0^inf 5^4 3^(7) 2^1 A> 1^1 2^2 5` |
| `fwp21_7` | ❌ | 1 | 16 777 216 | 147 | Budget exhausted, no match |
| `fwp21fw_4` | ✅ | 1 | 16 777 216 | 72 | `1^9 G> 1^1 0` |
| `fwp21fw_6` | ✅ | 1 | 16 777 216 | 71 | `0^inf 3^(1) 2^8 A> 1^2 4 0^inf` |
| `fwp22_4` | ✅ | 0 | 5 667 098 | 25 | `0^inf A\|C\|G> 3^2 0^2 1^1 2 0^inf` (with 13 states / 5 symbols and a 2e9 step budget: `0^inf A\|C\|G> 3^2 0^18 1^1 2 0^inf`) |
| `fwp22_5` | ❌ | 0 | 3 000 | 0 | Hit `--cap` on the first branch; no node was expanded |
| `fwp22_7` | ❌ | 1 | 16 777 216 | 65 | Budget exhausted, no match |
| `fwp22fw_4` | ✅ | 0 | 2 982 473 | 11 | `1^2 G> 1^2 0` |
| `fwp23_4` | ❌ | 0 | 5 040 451 | 19 | The blank tape does not reach it (see the table below) |
| `fwpw_5` | ✅ | 1 | 16 777 216 | 68 | `0^inf 3^(1) 2^4 A> 1^4 4 2` (with `--cap 200000 --budget 3e7`: `0^inf 3^(1) 2^4 A> 1^2 4 2`) |
| `dfwpw_5` | ✅ | 1 | 16 777 216 | 86 | `0^inf 3^(1) 2^23 A> 1^1 4 3 0^inf` (with `--cap 20000 --budget 2e7`: `0^inf 3^(27) 2^1 A> 1^2 4 3 0`; with `--cap 200000 --budget 3e9`: `0^inf 3^(1) 2^23 A> 1^1 4 3 0^inf`) |
| `fwpw1_5a` | ❌ | 0 | 1 | 0 | Halted after 1 step (the txt says to use a start state other than `A`) |
| `fwpw1_5b` | ❌ | 1 | 16 777 216 | 61 | Budget exhausted, no match |
| `fwpw1_6` | ❌ | 1 | 16 777 216 | 59 | Budget exhausted, no match |
| `fe0_5` | ❌ | 1 | 16 777 216 | 58 | Budget exhausted, no match (with `--start cfg --m 2 --n 5` the input configuration `1^2 B> 2^5 0` is recognised) |

**27 / 56 found the landmark** (`abort=1` means the budget ran out after the landmark was found, so the result may still be too small). The numbers are from a uniform re-measurement on 2026-11-19; the supplementary results of individual cores under a larger budget / more states are in their own sections.

> ⚠️ "❌ not reached within the budget" is **not** the same as "this core cannot produce this landmark". These cores have deeper construction phases that the DFS does not reach within the budget. Increasing `--budget` / `--cap`, adding `--states`, or switching to `--start cfg` may find them.

## Cores with no landmark found

| Core | What was observed | Suggested next attempt |
|---|---|---|
| `dfw_4` | Halted after 3 steps (`abort=0`) | `--startstate B`, or `--start cfg` |
| `tfw_5` | Halted after 17 steps (`abort=0`) | `--startstate B`, or `--start cfg` |
| `fw1_4` | Reached only 1 node, ended naturally (the newer 4-state core) | Increase `--budget`, try `--states 6` |
| `fw1_6` | Reached only 1 node, ended naturally | `--states 4`, `--symbols 7` |
| `fw2_3` | Reached only 2 nodes, ended naturally | `--states 8`, increase `--budget` |
| `fwm2_3` | Reached only 3 nodes, ended naturally | `--start cfg` (verified: it recognises the input configuration `1^3 B> 0 2^2 1^4 0`) |
| `fw2_4` | Reached 54 nodes, ended naturally (the newer 6-state core) | Increase `--budget`, try `--states 7` |
| `fw2_6` | 156 nodes, budget exhausted | `--states 5` |
| `fw2_7` | Reached only 1 node, ended naturally (the 3-state / 7-symbol core) | `--start cfg --p 1 --q 1` (verified: it recognises the input configuration `0^inf 3^(1) 2^16 A> 1^3 4 5 6` at step 0), `--states 4` |
| `fw4_5` | 145 nodes, budget exhausted | `--states 8` |
| `fwm2_6` | 96 nodes, budget exhausted | `--states 7` |
| `fwm21_3` | 115 nodes, budget exhausted (the landmark is now `1^m B> 0 2^k 1^n 0 1^p 2`) | `--start cfg --p 1` (verified: it recognises the input configuration `1^1 B> 2^16 1^3` at step 0), increase `--states` / `--symbols` (the txt says this is needed) |
| `fwm21_4` | 88 nodes, budget exhausted | `--start cfg` (verified: it recognises the input configuration `1^3 B> 0 2^2 1^4 0^2 3`) |
| `fwm21_6` | 81 nodes, budget exhausted | `--states 8` |
| `fwm21_7` | 159 nodes, budget exhausted | `--start cfg --p 1` (verified: it recognises the input configuration `0^inf 5 3^(1) 2^16 A> 1^3 2 5` at step 0), `--states 7` |
| `fwm22_3` | 171 nodes, budget exhausted | `--start cfg --p 1 --q 1` (verified: it recognises the input configuration `1^1 B> 2^16 1^3` at step 0), `--states 13` |
| `fwm22_8` | 118 nodes, budget exhausted | `--start cfg --p 1 --q 1` (verified: it recognises the input configuration `0^inf 5 3^(1) 2^16 A> 1^3 2 7 5` at step 0), `--states 7` |
| `fwp2_4` | 18 nodes, ended naturally | `--start cfg` (verified: it recognises the input configuration `0^inf A> 3^3 0`), increase `--states` / `--symbols` (the txt says this is needed) |
| `fw_3` | Reached only 1 node, ended naturally | `--states 6` (verified: it finds `1^3 A> 1`), `--symbols 4` (the txt says this is needed) |
| `fwp21_5` | 79 nodes, budget exhausted | Increase `--budget` / `--cap`, `--states 10` |
| `fwp21_7` | 147 nodes, budget exhausted | `--start cfg --p 1 --r 1` (verified: it recognises the input configuration `0^inf 5^1 3^(1) 2^16 A> 1^3 2 5` at step 0), `--states 8` |
| `fe0_5` | 58 nodes, budget exhausted | `--start cfg` (verified: it recognises the input configuration `1^2 B> 2^5 0`), increase `--budget` / `--cap` |
| `fwp23_4` | 19 nodes, ended naturally (with `--cap 200000 --budget 3e9` it still had not matched after 261 399 343 steps; `--states 13` / `--symbols 5` likewise did not match) | `--start cfg` (verified: it recognises the input configuration `0^inf A> 3^1 0^2 1^1 2^1 3`), increase `--budget` / `--cap` |
| `fwp2_5` | 101 nodes, budget exhausted | Increase `--budget` / `--cap`; `--start cfg --n 3` (verified: it recognises the input configuration `0^inf C> 3^3 0`, the start state is `C` because the landmark's head set is `C`/`G`) |
| `fwp22_5` | 0 nodes, hit `--cap` on the first branch | Increase `--cap` (the default 3 000 is far too small for this core), then `--budget`; `--start cfg --m 1 --n 2 --k 1` (verified: it recognises the input configuration `0^inf A> 3^1 4^2 0^1 4`) |
| `fwp22_7` | 65 nodes, budget exhausted | `--start cfg --p 1 --q 1 --r 1` (verified: it recognises the input configuration `0^inf 5^1 3^(1) 2^16 A> 1^3 2 5 6` at step 0), increase `--budget` / `--cap` |
| `fwpw1_5a` | Halted after 1 step (`abort=0`) | `--startstate` with a state other than `A` (the txt says so explicitly); `--start cfg --k 2 --n 3 --m 1 --p 2` (verified: it recognises the input configuration `0^inf 3^(3) 2^2 A> 1^3 4 2^2`) |
| `fwpw1_5b` | 61 nodes, budget exhausted | `--start cfg --k 2 --n 3 --m 1 --p 2` (verified: it recognises the input configuration `0^inf 3^(3) 2^2 A> 1^3 4`), increase `--budget` / `--cap` |
| `fwpw1_6` | 59 nodes, budget exhausted | `--start cfg --k 2 --n 3 --m 1 --p 2` (verified: it recognises the input configuration `0^inf 3^(3) 2^2 A> 1^3 4 2^2 5`), increase `--budget` / `--cap` |

## Special core notes

### `fw3_4`

`fw3_4` was replaced; `bbcores_new.txt` now contains only:

- **`fw3_4`**: 7 states / 4 symbols, landmark `1^m F> 0^n 1^k 2^p 3` (`KIND_ONES`), importance `p > k > n > m`

The old 8-state version (once named `fw3_4_old`, landmark `0^inf 3^(2m+1) 2^k A|E> 1^n 2 1^p 0^q 1^r 2`) has been removed from `bbcores_new.txt` and is no longer built into the code.

### `fw1_3` — the `f_(ω+1)` core whose head cell is the first cell of the `1^n` run

Landmark `1^m F> 1^n 2`:

- The head state is fixed to `F`; the head cell is the **first cell** of the `1^n` run (the notation convention "the segment after the marker starts at the head cell")
- The `1^m` to the left of the head and the `1^n` run are **the same run of `1`s**: `m` cells of `1` immediately to the left of the head, and `n` cells of `1` counted from the head cell rightwards (the `n` count includes the head cell) — the same convention as `fwp21fw_4` / `fwp22fw_4` (`1^n G> 1^m 0`), only with the roles of the two counters swapped
- The trailing `2` is a fixed single-cell segment (`right[1]` = `{2,1,-1,-1}`); the cfg does not end with `BLANK`, so by the **last-segment rule** it means "≥1 cell"
- Importance is `n > m` → slot order `{ SLOT_N, SLOT_M }`
- This core replaces the old `fw1_3a` / `fw1_3b` pair, whose head cell was a blank run (`0^n`) followed by a `1` sentinel (`1^m F> 0^n 1`)

In practice: under the default budget the blank-tape search finds `1^22 F> 1^3 2` (steps=16 777 216 / nodes=159 — the budget ran out, so the result may still be too small); `--start cfg --n 3 --m 2` recognises the input configuration at step 0 (the loose-mode print shows `1^2 F> 1^2 2`, because the printed right-hand shape starts **after** the head cell).

### `fwm2_3` / `fwm21_3` / `fwm21_4` / `fwm22_3` — the `KIND_ONES` cores whose head cell is a blank cell

Landmarks `1^m B> 0 2^k 1^n 0` (`fwm2_3`), `1^m B> 0 2^k 1^n 0 1^p 2` (`fwm21_3`), `1^m B> 0 2^k 1^n 0^p 3` (`fwm21_4`), `1^m B> 0 2^k 1^n 0 1^p 2^q 0` (`fwm22_3`):

- The head state is `B` (neither `A` nor `F`)
- The head cell is **exactly one** blank cell, not a counter segment (`0^1` is fixed)
- All counters are on the right: `2^k` and `1^n`; `fwm21_4` also has a `0^p` segment (counted into slot `p`), `fwm21_3` / `fwm22_3` instead have a **fixed** blank cell (`0^1`) followed by the counter `1^p`
- The trailing segment follows the **last-segment rule** when the cfg does not end with `BLANK`: `fwm21_3`'s `2` and `fwm21_4`'s `3` mean "≥1 cell", while `fwm22_3` ends with `BLANK` (so the whole right side after the `2^q` run must be blank)
- Importance: `fwm2_3` is `k > n > m`; `fwm21_3` / `fwm21_4` are `p > k > n > m`; `fwm22_3` is `q > p > k > n > m`

The zero segment has a fixed length of 1 for these four cores, so `right[0]` is `{0,1,1,-1}` (symbol 0, a fixed single cell, no slot), and the `0^p` segment is recorded separately by `SLOT_P` in `sp.right[]`. `fwm2_3`'s cfg ends with an explicit `0^1`, so it carries an extra blank sentinel `{0,1,1,-1}` at the end (only that one cell is checked for being blank). `fwm22_3`'s `right[]` is `{0,1,1,-1}`, `{2,1,-1,SLOT_K}`, `{1,1,-1,SLOT_N}`, `{0,1,1,-1}`, `{1,1,-1,SLOT_P}`, `{2,1,-1,SLOT_Q}` with `termBlank` true.

### `fwp2_4` / `fwp2_5` — the cores that use `KIND_THREE`

Landmarks `0^inf A|C|G> 3^n 0` (`fwp2_4`) and `0^inf C|G> 3^n 0` (`fwp2_5`):

- The head cell is the first cell of the `3^n` run and **the whole tape except that run of `3`s is blank** (`0^inf` on the left, `0^inf` on the right)
- The counter is the length of that run of `3`s, `n >= 3`; the txt's `slot` line has only one counter
- `fwp2_4` allows the head in state `A`, `C` or `G` (all three have the transition `3LG` on symbol `3`, i.e. write `3` back and move left); `fwp2_5` is the 5-symbol variant and allows only `C` or `G` — its `A` row is not the "turn left into the counter" transition, so the head state set is narrower
- `--start cfg` puts `n` cells of `3` under the head (`n` at least 3), and the start state is the lowest landmark-allowed state (`A` for `fwp2_4`, `C` for `fwp2_5`)

This is `KIND_THREE` in `bbsearch.cpp`; the difference from `KIND_BCFG` / `KIND_ONES` is that the counter segment sits right under the head and there is no structure on the left at all.

In practice: `fwp2_4` reaches only 18 nodes from a blank tape (the txt says the state / symbol count must be increased); `fwp2_5` exhausts the budget after 101 nodes. `--start cfg --n 3` recognises the input configuration `0^inf C> 3^3 0` for `fwp2_5` at step 0 (its start state is `C`, the lowest state of its `C`/`G` head set).

### `fwp21_4` / `fwp21_5` — multi-segment variants of `KIND_THREE`

Landmarks `0^inf A|C|G> 3^m 0^n 1` (`fwp21_4`) and `0^inf A|C|G> 3^m 0^n 4` (`fwp21_5`):

- The head can be in state `A`, `C` or `G`; the head cell is still the first cell of the `3^m` run and everything to the left must be blank
- On the right, after `3^m` comes a `0^n` segment (the second counter), then **exactly one** sentinel cell (`1` and `4` respectively), and after the sentinel everything must be blank
- Importance is `n > m` for both → slot order `{ SLOT_N, SLOT_M }`
- In the txt the sentinel is written as a bare symbol (a single symbol with no exponent at the end); by the notation rules it is an anchor, not a counter

The matching logic of `KIND_THREE` (`noteCfgThree()`) already counts segment by segment according to `right[]`, so these two cores **need no new kind**: it is enough to configure the right side as three segments in the txt's `cfg` line: `{3,1,-1,SLOT_M}`, `{0,<n minimum>,-1,SLOT_N}`, `{1|4,1,1,-1}`. The minimum of `n` comes from the `slot` line (`fwp21_4` is `n>=2` → `{0,2,...}`, `fwp21_5` is `n>=1` → `{0,1,...}`).

> Note: the template printed by `--cores` shows one extra `0` after the sentinel; that is the terminating blank cell required by `termBlank`, not part of the landmark.

### `fw_3` — a `KIND_ONES` core whose head cell is the symbol `1`

Landmark `1^n A> 1 0`:

- The head state is fixed to `A` (`fw_3`'s state `A` has the transition `1LE` on symbol `1`, i.e. write `1` back and turn left — the same "turn left into the counter" feature as `fw_4`'s `3LB` and `fw2_3`'s `0LE`)
- The head cell is **one `1`** (a sentinel); the `n` cells of `1` immediately to its left are the counter (`n >= 1`), and everything further left must be blank
- To the right of the head cell is **exactly one blank cell** (`right[1]` = `{0,1,1,-1}`), treated as a sentinel: only that one cell is checked for being blank, and the blank region after it does not count towards the length
- The txt's `slot` line has only one counter → importance is `n`
- It is also `KIND_ONES` (the "a run of `1^m` on the left" shape): the only difference from the `fw1_3` family is that the counter segment is on the **left** of the head, not under the head cell

`fw_3` **cannot** produce this shape on a blank tape with 5 states: both transitions into state `A` (`A`'s `1LE` and `D`'s `1RC`) write `2`, so while the machine runs, whenever the head is in state `A` the head cell is a `2` and can never be a `1`. Therefore `1^n A> 1` only holds in the **input configuration** (step 0); the txt explicitly says "Please increase state or symbol count to search!". In practice, `--states 6 --cap 1e7 --budget 3e8` finds `1^1` / `1^2` / `1^3 A> 1` (that run was aborted manually before reaching the budget limit), because the extra states give the DFS the chance to construct a machine that moves a `1` into `A`.

### `fe0_5` — a `KIND_ONES` core whose head cell is on the `2^n` segment

Landmark `1^m B> 2^n 0^inf`:

- The head state is fixed to `B` (`fe0_5`'s state `B` has the transition `2LC` on symbol `2`, i.e. write `2` back and turn left)
- The head cell is the **first cell** of the `2^n` run — the notation convention is "the segment after the marker starts at the head cell", consistent with `1^n A> 1 0` (`fw_3`) and `0^inf A|C|G> 3^n 0^inf` (`KIND_THREE`)
- The head has `m` cells of `1` **immediately to its left** (`m >= 1`), and everything further left must be blank; everything after the `2^n` run must be blank too
- `n` is the length of the `2^n` run (`n >= 3`), importance is `n > m` → slot order `{ SLOT_N, SLOT_M }`
- It is also `KIND_ONES`: the difference from `KIND_THREE` is that the left side is not blank but a run of `1^m` (`m` is the second counter); `right[]` needs only one segment `{2,3,-1,SLOT_N}`

`fe0_5` has 17 states (A..Q), exceeding the original `MAXST` of 16, so `MAXST` was raised to **25** (A..Y are ordinary states, `'Z'` is the halt marker and does not count as a state).

In practice: `--start cfg --m 2 --n 5` recognises the input configuration `1^2 B> 2^5 0` at step 0; starting from a blank tape the landmark was not reached within a 3e9 step budget (`abort=1`), which is **insufficient budget** rather than the core not producing the shape.

### `fwp21fw_4` / `fwp22fw_4` — `KIND_ONES` cores whose head cell is the first cell of the `1^m` segment

Landmark `1^n G> 1^m 0`:

- The head state is fixed to `G`, and the head cell is the **first cell** of the `1^m` run (the same notation convention as the `2^n` run of `fe0_5`)
- The head has `n` cells of `1` **immediately to its left** (`n >= 2`), and everything further left must be blank — this run of `1^n` is the counter being decremented: `G`'s transition on symbol `1` is `1LH` (write `1` back and turn left into `H`), and `H` then rewrites the `1`s to the left one by one into `2` (`2LH`)
- `right[]` is `{1,1,-1,SLOT_M}` plus a **single-cell** blank sentinel `{0,1,1,-1}` — the txt's input configuration is written `1^n G> 1^m 0`, and that `0` is that cell
- Importance is `n > m` → slot order `{ SLOT_N, SLOT_M }`

> Notationally, `1^n` and `1^m` are **the same run of `1`s**: the head cell lies inside it, with `n` cells of `1` to the left of the head and `m` cells of `1` from the head cell rightwards. So the count of `m` is "the number of `1`s counted from the head cell to the right (including the head cell)".

In practice: `fwp21fw_4` finds `1^9 G> 1^1 0` under the default budget (also `1^9 G> 1^1 0` with 13 states / 5 symbols and a 2e9 step budget; `--start cfg --n 3 --m 3` recognises the input configuration at step 0); `fwp22fw_4` finds `1^2 G> 1^2 0` already under the default budget (`--start cfg --n 3 --m 2` recognises the input configuration at step 0).

### `fwp22_4` — the three-counter variant of `KIND_THREE`

Landmark `0^inf A|C|G> 3^m 0^n 1^k 2`:

- The head can be in state `A`, `C` or `G`; the head cell is the first cell of the `3^m` run, everything to the left must be blank, and everything after the `2` must be blank too (`termBlank`)
- Three counter segments on the right — `3^m` (slot `m`), `0^n` (slot `n`, `n >= 2`), `1^k` (slot `k`) — and finally **exactly one** `2`: `{3,1,-1,SLOT_M}`, `{0,2,-1,SLOT_N}`, `{1,1,-1,SLOT_K}`, `{2,1,1,-1}`
- Importance is `k > n > m` → slot order `{ SLOT_K, SLOT_N, SLOT_M }`

In practice: the default budget `--cap 3000 --budget 3e6` finds `0^inf A|C|G> 3^2 0^2 1^1 2 0^inf` (at step 13); `--start cfg --m 1 --n 2 --k 1` recognises the input configuration at step 0.

### `fwp22_5` — the `f_(ω^2+2)` variant with a `4^n` segment

Landmark `0^inf A|C|G> 3^m 4^n 0^k 4`:

- The head can be in state `A`, `C` or `G`; the head cell is the first cell of the `3^m` run and everything to the left must be blank
- The right side is the counter `3^m` (slot `m`), the counter `4^n` (slot `n`, `n >= 2`), the counter `0^k` (slot `k`, `k >= 1`) and finally **exactly one** `4`: `{3,1,-1,SLOT_M}`, `{4,1,-1,SLOT_N}`, `{0,1,-1,SLOT_K}`, `{4,1,1,-1}`
- Importance is `k > n > m` → slot order `{ SLOT_K, SLOT_N, SLOT_M }`
- Compared with `fwp22_4` the second segment is a run of `4`s instead of a run of `0`s, and the trailing sentinel is a `4` instead of a `2`; the number of segments and their slots are unchanged, so `KIND_THREE` matches it without a new kind

In practice: the default budget is too small — the search hit `--cap` on the very first branch without expanding a single node, so no landmark was found. Use a larger `--cap` / `--budget`; `--start cfg --m 1 --n 2 --k 1` recognises the input configuration `0^inf A> 3^1 4^2 0^1 4` at step 0.

### `fwp23_4` — the four-counter variant of `KIND_THREE`

Landmark `0^inf A|C|G> 3^m 0^n 1^k 2^p 3`:

- The head can be in state `A`, `C` or `G`; the head cell is the first cell of the `3^m` run, everything to the left must be blank, and everything after the trailing `3` must be blank too (`termBlank`)
- Four counter segments on the right — `3^m` (slot `m`), `0^n` (slot `n`, `n >= 2`), `1^k` (slot `k`), `2^p` (slot `p`) — and finally **exactly one** `3`: `{3,1,-1,SLOT_M}`, `{0,2,-1,SLOT_N}`, `{1,1,-1,SLOT_K}`, `{2,1,-1,SLOT_P}`, `{3,1,1,-1}`
- Importance is `p > k > n > m` → slot order `{ SLOT_P, SLOT_K, SLOT_N, SLOT_M }`

In practice: `--start cfg --m 1 --n 2 --k 1 --p 1` recognises the input configuration `0^inf A> 3^1 0^2 1^1 2^1 3 0` at step 0; starting from a blank tape it did not match under `--cap 3000 --budget 3e6` (19 nodes, ended naturally), nor under `--cap 200000 --budget 3e9` (still no match after 261 399 343 steps), nor with `--states 13`, nor with `--symbols 5`.

### `fwpw_5` — `KIND_BCFG` with two fixed right-hand segments

Landmark `0^inf 3^(2m+1) 2^k A> 1^n 4 2`:

- The head state is fixed to `A`, and the head cell is the first cell of the `1^n` run; to its left are `2^k` and then `3^(2m+1)`, and further left everything is blank
- The txt's `cfg` reads `BCFG(A) 4^1 2^1`; neither symbol has an exponent, so the right side is two fixed segments: `{4,1,1,-1}` and `{2,1,-1,-1}` — the trailing `2` follows the **last-segment rule** and means "≥1 cell", not exactly one cell
- `termBlank` is false — no blank is required after the `2`, so cells after the `2` still count as a match
- Importance is `k > n > m` → slot order `{ SLOT_K, SLOT_N, SLOT_M }`

In practice: `--censusL` (with a 3e6 step budget) observed configurations whose left side complies and whose right side is `4 2 …` (e.g. `3^1 2^3 | 4 2 1^2`), confirming that these two fixed segments match the shapes the machine actually produces; the blank-tape search finds `0^inf 3^(1) 2^4 A> 1^4 4 2` under the default budget (`--cap 3000 --budget 3e6`) and `0^inf 3^(1) 2^4 A> 1^2 4 2` under `--cap 200000 --budget 3e7`.

### `dfwpw_5` — `KIND_BCFG` whose second right-hand segment is `3`

Landmark `0^inf 3^(2m+1) 2^k A> 1^n 4 3 0`:

- The head state is fixed to `A`, and the head cell is the first cell of the `1^n` run; to its left are `2^k` and then `3^(2m+1)`, and further left everything is blank
- The txt's `cfg` reads `BCFG(A) 4^1 3^1 BLANK`; the right side is two fixed segments of **exactly one cell each**: `{4,1,1,-1}` and `{3,1,1,-1}`
- `termBlank` is true — after the `3` the **whole right side** must be blank (the `BLANK` in the txt): checking a single cell is not enough, and `4 3 0 1` (writing `1` one cell after the `3`) is a different configuration and does not count as `4 3 0 0`. **Note**: you must not add an explicit `0^1` segment instead of `BLANK` — the `KIND_BCFG` matching branch counts the number of consecutive blank cells (far more than 1), so it would never match
- Importance is `k > n > m` → slot order `{ SLOT_K, SLOT_N, SLOT_M }`

In practice: the blank-tape search finds `0^inf 3^(1) 2^23 A> 1^1 4 3 0^inf` under the default budget (`--cap 3000 --budget 3e6`, steps=16 777 216 / nodes=86), `0^inf 3^(27) 2^1 A> 1^2 4 3 0^inf` under `--cap 20000 --budget 2e7` (steps=33 554 432 / nodes=30), and `0^inf 3^(1) 2^23 A> 1^1 4 3 0^inf` under `--cap 200000 --budget 3e9` (steps=3 003 121 664 / nodes=211); `--start cfg --k 3 --n 1 --m 0` recognises the input configuration `0^inf 3^(1) 2^3 A> 1^1 4 3` at step 0. The cells `H0` / `K0` in the core are FREE; completing them with `--fix H0=3RJ,K0=3LI` turns the core into a full Turing machine, from which `0^inf 3^(1) 2^3 A> 1^1 4 3 0^inf` is reachable from a blank tape.

**Different FREE completions of the same core produce different right-hand shapes**: another machine, `--fix B4=1LJ,H0=0LI,J0=4RB` (a different choice for the three cells `B4` / `H0` / `J0`), reaches `B(4,1,0) 4 3 0 1` — a `1` written one cell after the `3`, so that is not `4 3 0 0`; since 2026-11-15 the tool no longer treats it as a landmark (`termBlank` now scans the whole right side instead of checking only the first cell).

### `fwpw1_5a` / `fwpw1_5b` / `fwpw1_6` — the three `f_(ω^ω+1)` variants

Landmarks `0^inf 3^(2m+1) 2^k A> 1^n 4 2^p 0` (`fwpw1_5a`), `0^inf 3^(2m+1) 2^k A> 1^n 4 0^p 2` (`fwpw1_5b`), `0^inf 3^(2m+1) 2^k A> 1^n 4 2^p 5` (`fwpw1_6`):

- The head state is fixed to `A`, and the head cell is the first cell of the `1^n` run; to its left are `2^k` and then `3^(2m+1)`, and further left everything is blank
- All three have the same left-hand structure and the same single-cell `4` on the right; they differ in the middle segment:
  - `fwpw1_5a`: the counter `2^p` (`p >= 1`), followed by the blank tail — `{4,1,1,-1}`, `{2,1,-1,SLOT_P}`, `termBlank` true (the txt reads `BCFG(A) 4^1 2^p BLANK`)
  - `fwpw1_5b`: the counter `0^p` (a run of blanks, `p >= 1`) followed by a fixed `2` — `{4,1,1,-1}`, `{0,1,-1,SLOT_P}`, `{2,1,-1,-1}`; the trailing `2` follows the **last-segment rule** and means "≥1 cell"; `termBlank` is false
  - `fwpw1_6`: the counter `2^p` followed by a fixed `5` — `{4,1,1,-1}`, `{2,1,-1,SLOT_P}`, `{5,1,-1,-1}`; the trailing `5` also follows the **last-segment rule**; `termBlank` is false
- Importance is `p > k > n > m` → slot order `{ SLOT_P, SLOT_K, SLOT_N, SLOT_M }`
- The txt's comment for `fwpw1_5a` says "Please use other start state instead of A" — starting in state `A` halts the machine immediately (measured: 1 step, 0 nodes), so this core needs `--startstate` with a different state

The old `fwpw1_5` (14 states / 5 symbols, landmark `0^inf 3^(2m+1) 2^k A> 1^n 4 2^p 0`) has been replaced by these three blocks; the txt no longer contains a core named `fwpw1_5`.

In practice: none of the three reaches its landmark from a blank tape under the default budget (`fwpw1_5a` halts after 1 step, `fwpw1_5b` and `fwpw1_6` exhaust the budget after 61 / 59 nodes). `--start cfg --k 2 --n 3 --m 1 --p 2` recognises the input configuration of each at step 0: `0^inf 3^(3) 2^2 A> 1^3 4 2^2` (`fwpw1_5a`), `0^inf 3^(3) 2^2 A> 1^3 4` (`fwpw1_5b` — its `0^p` segment is a run of blanks and therefore invisible) and `0^inf 3^(3) 2^2 A> 1^3 4 2^2 5` (`fwpw1_6`). For `fwpw1_5a` a different start state than `A` is required, as the txt says: `--startstate B` makes the machine run (16 777 216 steps / 71 nodes) but the landmark is still not reached.

### `fw1_4` / `fw2_4` / `fw2_5` — the `KIND_BCFG` cores whose head is in state `B`

Landmarks `0^inf 3^(2m+1) 2^k B> 1^n 2^p 0` (`fw1_4`), `0^inf 3^(2m+1) 2^k B> 1^n 2^p 0^q 1` (`fw2_4`), `0^inf 3^(2m+1) 2^k B> 1^n 2^p 4^q 0` (`fw2_5`):

- The head state is fixed to `B`, not `A` — these three cores belong to the `B'(k, n, m) …` family (`B'` means "the head is in state B"), which differs from `fw_4`'s `B(k, n, m) …` family (head in `A`)
- The left-hand structure is exactly the same as `fw_4`'s: `3^(2m+1)` leftmost, `2^k` adjacent to the head, everything further left blank
- `fw1_4`'s right side has only a counter segment `2^p` (`p >= 1`): `{2,1,-1,SLOT_P}`; the trailing blank is covered by `termBlank` (the txt reads `BCFG(B) 2^p BLANK`, with no explicit blank segment)
- `fw2_4`'s right side is the counter `2^p`, the counter `0^q` (`q >= 1`) and one `1` segment: `{2,1,-1,SLOT_P}`, `{0,1,-1,SLOT_Q}`, `{1,1,-1,-1}` (the trailing segment follows the **last-segment rule** and means "≥1 cell"); `termBlank` is false, so nothing is required after the `1`
- `fw2_5`'s right side is the counter `2^p` and the counter `4^q` (`q >= 1`): `{2,1,-1,SLOT_P}`, `{4,1,-1,SLOT_Q}`, with `termBlank` true (the trailing blank cell is handled by the fallback)
- Importance: `fw1_4` is `p > k > n > m`, `fw2_4` and `fw2_5` are `q > p > k > n > m`

In practice: neither `fw1_4` nor `fw2_4` reached the landmark from a blank tape under the default budget (`fw1_4` reached only 1 node, `fw2_4` only 54 nodes, both ending naturally), and `fw1_4` still did not reach it under `--cap 200000 --budget 3e9`; with `--start cfg` both recognise the input configuration `0^inf 3^(1) 2^16 B> 1^3 2` at step 0. `fw2_5` finds the landmark already under the default budget, with the maximum configuration `0^inf 3^(1) 2^1 B> 1^3 2^2 4^3 0^inf`; `--start cfg` likewise recognises the input configuration at step 0.

### `fw1fw_4` / `fwm2fw_5` — the `f_(ω+1)+f_ω` and `f_(ω*2)+f_ω` machines

Landmark `0^inf 3^(2m+1) 2^k A> 1^n 0^inf` for both:

- The head state is fixed to `A`; the left-hand structure is the same as `fw_4`'s, and the right side is **exactly one blank cell** (a sentinel): `right[]` is empty and the trailing blank is covered by `termBlank`
- `fw1fw_4`'s cfg reads `BCFG(A) BLANK`; `fwm2fw_5`'s reads the same, but it is a 6-state / 5-symbol core of the `f_(ω*2)+f_ω` family
- Importance is `k > n > m` → slot order `{ SLOT_K, SLOT_N, SLOT_M }`
- For `fw1fw_4` the txt notes: the machine itself reaches the `B(5, 1, 0) 0^inf` configuration, so verification **starts in state `A`**; searching based on it requires more states and the start state set to `F`

In practice: under the default budget `--cap 3000 --budget 3e6` `fw1fw_4` immediately finds `0^inf 3^(1) 2^5 A> 1^1 0^inf`; `fwm2fw_5` finds `0^inf 3^(1) 2^3 A> 1^3 0^inf` (steps=2 232 425 / nodes=18, the search ended naturally), and `--start cfg` recognises its input configuration at step 0.

### `fwm2_5` — the `f_(ω*2)` core whose head is in state `D`

Landmark `0^inf 3^(2m+1) 2^k D> 1^n 0`:

- The head state is fixed to `D` — a third head state next to `A` (`fw_4`'s `B(k, n, m) …`) and `B` (`fw1_4` / `fw2_4` / `fw2_5`'s `B'(k, n, m) …`)
- The txt's cfg reads `BCFG(D) BLANK`: there is **no** right-hand segment at all — `right[]` is empty and the trailing blank is covered by `termBlank`
- Importance is `k > n > m` → slot order `{ SLOT_K, SLOT_N, SLOT_M }`
- The core has 6 states / 5 symbols; it replaces the previous 8-state `fwm2_5` (head in `A`, landmark `0^inf 3^(2m+1) 2^k A> 1^n 2`)

In practice: the blank-tape search finds `0^inf 3^(1) 2^10 D> 1^2 0^inf` under the default budget (steps=16 777 216 / nodes=160 — the budget ran out, so the result may still be too small); `--start cfg` recognises the input configuration `0^inf 3^(1) 2^16 D> 1^3 0^inf` at step 0.

### `fw2_7` / `fw4_8` / `fwm2_7` / `fwm21_7` / `fwm22_8` / `fwp21_7` / `fwp22_7` — the newer `KIND_BCFG` variants

All of them keep the standard left-hand structure `0^inf 3^(2m+1) 2^k` and put the head on the first cell of the `1^n` run; they differ only in the right-hand segments:

| Core | cfg | Right-hand segments | Importance |
|---|---|---|---|
| `fw2_7` | `BCFG(A) 4^1 5^p 6^q BLANK` | a fixed `4`, the counter `5^p`, the counter `6^q`, then the whole right side must be blank | `q > p > k > n > m` |
| `fw4_8` | `BCFG(A) 4^p 5^q 6^r 7^s BLANK` | four counters; the fourth is the `s` slot (there is **no `--s` CLI option**, so `--start cfg` always uses `s = 1`) | `s > r > q > p > k > n > m` |
| `fwm2_7` | `BCFG(A) 2^1` | a single trailing `2` ("≥1 cell" by the **last-segment rule**, `termBlank` false) | `k > n > m` |
| `fwm21_7` | `5^1 BCFG(A) 2^p 5^1 BLANK` | **one** `5` left of the `3` run, the counter `2^p`, a fixed `5`, then blank | `p > k > n > m` |
| `fwm22_8` | `5^1 BCFG(A) 2^p 7^q 5^1 BLANK` | the same single `5` on the left, the counters `2^p` and `7^q`, a fixed `5`, then blank | `q > p > k > n > m` |
| `fwp21_7` | `5^r BCFG(A) 2^p 5^1 BLANK` | a **run** of `5`s on the left (slot `r5`), the counter `2^p`, a fixed `5`, then blank | `p > r5 > k > n > m` |
| `fwp22_7` | `5^r BCFG(A) 2^p 5^q 6^1` | the left `5^r` run, the counters `2^p` and `5^q`, and a trailing `6` ("≥1 cell", `termBlank` false) | `q > p > r5 > k > n > m` |

The left-hand `5` marker has two spellings: `5^1` means **exactly one** `5` on the left (`L5_ONE`), `5^r` means a **run** of `5`s counted into slot `r5` (`L5_MANY`) — the same distinction as between `fwm2_6` / `fwm21_6` and `fwp2_6` / `fwp21_6`.

In practice: `fw4_8` (maximum `0^inf 3^(1) 2^1 A> 1^2 4^4 5^3 6^3 7^4 0^inf`) and `fwm2_7` (maximum `0^inf 3^(1) 2^2 A> 1^3 2`) reach their landmark under the default budget; the other five do not. All seven recognise their input configuration at step 0 with `--start cfg`:
`fw2_7` → `0^inf 3^(1) 2^16 A> 1^3 4 5 6`, `fw4_8` → `0^inf 3^(1) 2^16 A> 1^3 4 5 6 7`, `fwm2_7` → `0^inf 3^(1) 2^16 A> 1^3 2`, `fwm21_7` → `0^inf 5 3^(1) 2^16 A> 1^3 2 5`, `fwm22_8` → `0^inf 5 3^(1) 2^16 A> 1^3 2 7 5`, `fwp21_7` → `0^inf 5^1 3^(1) 2^16 A> 1^3 2 5`, `fwp22_7` → `0^inf 5^1 3^(1) 2^16 A> 1^3 2 5 6`.

### How trailing blanks are written (`termBlank`)

The txt has two spellings for a trailing blank:

- `BLANK` — after the last segment the whole right side must be blank, corresponding to `sp.termBlank = true` (e.g. `fw1_4`'s `BCFG(B) 2^p BLANK`)
- an explicit `0` segment — only that one cell is checked for being blank (a sentinel), and nothing is required after it (e.g. `fw_3`'s `1^n A> 1^1 0^1`, `fwm2_3`'s `1^m B> 0 2^k 1^n 0^1`)

The template printed by `--cores` always shows the trailing blank as `0` (`0^inf` when `termBlank` is true). This differs from the literal spelling in the txt but does not affect matching: one extra blank cell still satisfies the "the last cell is blank" check.

### `fwp2_6` / `fwp21_6` / `fwp21_7` / `fwp22_7` — the left-hand `5^r` segment

The landmark has a `5^r` segment on the left. That `r` corresponds to slot `r5` (`SLOT_R5`), which is a **different slot** from the `r` of the right-hand `1^r` segment (`SLOT_R`) in cores such as `fw3_5`. In `bbcores_new.txt` both are written as `r` and are disambiguated by context.

`fwp2_6` (`BCFG(A) 2^1`) and `fwp21_6` (`BCFG(A) 2^p 5^1`) are the two older members of this group; `fwp21_7` (`BCFG(A) 2^p 5^1 BLANK`) and `fwp22_7` (`BCFG(A) 2^p 5^q 6^1`) are the newer ones — see the table in "the newer `KIND_BCFG` variants" above. Note that the `5^r` on the left is a **run** (`L5_MANY`, slot `r5`), while `5^1` on the left of `fwm21_7` / `fwm22_8` means exactly one `5` (`L5_ONE`).
