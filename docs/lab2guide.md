---
title: CS 21 26.1 Lab Guide 2
---

<h2 align="center"> CS 21 26.1 Lab Guide 2 </h2>
<h1 align="center"> Refactoring Guide and Function Implementation in RISC-V </h1>

When mechanically converting C code to RISC-V assembly, apply the following rules. These rules make it easier to match each C statement with its corresponding assembly instructions.

!!! tip "Rule 1 — Use exactly one operation per line"

    Each line of C code should contain only one arithmetic operation. This creates a clearer correspondence between the C statements and the assembly instructions.

    **Original:**

    ```c
    int x = a + b + c;
    ```

    **Convert to:**

    ```c
    int x = a + b;
    x = x + c;
    ```

!!! tip "Rule 2 — Associate each variable with a register"

    Rename each variable to show the register that will hold its value.

    **Original:**

    ```c
    int n;
    ```

    **Convert to:**

    ```c
    int n_s0;  // Register s0 holds the value of n
    ```

    For example, a variable named `sum` stored in register `s1` may be renamed to `sum_s1`.

!!! tip "Rule 3 — Rename function parameters according to argument registers"

    Function parameters should be renamed according to the RISC-V argument registers assigned to them. The first parameter uses `a0`, the second uses `a1`, and so on.

    **Original:**

    ```c
    int add(int x, int y, int z)
    ```

    **Convert to:**

    ```c
    int add(int x_a0, int y_a1, int z_a2)
    ```
    

    This naming convention makes it clear which RISC-V register contains each parameter.

!!! tip "Rule 4 — Pass arguments using argument registers"

    Each function call should pass its arguments using `a0`, `a1`, and the other RISC-V argument registers.

    **Original:**

    ```c
    f(a, b);
    ```

    **Convert to:**

    ```c
    f(a0, a1);
    ```

!!! tip "Rule 5 — Copy function arguments into local variables"

    At the beginning of a function, copy each argument into the variable or saved register that will hold its value.

    **Original:**

    ```c
    int f(int a0, int a1) {
        // Function body
    }
    ```

    **Convert to:**

    ```c
    int f(int a0, int a1) {
        int s0 = a0;
        int s1 = a1;

        // Function body
    }
    ```

!!! tip "Rule 6 — Return the value through `a0`"

    A function's return value must be placed in `a0`. Ideally, do this near the end of the function.

    **Original:**

    ```c
    return 123;
    ```

    **Convert to:**

    ```c
    int a0 = 123;
    return a0;
    ```

!!! tip "Rule 7 — Convert each branch into its own `if`–`else` statement"

    Rewrite an `if`–`else if`–`else` chain as nested `if`–`else` statements. Compute complex conditions separately before evaluating them.

    **Original:**

    ```c
    void f() {
        if (x + y >= 0) {
            // First case
        } else if (x + y + z >= 0) {
            // Second case
        } else if (will_crash() == 0) {
            // Third case
        } else {
            // Final case
        }
    }
    ```

    **Convert to:**

    ```c
    void f() {
        int temp1 = x + y;

        if (temp1 >= 0) {
            // First case
        } else {
            int temp2 = x + y;
            temp2 = temp2 + z;

            if (temp2 >= 0) {
                // Second case
            } else {
                int temp3 = will_crash();

                if (temp3 == 0) {
                    // Third case
                } else {
                    // Final case
                }
            }
        }
    }
    ```

## C-to-RISC-V Refactoring Example

Suppose we are given the following C program to convert into RISC-V.

Before attempting to convert C code into RISC-V, it is a good idea to refactor the C code so that it follows the rules above.

!!! note "Optional requirements"

    The requirements above are entirely **optional**. You are free to use a different approach that you are more comfortable with unless otherwise stated.

Verify that the refactored code is functionally equivalent to the original C program.

<style>
.code-comparison {
    display: grid;
    grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
    gap: 1.5rem;
    align-items: start;
    width: 100%;
}

.code-column {
    min-width: 0;
}

.code-column h3 {
    margin-top: 0;
    padding: 0.6rem;
    text-align: center;
    background-color: #eeeeee;
    border-radius: 0.25rem;
}

.code-column pre {
    max-height: 75vh;
    overflow: auto;
    font-size: 0.75rem;
}

@media (max-width: 900px) {
    .code-comparison {
        grid-template-columns: 1fr;
    }
}
</style>

<div class="code-comparison" markdown="1">

<div class="code-column" markdown="1">

### Original C Program

```c
#include <stdio.h>

int g(int n) {
    return n * (n - 1);
}

int f(int n) {
    printf("%d\n", n);

    int k = g(n);

    if (n <= 1) {
        return k;

    } else if (n == 2) {
        return 1;

    } else {
        int ret = k + f(n - 1);
        printf("%d\n", n);

        return ret;
    }
}

int main() {
    for (int k = -5; k <= 5; k++) {
        printf("f(%d): %d\n", k, f(k));
    }
}
```

</div>

<div class="code-column" markdown="1">

### Refactored C Program

```c
#include <stdio.h>

int g(int a0) {
    int n_s0 = a0;

    int s1 = n_s0 - 1;
    s1 = n_s0 * s1;

    a0 = s1;
    return a0;
}

int f(int a0) {
    int n_s0 = a0;

    printf("%d", n_s0);
    printf("\n");

    a0 = n_s0;
    a0 = g(a0);
    int k_s1 = a0;

    int tmp_s2 = 1;

    if (n_s0 <= tmp_s2) {
        a0 = k_s1;
        return a0;

    } else {
        tmp_s2 = 2;

        if (n_s0 == tmp_s2) {
            a0 = 1;
            return a0;

        } else {
            a0 = n_s0 - 1;
            a0 = f(a0);
            int ret_s3 = k_s1 + a0;

            printf("%d", n_s0);
            printf("\n");

            a0 = ret_s3;
            return a0;
        }
    }
}

int main() {
    int k_s0 = -5;
    int _5_s2 = 5;

    while (k_s0 <= _5_s2) {
        int a0 = k_s0;
        a0 = f(a0);
        int fk_s1 = a0;

        printf("f(");
        printf("%d", k_s0);
        printf("): ");
        printf("%d", fk_s1);
        printf("\n");

        k_s0++;
    }
}
```

</div>

</div>

Having a C program in this form ensures we _(roughly)_ have a one-to-one correspondence between lines of code in C and RISC-V instructions, turning the remaining steps into a mostly _mechanical_ process.

Let us start with `main`, taking note of all registers that are **modified**:

## Converting `main` to RISC-V

The refactored C implementation of `main` and its corresponding RISC-V assembly are shown side by side.

<div class="code-comparison" markdown="1">

<div class="code-column" markdown="1">

### RISC-V Assembly

```asm
.text

# Modifies ra, s0, s1, s2, a0, and a7

main:
    # int k_s0 = -5;
    li s0, -5

    # int _5_s2 = 5;
    li s2, 5

main__loop:
    # while (k_s0 <= _5_s2)
    blt s2, s0, main__after

    # int a0 = k_s0;
    # a0 = f(a0);
    # int fk_s1 = a0;
    mv a0, s0
    call f
    mv s1, a0

    # printf("f(");
    li a7, 4
    la a0, str1
    ecall

    # printf("%d", k_s0);
    li a7, 1
    mv a0, s0
    ecall

    # printf("): ");
    li a7, 4
    la a0, str2
    ecall

    # printf("%d", fk_s1);
    li a7, 1
    mv a0, s1
    ecall

    # printf("\n");
    li a7, 11
    li a0, 10
    ecall

    # k_s0++;
    addi s0, s0, 1

    # Repeat the loop
    j main__loop

main__after:
    # Exit the program
    li a7, 10
    ecall

.data

str1: .asciz "f("
str2: .asciz "): "
```

</div>

<div class="code-column" markdown="1">

### Equivalent C Code

```c
int main() {
    int k_s0 = -5;
    int _5_s2 = 5;

    while (k_s0 <= _5_s2) {
        int a0 = k_s0;
        a0 = f(a0);
        int fk_s1 = a0;

        printf("f(");
        printf("%d", k_s0);
        printf("): ");
        printf("%d", fk_s1);
        printf("\n");

        k_s0++;
    }
}
```

</div>

</div>

For now, we can define `f` as a _stub_ ([Wikipedia](https://en.wikipedia.org/wiki/Test_stub)) that prints the argument it is given and returns it just so we can check the correctness of the other parts of our code so far:

```mips
.text

# `main` omitted for brevity

# Modifies a7, a0
f:           # int f(int a0) {
  li a7, 1   #   printf("%d");
  ecall

  li a7, 11  #   printf("\n");
  li a0, 10
  ecall

  ret        #   return a0;
             # }

# `.data` omitted for brevity
```

Verify that the program works as intended stub-wise.

As we are fairly confident that `main` works as intended at this point, we can now proceed with the mechanical translation of the lines of `f`.

!!! tip "Tip: Define a unique return label per function"

    One trick to simplify function conversion is to **have only one `ret` at the end of the function body** and to have a label that refers to _"the end of the function"_.

    A suggestion for the label name is the function name with the suffix `__ret` _(e.g., `f__ret`)_.

    Jumping to the label should be synonymous for the function returning the value in `a0`.

Applying the said tip and doing things step-by-step, we would have the following _(note that this is still **incorrect**)_:

## Converting Function `f` to RISC-V

The refactored C implementation and its corresponding RISC-V assembly are shown side by side.

<div class="code-comparison" markdown="1">

<div class="code-column" markdown="1">

### RISC-V Assembly

```asm
# Modifies ra, a0, a7, s0, s1, s2, s3

f:
    # int n_s0 = a0;
    mv s0, a0

    # printf("%d", n_s0);
    li a7, 1
    mv a0, s0
    ecall

    # printf("\n");
    li a7, 11
    li a0, 10
    ecall

    # a0 = n_s0;
    # a0 = g(a0);
    # int k_s1 = a0;
    mv a0, s0
    call g
    mv s1, a0

    # int tmp_s2 = 1;
    li s2, 1

    # if (n_s0 <= tmp_s2)
    blt s2, s0, f__else

    # a0 = k_s1;
    # return a0;
    mv a0, s1
    j f__ret

f__else:
    # tmp_s2 = 2;
    li s2, 2

    # if (n_s0 == tmp_s2)
    bne s0, s2, f__else_2

    # a0 = 1;
    # return a0;
    li a0, 1
    j f__ret

f__else_2:
    # a0 = n_s0 - 1;
    # a0 = f(a0);
    addi a0, s0, -1
    call f

    # int ret_s3 = k_s1 + a0;
    add s3, s1, a0

    # printf("%d", n_s0);
    li a7, 1
    mv a0, s0
    ecall

    # printf("\n");
    li a7, 11
    li a0, 10
    ecall

    # a0 = ret_s3;
    # return a0;
    mv a0, s3
    j f__ret

f__ret:
    ret
```

</div>

<div class="code-column" markdown="1">

### Equivalent C Code

```c
int f(int a0) {
    int n_s0 = a0;

    printf("%d", n_s0);
    printf("\n");

    a0 = n_s0;
    a0 = g(a0);
    int k_s1 = a0;

    int tmp_s2 = 1;

    if (n_s0 <= tmp_s2) {
        a0 = k_s1;
        return a0;

    } else {
        tmp_s2 = 2;

        if (n_s0 == tmp_s2) {
            a0 = 1;
            return a0;

        } else {
            a0 = n_s0 - 1;
            a0 = f(a0);
            int ret_s3 = k_s1 + a0;

            printf("%d", n_s0);
            printf("\n");

            a0 = ret_s3;
            return a0;
        }
    }
}
```

</div>

</div>

!!! example "Self-check"

    The comment says that `f` modifies `ra`. Where exactly does `f` modify `ra`?

!!! example "Self-check"

    Why is the first `mv a0, s0` redundant? Why would it still be beneficial to retain it despite the redundancy?

Recall that in RISC-V, registers _("variables")_ are **shared** across functions. To help bridge things with C, you may think of **all** RISC-V registers being global variables in C.

In the scenario that `f` calls `g`, `f` is the **caller** while `g` is the **callee**. You may think of the callee as **borrowing all the registers** from the caller, and that when the registers are "returned" to the caller, the callee must revert the registers _(except `a0` and `sp`)_ into their old values.

A caller may do this by **pushing** all the original values of all registers it will be overwriting into _stack memory_ in order to _save_ their values. 

Recall that the _stack pointer_ `sp` points to the top element of the stack, pushing or saving the value of `ra` may be done as follows:

```mips
addi sp, sp, -4
sw ra, 0(sp)
```

As a simplification, we adopt the practice that all registers that _may_ be overwritten by a function _(except `a0` and `sp`)_ are pushed into the stack at the very start of the function:

## Adding a Stack Frame to Function `f`

The code below attempts to preserve registers using the stack. This version is intentionally incomplete and still contains errors that must be identified and corrected.

!!! warning "This implementation is still incorrect"

    Compare the RISC-V assembly with the equivalent C code. Determine which registers are stored incorrectly or not stored at all, and whether the stack frame is properly removed before returning.

```mips
# `main` omitted for brevity 

# Modifies ra, a0, a7, s0, s1, s2
f:                       # int f(int a0) {
  addi sp, sp, -4        # PUSH ra
  sw ra, 0(sp)
  addi sp, sp, -4        # PUSH a7
  sw a7, 0(sp)
  addi sp, sp, -4        # PUSH s0
  sw s0, 0(sp)
  addi sp, sp, -4        # PUSH s1
  sw s1, 0(sp)
  addi sp, sp, -4        # PUSH s2
  sw s2, 0(sp)

  mv s0, a0              #   int n_s0 = a0;

  li a7, 1               #   printf("%d", n_s0);
  mv a0, s0
  ecall

  li a7, 11              #   printf("\n");
  li a0, 10
  ecall

  mv a0, s0              #   a0 = n_s0;
  call g                 #   a0 = g(a0);
  mv s1, a0              #   int k_s1 = a0;

  li s2, 1               #   int tmp_s2 = 1;

  blt s2, s0, f__else    #   if (n_s0 <= tmp_s2) {
  mv a0, s1              #     a0 = k_s1;
  j f__ret               #     return a0;
f__else:                 #   } else {
  li s2, 2               #     tmp_s2 = 2;
  bne s0, s2, f__else_2  #     if (n_s0 == tmp_s2) {
  li a0, 1               #       a0 = 1;
  j f__ret               #       return a0;
f__else_2:               #     } else {
  addi a0, s0, -1        #       a0 = n_s0 - 1;
  call f                 #       a0 = f(a0);
  add s3, s1, a0         #       int ret_s3 = k_s1 + a0;
  
  li a7, 1               #       printf("%d", n_s0);
  mv a0, s0
  ecall

  li a7, 11              #       printf("\n");
  li a0, 10
  ecall

  mv a0, s3              #       a0 = ret_s3;
  j f__ret               #       return a0;
                         #     }
                         #   }
                         # }
f__ret:
  lw s2, 0(sp)           # s2 = POP
  addi sp, sp, 4
  lw s1, 0(sp)           # s1 = POP
  addi sp, sp, 4
  lw s0, 0(sp)           # s0 = POP
  addi sp, sp, 4
  lw a7, 0(sp)           # ra = POP
  addi sp, sp, 4
  lw ra, 0(sp)           # s2 = POP
  addi sp, sp, 4
  ret

# `.data` omitted for brevity 
```

Before a function returns, it must restore the original values of the registers it "borrowed" from its caller.

Notice that the last value pushed by `f` into the stack is that of `s2`. It then follows that the top of the stack contains the original value of `s2`.

If we pop from the stack, we will get the value of `s2`. Popping this value and placing it in `s2` can be done as follows:

```mips
lw s2, 0(sp)           # s2 = POP
addi sp, sp, 4
```

Restoring the original values of the registers borrowed by a callee can be done in its return label. We can pop the values we pushed for `f` and restore them in **reverse order** _(last in, first out)_: 

Verify from the code above that the proper storing and restoring of values are done for function `f`.

### Converting Function `g` to RISC-V

With `f` a proper function now, we can continue the mechanical conversion for `g`. Don't forget to also create its own stack frame.

```mips
# `main` and `f` omitted for brevity 

# Modifies s0, s1
g:                 # int g(int a0) {
  addi sp, sp, -4  # PUSH s0
  sw s0, 0(sp)
  addi sp, sp, -4  # PUSH s1
  sw s1, 0(sp)
  
  mv s0, a0        #   int n_s0 = a0;
  addi s1, s0, -1  #   int s1 = n_s0 - 1;
  mul s1, s0, s1   #   s1 = n_s0 * s1;

  mv a0, s1        #   a0 = s1;
g__ret:
  lw s1, 0(sp)     # s1 = POP
  addi sp, sp, 4
  lw s0, 0(sp)     # s0 = POP
  addi sp, sp, 4
  ret              #   ret
                   # }

# `.data` omitted for brevity 
```

Verify that the entire program now works.

