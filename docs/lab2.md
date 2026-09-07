---
title: Memory and Functions in RISC-V
---

<h2 align="center"> CS 21 26.1 Laboratory Exercise 2 </h2>
<h1 align="center"> Memory and Functions in RISC-V </h1>

## Overview
In the previous week, we tackled the B-type instructions for interfacing with the memory and how J-type instructions are used for calling a function in RISC-V. In this laboratory activity, you will test your understanding in last week's lectures and implement a bit more complex RISC-V programs. 

In Lecture 4, you learned how to write a function in RISC-V using the proper storing and restoring of RISC-V registers. A function in RISC-V uses registers `a0` to `a7` for input parameters, while `a0` and `a1` are used to contain the return values of functions.

The stack frame of RISC-V serves as a memory space for 'scratch' work of functions where values of registers may be temporarily stored and restored using the stack pointer `sp` to track the top of the stack. It is good practice for all functions in a RISC-V program to have the same rules for storing and restoring register values in the stack. For simplicity, registers whose values may be overwritten or used by a function are usually temporarily stored in the stack frame before executing the function body. The function restores these values prior to returning to the function's caller. Allocation and deallocation is done by moving the `sp`.

With proper storing and restoring of registers, the implementation of recursive functions is no different from that of nonrecursive functions. Recursive functions will be reserved for the next lab activity. 

## Relevant links

- **RISC-V Green Card** [https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view](https://drive.google.com/file/d/1xSll1ON5cSaOQhoGxpkvr4fKe7lGQFy3/view)

- **RIPES (DCS Fork) Simulator** [https:ripes.upd-dcs.work/](https://ripes.upd-dcs.work/). 

!!! important Single-cycle Processor Setting 
    For the first few lab activities on assembly programming, set the processor to **Single-cycle processor** via the Select processor menu button (a black chip-like image below the File button). Failing to do this may lead to confusing behavior (for now) with the `Step` operation.


## General Instructions

For this laboratory activity, you are to work on **2 Checkpoint Items** that will aid in your understanding of writing instructions that interface with the memory and writing functions. 

!!! warning "Important Reminder"
    You **must finish and submit** your working checkpoint outputs in the Google Classroom **by the end of the laboratory period**. Save your outputs as `lab02_item1_lastname.asm` and `lab02_item2_lastname.asm`. In each of your code submissions, write your name within comment blocks.
