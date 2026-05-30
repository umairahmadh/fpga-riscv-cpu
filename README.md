# 04 · RISC-V CPU ⭐

**Part of [FPGA Journey](https://github.com/umairahmadh/fpga-journey)**

> The flagship project: a pipelined, verified RV32I RISC-V core that runs real compiled programs. This is the project that makes the portfolio real.

Building a CPU from scratch forces you to understand every layer of the computing stack simultaneously — ISA, microarchitecture, hazards, timing, and verification. There is no shortcut and that is the point.

---

## 🎯 What's Built Here

| Stage | Description |
|-------|-------------|
| **v1 — Single-cycle** | Every instruction in one clock cycle. Correct, slow, simple. |
| **v2 — Pipelined** | 5-stage pipeline: IF → ID → EX → MEM → WB |
| **v3 — Hazard handling** | Data forwarding, load-use stall, branch flush |
| **v4 — Verification** | Full riscv-tests suite, comparison against Spike ISA sim |

The pipeline is the deliverable. v1 is scaffolding.

---

## 📁 Structure

```
fpga-04-riscv-cpu/
├── docs/
│   ├── block_diagram_single_cycle.png
│   ├── block_diagram_pipeline.png
│   ├── hazard_unit.png
│   └── design_notes.md         # Why you made specific microarch decisions
├── rtl/
│   ├── single_cycle/
│   │   ├── rv32i_sc_top.sv
│   │   ├── alu.sv
│   │   ├── register_file.sv
│   │   ├── control_unit.sv
│   │   ├── imm_extend.sv
│   │   └── data_mem.sv
│   └── pipeline/
│       ├── rv32i_top.sv        # Top-level connecting all stages
│       ├── fetch.sv            # IF: PC register, instruction memory
│       ├── decode.sv           # ID: register read, control signals, imm
│       ├── execute.sv          # EX: ALU, branch target, jump
│       ├── memory.sv           # MEM: data memory read/write
│       ├── writeback.sv        # WB: mux for write-back source
│       ├── hazard_unit.sv      # Forwarding + stall + flush logic
│       ├── pipeline_regs/      # IF/ID, ID/EX, EX/MEM, MEM/WB registers
│       ├── alu.sv
│       ├── register_file.sv    # 32 × 32-bit, x0 hardwired to 0
│       ├── control_unit.sv
│       └── imm_extend.sv
├── tb/
│   ├── rv32i_sc_tb.sv          # Single-cycle: instruction-by-instruction
│   ├── rv32i_pipeline_tb.sv    # Pipeline: program-level tests
│   └── cocotb/
│       ├── test_alu.py
│       ├── test_register_file.py
│       └── test_cpu_programs.py  # Load ELF, run, compare to Spike
├── programs/
│   ├── asm/
│   │   ├── fibonacci.s
│   │   ├── bubble_sort.s
│   │   └── sieve.s
│   └── c/
│       ├── hello.c             # Prints via memory-mapped UART
│       └── Makefile            # riscv32-unknown-elf-gcc + objcopy to .hex
├── riscv-tests/                # git submodule: riscv/riscv-tests
├── scripts/
│   ├── run_riscv_tests.sh      # Run the official test suite
│   └── spike_compare.py        # Run both Spike + RTL, diff register state
├── sim/
│   └── waveforms/
├── Makefile
└── README.md
```

---

## ✅ Build Checklist

**Single-cycle (v1)**
- [ ] ALU: all R-type, I-type, shift operations
- [ ] Load/store: LW, SW (word only first)
- [ ] Branches: BEQ, BNE, BLT, BGE, BLTU, BGEU
- [ ] Jumps: JAL, JALR
- [ ] Upper immediates: LUI, AUIPC
- [ ] Passes handwritten assembly programs

**Pipeline (v2)**
- [ ] IF/ID → ID/EX → EX/MEM → MEM/WB registers
- [ ] Correct output on no-hazard programs

**Hazard handling (v3)**
- [ ] Data forwarding: EX→EX and MEM→EX paths
- [ ] Load-use hazard stall (1 cycle)
- [ ] Branch: flush (predict not-taken), no branch delay slot
- [ ] Passes all programs that v1 passed

**Verification (v4)**
- [ ] Passes `riscv-tests` rv32ui suite (all RV32I instructions)
- [ ] Matches Spike register state after every instruction on ≥3 programs
- [ ] Byte and halfword loads/stores (LB, LH, LBU, LHU, SB, SH)
- [ ] All 32 extension instructions (if extending to RV32IM)

---

## 🛠️ Tools

| Tool | Purpose |
|------|---------|
| Verilator | Fast simulation of the full pipeline |
| cocotb + riscv-tests | Automated instruction-level testing |
| Spike (riscv-isa-sim) | Golden reference ISA simulator |
| riscv-gnu-toolchain | Compile C and assembly to RV32I |
| GTKWave | Pipeline waveform debugging |
| SymbiYosys | Formal: prove register file never reads x0 as non-zero |

---

## 🔑 Hardest Parts (and where to focus)

**Hazard detection logic** is where most people stall. The forwarding unit needs to correctly identify when EX or MEM stage results should bypass the register file, and the load-use stall must inject a bubble without corrupting the pipeline. Draw the forwarding paths on paper before writing a line of HDL.

**Branch handling:** the simplest correct approach is predict-not-taken, flush on taken. Implement this first. Understand it fully. Only add a branch predictor if you want it as a research extension.

**Verification matters more than optimization.** A 5-stage pipeline that passes the full `riscv-tests` suite is massively more impressive than a "10-stage pipeline" that can't prove correctness.

---

## 📖 Resources

- *Computer Organization and Design: RISC-V edition* (Patterson & Hennessy) — chapters 4–5 are your bible
- [riscv-tests](https://github.com/riscv-software-src/riscv-tests) — the official test suite
- [Spike ISA simulator](https://github.com/riscv-software-src/riscv-isa-sim)
- [picorv32](https://github.com/YosysHQ/picorv32) — study it, don't copy it
- [ZipCPU blog — CPU design](https://zipcpu.com/blog/2019/02/09/cpu-bkground.html)

---

**← Previous:** [03 · Peripherals](https://github.com/umairahmadh/fpga-03-peripherals) &nbsp;|&nbsp; **Next →** [05 · SoC](https://github.com/umairahmadh/fpga-05-soc)
