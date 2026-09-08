---
title: Memory and Functions in RISC-V
---

<h2 align="center"> CS 21 26.1 Laboratory Exercise 2 </h2>
<h1 align="center"> Memory and Functions in RISC-V </h1>

## Overview
In the previous week, we tackled the B-type instructions for interfacing with the memory and how J-type instructions are used for calling a function in RISC-V. In this laboratory activity, you will test your understanding in last week's lectures and implement a bit more complex RISC-V programs. 

In Lecture 4, you learned how to write a function in RISC-V using the proper storing and restoring of RISC-V registers. A function in RISC-V uses registers `a0` to `a7` for input parameters, while `a0` and `a1` are used to contain the return values of functions.

The stack frame of RISC-V serves as a memory space for 'scratch' work of functions where values of registers may be temporarily stored and restored using the stack pointer `sp` to track the top of the stack. It is good practice for all functions in a RISC-V program to have the same rules for storing and restoring register values in the stack. For simplicity, registers whose values may be overwritten or used by a function are usually temporarily stored in the stack frame before executing the function body. The function restores these values prior to returning to the function's caller. Allocation and deallocation is done by moving the `sp`.

!!! warning "Saving/Restoring Convention"

    In CS 21, we will use the following convention for saving and restoring registers when used in functions:

    - Save and restore all **`s`** and **`t`** registers that are written to by the function.
    - There is no need to save and restore **`a0`** and **`sp`**.

With proper storing and restoring of registers, the implementation of recursive functions is no different from that of nonrecursive functions. Recursive functions will be reserved for the next lab activity. 

## Relevant links

- **RISC-V Green Card** [https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view](https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view)

- **RIPES (DCS Fork) Simulator** [https:ripes.upd-dcs.work/](https://ripes.upd-dcs.work/). 

!!! important Single-cycle Processor Setting 
    For the first few lab activities on assembly programming, set the processor to **Single-cycle processor** via the Select processor menu button (a black chip-like image below the File button). Failing to do this may lead to confusing behavior (for now) with the `Step` operation.


## General Instructions

For this laboratory activity, you are to work on:

1. **Two Checkpoint Items** (due by the end of the lab period)
2. **One Take-Home Item** (due before the next lab meeting)

Save your outputs as `lab02_item1_lastname.asm`, `lab02_item2_lastname.asm` and `lab02_takehome_lastname.asm`. In each of your code submissions, write your name within comment blocks. All files must must be submitted via Google Classroom.

These activities will aid in your understanding of writing instructions that interface with memory and writing functions.

!!! warning "Important Reminder"
    You **must finish and submit** your working checkpoint outputs and **show them to the lab instructor before the end of the lab meeting**. 

## Practice Task

Before doing this task, go through the **Lab 2 Guide** first. Be sure that you write your functions properly so that it follows our conventions for the storing and retoring or registers in the stack frame.

Convert the following C program into a RISC-V program such that `f` and `g` are proper functions _(i.e., they have proper stack frames)_:

```c
#include <stdio.h>

int increase(int n) {
    return n + 3;
}

int combine(int n) {
    int first = increase(n);
    int second = increase(first);

    return first + second;
}

int main() {
    int result = combine(4);
    printf("%d\n", result);
}
```


## Checkpoint Tasks
### Checkpoint Item 1: Compute GCD(a,b) (40/100)

The greatest common divisor (GCD) of two positive integers `a` and `b` can be computer in C using the following C-code:

```c
int gcd(int a, int b) {
    if (a < 0) {
        a = -a;
    }

    if (b < 0) {
        b = -b;
    }

    while (b != 0) {
        int remainder = a % b;
        a = b;
        b = remainder;
    }

    return a;
}
```

Write a RISC-V program that will take two integers from the, say $a$ and $b$, and print the result of $gcd(a,b)$ on the console. In the `gcd(a,b)` function above, it is assumed that $a \ge b$.

**Checkpoint 1 Submission Template**

Copy the following template into Ripes. Implement the three incomplete functions without changing the provided `main` function.

```asm
# Name: YOUR NAME HERE
# Student number: YOUR STUDENT NUMBER HERE
# Lab 1 Checkpoint 1: Greatest Common Divisor 

.text
main:
    # Read the first integer.
    # TODO : add your codes here

    # Read the second integer.
    # TODO : add your codes here

    # a0 = gcd(a0, a1)
    call gcd

    # Print the returned GCD.
    li a7, 1
    ecall

    # Print a newline.
    li a0, 10
    li a7, 11
    ecall

    # Exit the program.
    li a7, 10
    ecall

gcd:
    # TODO: implement this function.

    ret

```

### Checkpoint Item 2 : Print_String from Memory (30/100)
Refer to the `void print_message(char *message);` of the Take-Home Activity below

**Checkpoint 2 Submission Template**

You must submit a `.asm` code following this template:

```asm
# Name: YOUR NAME HERE
# Student number: YOUR STUDENT NUMBER HERE
# Lab 1 Checkpoint 2: Print_message Implementation 

.text
main:
    #--------------------------------------
    #--- DO NOT CHANGE ANYTHING IN MAIN ---
    #--------------------------------------
    # printf("Original message:  ");
    li a7, 4
    la a0, original_label
    ecall

    # print_message(message);
    la a0, message
    call print_message

    # printf("\n");
    li a7, 11
    li a0, 10
    ecall

    # Exit the program.
    li a7, 10
    ecall

print_message:
# TODO: Implement this function.

.data
message:          .asciz "Hello, RISC-V!" # Writable string that will be encrypted in place.
original_label:   .asciz "Original message:  "

```

## Take-Home Activity: In-Place Caesar Cipher (30/100)

The Caesar cipher encrypts a message by replacing every letter with the letter located \(k\) positions to its right in the alphabet.

For a letter with alphabet index \(i\):

\[
i_{\text{encrypted}} = (i + k) \bmod 26
\]

Consider that the capital letter `A` and the small letter `a` correspond to index 0, while capital letter `B` and small letter `b` correspond to index 1, and so on.

### Objectives

Complete the provided RISC-V program template below by implementing three functions:

```c
void print_message(char *message);
char shift_character(char c, int k);
void encrypt_message(char *message, int k);
```

The `main` function and the required data are already provided. You should not need to modify them.

Your implementation must:

1. Print the original message one character at a time.
2. Encrypt the message using a Caesar cipher.
3. Modify the original message directly in memory.
4. Print the encrypted message one character at a time.
5. Use proper function calls and stack-frame management.

### Expected behavior

The provided program uses:

```text
Message: Hello, RISC-V!
Shift:   3
```

The completed program must print:

```text
Original message:  Hello, RISC-V!
Encrypted message: Khoor, ULVF-Y!
```

The encryption must:

- Preserve uppercase letters as uppercase.
- Preserve lowercase letters as lowercase.
- Wrap around at the end of the alphabet.
- Leave spaces, digits, and punctuation unchanged.

Note that for a shift value of 3, the letters in the original message will be encrypted as follows:

| Original | Encrypted |
|---|---|
| `A` | `D` |
| `X` | `A` |
| `a` | `d` |
| `z` | `c` |
| `7` | `7` |
| `!` | `!` |

### Submission Template

Copy the following template into Ripes. Implement the three incomplete functions without changing the provided `main` function.

```asm
# Name:
# Student number:
# Take-Home Activity: In-Place Caesar Cipher

.text
main:
    # printf("Original message:  ");
    li a7, 4
    la a0, original_label
    ecall

    # print_message(message);
    la a0, message
    call print_message

    # printf("\n");
    li a7, 11
    li a0, 10
    ecall

    # encrypt_message(message, 3);
    la a0, message
    li a1, 3
    call encrypt_message

    # printf("Encrypted message: ");
    li a7, 4
    la a0, encrypted_label
    ecall

    # print_message(message);
    la a0, message
    call print_message

    # printf("\n");
    li a7, 11
    li a0, 10
    ecall

    # Exit the program.
    li a7, 10
    ecall


# --------------------------------------------------
# void print_message(char *message)
#
# a0 = address of the null-terminated string
# --------------------------------------------------

print_message:
    # TODO: Implement this function.

    ret


# --------------------------------------------------
# char shift_character(char c, int k)
#
# a0 = character to encrypt
# a1 = shift value
#
# Return:
#   a0 = encrypted character
# --------------------------------------------------

shift_character:
    # TODO: Implement this function.

    ret


# --------------------------------------------------
# void encrypt_message(char *message, int k)
#
# a0 = address of the writable string
# a1 = shift value
# --------------------------------------------------

encrypt_message:
    # TODO: Implement this function.

    ret

.data
# Writable string that will be encrypted in place.
message:         .asciz "Hello, RISC-V!"
original_label:  .asciz "Original message:  "
encrypted_label: .asciz "Encrypted message: "
```

### Function specifications

#### 1. `print_message`

```c
void print_message(char *message);
```

The address of the string is passed through `a0`.

The function must:

1. Load one character from memory using `lb` or `lbu`.
2. Stop when it encounters the null terminator.
3. Print the character using environment call 11.
4. Advance to the next byte.
5. Continue until the entire string has been printed.

The print-character environment call uses:

```asm
li a7, 11
# Place the character in a0.
ecall
```

!!! warning "Do not use the print-string environment call"

    Inside `print_message`, you may not use environment call 4 to print the complete string. You must traverse memory and print the string one character at a time.

#### 2. `shift_character`

```c
char shift_character(char c, int k);
```

The function receives:

| Register | Value |
|---|---|
| `a0` | Character to encrypt |
| `a1` | Shift value |

It returns the encrypted character through `a0`.

The function must:

- Shift uppercase letters within `A` through `Z`.
- Shift lowercase letters within `a` through `z`.
- Apply alphabet wraparound.
- Return spaces, digits, and punctuation unchanged.

#### 3. `encrypt_message`

```c
void encrypt_message(char *message, int k);
```

The function receives:

| Register | Value |
|---|---|
| `a0` | Address of the writable message |
| `a1` | Shift value |

For each character, the function must:

1. Load the character from memory.
2. Check for the null terminator.
3. Pass the character and shift value to `shift_character`.
4. Call `shift_character`.
5. Store the returned character into the original memory location.
6. Advance to the next byte.

Because `encrypt_message` calls `shift_character`, it is a **non-leaf function** and must use a proper stack frame.

#### Memory requirements

!!! important "Modify the original message"

    The ciphertext must overwrite the original `message` buffer. A solution that prints a separate or hard-coded ciphertext does not satisfy the requirements.

Your program must:

- Use `lb` or `lbu` to read characters from memory.
- Use `sb` to write encrypted characters to memory.
- Preserve the null terminator.
- Not create a second message buffer.
- Not hard-code the string length.
- Not hard-code the ciphertext.
- Continue to work if the contents of `message` are changed.
- Print both versions of the message using `print_message`.

#### Calling-convention requirements

Your functions must:

1. Pass arguments using `a0` and `a1`.
2. Return the encrypted character through `a0`.
3. Use `call shift_character` inside `encrypt_message`.
4. Preserve all modified callee-saved registers.
5. Restore `sp` before executing `ret`.

The `encrypt_message` function must additionally:

1. Allocate a stack frame.
2. Save and restore `ra`.
3. Preserve the current message address across calls.
4. Preserve the shift value across calls.
5. Deallocate its complete stack frame before returning.

!!! note "Leaf and non-leaf functions"

    `print_message` and `shift_character` are leaf functions because they do not call other functions.

    `encrypt_message` is a non-leaf function because it calls `shift_character`. Therefore, it must save and restore `ra`.

#### Equivalent C program

The completed assembly program should behave like this C program:

```c
#include <stdio.h>

void print_message(char *message) {
    int i = 0;

    while (message[i] != '\0') {
        printf("%c", message[i]);
        i++;
    }
}

char shift_character(char c, int k) {
    k = k % 26;

    if (c >= 'A' && c <= 'Z') {
        c = c + k;

        if (c > 'Z') {
            c = c - 26;
        }

    } else if (c >= 'a' && c <= 'z') {
        c = c + k;

        if (c > 'z') {
            c = c - 26;
        }
    }

    return c;
}

void encrypt_message(char *message, int k) {
    int i = 0;

    while (message[i] != '\0') {
        char current = message[i];
        char encrypted = shift_character(current, k);

        message[i] = encrypted;
        i++;
    }
}

int main() {
    char message[] = "Hello, RISC-V!";
    int shift = 3;

    printf("Original message:  ");
    print_message(message);
    printf("\n");

    encrypt_message(message, shift);

    printf("Encrypted message: ");
    print_message(message);
    printf("\n");
}
```

#### Test cases

After your program works with the provided message, test it by changing `message` and the shift value `k` in `main`.

| $k$ | $message$ | Ciphertext  |
|:-:|:-:|:-:|
| $3$ | `Hello` | `Khoor` |
| $5$ | `BUZZ` | `GZEE` |
| $12$ | `SuperCaliFragilisticExpialidocious` | `EgbqdOmxuRdmsuxuefuoQjbumxupaouage`|
| $0$ | `EXTraOrdinary` | `EXTraOrdinary`|

#### Take Home Submission requirements

Submit one assembly source file named:

```text
caesar_lastname.asm
```

Include the following comment block at the beginning of the file:

```asm
# Name:
# Student number:
# Section:
# Take-Home Activity: In-Place Caesar Cipher
```

Your submission must contain working implementations of:

```text
print_message
shift_character
encrypt_message
```

Do not change the behavior of the provided `main` function.


