# CSC-3643 Single Cycle RISC-V Processor
Final project for CSC-364 Computer Architecture. 
A single-cycle CPU implementing a subset of the RISC-V RV32I instruction set, designed and simulated in Logisim.

---

## Overview

This processor fetches, decodes, executes, and writes back results for every supported instruction in a single clock cycle. The design includes a complete datapath, immediate generator, ALU, control unit, and PC update logic.

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

### Control Unit
Decodes the instruction opcode and converts it into the corresponding control signals based on the instruction type. It drives signals such as `RegisterWrite`, `ALUSrc`, `Read`, `Write`, `WBSel`, `Branch`, `Jump`, `ALUControl`, and `ImmSel` to ensure the rest of the datapath behaves correctly for each instruction class.

### Instruction Memory 
A read-only memory block that stores the program instructions. It is word-aligned, meaning each instruction occupies a 4-byte address. On every cycle, it outputs the 32-bit instruction at the current PC address.

### Register File
Contains 32 general-purpose 32-bit registers. Register `x0` is hardwired to 0 and cannot be overwritten. The register file supports two simultaneous read ports (for `rs1` and `rs2`) and one write port (for `rd`), all accessed within a single cycle.

### Immediate Generator
Reconstructs the immediate value from the instruction bits and sign-extends it to 32 bits. It handles all four immediate formats used in this design — I, S, B, and J — selecting the correct bit layout based on the `ImmSel` control signal.

### ALU
The Arithmetic Logic Unit performs all computations in the datapath. It supports ADD, SUB, AND, OR, XOR, and SLT operations, as well as a zero flag output for branch instructions. The operation performed is determined by the `ALUControl` signal from the ALU_Control block.

### ALU_Control
The logical unit that reads the ALU_op bits from the Control Unit, the funct7 bit, and the 3 funct3 bits from the Instruction Memory. It directs output to control based on these values, matching up the logic from the ALU. This unit forms instructions for the ALU from the table below: 

| ALU_op | Funct7 | Funct3 | Instruction |
|--------|--------|--------|-------------|
| 00     | X      | X      | `add (lw and sw)` |
| 01     | X      | X      | `sub (branch)` |
| 10     | 0      | 000    | `add`       |
| 10     | 1      | 000    | `sub`       |
| 10     | 0      | 111    | `AND`       |
| 10     | 0      | 110    | `OR`        |
| 10     | 0      | 100    | `XOR`       |
| 10     | 0      | 010    | `SLT`       |

### Data Memory (ByteAddrMemory and MemoryLogic)
A read/write memory block used by `lw` and `sw` instructions. All accesses are word-aligned. `Read` enables a load and routes the result to the write-back mux, while `Write` enables a store from `rs2` into the computed address.

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

Test programs are padded to word align instructions in instruction memory. This is intentional to ensure correct logic for PC incrementation, jumps, and branches.

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
- Branch condition uses the ALU Zero flag instead of a dedicated comparator
