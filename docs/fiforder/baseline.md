# FIFOrder baseline

## Environment
- AWS c5.2xlarge, x86_64, Amazon Linux 2023
- 16 GB RAM, 100 GiB root disk
- Project directory: ~/FIFOrder-project/chipyard
- Configuration: SmallBoomV3Config
- INT, FP, MEM issue queues: 8 entries each, issue width 1
- ROB: 32 entries

## Setup
Install Miniforge, activate its base environment, then run:

./build-setup.sh riscv-tools --use-lean-conda --skip-firesim --skip-marshal --skip-ctags --skip-precompile --skip-clean

## Build
cd ~/FIFOrder-project/chipyard
source ./env.sh
cd sims/verilator
make -j2 CONFIG=SmallBoomV3Config

## Verified tool versions
- Java: OpenJDK 20.0.2-internal
- Verilator: 5.022
- CIRCT: firtool-1.75.0
- RISC-V GCC: 16.1.0
- sbt: 1.8.2

## Test status
- Simulator build succeeded.
- rv64ui-p-simple smoke test passed with exit code 0.
- Full assembly suite was interrupted; full regression is not verified.
- No FIFOrder hardware modifications yet.

## Run the smoke test
From sims/verilator, after sourcing env.sh:

make CONFIG=SmallBoomV3Config run-binary BINARY="$RISCV/riscv64-unknown-elf/share/riscv-tests/isa/rv64ui-p-simple"

## Existing logs
Outside the repository, in ~/FIFOrder-project:
- setup-linux.log
- baseline-build.log
- baseline-asm-tests.log

## Outstanding setup issues
Inspect dirty/untracked content in constellation and toolchain submodules.
Investigate BramDwc symbolic-link warnings.
Verify BOOM's working branch before editing.
