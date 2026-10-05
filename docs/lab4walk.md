---
title: CS 21 26.1 Lab 4 Walkthrough
---

<h2 align="center"> CS 21 26.1 Lab 4 Walkthrough </h2>
<h1 align="center"> Partial Single Cycle Processor in Logisim </h1>

### Overview
The RISC-V single-cycle processor is a sequential circuit made up of registers, memory, multiplexers, and other combinational components. In the lecture, we have derived the Single Cycle Processor Datapath. In this lab activity, you will be using Logisim to modify an existing single-cycle processor implementation to support other kinds of instructions.

!!! warning "Ripes version"

    For uniformity, use the [https://ripes.upd-dcs.work](https://ripes.upd-dcs.work) instead of a locally installed RIPES.

- In this lab and the following lab activities, we will be using Logisim. Install [Logisim-Evolution](https://github.com/UPD-DCS/logisim-evolution) in your computers.
- Download the **Lab 4 Single Cycle Processor Template** here: [https://drive.google.com/file/d/1-WUrZY4d_uMD2QblyLCW-kN8QcwqnL-S/view?usp=sharing](https://drive.google.com/file/d/1-WUrZY4d_uMD2QblyLCW-kN8QcwqnL-S/view?usp=sharing)

The RISC-V single cycle processor in the provided logisim template supports the following instructions:

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

Understand the single-cycle processor template. Note that the ALU is **incomplete**, causing R-type and I-type instructions not listed above to yield incorrect results.

### Walkthrough #1: Running a single instruction

Suppose we want the processor to execute the following instruction:

```mips
addi  x1, zero, 21
```

To do this, its machine instruction equivalent should be loaded into the **instruction memory**.

!!! example "Do this now!"

    Convert `addi x1, zero, 21` into its machine instruction equivalent in hexapkill -f "mkdocs serve"decimal. You may use Ripes to do this quickly.

Follow these steps to run `addi x1, zero, 21` in the processor:

1. Using the hand tool, double-click on the Instruction Memory component. This should allow you to enter and view the implementation of the `instmem` subcircuit. In this circuit, you should find an ROM component. 

2. Using the hand tool, click the first top-left `00000000` word _(address `0`)_ on the ROM component. This should have the component circled in pink with the word boxed in red.

3. Typing hexadecimal symbols here will change the value boxed in red. Type the hexadecimal representation of `addi x1, zero, 21` to make address `0` contain the said instruction.

4. Return to the main circuit by double-clicking the `scp`.

5. Verify that the entered machine instruction is shown as the output of the `instmem` component and is shown as the value of the `Instr` probe.

6. Observe the changes in the datapath values. Trace the flow of the data bits and control bits across the circuit and verify that the datapath is executing the correct instruction.

!!! tip "Tip: Alternative method"

    Copy-pasting values can be done by instead right-clicking the ROM component, selecting `Edit Contents`, selecting a box to fill up, then hitting `Ctrl-v` or `⌘-v`.

Note that the instruction `addi x1, zero, 21` under `PC` will only take effect after the next triggering edge. Use the `State` tab (lower left panel in Logism, beside `Properties`) to verify that `x1` still contains the value `0`.

Here's how you make the changes in the register file take effect:

1. To make the clock signal tick _(i.e., make one full cycle)_, hit `Ctrl-T` or `⌘-T` twice _(or run `Simulate > Manual Tick Full Cycle` once)_. Make the clock tick once, then use the `State` tab to verify that `x1` now contains the value `21` _(`0x15`)_.

2. Look for the `PC` register laid out in the circuit and verify that its current value is now `4` with the current instruction being `00000000`. Can you reason out why the current instruction is `00000000`?

3. Hit `Ctrl-r` or `⌘-r` _(or run the `Simulate > Reset Simulation` menu item once)_ to reset all non-ROM data back to zero. Verify that `PC` is once again `0` and that all registers are zeroed out.

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

To easily load several instructions into the instruction memory component, copy the machine translated version of your code from the RIPES simulator and save it to a `.txt`. For this example, the following contents should be saved into an `assembly.txt` file. Note that the extra lines containing the labels are omitted.

```txt
v3.0 hex words
    0:        ff000113        addi x2 x0 -16
    4:        000001b3        add x3 x0 x0
    8:        000002b3        add x5 x0 x0
    c:        00a00313        addi x6 x0 10
    10:        fe010113        addi x2 x2 -32
    14:        02628263        beq x5 x6 36 <done>
    18:        00412383        lw x7 4 x2
    1c:        00a38393        addi x7 x7 10
    20:        00712223        sw x7 4 x2
    24:        00812e03        lw x28 8 x2
    28:        064e0e13        addi x28 x28 100
    2c:        01c12423        sw x28 8 x2
    30:        00128293        addi x5 x5 1
    34:        fe1ff06f        jal x0 -32 <loop>
    38:        40010eb3        sub x29 x2 x0
    3c:        fffece93        xori x29 x29 -1
```

!!! info "Memory image format"

    The header line `v3.0 hex words` should be written as is. Note that the instructions in hex should __not__ have `0x`.

!!! tip "Tip: Quick conversion shell script"

    Supposing the output of the Ripes assembly is in `assembly.txt`, the following command automatically transforms the said output into its Logisim-ready form in `logisim.txt`:

    ```bash
    awk '
    BEGIN { print "v3.0 hex words" }
    $1 ~ /^[[:xdigit:]]+:$/ &&
    $2 ~ /^[[:xdigit:]]+$/ &&
    length($2) == 8 { print $2 }
    ' assembly.txt > logisim.txt
    ```
Verify that the `logisim.txt` file should convert the contents of the `assembly.txt` file to a text file that contains only the machine code instruction in each line without the `0x`. A general format will be something like this:

```txt
v3.0 hex words
ff000113
000001b3
```

Go back to your Single Cycle Logisim circuit and follow these instructions to simulate all lines of code:

1. Open the `instmem` subcircuit, right-click the ROM component, select `Load Image`, then select the newly created `logisim.txt` file.

2. Verify that the ROM component indeed contains the correct sequence of instructions and head back to the main `scp` circuit.

3. Make sure that your `PC = 00000000` and simulate the program. While you could manually tick the clock until the execution of the last instruction of the loaded program, it is likely more convenient to make the clock tick automatically by hitting `Ctrl-g` or `⌘-g` _(or ticking `Simulate > Auto-Tick Enabled`)_. Using this approach, `PC` will eventually point to locations outside the scope of the loaded program which contains the instruction `00000000` by default–you may take this to mean that the program has finished execution.

    !!! tip "Clock rate"

        You may adjust the frequency of clock ticks via `Simulate > Auto-Tick Frequency`.

4. At any time, feel free to reset the simulation state, then run the program until its completion or manually tick the clock every cycle. Verify that the register values correctly change accordingly.

    !!! tip "Tip: State tab"
        You may use the **State tab** located at the left side of the Logisim window to examine register values without having to open up the register file.

5. Run the same program **using Ripes** and take note of its register values. Verify that all but one register is consistent between the two sets of register values–this is because `xori` is __not__ yet supported by the processor.



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

   Choose an unused (free) `ALUControl` value, then modify the ALU to make it perform `SrcA XOR SrcB` for that particular `ALUControl`.

The computation for the `ALUControl` signal itself should also be fixed.

!!! example "Do this now!"

    Return to the `main` circuit, trace where the `ALUControl` signal is being generated, and enter that subcircuit.

    Ensure you know how priority encoders work before moving forward.

Using the `ALUControl` value you selected earlier, create an `IsXor` tunnel in the subcircuit and connect it to the correct input port of the priority encoder.

Create another `IsXor` tunnel, then connect it to a combinational circuit that:

- Outputs `1` if `ALUOp == 10` _(base 2)_ and `funct3 == 100` _(base 2)_
- Outputs `0` otherwise

Verify that `ALUControl` now outputs a valid value for any `xori` instruction and that the resulting register values of the program loaded earlier are now consistent with those of the Ripes execution.

