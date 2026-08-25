---
title: Lab Exercise 1 (Introduction to RISC-V Programming)
---

<h2 align="center"> CS 21 26.1 Laboratory Exercise 1 </h2>
<h1 align="center"> Introduction to RISC-V Programming </h1>

## Overview
Ripes is an open-source RISC-V simulator with a web-based version available over at [https://ripes.me/](https://ripes.me/). If you want a locally installed Ripes in your computer, you may install them from this link: [https://github.com/mortbopet/Ripes/releases/tag/v2.2.6](https://github.com/mortbopet/Ripes/releases/tag/v2.2.6). For our lab classes, we will be using a modified Ripes version accessible at [https:ripes.upd-dcs.work/](https://ripes.upd-dcs.work/). 

!!! important Single-cycle Processor Setting 
    For the first few lab activities on assembly programming, set the processor to **Single-cycle processor** via the Select processor menu button (a black chip-like image below the File button). Failing to do this may lead to confusing behavior (for now) with the `Step` operation.

## General Instructions

For this laboratory activity, you are to work on **2 Checkpoint Items** and **2 TakeHome Items** that will aid in your understanding of basic R-type instructions, I-type, and B-type instructions for writing simple programs involving integers and loops.

!!! warning "Important Reminder"
    You **must finish and show** your working checkpoint tasks to your lab handler before the end of the laboratory period. 

A practice task is provided in the next section to help you warm up prior to the Checkpoint task and be comfortable with using the Ripes simulator. The practice task is not required and will not be checked.

## Relevant links

- **RISC-V Green Card** [https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view](https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view)

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
### Checkpoint Item 1: Warm Up (20/100)
As a warm up, let `t0` = `x`, `t1` = `y`, `t2` = `z`, and `t3` = `r`. Manually load a user-defined value into the variables `x`, `y` and `z` and write a RISC-V program that assigns a value to `r` based on the following C code.
```c
#include <stdio.h>

int main() {
  if (x != 0 && y < z) {
      r = 1;
  } else {
      r = 0;
  }
}
```
### Checkpoint Item 2: A Very Boring Start (30/100)
Write a RISC-V program that will change the value of `s0` based on the value stored in `t1` based on the given C code.

```c
#include <stdio.h>

int main() {
    int t1 = 12; // Only this line should be manually changed for testing
    int s0 = 0;

    if (t1 > 15){
      s0 = 50465;
    }
    else{
      s0 = t1 * t1 * t1 * t1;
    }
}
```
### Take Home Item: Tribonacci in Assembly (50/100)
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

#### Restrictions

- Only one instance of the print integer syscall is allowed to be present in your solution
- `or` must be used when converting `n == 1 || n == 2`

#### Constraints

- $-30 \le n \le 30$

#### Test Cases

| $n$ | $T_n$ |
|:-:|:-:|
| $0$ | $0$ |
| $10$ | $149$ |
| $20$ | $66012$ |
| $30$ | $29249425$ |
| $-21$ | $-1$ |

## TakeHOPE Problems

### Item 1: More Practice Exercises

1. For the following subitems, let `s0` = `a`, `s1` = `b`, `s2` = `c`, and `s3` = `d`. Use at most two (2) temporary registers. Create RISC-V programs that can compute the following:
      1. `(a + b) - (c + d)`
      2. `(a * b) + c`
      3. `a + (b - (c * d))`
      4. `(a + b)`<sup>2</sup>
      5. `(a + b)`<sup>2</sup> - `(c + d)`<sup>2</sup>

2. Create a RISC-V program that swaps the values in registers `a0` and `a1` in place without using temporary registers or pseudoinstructions.

3. Let `t0` = `x` and `t1` = `y`. Create a RISC-V program that evaluates:
```c
if (x < 0) {
    y = -1;
} else if (x == 0){
    y = 0;
} else {
    y = 1;
}
```



5. Let `t0` = `x`, `t1` = `y`, `t2` = `z`, and `t3` = `r`. Create a RISC-V program that evaluates:
```c
if (!(x < y || z == 0)) {
    r = 1;
} else {
    r = 0;
}
```

6. The following code is intended to count from `0` to `N`, where `N` is store in `a0`. However, it does not work for all `N`. For which `N` does the code fail? How can you fix the code such that it works for all `N`?
```asm
li t0, 0

loop:
    addi t0, t0, 1
    bne  t0, a0, loop
```

7. Create a RISC-V program that gets the count of numbers in `[0...N-1]` that are less than `K`, where `N` is stored in `a0` and `K` is stored in `a1`. Assume `0 < K < N` and `0 < N`.

### Item 2: C to Assembly

Convert the C program below into an equivalent RISC-V program:

!!! warning "Print character syscall"

    This item requires the print character syscall _(syscall code 11)_ that will be introduced in Lecture 3.

    For now, you may refer to **Help > System calls** in Ripes to information on how it is used.

```c
#include <stdio.h>

int main() {
  unsigned int n = 5;
  unsigned int size = 3;

  size = (size & 7) + 1;
  int height = size << 1;
  char ch = '\0';
  int level = n & 15;

  do {
    int r = height;

    while (r >= 0) {
      int num = r;
      int left = (r > size) ? r - size : size - r;
      int right = (r > size) ? size + height - r : r - size + height;

      for (int c = 0; c <= height; c++) {
        ch = ' ';

        if (c == left) {
          if (r == height) {
            ch = 'A';
          } else if (r == size) {
            ch = '<';
          } else if (r == 0) {
            ch = 'V';
          } else if (r > size) {
            ch = '/';
          } else {
            ch = '\\';
          }
        } else if (c == right) {
          if (r == size) {
            ch = '>';
          } else if (r > size) {
            ch = '\\';
          } else {
            ch = '/';
          }
        } else if (c > left && c < right) {
          if (level >= r + 1) {
            if (level > 0) {
              ch = '0' + num;

              if (ch > '9') {
                ch += 39;
              }
            }
          }
        } else if (c > right) {
          break;
        }

        printf("%c", ch);
      }

      ch = '\n';
      printf("%c", ch);
      r -= 1;
    }

    level--;
  } while (level > 0 && size > 0);
}

```

### Item 3: Assembly to C

Convert the RISC-V program below into an equivalent C program:

```mips
main:
  li a1, 123456
  li a2, 765432
  li a3, 6

  li s11, 10

  li s0, 0
  mv s1, a1
  
  li t0, 0
L1: 
  bge t0, a3, L1_end

  rem s2, s1, s11
  mv s3, a2
  li t1, 0

L2:
  rem s4, s3, s11
  bne s2, s4, L3
  addi s0, s0, 1
L3:
  div s3, s3, s11
  addi t1, t1, 1
  blt t1, a3, L2

  div s1, s1, s11

  addi t0, t0, 1
  beq zero, zero, L1
L1_end:

  li a7, 1
  mv a0, s0
  ecall

  li a7, 11
  li a0, 10
  ecall

  li a7, 10
  ecall
```

### Item 4: The Smoking Pearl

#### Problem Statement

Pearl has $n$ cigarettes. Each time she smokes a cigarette, she obtains one cigarette butt. With $k$ cigarette butts, she can make one (1) new cigarette. She can smoke this new cigarette and obtain *another* cigarette butt.

Your task is to implement a RISC-V program prints out the the number of cigarettes Pearl can smoke in total, where the values of $n$ and $k$ are hardcoded in registers `a0` and `a1` at the start of the program.

#### Sample Test Cases

| $N$ | $K$ | Expected Output |
|:-:|:-:|:-:|
| $10$ | $2$ | $19$ |
| $4$ | $3$ | $5$ |
| $10$ | $3$ | $14$ |

#### Restrictions

- You are **not** allowed to use pseudoinstructions

#### Constraints

- $0 \le n \le 300$
- $2 \le k \le 300$