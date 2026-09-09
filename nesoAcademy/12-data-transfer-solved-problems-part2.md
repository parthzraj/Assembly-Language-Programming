# 8085 Microprocessor — Data Transfer Instructions: Solved Problems (Part 2)

> This session revisits **op-code formation** by deriving the op-codes for three specific instructions using the register code chart: `MOV B, C`, `MOV M, D`, and `MOV A, M`.

---

## Table of Contents
1. [Register Code Chart Recap](#register-code-chart-recap)
2. [Problem 1: MOV B, C](#problem-1-mov-b-c)
3. [Problem 2: MOV M, D](#problem-2-mov-m-d)
4. [Problem 3: MOV A, M](#problem-3-mov-a-m)
5. [How to Remember the Register Code Chart Without Having It](#how-to-remember-the-register-code-chart-without-having-it)
6. [Session Summary](#session-summary)

---

## Register Code Chart Recap

All three problems in this session rely on the register code chart established in earlier sessions:

| Register/Memory | Code |
|---|---|
| B | `000` |
| C | `001` |
| D | `010` |
| E | `011` |
| H | `100` |
| L | `101` |
| M (memory via HL) | `110` |
| A (Accumulator) | `111` |

Also recall: Intel's fixed code for the `MOV` mnemonic (register-to-register or register-to-memory) is **`01`**, and all three instructions below are **1-byte long**, following the pattern:

```
01  DDD  SSS
```
where `DDD` = destination register code, `SSS` = source register code.

---

## Problem 1: MOV B, C

**Instruction:** `MOV B, C` — destination = B, source = C

| Part | Code |
|---|---|
| MOV mnemonic | `01` |
| Destination (B) | `000` |
| Source (C) | `001` |

**Assembling the 8 bits:**
```
01   000   001
MOV   B     C
```
Full byte: `01000001`

**Convert to hex** (group into nibbles from LSB to MSB):
- Least significant nibble: `0001` = **1**
- Most significant nibble: `0100` = **4**

**Op-code: `41H`**

---

## Problem 2: MOV M, D

**Instruction:** `MOV M, D` — destination = M (memory via HL), source = D

| Part | Code |
|---|---|
| MOV mnemonic | `01` |
| Destination (M) | `110` |
| Source (D) | `010` |

**Assembling the 8 bits:**
```
01   110   010
MOV   M     D
```
Full byte: `01110010`

**Convert to hex:**
- Least significant nibble: `0010` = **2**
- Most significant nibble: `0111` = **7**

**Op-code: `72H`**

### Addressing mode note
In `MOV M, D`, the value in register **D** (the source) is being moved to the memory location pointed to by the **HL register pair**. Even though HL is never explicitly named in the instruction, it is referenced *indirectly* through the alphabet **M**. This makes `MOV M, D` an example of **register indirect addressing mode**.

---

## Problem 3: MOV A, M

**Instruction:** `MOV A, M` — destination = A (accumulator), source = M (memory via HL)

| Part | Code |
|---|---|
| MOV mnemonic | `01` |
| Destination (A) | `111` |
| Source (M) | `110` |

**Assembling the 8 bits:**
```
01   111   110
MOV   A     M
```
Full byte: `01111110`

**Convert to hex:**
- Least significant nibble: `1110` = **14** = **E**
- Most significant nibble: `0111` = **7**

**Op-code: `7EH`**

---

## Summary of Derived Op-Codes

| Instruction | Op-Code |
|---|---|
| `MOV B, C` | `41H` |
| `MOV M, D` | `72H` |
| `MOV A, M` | `7EH` |

---

## How to Remember the Register Code Chart Without Having It

Some exam/problem questions may **not provide the register code chart directly** — so it's useful to know how to reconstruct it from memory:

1. **3 bits are enough** to represent all registers plus the memory element, since $2^3 = 8$ possible patterns are needed for 7 registers + 1 memory reference.
2. **Order of assignment:**
   - First, assign codes to the **6 general-purpose registers (GPRs)** in this order: **B, C, D, E, H, L** — these naturally take the sequences `000` through `101`.
   - The code `110` (decimal 6) is reserved for **M** — the memory element (pointed to by HL).
   - The **last** sequence, `111`, is reserved for the **accumulator (A)**.

This ordering (`B, C, D, E, H, L → M → A`) is the easiest way to reconstruct the full chart from memory whenever it isn't explicitly given.

---

## Session Summary

- This session reinforced the **op-code derivation process** using the register code chart, applying it to three new instructions: `MOV B, C` (`41H`), `MOV M, D` (`72H`), and `MOV A, M` (`7EH`).
- All three followed the general 1-byte pattern: `01 DDD SSS`.
- `MOV M, D` was highlighted as an example of **register indirect addressing mode**, since the HL pair is referenced indirectly through `M`.
- Even without a provided register code chart, it can be reconstructed by remembering the assignment order: **B, C, D, E, H, L (GPRs) → M (`110`) → A (`111`)**.

---

*Next session: Beginning the Arithmetic Group of Instructions in 8085 microprocessor.*
