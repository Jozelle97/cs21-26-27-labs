---
title: CS 21 26.1 Lab 7
---

<h2 align="center"> CS 21 26.1 Lab 7 </h2>
<h1 align="center"> Interfacing with Memory via LED displays in the RIPES Simulator </h1>

## Overview
In this laboratory activity, you will learn how to interface with the LED matrix in the RIPES simulator by writing RISC-V codes to control a **5x5 LED matrix** through memory-mapped I/O .

By the end of the activity, you should be able to write your own codes to change the values in the RIPES simulator memory to manipulate the lights in the simulator's built-in LED matrix.
This laboratory is meant to prepare you in accomplishing your Project 2 task which is a GUI implementation of your snake game in Project 1.

## General Instructions

For this laboratory activity, **you may work in pairs** to complete the following:

1. **2 Guided Walkthroughs** to learn LED addressing and keyboard-controlled movement.
2. **2 Checkpoint Items** to be demonstrated and submitted by the end of the lab period.


## Relevant Links
- **Extra Reference: Ripes Memory-Mapped I/O:** [Device setup and exported symbols](https://github.com/mortbopet/Ripes/blob/master/docs/mmio.md)
- **Optional Reference : Ripes Console Input Example:** [Reading from standard input in assembly](https://github.com/mortbopet/Ripes/blob/master/examples/assembly/consoleReading.s)

## Guided Walkthrough
### Setting up the LED matrix
1. Open RIPES and make sure that your processor setup uses a **Single-cycle processor**.
2. Open the **I/O** tab from the left-most panel. Select LED Matrix from the list of Devices. This should show a new window displaying a matrix of black circles (LEDs). 
3. Change the dimensions of the LED matrix to a 5x5 matrix by setting the  **Width = 5** and **Height = 5** at the left panel on your screen. Increase the LED size if needed.
4. Find the device's exported symbols in the **Exports** window. With one matrix, its base symbol should be `LED_MATRIX_0_BASE`. 
5. Inspect the **Register Map**. There should be 25 LED entries, starting at byte offsets 0 or `0x00`, 4 or `0x04`, 8 or `0x08`, and ending at 96 or `0x60`.

!!! warning "LED device values may not appear in the Memory viewer"

    The LED matrix is a **memory-mapped I/O device**. Its registers may not appear in Ripes' regular **Memory** viewer. If its base address is `0xF0000000`, navigating to that address may show nothing, even when the device is correctly configured and working.

    Find the actual base address in **I/O > LED Matrix > Exports**, and use the device's **register map** to identify each LED's offset. Do not assume that `0xF0000000` is always the base address. Observe the visible matrix after executing `sw`; to check a stored color numerically, execute `lw` from the same LED address and inspect the destination register.

### Understanding the LED addresses
The LED matrix is a **memory-mapped I/O device**. Ripes assigns it a range of addresses that your program can access using memory instructions. Writing a color to an address in this range changes the corresponding LED.

`LED_MATRIX_0_BASE` is the **base address**: the address of the first LED, at the upper-left corner `(row 0, column 0)`. The offsets in the register map are distances **in bytes from this base**, rather than complete addresses. Each LED occupies a four-byte slot, so consecutive LEDs have addresses four bytes apart.

For example, if the **Exports** window shows a base address of `0xFFFF0000`, the addresses are:

| LED position | Byte offset | Complete address |
|---|---|---|
| Upper-left `(0,0)` | `0x00` | `0xFFFF0000` |
| Next LED to the right `(0,1)` | `0x04` | `0xFFFF0004` |
| First LED in the next row `(1,0)` | `0x14` | `0xFFFF0014` |
| Lower-right `(4,4)` | `0x60` | `0xFFFF0060` |

The base above is an example; use the actual value shown in your Ripes session. The 25 LEDs occupy 100 bytes in total. The final LED's four-byte slot starts at `base + 0x60` and ends at `base + 0x63`.

### Lighting the Upper-left LED

Enter the complete program below in the Ripes assembly editor and run it once. Go back to your LED matrix and you should see one **green LED at the upper-left corner** and all other LEDs black.

```asm
.text
main:
    li s0, LED_MATRIX_0_BASE

    # Clear all 25 LEDs first.
    mv t0, s0
    li t1, 25
clear:
    sw zero, 0(t0)
    addi t0, t0, 4
    addi t1, t1, -1
    bnez t1, clear

    li t2, 0x0000FF00      # Green
before_light:
    sw t2, 0(s0)           # Upper-left LED
after_light:
    nop
    li a7, 10
    ecall
```

### Using Breakpoints to Observe the Behavior of a code one step at a time
A **breakpoint** tells RIPES to pause execution when it reaches a selected instruction. Note that the selected instruction has **not yet executed** when the processor stops there.

To add a **breakpoint** to an instruction of code, locate the corresponding assembled code (to the right of the editor) of the instruction and click on the **narrow breakpoint gutter** (blue vertical bar to the left of the line of code). Once clicked, you should see a red dot on the breakpoint gutter aligned with that line of code. Click it again to remove the breakpoint.

To understand the code, observe what happens after running the code until a few breakpoints:

1. Put breakpoints on the instructions labeled as `before_light` and `after_light`.
2. Click **Run**. RIPES should executes the initialization and `clear` loop, then pauses at the first breakpoint on `sw t2, 0(s0)`. 
3. Open the LED matrix and observe that the **upper-left LED should still be off**, because the `sw` has not executed yet.
4. Open the register display and inspect **`s0` (`x8`)** and **`t2` (`x7`)`s0` should contain the actual LED base address and `t2` should contain `0x0000FF00`, the green color value.
3. Calculate the destination of the store: `s0 + 0`. Find this address in the LED device's I/O register map. It is the upper-left LED. 
4. Click **Run** again and confirm that the current instruction has advanced to the `nop` at `after_light`. 
5. Look at the LED matrix and observe that the **the upper-left LED should now be green**, while all other LEDs remain off. 

### How the addresses map to LEDs
Coordinates of the LED matrix are referred to in this lab using `(row, column)` notation, starting at index zero. Rows increase downward; columns increase to the right. Let `BASE` be the LED base address of the LED matrix.

```
cell index  = row * 5 + column
LED address = BASE + 4 * (row * 5 + column)
```

Each LED occupies a four-byte slot. The color uses the lower 24 bits as `0x00RRGGBB`.

| Color | Word to store |
|---|---|
| Off/black | `0x00000000` |
| Red | `0x00FF0000` |
| Green | `0x0000FF00` |
| Blue | `0x000000FF` |
| Yellow | `0x00FFFF00` |

Here is the **byte offset** of every LED. Add `BASE` to get the actual address.

```
             column
            0   1   2   3   4
row 0       0   4   8  12  16
row 1      20  24  28  32  36
row 2      40  44  48  52  56
row 3      60  64  68  72  76
row 4      80  84  88  92  96
```

### Walkthrough #1: Move vertically, then draw a diagonal
#### Task A. Select the LED below
1. Save a copy of the program that lights up the upper-left LED.
2. Change only `sw t2, 0(s0)` to `sw t2, 20(s0)`.
3. Predict the LED position, reset, and run. The green LED should now be at `(1,0)`.
4. Change the offset to 40. Predict and test again. It should appear at `(2,0)`.

These changes select a different starting location. To make an LED **move during the same execution**, you must also erase its old location.

#### Task B. Move one LED down, then up
Now, use this assembly code:
```asm
.text
main:
    li s0, LED_MATRIX_0_BASE

    # Clear all 25 LEDs first.
    mv t0, s0
    li t1, 25
clear:
    sw zero, 0(t0)
    addi t0, t0, 4
    addi t1, t1, -1
    bnez t1, clear

    li t2, 0x0000FF00
    sw t2, 0(s0)          # Start at (0,0)
pause_top:
    nop

    sw zero, 0(s0)        # Erase old location
    sw t2, 20(s0)         # Draw one row below: (1,0)
pause_below:
    nop

    sw zero, 20(s0)       # Erase (1,0)
    sw t2, 0(s0)          # Draw one row above: back to (0,0)
pause_back:
    nop

    li a7, 10
    ecall
```

Set breakpoints at all three `pause_...` instructions. Run between them and observe the changes that happen in the LED matrix at every break point. Without breakpoints, the program may finish too quickly and you will not be able to observe the changes in the LED matrix.

##### Self-Check Task:
1. Modify this code so that the light start at `(2,2)`, move down to `(3,2)`, then return upward. Calculate the offsets yourself.
2. Temporarily remove one erase instruction. What stays lit? Restore the instruction afterward.

!!! tip "Moving one LED"

    Down adds 20 bytes; up subtracts 20 bytes, provided the destination stays inside the board. Clear the old LED before lighting the new one.

    7. Observe what happens to the LED and understand how the code works.

### Walkthrough #2: Move one LED right using "R"

In this walkthrough, an LED on the upper right corner is initially ON. Everytime a user inputs an **R** on the keyboard, the LED light moves one cell to the right. When it reaches the the upper-right corner, the next **R** returns it to the upper-left corner. The program keeps asking for user input and terminates the program when a letter **X** is entered.

1. Keep the LED matrix configured as 5x5.
2. Copy the RISC-V code for `Complete Program for Moving "R"` below.
3. Remove old breakpoints, reset, and click **Run**. One yellow LED should appear at the upper-left `(0,0)`.
4. Focus the Ripes console input, type **R**, and press **Enter**. The yellow LED should move to `(0,1)`.
5. Continue entering one **R** per line. The positions should be `(0,2)`, `(0,3)`, `(0,4)`, and then `(0,0)` again. The LED at `(0,4)` remains visible while the program waits for the next command.
6. Type **X** and press **Enter**. The program should terminate without moving or clearing the last LED.

!!! note "Console input"

    Submit one uppercase letter followed by Enter. This program reads a line into a RAM buffer using Ripes' `read` system call 63 and uses its first character. A line containing `RR` causes only one move. Lowercase letters, empty lines, and other commands cause no movement; the program asks again.

#### Complete Program for Moving "R"

```asm

.data
prompt: .string "\nEnter R to move right, or X to exit: "
buffer: .zero 64

.text
main:
    li s0, LED_MATRIX_0_BASE
    li s1, 0                 # Current column; row stays 0
    li s2, 0x00FFFF00        # Yellow

    # Clear the display.
    mv t0, s0
    li t1, 25
clear:
    sw zero, 0(t0)
    addi t0, t0, 4
    addi t1, t1, -1
    bnez t1, clear

    sw s2, 0(s0)            # Start at upper-left (0,0)

read_command:
    la a0, prompt
    li a7, 4                 # Print prompt
    ecall

    li a0, 0                 # Read from stdin
    la a1, buffer
    li a2, 64                # Maximum bytes to read
    li a7, 63                # Read submitted console input
    ecall
    blez a0, finish          # Stop safely on EOF/error

    la t0, buffer
    lbu t1, 0(t0)            # First character of input
    li t2, 88                # ASCII X
    beq t1, t2, finish
    li t2, 82                # ASCII R
    bne t1, t2, read_command # Other input: ask again; no move

    # Erase the LED at its current position.
    slli t0, s1, 2           # Byte offset = column * 4 (row 0)
    add t0, s0, t0
    sw zero, 0(t0)

    # Advance one column. After column 4, return to column 0.
    addi s1, s1, 1
    li t2, 5
    blt s1, t2, draw
    li s1, 0

draw:
    slli t0, s1, 2
    add t0, s0, t0
    sw s2, 0(t0)            # Light the new position yellow
    j read_command

finish:
    li a7, 10
    ecall
```
The following guide can help you understand the program above. Read the code and follow through these details:
    
1. **Initialization**:
    - `s0` is the LED base address
    - `s1` is the current column
    - `s2` is the yellow color. 
    - The row is always zero, so there is no row register yet.
2. **Prompt for a user input**. Note the following `ecall` codes:
    - `ecall` 4 prints the prompt.
    - `ecall` 63 reads console input into `buffer`, which is ordinary RAM.
    - `lbu` loads its first character.
3. **Choose an action based on user input**
    - ASCII X is 88 and ASCII R is 82. 
    - A letter X input exits the program while an R moves the LED light one cell to the right.
    - Every other input returns to the prompt without changing the LED.
4. **Turn off the previous LED cell:** 
    - Every after movement, calculate the old LED address and store zero there. 
    - For row zero, the address formula simplifies to `base + (4 * column)`.
5. **Turn on the next LED cell:**
    - For an R movement, increase the column by one. If it becomes 5, set it back to zero.
6. **Draw and repeat:**
    - After moving, calculate the new LED address and store the color `yellow` to that LED address then jump back to `read_command`.

The following three instructions locate the current LED and turn it off:

```asm
    slli t0, s1, 2         # Multiply column by four
    add t0, s0, t0         # Add the device base address
    sw zero, 0(t0)        # Turn off the old LED
```


## Checkpoint Tasks
### Checkpoint Item 1: Light all five LEDs on the main diagonal (40/100 points)
For the first checkpoint task, write a RISC-V program that will light up all the LEDs along the diagonal of the matrix. That is, the LEDs at `(0,0)`, `(1,1)`, `(2,2)`, `(3,3)`, and `(4,4)` should be color `RED`.

To get the full 40 points, you should use a loop to light up all 5 LEDs. You will only get 20 points if you light up the LEDs one by one without using a loop.

### Checkpoint Item 2: Extend keyboard control to W/S/A/D (60/100 points)
Extend the R-only example from Walkthrough #2 so that one yellow LED can move in all four directions. The position of LED should initially be at the center of the matrix `(row 2, column 2)`. Keep exactly one LED lit throughout the duration of the program.

| Input | Required action |
|---|---|
| W | Move up one cell: row decreases by 1 |
| S | Move down one cell: row increases by 1 |
| A | Move left one cell: column decreases by 1 |
| D | Move right one cell: column increases by 1 |
| X | End the program without applying a move |

Use uppercase letters and submit one command followed by Enter. Replace R with D for rightward movement; R is no longer a movement command in this checkpoint.

Your program must satisfy the following specifications:

1. Clear the 5x5 display, then light the center LED yellow.
2. Print a prompt and repeatedly read a command from the console until `X` is entered.
3. The valid inputs from the user is `W`, `S`, `A`, `D` and `X` only. For an invalid command, leave the display and position unchanged and ask again. A short invalid-command message is recommended.
4. If a move would cross any matrix boundary, leave the LED and current coordinates unchanged, then ask for another user input. The LED must **not wrap** to another row or the opposite edge.
5. Hint: Use `LED_MATRIX_0_BASE + 4 * (row * 5 + column)` to find a destination address.
6. When `X` is entered, terminate without moving or clearing the last LED.

#### Suggested Test Moves
You may use the following sequence of moves to test that your program works as intended.

| Test | Commands from center `(2,2)` | Expected result |
|---|---|---|
| Four directions | D, S, A, W | `(2,3)`, `(3,3)`, `(3,2)`, then `(2,2)` |
| Top boundary | W, W, W | `(1,2)`, `(0,2)`, then remain at `(0,2)` |
| Bottom boundary | S, S, S | `(3,2)`, `(4,2)`, then remain at `(4,2)` |
| Left boundary | A, A, A | `(2,1)`, `(2,0)`, then remain at `(2,0)` |
| Right boundary | D, D, D | `(2,3)`, `(2,4)`, then remain at `(2,4)` |
| Invalid input | Q, R | No movement for either input |
| Exit | X | Program stops without applying a move |








