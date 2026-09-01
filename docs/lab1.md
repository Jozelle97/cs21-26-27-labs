---
title: Lab Exercise 1 (Introduction to RISC-V Programming)
---

<h2 align="center"> CS 21 26.1 Laboratory Exercise 1 </h2>
<h1 align="center"> Introduction to RISC-V Programming </h1>

## Overview
Ripes is an open-source RISC-V simulator with a web-based version available over at [https://ripes.me/](https://ripes.me/). If you want a locally installed Ripes in your computer, you may install them from this link: [https://github.com/mortbopet/Ripes/releases/tag/v2.2.6](https://github.com/mortbopet/Ripes/releases/tag/v2.2.6). For our lab classes, we will be using a modified Ripes version accessible at [https:ripes.upd-dcs.work/](https://ripes.upd-dcs.work/). 

## Relevant links

- **RISC-V Green Card** [https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view](https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view)

- **RIPES (DCS Fork) Simulator** [https:ripes.upd-dcs.work/](https://ripes.upd-dcs.work/). 

!!! important Single-cycle Processor Setting 
    For the first few lab activities on assembly programming, set the processor to **Single-cycle processor** via the Select processor menu button (a black chip-like image below the File button). Failing to do this may lead to confusing behavior (for now) with the `Step` operation.


## General Instructions

For this laboratory activity, you are to work on **2 Checkpoint Items** that will aid in your understanding of basic R-type instructions, I-type, and B-type instructions for writing simple programs involving conditionals and loops.

!!! warning "Important Reminder"
    You **must finish and submit** your working checkpoint outputs in the Google Classroom **by 5pm of Semptember 1, 2026**. Save your outputs as `lab01_item1_lastname.asm` and `lab01_item2_lastname.asm`. In each of your code submissions, write your name within comment blocks.

A practice task is provided in the next section to help you warm up prior to the Checkpoint task and be comfortable with using the Ripes simulator. The practice task is not required and will not be checked.

## Practice Task

This task is meant to familiarize you with basic R-type instructions and help you be comfortable with using the Ripes simulator.

Create a RISC-V program that performs the following operations consecutively:

1. Assigns some integer to `s0` expresed in hexadecimal (i.e., with a `0x` prefix)

2. Assigns some integer to `s1` expressed in base 10

3. Sets `s2` to the sum of `s0` and `s1`

4. Sets `s3` to the difference of `s0` and `s1`

5. Sets `s7` to the bitwise OR of `s0` and `s1`

6. Sets `s8` to the bitwise XOR of `s0` and `s1`

7. Sets `s9` to the bitwise negation of `s0`

Confirm that the correct results have been saved to the corresponding registers in the Register Panel.


## Checkpoint Task
### Checkpoint Item 1: Bi-Conditional If-Statement (40/100)
As a warm up, let `t0` = `x`, `t1` = `y`, `t2` = `z`, and `t3` = `r`. Using the correct RISC-V instructions, inject user-defined integer values into the variables `x`, `y` and `z` and write a RISC-V program that assigns a value to `r` based on the following C code.
```c
#include <stdio.h>

int main() {
    int x = 0;
    int y = 5;
    int z = 8;
    int r = 0;
  if (x != 0 && y < z) {
    r = 1;
    printf("%d", x);
  } else {
      for (int i = 0; i < z; i = i+2){
        printf("%d", i);
      }
  }

}
```
### Checkpoint Item 2: Tribonacci in Assembly (60/100)
The _Fibonacci sequence_ is a series of numbers wherein each number in the series is the sum of the two numbers preceding it. The _Tribonacci sequence_ is a similar sequence wherein each term is the sum of the _three_ preceding terms.

The first three numbers of the Tribonacci sequence are $0$, $1$, and $1$. The first Tribonacci number is denoted by $T_0$, the second by $T_1$, and so on, such that $T_3 = 2$ and $T_4 = 4$.

You are to convert the following C code which prints out $T_n$ for `n` any nonnegative value, and `-1` otherwise.

At the start of your solution, the register `a0` must be set to `n`. The use of pseudoinstructions is allowed.

```c
#include <stdio.h>

int main() {
    int n = 10;  // Only this line should be manually changed for testing
    int ans;

    if (n < 0) {
        ans = -1;
    } else if (n == 0) {
        ans = 0;
    } else if (n == 1 || n == 2) {
        ans = 1;
    } else {
        int t0 = 0;
        int t1 = 1;
        int t2 = 1;

        for (int i = 3; i <= n; i++) {
            ans = t0 + t1 + t2;
            t0 = t1;
            t1 = t2;
            t2 = ans;
        }
    }

    printf("%d", ans);  // No need for trailing newline
}
```

**Restrictions**

- Only one instance of the print integer syscall is allowed to be present in your solution
- `or` must be used when converting `n == 1 || n == 2`

**Constraints**

- $-30 \le n \le 30$

**Test Cases**

| $n$ | $T_n$ |
|:-:|:-:|
| $0$ | $0$ |
| $10$ | $149$ |
| $20$ | $66012$ |
| $30$ | $29249425$ |
| $-21$ | $-1$ |

## Challenge Problems (non-graded)
These challenge problems are extra problems that you can use for practice. Some of these challenge problems may require new knowledge that you will encounter throughout the lectures this week. Advanced reading may also help you finish these tasks earlier.

#### Challenge Problem # 1 

Create a RISC-V program that gets the count of numbers in `[0...N-1]` that are less than `K`, where `N` is stored in `a0` and `K` is stored in `a1`. Assume `0 < K < N` and `0 < N`.

#### Challenge Problem # 2

The Banker's Rounding is a floating-point rounding algorithm specified by the IEEE 754 standard in which a number equidistant from the two nearest integers are rounded to the nearest even integer. The banker's rounding has the following rules:

1. When the number to be rounded (the rounding digit) is followed by `0`,`1`,`2`,`3` or `4`, the rounding digit remains the same.
2. When the number to be rounded (the rounding digit) is followed by a `6`,`7`,`8` or a `9`, the number is rounded up. 
3. When the number to be rounded (the rounding digit) is follwed by a `5`, the rounding rule is determined by the number after `5`:
    - When there is a number after `5`, round up. 
    - When there is no number after `5`, check the number before `5`. If it is an odd number, round up, otherwise round down.

Using the read string syscall, ask the user to input a 6-character number input (5 digits and 1 decimal point) and print the output of the Banker's rounding. The code below is a sample of how you can get a user input string using the console in RIPES.

!!! tip
    You may use the  `Console` and `Memory` panel to view the console or the contents of the RISC-V memory contents

```asm
.text
main:
    # execute a Read String syscall
    li a7, 63          # syscall 63 = read
    li a0, 0           # file descriptor 0 = stdin
    la a1, input       # a1 will hold the address where input is stored
    li a2, 100         # reserves a maxinmum of 100 bytes to read
    ecall

    # execute Exit Syscall
    li a7, 10          # syscall 93 = exit
    li a0, 0           # exit status
    ecall
    
.data
prompt: .string "Enter a string: "
input:  .string 100
```

#### Challenge Problem # 3

Pearl has $n$ cigarettes. Each time she smokes a cigarette, she obtains one cigarette butt. With $k$ cigarette butts, she can make one (1) new cigarette. She can smoke this new cigarette and obtain *another* cigarette butt.

Your task is to implement a RISC-V program prints out the the number of cigarettes Pearl can smoke in total, where the values of $n$ and $k$ are hardcoded in registers `a0` and `a1` at the start of the program.

**Sample Test Cases**

| $N$ | $K$ | Expected Output |
|:-:|:-:|:-:|
| $10$ | $2$ | $19$ |
| $4$ | $3$ | $5$ |
| $10$ | $3$ | $14$ |

**Restrictions**

- You are **not** allowed to use pseudoinstructions

**Constraints**

- $0 \le n \le 300$
- $2 \le k \le 300$
