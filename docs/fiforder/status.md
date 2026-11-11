# FIFOrder current state

Recorded 2026-11-10. Synthesis results were added the same day. No FIFOrder scheduler changes are in the tree.
BOOM v3 is the core under test. v4 sources are present and are not used by the configs below.

## Commits

- Chipyard branch `fiforder-main`. The simulation note is `7061ed71`. The commit before that note is `7f9ac35377b427a3f26050e89b621338237976f3`
- BOOM submodule `generators/boom`, branch `fiforder-main`: `ba03175e0a3cf5c0d5a7f14d385ac1c8998c4428`
- BOOM working tree is clean. The earlier pointer in `baseline-commits.txt` (`a3d8de65`) is the commit before the Small baseline note.

These Chipyard entries were already dirty and were not included in this commit:

- `generators/constellation` has local submodule changes
- `toolchains/libgloss`, `riscv-isa-sim`, `riscv-pk`, and `riscv-tests` have untracked submodule content
- `git status` still warns that `behav/BramDwc/.gitignore` and `synth/BramDwc/.gitignore` are symlink loops

## Host

This checkout was exercised on the 8-vCPU, 15 GiB, no-swap machine (about 64 GiB free on a 100 GiB disk).
`source env.sh` requires Miniforge on `PATH` first. The Makefile default Java heap is 8 GiB.
Verilator runtime threads stay at 1. Builds below used `make -j2` and did not enable waveform tracing.

## SmallBoomV3Config

Queues are 8 entries, issue width 1, for INT, FP, and MEM. ROB is 32.
The simulator was already built. `rv64ui-p-simple` had previously printed `*** PASSED ***`.
The rest of that assembly log reached `$finish` without a pass line, and the suite was interrupted. It is not a completed regression.

## MegaBoomV3Config

From `boom.v3.common.WithNMegaBooms`: INT 40/4, FP 32/2, MEM 24/2 entries/issue-width. ROB is 128. Decode width is 4.

Fresh non-tracing build on 2026-11-10, command `make -j2 CONFIG=MegaBoomV3Config`:

- Exit code 0
- Wall time 6:19.71
- `/usr/bin/time -v` maximum resident set 2,369,256 kB
- Binary `sims/verilator/simulator-chipyard.harness-MegaBoomV3Config` (gitignored)
- Generated sources about 635 MB (gitignored)

Both runs used `make CONFIG=MegaBoomV3Config run-binary` with a 5-minute timeout. Neither timed out.

- `rv64ui-p-simple`: exit 0, `*** PASSED ***`, 31,616 simulator cycles, wall time 10.68 s. This test does not print `mcycle` or `minstret`.
- `median.riscv`: exit 0, `*** PASSED ***`, 582,896 simulator cycles including load and startup, wall time 3:14.14. The benchmark window printed `mcycle = 5492` and `minstret = 4658`, so IPC is 4658/5492 = 0.848.

Logs outside the repository:

- `~/FIFOrder-project/mega-build.log`
- `~/FIFOrder-project/mega-simple-make.log`
- `~/FIFOrder-project/mega-median-make.log`

Program output is under `sims/verilator/output/chipyard.harness.TestHarness.MegaBoomV3Config/`.

## Mega scheduler synthesis

Standalone synthesis of the baseline INT and FP issue units. The BOOM sources and the Mega simulator binary were left unchanged. RTL for Yosys is in `sims/verilator/generated-src-yosys/`, separate from the simulator tree. Tools and the PDK are outside this repository, under `~/FIFOrder-project/synth-env` and `~/FIFOrder-project/sky130-pdk`.

Hammer 1.2.0 and Yosys 0.69 map to Sky130 HD `sky130_fd_sc_hd`, corner `tt_025C_1v80` (1.80 V, 25 °C). OpenSTA 2.2.0 measures slack. The clock port is `clock`, period 20 ns, uncertainty 1 ns. Yosys `abc -D 20000` is that period in picoseconds and is the mapper target. Hammer does not report slack.

Sky130 HD has no liberty match for the synchronous-reset flops, so a Yosys `dfflegalize` pass mapped those cells before the area and timing numbers below. Both checks report 0 problems. There are no memories in either netlist.

- INT `IssueUnitCollapsing_2` plus `IssueSlot_32`, 40 slots: 185,524 cells, area 1,363,881.82 µm², pre-layout slack −32.37 ns.
- FP `IssueUnitCollapsing` plus `IssueSlot`, 32 slots: 136,352 cells, area 1,017,363.23 µm², pre-layout slack −24.68 ns.

Reports are under `~/FIFOrder-project/synth/int` and `~/FIFOrder-project/synth/fp`.

## Not done

No hybrid scheduler, new counters, or new Chipyard configs exist.
Medium, Large, and Giga simulators have not been built here.
Timing is pre-layout and misses the 20 ns period. Power has not been measured.
