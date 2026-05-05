# CSC-3643 Single Cycle RISC-V Processor
Final project for CSC-364 Computer Architecture. 
A single-cycle CPU implementing a subset of the RISC-V RV32I instruction set, designed and simulated in Logisim.

---

## Overview

This CPU fetches, decodes, executes, and writes back results for every supported instruction in a single clock cycle. The design includes a complete datapath, immediate generator, ALU, control unit, and PC update logic.

---

## Supported Instructions

| Class | Instructions |
|---|---|
| ALU (reg-reg) | `add`, `sub`, `and`, `or`, `xor`, `slt` |
| ALU (immediate) | `addi`, `slti`, `andi`, `ori`, `xori` |
| Load/Store | `lw`, `sw` |
| Branch | `beq`, `bne` |
| Jump | `jal` |

---

## Datapath Components

- **PC register** — holds current instruction address, increments by 4
- **Instruction Memory** — read-only, word-aligned
- **Register File** — 32 × 32-bit registers, x0 hardwired to 0, two read ports, one write port
- **Immediate Generator** — reconstructs and sign-extends immediates for I, S, B, and J formats
- **ALU** — supports ADD, SUB, AND, OR, XOR, SLT, and equality comparison
- **Data Memory** — supports word-aligned `lw` and `sw`
- **PC Adders** — dedicated adders for PC+4 and PC+imm
- **Multiplexers** — steer ALU operands, write-back data, and next PC

---

## Control Signals

| Signal | Purpose |
|---|---|
| `RegWrite` | Enables write to destination register `rd` |
| `ALUSrc` | Selects ALU input B: `0` = rs2, `1` = immediate |
| `MemRead` | Enables data memory read (`lw`) |
| `MemWrite` | Enables data memory write (`sw`) |
| `WBSel` | Write-back source: `0` = ALU result, `1` = memory data, `2` = PC+4 |
| `Branch` | Indicates a conditional branch |
| `Jump` | Indicates `jal` |
| `ALUCtrl` | Encodes ALU operation |
| `ImmSel` | Selects immediate format for ImmGen (I, S, B, J) |

---

## Test Programs

### Program A — ALU & Immediate Operations
```
addi x1, x0, 10
addi x2, x0, 3
add  x3, x1, x2    # x3 = 13
sub  x4, x1, x2    # x4 = 7
andi x5, x1, 6     # x5 = 2
ori  x6, x2, 8     # x6 = 11
xori x7, x1, 15    # x7 = 5
slti x8, x2, 4     # x8 = 1
```

### Program B — Load & Store
```
addi x10, x0, 0x20
addi x11, x0, 99
sw   x11, 0(x10)   # MEM[0x20] = 99
lw   x12, 0(x10)   # x12 = 99
```

### Program C — Branches & Jump
```
addi x1, x0, 5
addi x2, x0, 5
beq  x1, x2, L1    # taken — x3 = 222
addi x3, x0, 111   # skipped
L1: addi x3, x0, 222
bne  x1, x2, L2    # not taken — x4 = 7
addi x4, x0, 7
L2: jal x5, L3     # x5 = return address
addi x6, x0, 9     # skipped
L3: addi x6, x0, 10  # x6 = 10
```

---

## Tools

- **Logisim Evolution v4.1.0**
- [Logisim GitHub](https://github.com/logisim-evolution/logisim-evolution)

---

## Notes

- All registers are 32-bit; x0 is hardwired to 0
- Instruction addresses are byte-addressed; PC increments by 4
- Data memory accesses are word-aligned
- `jal` takes PC redirect priority over branch logic
- Branch condition uses the ALU Zero flag or a dedicated comparator
