---
title: Recursions and Memory Interfacing in RISC-V
---

<h2 align="center"> CS 21 26.1 Laboratory Exercise 3 </h2>
<h1 align="center"> Recursions and Memory Interfacing in RISC-V </h1>

## Overview
In the lecture, we've emphasized that writing recursions is no different from writing functions in RISC-V. This lab activity is meant to test your understanding and skills in writing recursive functions in assembly language to solve programming tasks. 

!!! warning "Saving/Restoring Convention"

    In CS 21, we will use the following convention for saving and restoring registers when used in functions:

    - Save and restore all **`s`** and **`t`** registers that are written to by the function.
    - There is no need to save and restore **`a0`** and **`sp`**.

## Relevant links

- **RISC-V Green Card** [https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view](https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view)

- **RIPES (DCS Fork) Simulator** [https:ripes.upd-dcs.work/](https://ripes.upd-dcs.work/). 

!!! important Single-cycle Processor Setting 
    For the first few lab activities on assembly programming, set the processor to **Single-cycle processor** via the Select processor menu button (a black chip-like image below the File button). Failing to do this may lead to confusing behavior (for now) with the `Step` operation.


## General Instructions

For this laboratory activity, you are to work on:

1. **Two Checkpoint Items** (due by the end of the lab period)
2. **One Take-Home Item** (due before the next lab meeting)

Save your outputs as `lab03_item1_lastname.asm`, `lab03_item2_lastname.asm` and `lab03_takehome_lastname.asm`. In each of your code submissions, write your name within comment blocks. All files must must be submitted via Google Classroom.

These activities will aid in your understanding of writing instructions that interface with memory and writing functions.

!!! warning "Important Reminder"
    You **must finish and submit** your working checkpoint outputs and **show them to the lab instructor before the end of the lab meeting**. 

## Overview

An arithmetic expression can contain smaller expressions. For example, evaluating

```text
((8+2)*(9-4))
```

requires evaluating `(8+2)` and `(9-4)` before multiplying their results.

In this laboratory activity, you will implement two separate recursive functions:

1. A recursive function that converts decimal digits into an integer.
2. A recursive function that evaluates nested arithmetic expressions.

Once both functions work, they will be combined to create a complete arithmetic-expression evaluator.

## Supported Expression Format

Expressions follow this grammar:

```text
expression := number
            | '(' expression operator expression ')'

operator   := '+' | '-' | '*' | '/'

number     := digit number
            | digit

digit      := '0' | '1' | ... | '9'
```

All arithmetic operations must be enclosed in parentheses.

Valid examples:

```text
42
(8+2)
((8+2)*(9-4))
(20/(2+3))
((10-4)+(6*2))
```

Invalid examples:

```text
8+2
(8+)
()
((8+2)*5
```

Spaces do not need to be supported.

## Checkpoint Tasks
### Checkpoint Item 1: Recursive Number Parser (40/100)

A number in the expression is stored as a sequence of ASCII characters. For example, the integer `527` is stored as a sequence of digits "527" or a sequence of characters '5', '2', '7' in the memory.

Write a recursive RISC-V function named `parse_number` that will convert a string of consecutive decimal digits into their integer value. Assume that this sequence of digits is stored as characters in the memory defined in the `.data` of your assembly file.

#### Input to the Function

```text
a0 = address of the current character
a1 = accumulated value
a2 = number of digits processed
```

#### Output of the Function

```text
a0 = address of the first character that is not a digit
a1 = converted integer value
```

The initial call must use:

```text
a0 = address of the first character
a1 = 0
a2 = 0
```

#### Expected behavior
The accumulated value holds the integer value obtained by processing each digit in the string of characters. For every digit, calculate:

```text
new_accumulated_value = accumulated_value × 10 + digit_value
```

The ASCII digit can be converted to an integer using:

```text
digit_value = character - '0'
```

The function must then call itself recursively to process the next character.

Example 1

Given:

```text
Input string: "527+3"
a0 = address of '5'
a1 = 0
a2 = 0
```

The recursive calls process the number as follows:

```text
0 × 10 + 5 = 5
5 × 10 + 2 = 52
52 × 10 + 7 = 527
```

The function must return:

```text
a0 = address of '+'
a1 = 527
```

#### Example 2

Given:

```text
Input string: "84)"
```

The function must return:

```text
a0 = address of ')'
a1 = 84
```

#### Output

```text
a0 = address of the first character that is not a digit
a1 = converted integer value
```

#### Restrictions

- `parse_number` must call itself recursively.
- A loop may not be used to process the digits.
- Multiplication instructions or repeated addition may be used.
- The function must work for numbers containing more than one digit.
- The function must stop at the first nondigit character.
- The function must save and restore `ra`.
- The stack must be restored before returning.

#### Sample I/O for Checkpoint 1

| Input string | Returned value | Cursor stops at |
|---|---:|---|
| `"7"` | `7` | Null terminator |
| `"42"` | `42` | Null terminator |
| `"527+3"` | `527` | `+` |
| `"84)"` | `84` | `)` |
| `"1234*5"` | `1234` | `*` |
| `"+7"` | Error | `+` |

## Checkpoint 2: Recursive Expression Evaluator (60/100)

After completing `parse_number`, write a second recursive RISC-V function named `parse_expression`.The function must evaluate either:

1. A decimal number.
2. A fully parenthesized arithmetic expression.

Assume that a valid expression that follows the grammar discussed above is stored as a sequence of characters in the memory defined in the `.data` of your assembly file.

#### Input to the Function

```text
a0 = address of the current character
```

#### Output of the Function

```text
a0 = address of the first unprocessed character
a1 = evaluated integer result
```
#### Expected Behavior of the Function
##### Case 1: The Expression Is a Number

If the current character is a digit, `parse_expression` must call `parse_number` to process the string of digits and convert it to its integer value equivalent.

For example:

```text
Input:  "125)"
Output: value = 125
Cursor: address of ')'
```

##### Case 2: The Expression Begins with `(`

When the current character is `(`, the function must perform the following operations:

1. Move to the character after `(`.
2. Recursively call `parse_expression` to evaluate the left operand. If the operand is a number, parse_expression calls `parse_number`.
3. Read and save the operator.
4. Recursively call `parse_expression` to evaluate the right operand. If the operand is a number, parse_expression calls `parse_number`.
5. Verify that the next character is `)`.
6. Apply the saved operator.
7. Return the calculated result.

For example:

```text
Expression: ((8+2)*(9-4))
```

The recursive structure is:

```text
                 *
               /   \
              +     -
             / \   / \
            8   2 9   4
```

The evaluation proceeds as follows:

```text
(8+2) = 10
(9-4) = 5
10*5  = 50
```

