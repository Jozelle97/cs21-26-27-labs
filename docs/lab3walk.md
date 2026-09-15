---
title: CS 21 26.1 Lab 3 Example Walkthrough
---

<h2 align="center"> CS 21 26.1 Lab 3 Walkthrough </h2>
<h1 align="center"> Function Call Sequence for `parse_expression("(2+10)*2")` </h1>

## Relevant links

- **Function Call Visualizer** [Open the Recursive Parser Stack Visualizer](demos/index2.html)

## Important Result

Under the laboratory grammar, every operation must be fully parenthesized:

```text
expression := number
            | '(' expression operator expression ')'
```

Therefore, the exact input

```text
(2+10)*2
```

is **not a complete valid expression**. The call to `parse_expression` successfully evaluates `(2+10)` as `12`, but it stops at `*`. The main program then detects the unprocessed `*2` and reports a syntax error.

To calculate the entire expression, the input must be:

```text
((2+10)*2)
```

---

## Input Character Positions

The exact requested input is stored as a null-terminated string:

```text
(2+10)*2\0
```

| Index | Character |
|---:|:---:|
| 0 | `(` |
| 1 | `2` |
| 2 | `+` |
| 3 | `1` |
| 4 | `0` |
| 5 | `)` |
| 6 | `*` |
| 7 | `2` |
| 8 | `\0` |

The initial call is:

```text
parse_expression(cursor = 0)
```

---

## Step 1: Begin Parsing `(2+10)`

`parse_expression` examines the character at index 0.

```text
Current character = '('
```

Because the current character is `(`, the function:

1. Advances the cursor to index 1.
2. Recursively calls `parse_expression` to evaluate the left operand.

```text
parse_expression(cursor = 0)
└── parse_expression(cursor = 1)
```

The first call remains on the program stack while the recursive call processes the left operand.

---

## Step 2: Parse the Left Operand `2`

The recursive call examines index 1.

```text
Current character = '2'
```

Because `2` is a digit, `parse_expression` calls `parse_number`:

```text
parse_number(cursor = 1, accumulated = 0, digit_count = 0)
```

`parse_number` calculates:

```text
digit_value           = '2' - '0' = 2
new_accumulated_value = 0 × 10 + 2 = 2
```

It then recursively calls itself for index 2:

```text
parse_number(cursor = 1, value = 0, count = 0)
└── parse_number(cursor = 2, value = 2, count = 1)
```

Index 2 contains `+`, which is not a digit. This is the base case, so `parse_number` returns:

```text
a0 = address of index 2
a1 = 2
a2 = 0
```

The recursive `parse_expression` call also returns:

```text
Left operand = 2
Cursor       = index 2
```

---

## Step 3: Read the `+` Operator

Control returns to the `parse_expression` call that started at index 0.

The function reads the character at index 2:

```text
Operator = '+'
```

Before making another function call, it must preserve:

```text
Left operand = 2
Operator     = '+'
```

The function advances the cursor to index 3 and recursively calls `parse_expression` for the right operand:

```text
parse_expression(cursor = 0)
└── parse_expression(cursor = 3)
```

---

## Step 4: Parse the Right Operand `10`

The recursive call examines index 3.

```text
Current character = '1'
```

Because `1` is a digit, `parse_expression` calls:

```text
parse_number(cursor = 3, accumulated = 0, digit_count = 0)
```

### Process the digit `1`

```text
digit_value           = '1' - '0' = 1
new_accumulated_value = 0 × 10 + 1 = 1
```

The function recursively processes index 4:

```text
parse_number(cursor = 4, accumulated = 1, digit_count = 1)
```

### Process the digit `0`

```text
digit_value           = '0' - '0' = 0
new_accumulated_value = 1 × 10 + 0 = 10
```

The function recursively processes index 5:

```text
parse_number(cursor = 5, accumulated = 10, digit_count = 2)
```

Index 5 contains `)`, which is not a digit. The base case returns:

```text
a0 = address of index 5
a1 = 10
a2 = 0
```

The complete number-parser call sequence is:

```text
parse_number(index 3, value 0, count 0)
└── parse_number(index 4, value 1, count 1)
    └── parse_number(index 5, value 10, count 2)
        └── returns value 10 at ')'
```

The recursive `parse_expression` call returns:

```text
Right operand = 10
Cursor        = index 5
Status        = success
```

---

## Step 5: Complete `(2+10)`

The original `parse_expression` call now has:

```text
Left operand  = 2
Operator      = '+'
Right operand = 10
Cursor        = index 5
```

It verifies that index 5 contains the required closing parenthesis:

```text
Current character = ')'
```

The function advances the cursor to index 6 and performs the operation:

```text
2 + 10 = 12
```

It returns:

```text
a0 = address of index 6
a1 = 12
a2 = 0
```

At this point, `parse_expression` has successfully evaluated only:

```text
(2+10)
```

---

## Step 6: Detect the Remaining `*2`

The main program examines the character at the returned cursor.

```text
Returned cursor = index 6
Character       = '*'
```

A successful complete parse must return with the cursor pointing to `\0`. Because the cursor points to `*`, some input remains:

```text
*2
```

The main program therefore reports:

```text
Error: invalid expression
```

The value `12` is only an intermediate result and must not be printed as the final answer for the complete input.

---

## Condensed Call Sequence

```text
parse_expression("(2+10)*2")
│
├── sees '('
│
├── parse_expression("2+10)*2")
│   └── parse_number("2+10)*2", 0, 0)
│       └── parse_number("+10)*2", 2, 1)
│           └── returns value 2 at '+'
│
├── saves left operand 2
├── reads '+'
│
├── parse_expression("10)*2")
│   └── parse_number("10)*2", 0, 0)
│       └── parse_number("0)*2", 1, 1)
│           └── parse_number(")*2", 10, 2)
│               └── returns value 10 at ')'
│
├── verifies ')'
└── returns value 12 with cursor at '*'

main
└── detects remaining "*2"
    └── reports syntax error
```

---

## Correct Form: `((2+10)*2)`

To include multiplication in the expression parsed by the outermost call, add another pair of parentheses:

```text
((2+10)*2)
```

Its high-level call sequence is:

```text
parse_expression("((2+10)*2)")
│
├── parse_expression("(2+10)*2)")
│   ├── parse_expression("2+10)*2)")
│   │   └── parse_number(...) returns 2
│   ├── reads '+'
│   ├── parse_expression("10)*2)")
│   │   └── parse_number(...) returns 10
│   └── returns 2 + 10 = 12
│
├── reads '*'
│
├── parse_expression("2)")
│   └── parse_number(...) returns 2
│
└── returns 12 × 2 = 24 at '\0'
```

Final result:

```text
Expression: ((2+10)*2)
Result: 24
Status: success
```

## Summary

| Input | `parse_expression` result | Remaining input | Final program result |
|---|---:|---|---|
| `(2+10)*2` | `12` | `*2` | Syntax error |
| `((2+10)*2)` | `24` | None | Success |
