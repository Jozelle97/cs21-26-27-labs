---
title: RISC-V Single Cycle Processor
---

<h2 align="center"> CS 21 26.1 Laboratory Exercise 4 </h2>
<h1 align="center"> RISC-V Single Cycle Processor </h1>

## Overview

The RISC-V single-cycle processor is a sequential circuit made up of registers, memory, multiplexers, and other combinational components. In the lecture, we have derived the Single Cycle Processor Datapath. In this lab activity, you will be using Logisim to modify an existing single-cycle processor implementation to support other kinds of instructions.

## General Instructions

For this laboratory activity, you are to work on:

1. **Two Checkpoint Items** (due by the end of the lab period)
2. **One Take-Home Item** (due before the next lab meeting)

Save your outputs as `lab04_item1.circ`, `lab04_item2.circ`. All files must must be submitted via Google Classroom.

!!! warning "Important Reminder"
    You **must finish and submit** your working checkpoint outputs and **show them to the lab instructor before the end of the lab meeting**. 

- **Lab 5 Logisim Template:** [https://drive.google.com/file/d/1-WUrZY4d_uMD2QblyLCW-kN8QcwqnL-S/view?usp=sharing](https://drive.google.com/file/d/1-WUrZY4d_uMD2QblyLCW-kN8QcwqnL-S/view?usp=sharing)


## Guided Walkthrough

### Overview

A [Logisim-Evolution](https://github.com/UPD-DCS/logisim-evolution) template for a RISC-V single-cycle processor can be downloaded via the link in the previous section. This processor implementation supports the following instructions:

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

Understand the single-cycle processor template. Note that the ALU is _incomplete_, causing R-type and I-type instructions not listed above to yield incorrect results.

### Walkthrough #1: Running a single instruction

Suppose we want the processor to execute the following instruction:

```mips
addi  x1, zero, 21
```

To do this, its machine instruction equivalent should be loaded into _instruction memory_.

!!! example "Do this now!"

    Convert `addi x1, zero, 21` into its machine instruction equivalent in hexadecimal. You may use Ripes to do this quickly.

Select the hand tool _(`Ctrl-1` or `⌘-1`)_ and click the `instmem` component laid out in the circuit once for a pink circle and magnifying glass icon to appear at the center of the component.

Next, double-click the magnifying glass icon to enter into the implementation of the `instmem` subcircuit. You may need to scroll up to find the _ROM component_.

Use the hand tool to click the first top-left `00000000` word _(address `0`)_ on the ROM component–this should have the component circled in pink with the word boxed in red.

Typing hexadecimal symbols here will change the value boxed in red. Type the hexadecimal representation of `addi x1, zero, 21` to make address `0` contain the said instruction.

!!! tip "Tip: Alternative method"

    Copy-pasting values can be done by instead right-clicking the ROM component, selecting `Edit Contents`, selecting a box to fill up, then hitting `Ctrl-v` or `⌘-v`.

Double-click `main` under either the `Design` or `Simulate` tab _(or hit `Ctrl-Left key` or `⌘-Left key`)_ to return to the main circuit.

!!! example "Do this now!"

    Verify that the entered machine instruction is shown as the output of the `instmem` component and is shown as the value of the `Instr` probe.

Note that the instruction `addi x1, zero, 21` under `PC` will only take effect after the next triggering edge. Use the `State` tab to verify that `x1` still contains the value `0`.

Trace the flow of bits across the circuit and verify that the value `21` will be written to `x1` when the register file is triggered.

To make the clock signal tick _(i.e., make one full cycle)_, hit `Ctrl-T` or `⌘-T` twice _(or run `Simulate > Manual Tick Full Cycle` once)_.

Make the clock tick once, then use the `State` tab to verify that `x1` now contains the value `21` _(`0x15`)_.

Look for the `PC` register laid out in the circuit and verify that its current value is now `4` with the current instruction being `00000000`.

Hit `Ctrl-r` or `⌘-r` _(or run the `Simulate > Reset Simulation` menu item once)_ to reset all non-ROM data back to zero. Verify that `PC` is once again `0` and that all registers are zeroed out.

!!! tip "Reset Simulation"

    The `Reset Simulation` menu option of Logisim will zero-out all register and main memory values, but __not__ ROM values _(i.e., instruction memory)_.

    This convenience allows you to perform multiple reruns of the same program with all registers and main memory values zeroed out without having to reload the same set of machine instructions into the ROM component over and over.


### Walkthrough #2: Running an entire program

!!! example "Do this now!"

    Use Ripes to assemble the instructions below into their hexadecimal equivalents.

```mips
.text
       addi  sp, zero, 0xff0
       add   gp, zero, zero
       add   t0, zero, zero
       addi  t1, zero, 10
       addi  sp, sp, -32
loop:  beq   t0, t1, done
       lw    t2, 4(sp)
       addi  t2, t2, 10
       sw    t2, 4(sp)
       lw    t3, 8(sp)
       addi  t3, t3, 100
       sw    t3, 8(sp)
       addi  t0, t0, 1
       jal   zero, loop
done:  sub   t4, sp, zero
       not   t4, t4
```
!!! warning "Ripes version"

    If the locally installed Ripes on your TL machine is unable to assemble `addi sp, zero, 0xff0`, kindly switch to [https://ripes.upd-dcs.work](https://ripes.upd-dcs.work).

To easily load several instructions into the instruction memory component, create a `.txt` file as follows:

```txt
v3.0 hex words
<eight-hex symbol instruction 1>
<eight-hex symbol instruction 2>
...
<eight-hex symbol instruction n>
```

!!! info "Memory image format"

    The header line `v3.0 hex words` should be written as is while the instructions in hex should __not__ have `0x`.

!!! tip "Tip: Quick conversion shell script"

    Supposing the output of the Ripes assembly is in `assembly.txt`, the following command automatically transforms the said output into its Logisim-ready form in `logisim.txt`:

    ```bash
    printf "v3.0 hex words\n%s" "$(cat assembly.txt | grep "^\s" | tr -s " " | cut -d " " -f 3)" > logisim.txt
    ```

Open the `instmem` subcircuit, right-click the ROM component, select `Load Image`, then select the newly created `.txt` file.

After verifying that the ROM component indeed contains the correct sequence of instructions, head back to the `main` circuit.

While you could manually tick the clock until the execution of the last instruction to run the loaded program, it is likely more convenient to make the clock tick automatically by hitting `Ctrl-g` or `⌘-g` _(or ticking `Simulate > Auto-Tick Enabled`)_.

!!! tip "Clock rate"

    You may adjust the frequency of clock ticks via `Simulate > Auto-Tick Frequency`.

Using this approach, `PC` will eventually point to locations outside the scope of the loaded program which contains the instruction `00000000` by default–you may take this to mean that the program has finished execution.

Reset the simulation state, then run the program until its completion. Take note of the register values in particular.

!!! tip "Tip: State tab"

    You may use the **State tab** located at the left side of the Logisim window to examine register values without having to open up the register file.

Run the same program **using Ripes** and take note of its register values.

Verify that all but one register is consistent between the two sets of register values–this is because `xori` is __not__ yet supported by the processor.

!!! question "Self-check"

    `xori` does **not** appear in the assembly version of the program, but why is it present in the assembled version?


### Walkthrough #3: Support for `xori`

Clear instruction memory and load `xori x5, zero, -1` into address `0`.

!!! tip "Clear instruction memory"

    You may select `Clear Contents` to zero-out the ROM component of `instmem`.

Notice that the `ALUControl` signal which controls the ALU does __not__ have a valid value.

!!! example "Do this now!"

    Zoom into the ALU component by double-clicking it with the hand tool, trace where exactly the ALU uses the `ALUControl` signal, and examine its purpose in the operation of the ALU.

    Ensure you understand why `ALUControl = 000` makes the ALU add, `001` makes it subtract, `010` makes it do `AND`, `011` makes it do `OR`, and `101` makes it do `SLT`.

The `XOR` operation is not yet supported by the ALU–there is no `XOR` gate in the ALU and it is not mapped to any `ALUControl` value.

!!! example "Do this now!"

    Select a free `ALUControl` value, then modify the ALU to make it perform `SrcA XOR SrcB` for that particular `ALUControl`.

The computation for the `ALUControl` signal itself should also be fixed.

!!! example "Do this now!"

    Return to the `main` circuit, trace where the `ALUControl` signal is being generated, and enter that subcircuit.

    Ensure you know how priority encoders work before moving forward.

Using the `ALUControl` value you selected earlier, create an `IsXor` tunnel in the subcircuit and connect it to the correct input port of the priority encoder.

Create another `IsXor` tunnel, then connect it to a combinational circuit that:

- Outputs `1` if `ALUOp == 10` _(base 2)_ and `funct3 == 100` _(base 2)_
- Outputs `0` otherwise

Verify that `ALUControl` now outputs a valid value for any `xori` instruction and that the resulting register values of the program loaded earlier are now consistent with those of the Ripes execution.


## HOPELEx Checkpoint

### Checkpoint Task: `auipc`

Modify the given Logisim circuit to support the U-type instruction `auipc`.

Ensure that all other instructions that were supported before by the template still work as intended.

You are allowed to use tunnels.

Save the resulting circuit as `lab05.circ`.

### Demo

Show that the following code:

```mips
main:
  auipc x31, 0xf0000
  addi x1, x0, 10

loop:
  auipc x30, 0x12345
  add x31, x31, x30
  addi x1, x1, -1
  beq x1, x0, done
  beq x0, x0, loop

done:
  add x0, x0, x0
```

results in the following register values _(with all unstated registers equal to `0x00000000`)_:

```
x30 = 0x12345008
x31 = 0xa60b2050
```


## TakeHOPE Problems

### Item 1: R-type instructions

Add microarchitecture support for the following instructions:

1. `sub`
1. `and`
1. `or`
1. `xor`
1. `sll`
1. `srl`
1. `sra`
1. `slt`
1. `sltu`

### Item 2: I-type instructions

Add microarchitecture support for the following instructions:

1. `andi`
1. `ori`
1. `xori`
1. `slli`
1. `srli`
1. `srai`
1. `slti`
1. `sltiu`

### Item 3: Branch instructions

Add microarchitecture support for the following instructions:

1. `bne`
1. `bge`
1. `blt`
1. `bgeu`
1. `bltu`

### Item 4: Other instructions

Add microarchitecture support for the following instructions:

1. `lui`
1. `lb`
1. `lbu`
1. `lh`
1. `lhu`
1. `sb`
1. `sh`

### Item 5: Pseudoinstructions

Turn the following pseudoinstructions into basic instructions and add microarchitecture support for them:

1. `not`
1. `ble`
1. `bleu`
1. `bgt`
1. `bgtu`
1. `beqz`
1. `bnez`
1. `blez`
1. `bgez`
1. `bltz`
1. `bgtz`