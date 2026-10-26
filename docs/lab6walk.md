---
title: CS 21 26.1 Lab 6 Walkthrough
---

<h2 align="center"> CS 21 26.1 Lab 6 Walkthrough </h2>
<h1 align="center"> Pipelined Processor in Logisim </h1>

## Overview

In the lecture, we introduced the RISC-V Pipelined Processor.  In this lab activity, you will be using Logisim to modify an existing pipelined RISC-V processor implementation to support other kinds of instructions.

Your main task for Lab 6 (details to follow) is to implement the forwarding mechanism by creating the hazard unit for the given Pipeline Processor implementation in Logisim. But before that, this walkthrough will familiarize you with the given Pipeline Processor Logisim implementation.

## Relevant Links

- **Ripes Fork:** [https://ripes.upd-dcs.work](https://ripes.upd-dcs.work)
- **Lab 6 Logisim Template:** [https://drive.google.com/file/d/1YIG0GWN8k4u7n-qk0t6r8i7NrUMPpal9/view?usp=sharing](https://drive.google.com/file/d/1YIG0GWN8k4u7n-qk0t6r8i7NrUMPpal9/view?usp=sharing)
- **Instruction Memory Image File:** [Instruction memory image](https://drive.google.com/file/d/1dz_vNA9nIndfM-17Q-lqdOc_KuWycT-2/view?usp=sharing)
- **Data Memory Image File:** [Data memory image](https://drive.google.com/file/d/1QqreTP0v39KNsuxY7TQCXeuB4oGMQKib/view?usp=sharing)

## General Instructions
1. Work on this walkthrough and write your answers to **all self-check QUESTIONS** in a document. Save this as `lab06-lastname-self-check.pdf`. Submit this file **before 11:59 PM on October 27, 2026** in Google Classroom. You may work or discuss among your classmates, but you should write your asnwers to the questions and submit your output individually. This output is worth **30/100** points of your Lab 6 score.

2. The main Lab 6 activity worth **70/100** points will be released within this week and will be due a week after.


Download the Lab 6 Logisim Template using the link above. This processor implementation supports the following instructions:

- `lw`
- `sw`
- `add`
- `sub`
- `and`
- `or`
- `slt`
- `addi`
- `beq`
- `jal`

!!! info "Recommended zoom level"

    The given template is meant to be viewed at a zoom level of **x0.9**.

The bottom part of the template consists of the pipelined datapath and control while the top part provides a view of important values arranged by pipeline stage.

Save a working copy of the circuit as `lab06.circ` before starting. Do not modify the datapath during this activity.

!!! info "No forwarding required for this first trace"

    These three instructions have no read-after-write dependency on one another. They can execute correctly in the original template with all manual stall and flush controls set to `0`.

### Walkthrough #1: Familiarizing Yourself with the Pipeline Logisim Implementation
This walkthrough is a guided activity that will help you become familiar and comfortable with using the Logisim implementation of a RISC-V 5-stage Pipeline processor. 

Open the Logisim circuit, select `main`, and disable automatic clock ticking. Set the zoom to approximately **x0.9**.

Identify the five stages in the lower datapath and their corresponding signal displays in the upper portion of the circuit.

| Suffix | Stage | Main purpose |
| :-: | - | - |
| `F` | Fetch | Read an instruction using the program counter |
| `D` | Decode | Decode the instruction and read registers |
| `E` | Execute | Perform arithmetic or calculate an address |
| `M` | Memory | Read or write data memory |
| `W` | Writeback | Select a result for writing to a register |

For example, `RD1E` is a register operand available in Execute, while `ALUResultM` is an ALU result carried into Memory.

In this walkthrough, we will simulate this RISC-V code:

```mips
add  s2, s0, s1
lw   s4, 4(s3)
addi s3, s1, 2
```

Download the instruction memory image from the link above.

Take note of the following initial register values:

| Register | Register number | Decimal | Hexadecimal |
| :-: | :-: | :-: | :-: |
| `s0` | `x8` | 10 | `0000000a` |
| `s1` | `x9` | 7 | `00000007` |
| `s2` | `x18` | 0 | `00000000` |
| `s3` | `x19` | 16 | `00000010` |
| `s4` | `x20` | 0 | `00000000` |

Download the data memory image from the link above. The data memory image given above  places the value **42** (`0x2a`) at **byte address 20** (`0x14`).

Before running the program, make sure that you know what should be the final state of the register file after the execution of the entire program. 

#### Step 1: Load the Instruction

1. Before you start, make sure that the circuit is at the Reset state. Select **Simulate > Reset Simulation**.

2. Using the **Poke** tool, ensure that the four manual controls are set to `0`:

```text
StallF = 0
StallD = 0
FlushD = 0
FlushE = 0
```
3. Open the **Instruction Memory** labeled `InstMem` and load the instruction to the ROM using the image file `lab6-program.txt`. After this, return to `main`.

The first three instruction words are:

| Instruction byte address | ROM word index | Machine code | Instruction |
| :-: | :-: | :-: | - |
| `0x00` | `0` | `00940933` | `add s2, s0, s1` |
| `0x04` | `1` | `0049aa03` | `lw s4, 4(s3)` |
| `0x08` | `2` | `00248993` | `addi s3, s1, 2` |

The image also contains trailing `nop` instructions (`addi zero, zero, 0`) so that you can continue stepping while the three instructions finish.

!!! info "Byte addresses and word indices"

    The PC increases by **4 bytes** for each sequential instruction. The instruction ROM stores 32-bit words, so instruction byte addresses `0`, `4`, and `8` correspond to ROM word indices `0`, `1`, and `2`.

#### Step 2: Load data memory

Enter the **data memory** labeled `DataMem` and load `lab6-data.txt` to the RAM. Verify that RAM word index `5` contains `0000002a`.

In this circuit, the RAM index is obtained by removing the two lowest byte-address bits. Therefore:

```text
Byte address 20 / 4 bytes per word = RAM word index 5
```

#### Step 3: Initialize registers

Enter the **register-file instance used by `main`**. With the Poke tool, select and edit the stored values of the following registers:

```text
x8  = 0000000a
x9  = 00000007
x19 = 00000010
```

Verify that `x18` and `x20` contain zero, then return to `main`.

!!! tip "Edit the live instance"

    Enter the register-file instance from `main` when changing simulation state. Opening only the subcircuit definition from the project list may show a separate simulation context.

!!! warning "Reset before initialization"

    Do not reset after entering the register values. If you reset again, reinitialize the registers and verify the memory contents before continuing.

#### Step 4: Practice Clock Stepping

Before ticking the clock, locate `PCF`, `PC4F`, and `InstrF`. Verify the following:

```text
PCF    = 00000000
PC4F   = 00000004
InstrF = 00940933
```

1. Select **Simulate > Manual Tick Half Cycle** once.
2. Observe the clock level and pipeline displays.
3. Select **Manual Tick Half Cycle** again.
4. Observe which pipeline values changed at each clock transition.

A half-cycle toggles the clock once. A full cycle includes both transitions. Pipeline registers update on their triggering edge.

For the rest of the main trace, use **Manual Tick Full Cycle** consistently. Count the two half-cycle ticks you just performed as the first full cycle.

Track the first instruction using its `PC + 4` value:

```text
PC4D → PC4E → PC4M → PC4W
```

For the first instruction, that value remains `00000004` as it moves through the pipeline.

**Self-Check:** 
At this point, make sure that you observe the following:

- The PC normally advances by `4` bytes.
- An instruction's PC-related value moves one stage on each advancing pipeline edge.
- Several instructions occupy different stages simultaneously.
- Combinational outputs can settle after an input changes without another clock tick.
- An ALU result appearing does not mean the destination register has already changed.

To check your understanding, answer this: If `PC4E = 00000008`, which instruction is in Execute?

### Step 5: Trace the Instructions
Use this table to track the instructions. **Step 0** is the initial state before the first rising edge, with `add` already visible in Fetch. Each later row describes the state after another rising edge. Your earlier clock-stepping exercise has already advanced the simulation; do not restart the count accidentally.

| Step | Fetch | Decode | Execute | Memory | Writeback |
| :-: | :-: | :-: | :-: | :-: | :-: |
| 0 | `add` | — | — | — | — |
| 1 | `lw` | `add` | — | — | — |
| 2 | `addi` | `lw` | `add` | — | — |
| 3 | `nop` | `addi` | `lw` | `add` | — |
| 4 | `nop` | `nop` | `addi` | `lw` | `add` |
| 5 | `nop` | `nop` | `nop` | `addi` | `lw` |
| 6 | `nop` | `nop` | `nop` | `nop` | `addi` |

A dash means that none of the three target instructions occupies that stage.

#### Observation A: Arithmetic in Execute

Stop when `PC4E = 00000004`. Inspect:

| Signal | Expected value | Meaning |
| :-: | :-: | - |
| `RD1E` | `0000000a` | Value of `s0` |
| `RD2E` | `00000007` | Value of `s1` |
| `RdE` | 18 decimal | Destination register `s2` |
| `ALUSrcE` | `0` | Select the register operand |
| `ALUResultE` | `00000011` | Result: 17 decimal |

!!! tip "Check the display radix"

    Register numbers and register contents are different quantities. If `RdE` is displayed in hexadecimal, register number 18 appears as `12`. The result `00000011` is hexadecimal for decimal 17.

**Self-Check:** Why might `s2` still contain zero even though `ALUResultE` already displays 17?

#### Observation B: A load calculates an address

Advance until `PC4E = 00000008`. Inspect:

| Signal | Expected value |
| :-: | :-: |
| `RD1E` | `00000010` |
| `ImmExtE` | `00000004` |
| `ALUSrcE` | `1` |
| `ALUResultE` | `00000014` |

The ALU calculates:

```text
Address = old value of s3 + offset
        = 16 + 4
        = 20 decimal
        = 0x14
```

Advance one cycle so that the load enters Memory. Verify:

```text
ALUResultM = 00000014
ReadDataM  = 0000002a
MemWriteM  = 0
```

**Self-Check:** Should `s4` receive `0x14` or `0x2a`? What does each value represent?

#### Observation C: Immediate arithmetic overlaps the load

While the load is in Memory, `addi` is in Execute. Inspect:

```text
PC4E       = 0000000c
RD1E       = 00000007
ImmExtE    = 00000002
ALUSrcE    = 1
ALUResultE = 00000009
```

**Self-Check:** Why does the ALU use the immediate value 2 instead of `RD2E`?

The displayed `RD2E` is not a meaningful source operand for this instruction because the immediate input is selected.

#### Observation D: Writeback and architectural state

As each instruction reaches Writeback, inspect `RegWriteW`, `RdW`, and `ResultW`.

| Instruction | `RegWriteW` | Destination, decimal | `ResultW` |
| - | :-: | :-: | :-: |
| `add` | `1` | 18 (`s2`) | `00000011` |
| `lw` | `1` | 20 (`s4`) | `0000002a` |
| `addi` | `1` | 19 (`s3`) | `00000009` |

!!! info "Observe both clock edges"

    The supplied circuit inverts the clock entering the register file. Register-file writes therefore occur on the opposite clock edge from the main pipeline-register updates. Observe two half-cycle ticks during Writeback to distinguish a value being present in the W stage from the destination register receiving it.

Continue until you have completed **seven full clock cycles from the initial reset**, allowing all three writes to complete. Inspect the **State** tab or the live register-file instance:

| Register | Expected decimal value | Expected hexadecimal value | Your observation |
| :-: | :-: | :-: | :-: |
| `s2` / `x18` | 17 | `00000011` | __________ |
| `s4` / `x20` | 42 | `0000002a` | __________ |
| `s3` / `x19` | 9 | `00000009` | __________ |


### Walkthrough #2: Stalls and Flushes
The template includes a manual control pins for the following control signals:

1. `StallF`
2. `StallD`
3. `FlushD`
4. `FlushE`

Locate this in the upper left portion of the `main` circuit just beside the `Ticks` connected to a clock. Use these manual control pins for the following guided activity. Make sure to reinitialize registers and verify or reload the memory images every time you reset the simulation.

#### Experiment A: Stall Fetch only

1. Reset and initialize the processor.
2. Advance until the load is in Decode and `addi` is in Fetch.
3. Record `PCF` and `PC4D`.
4. Set only `StallF = 1`.
5. Advance one full cycle.

**Observe the following:**

- `PCF` remains unchanged.
- Decode still updates because `StallD = 0`.
- The instruction held in Fetch can enter Decode.

**Self-Check:** Does stalling Fetch freeze the entire processor?

!!! warning "This is a control experiment"

    Stalling Fetch alone can cause an instruction to enter Decode more than once. Do not use this experimental run to validate the program's final results.

#### Experiment B: Hold Fetch and Decode and insert a bubble

1. Reset and initialize again. Advance until the `load` isntruction is in Decode and `addi` is in Fetch. Set:
```text
StallF = 1
StallD = 1
FlushD = 0
FlushE = 1
```
2. Advance one full cycle.

    **Observe the following:**
    - Fetch remains on the same instruction.
    - Decode retains the load.
    - Execute is cleared to a bubble.
    - The older instruction advances into Memory.

3. Return all four controls to `0`, then advance once more. Verify that the held load advances into Execute.

!!! info "Connection to the checkpoint"

    These are the control actions used for a load-use stall. The three instructions in this activity do not require that stall; you are applying it manually to learn what the controls do.

#### Experiment C: Flush Decode

Set only `FlushD = 1`, advance one cycle, then return it to `0`.

**Observe:** The Fetch/Decode register is cleared, including its stored instruction and PC-related values. The PC continues advancing because `StallF = 0`.

**Self-Check:** How does this differ from setting `StallD = 1`?

Before proceeding to the Checkpoint Tasks for Lab 6, ensure that you can:

- Locate the Fetch, Decode, Execute, Memory, and Writeback displays
- Advance the clock without automatic ticking
- Follow an instruction using its PC-related values
- Explain why a load's address differs from its loaded value
- Distinguish a pending writeback from an updated register
- Demonstrate holding a pipeline register and clearing it to a bubble
