# 8085 Microprocessor — Register Codes

> This session is an **introduction to how op-codes are formed**. Before diving into op-code formation (next session), we first need to understand **Register Codes** — the bit patterns used to refer to each register.

---

## Table of Contents
1. [Why Do We Need Register Codes?](#why-do-we-need-register-codes)
2. [Analogy: Addressing Memory](#analogy-addressing-memory)
3. [How Many Bits Are Needed to Address the Registers?](#how-many-bits-are-needed-to-address-the-registers)
4. [The Register Code Table](#the-register-code-table)
5. [The Leftover Sequence — Referring to Memory (M)](#the-leftover-sequence--referring-to-memory-m)
6. [Session Summary](#session-summary)

---

## Why Do We Need Register Codes?

Up to this point, several **Data Transfer Group** instructions have been covered (e.g., `MOV`, `MVI`, `LXI`, `LDA a16`, `STA a16`, `XCHG`, etc.).

To understand **how the actual op-codes (machine instructions) for these instructions are formed**, it's first necessary to understand how the **8085 microprocessor internally refers to each register** using a fixed binary pattern — this is called a **register code**.

This session builds the foundation; the **next session** will use these register codes to show how op-codes are actually constructed.

---

## Analogy: Addressing Memory

Recall from earlier study of the 8085 architecture:

- The 8085 has an **address bus** made up of:
  - Pins **28 to 21** and **19 to 12**, corresponding to address lines **A15–A8** and **A7–A0**.
  - That's **16 address pins total**, all working **in parallel**.
- These 16 bits are sent to the **address decoder** of the memory.
- With 16 bits, the microprocessor can address:
$$2^{16} = 65{,}536 \text{ locations} = 64K \text{ locations}$$

This is the same underlying idea used for **registers** — just on a much smaller scale, since there are far fewer registers than memory locations.

---

## How Many Bits Are Needed to Address the Registers?

The 8085 has registers that are **accessible to the programmer**:
- **6 General Purpose Registers (GPRs):** B, C, D, E, H, L
- **1 Special Purpose Register:** Accumulator (A)

**Total = 7 programmer-accessible registers.**

To uniquely address 7 registers, we need enough bits to generate at least 7 distinct binary patterns.

- **3 bits** → gives $2^3 = 8$ possible patterns (from `000` to `111`).
- 8 patterns is enough to cover all 7 registers (with **1 pattern left over**).

So, **3-bit register codes** are used in the 8085 to identify registers.

---

## The Register Code Table

Intel assigned the following 3-bit codes to the GPRs and the accumulator:

| Register | Binary Code |
|---|---|
| B | `000` |
| C | `001` |
| D | `010` |
| E | `011` |
| H | `100` |
| L | `101` |
| A (Accumulator) | `111` |

> Note: These codes follow a fixed, standardized scheme used internally by the 8085 to reference registers within op-codes.

---

## The Leftover Sequence — Referring to Memory (M)

Since 3 bits give **8 total patterns** but only **7 registers** exist, there is **1 unused pattern**: `110`.

Instead of wasting this pattern, **Intel cleverly reused it**:

- Recall that the **HL register pair** can be used to **point to a memory location** (as seen with instructions like `MOV A, M`).
- The leftover code `110` is assigned to represent **M** — i.e., **the memory location pointed to by the HL register pair.**

| Code | Refers To |
|---|---|
| `110` | **M** — memory location pointed to by HL |

This means the same 3-bit addressing scheme covers **all 6 GPRs + accumulator + a reference to memory (via HL)** — a total of **8 distinct references** using just 3 bits, with **zero wastage**.

---

## Session Summary

- Just as **16-bit addresses** are used to reference **64K memory locations**, **3-bit register codes** are used to reference the 8085's programmer-accessible registers.
- There are **7 programmer-accessible registers**: B, C, D, E, H, L (GPRs) + A (accumulator).
- **3 bits** produce 8 possible codes — 7 are assigned to the registers, and the **8th (`110`) is assigned to M**, representing the memory location pointed to by the HL register pair.

### Full Reference Table

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

---

*Next session topic: Formation of Op-Codes using Register Codes.*
