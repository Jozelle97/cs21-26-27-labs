---
title: Lab Exercise 0 (Review on C Programming)
---

<h2 align="center"> CS 21 26.1 Laboratory Exercise 0 </h2>
<h1 align="center"> Review on C Programming </h1>

## Overview

In CS 21, you will be introduced to machine language which is comprised of instructions that computer processors are designed to execute. All computer programs are essentially translated into **machine language**, directly or indirectly, by compilers and interpreters.

High-level programming languages such as Python provide a high level of abstraction that hides the complexity of the underlying machine instructions from programmers; this allows for the creation of complex programs without requiring programmers to delve into the lower-level details of machine language. Machine language is generally cumbersome for humans to process as it is primarily designed for processors. A more human-readable bridging language called assembly language will be introduced so that the study of machine language is made more comprehensible.

Since all programs are translated into machine language (or equivalently, assembly language), understanding the principles behind low-level programs
enables programmers to craft high-level code that results in a more efficient set of machine instructions. Towards this end, you will be writing programs in assembly language; this is relatively more involved as the layers of abstraction of high-level languages that make programming much more manageable are not present at this level.

CS 21 starts off by introducing the C programming language which provides a lower level of abstraction compared to a language such as Python. While fewer abstractions requires more manual effort and attention to detail from the programmer, it enables more fine-grained control in how programs are translated into machine instructions; this makes it ideal for systems programming (e.g., operating systems, limited-resource devices) in which efficient processor execution is usually of greater concern. C shares much more functionally with assembly and machine language than Python does; knowing the fundamentals of C makes the study of assembly and machine language much more manageable, especially coming from a high-level language such as Python.

## General Instructions

In this laboratory activity, you are to work on **1 Checkpoint Task** and **1 Take Home Task** that will review your skills on C programming.

!!! info "Important Reminder"
    You **must finish and show** your working **Checkpoint Task**  to your lab handler before the end of the laboratory period. This lab activity is **non-graded**.

### 
### Checkpoint Item: Banker's Rounding Rule
The Banker's Rounding is a floating-point rounding algorithm specified by the IEEE 754 standard in which a number equidistant from the two nearest integers are rounded to the nearest even integer. The banker's rounding has the following rules:

1. When the number to be rounded (the rounding digit) is followed by `0`,`1`,`2`,`3` or `4`, the rounding digit remains the same.
2. When the number to be rounded (the rounding digit) is followed by a `6`,`7`,`8` or a `9`, the number is rounded up. 
3. When the number to be rounded (the rounding digit) is follwed by a `5`, the rounding rule is determined by the number after `5`:
    - When there is a number after `5`, round up. 
    - When there is no number after `5`, check the number before `5`. If it is an odd number, round up, otherwise round down.

To illustrate, here are some examples when the following numbers are rounded to the `nearest hundredths`:

| `Number`   | `Banker's Rounding Result`  |
| ---------- | --------------------------- | 
| `0.005`    | `0.00`                         | 
| `0.015`    | `0.02`                      | 
| `0.027`    | `0.03`                      | 
| `1.025`    | `1.02`                      | 
| `1.0256`   | `1.03`                      | 
| `5.0287`   | `5.03`                      | 
| `4`        | `4.00`                      | 

Write a C Program that will ask for a number input `N` from the user and print the number rounded to the nearest hundredths using the Banker's Rounding Rule.


### Take Home Task : Replacing Substrings

Implement a function `replace_substring` that takes a string `source` as well as substrings `target` and `replacement`, and returns a string with all instances of `target` in `source` with `replacement`.

Replacements must be done left-to-right. If `target` is not found in `source`, the original `source` string must be returned. If `target` is an empty string, the original `source` string must also be returned.

!!! tip

    The address of a string is the same as the address of the first character of the string in memory.

    The end of a string is always the `NULL` character with a byte value of `0`.

Your function must follow the following template:

#### Restrictions

1. The only headers that may be included are:
   - `<stdio.h>`
   - `<stdlib.h>`
   - `<assert.h>`
   - `<string.h>` - only `strlen()` is allowed to be used
2. Searching via regular expressions is not allowed.
3. Neither `source`, `target`, nor `replacement` can be modified.
4. The substring replacement is case-sensitive.

#### Example Usage

```c
char* replace_substring(const char *source,
                        const char *target,
                        const char *replacement){
    //insert your code here.
}
```

You may use the following testcase to validate your code:

```c
int evaluate_replace_string(char *test_string, char *test_result){
    int i = 0;
    while (test_string[i]||test_result[i]){
        if (test_string[i]!=test_result[i]){
            return 1;
        }
        ++i;
    }
    return 0;
}
int main(){
    const char *test_string = "Thirty-three thirsty, thundering thoroughbreds thumped Mr. Thurber on Thursday.";
    const char *test_substring = "th";
    const char *test_replace = "squw";
    char *test_result = "Thirty-squwree squwirsty, squwundering squworoughbreds squwumped Mr. Thurber on Thursday.";
    char* replaced_string = replace_substring(test_string, test_substring, test_replace);
    printf("Test case passed: %s",evaluate_replace_string(replaced_string, test_result) ? "False" : "True");
}

```

#### Test Cases

You may use these additional test cases to ensure your function is properly implemented.

| `source`                   | `target`  | `replacement`    | Expected String `replaced_string` |
| --------------------- | --------- | ---------------- | --------------------------------- |
| `"Hello hello CS21!"` | `"Hello"` | `"Goodbye"`      | `"Goodbye hello CS21!"`           |
| `"Hello hello CS21!"` | `"hEllO"` | `"goodBye"`      | `"Hello hello CS21!"`             |
| `"ababABABabab"`      | `"ba"`    | `"cal"`          | `"acalbABABacalb"`                |
| `"ggGGGggGgGGGGGGgG"` | `"gg"`    | `"All"`          | `"AllGGGAllGgGGGGGGgG"`           |
| `"aa aaa aaaa aaaaa"` | `"aa"`    | `"rt"`           | `"rt rta rtrt rtrta"`             |
| `"Agree"`             | `"E"`     | `"EEEEEE"`       | `"Agree"`                         |
| `"hello"`             | `"hello"` | `"Hello World!"` | `"Hello World!"`                  |
| `"hello"`             | `"hello"` | `""`             | `""`                              |
| `"run"`               | `"walk"`  | `"sprint"`       | `"run"`                           |
| `"Zero,one"`          | `"o"`     | `"0"`            | `"Zer0,0ne"`                      |
| `"Zer0,0ne"`          | `"0"`     | `"o"`            | `"Zero,one"`                      |
| `"pointer"`           | `""`      | `""`             | `"pointer"`                       |
| `"spa ce dou t"`      | `"space"` | `"tab"`          | `"spa ce dou t"`                  |
| `"0123456789ABCDEF"`  | `"9AB"`   | `"Gabby"`        | `"012345678GabbyCDEF"`            |
| `""`                  | `"9AB"`   | `"Gabby"`        | `""`                              |


