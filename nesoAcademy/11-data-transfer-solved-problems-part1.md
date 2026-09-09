# 8085 Microprocessor — Data Transfer Instructions: Solved Problems (Part 1)

> This session applies the Data Transfer Group concepts learned so far by **distinguishing between 4 pairs of easily-confused instructions**.

---

## Table of Contents
1. [Problem 1: LXI H, 1234H vs LHLD 1234H](#problem-1-lxi-h-1234h-vs-lhld-1234h)
2. [Problem 2: LDA F900H vs STA F900H](#problem-2-lda-f900h-vs-sta-f900h)
3. [Problem 3: MVI M, ADH vs LXI H, 008DH](#problem-3-mvi-m-adh-vs-lxi-h-008dh)
4. [Problem 4: LHLD FA00H vs SHLD FA00H](#problem-4-lhld-fa00h-vs-shld-fa00h)
5. [Session Summary](#session-summary)

---

## Problem 1: LXI H, 1234H vs LHLD 1234H

> Distinguish between `LXI H, 1234H` and `LHLD 1234H`.

### `LXI H, 1234H`
- **LXI** = Load extended register (register pair) with an **immediate** value.
- `H` here refers to the **HL register pair** (H is the register name; L is its extension).
- `1234H` is the **immediate 16-bit data** sent directly within the instruction.

**Execution:**
- The higher-order byte (`12`) is loaded into the **H register**.
- The lower-order byte (`34`) is loaded into the **L register**.
- After this, the value inside HL (`1234H`) can be **treated as an address** — i.e., the microprocessor now knows HL is "pointing to" memory location `1234H`.

> **Key point:** `1234H` here is **data** that gets loaded directly into the HL pair.

### `LHLD 1234H`
- **LHLD** = Load HL pair using **direct addressing** from memory.
- `1234H` here is the **starting address** in memory — not data to be loaded directly into HL.

**Example setup:** Suppose memory locations `1234H` and `1235H` (consecutive) contain `CD` and `AB` respectively.

**Execution:**
1. The microprocessor points to memory location `1234H` (the address given in the instruction).
   - The value there (`CD`) is loaded into the **L register**.
2. The microprocessor then points to the **next consecutive location**, `1235H`.
   - The value there (`AB`) is loaded into the **H register**.

> **Key point:** `1234H` here is a **memory address**, and the actual data loaded into HL comes from **that memory location and the next one**.

### Key Difference

| | `LXI H, 1234H` | `LHLD 1234H` |
|---|---|---|
| What `1234H` represents | **Data** — loaded directly into HL | **Address** — points to where the data resides in memory |
| Where HL's final value comes from | Directly from the instruction | From two consecutive memory locations starting at `1234H` |

---

## Problem 2: LDA F900H vs STA F900H

> Distinguish between `LDA F900H` and `STA F900H`.

### `LDA F900H`
- **LDA** = Load the **accumulator** with the content of the specified memory location.
- This is a **direct addressing mode** instruction (the address is given directly in the instruction).

**Example:** Suppose memory location `F900H` contains the value `AB`.

**Execution:**
- The microprocessor points directly to `F900H` (from the address in the instruction).
- The value there (`AB`) is loaded into the **accumulator**.

> **Direction of data flow:** Memory → Accumulator

### `STA F900H`
- **STA** = Store the content of the **accumulator** into the specified memory location.
- Also a **direct addressing mode** instruction.

**Example:** Suppose the accumulator currently holds the value `12`.

**Execution:**
- The microprocessor points directly to `F900H` (from the address in the instruction).
- The accumulator's content (`12`) is stored into that memory location.

> **Direction of data flow:** Accumulator → Memory

### Key Difference

| | `LDA F900H` | `STA F900H` |
|---|---|---|
| Data flow direction | Memory → Accumulator | Accumulator → Memory |
| Function | **Loads** the accumulator from memory | **Stores** the accumulator into memory |

---

## Problem 3: MVI M, ADH vs LXI H, 008DH

> Distinguish between `MVI M, ADH` and `LXI H, 008DH`.

### `MVI M, ADH`
- **MVI** = Move Immediate. Moves an 8-bit value directly (given in the instruction) into a destination.
- Destination here is `M` — the memory location pointed to by the **HL register pair**.
- **Precondition:** The HL register pair must **already** contain the intended address before this instruction executes.

**Example:** Suppose HL already contains `1234H` (so the memory location `1234H` is being pointed to).

**Execution:**
- The 8-bit immediate value `ADH` is copied into memory location `1234H` (the location pointed to by HL).

> **Key point:** `ADH` is **data** that gets written into memory (at the address currently held in HL).

### `LXI H, 008DH`
- **LXI** = Load extended register (HL pair) with an immediate 16-bit value.
- This instruction loads the value **directly into the HL register pair itself.**

**Execution:**
- HL is loaded with the value `008DH`.
- This value can later be treated as an **address** — i.e., HL now "points to" memory location `008DH`.

> **Key point:** `008DH` is **data loaded into HL**, which HL will later use/interpret as a memory address.

### Key Difference

| | `MVI M, ADH` | `LXI H, 008DH` |
|---|---|---|
| Destination | Memory location pointed to by HL | The HL register pair itself |
| What the value represents | 8-bit **data** written into memory | 16-bit **data** loaded into HL (may later serve as an address) |
| Precondition | HL must already hold the target address | None — this instruction sets HL's value |

---

## Problem 4: LHLD FA00H vs SHLD FA00H

> Distinguish between `LHLD FA00H` and `SHLD FA00H`.

### `LHLD FA00H`
- **LHLD** = Load HL pair using **direct addressing** — loading FROM memory INTO HL.

**Example setup:** Suppose memory location `FA00H` contains `34`, and the consecutive location `FA01H` contains `12`.

**Execution:**
1. The microprocessor points to `FA00H` (the address given in the instruction).
   - The value there (`34`) is loaded into the **L register**.
2. The microprocessor then points to the consecutive location `FA01H`.
   - The value there (`12`) is loaded into the **H register**.

> **Direction of data flow:** Memory → HL pair

### `SHLD FA00H`
- **SHLD** = Store HL pair using **direct addressing** — storing FROM HL INTO memory.

**Example setup:** Suppose the HL register pair currently holds `ABCD` (H = `AB`, L = `CD`).

**Execution:**
1. The microprocessor points to `FA00H` (the address given in the instruction).
   - The content of the **L register** (`CD`) is stored there **first**.
2. The microprocessor then points to the consecutive location `FA01H`.
   - The content of the **H register** (`AB`) is stored there.

> **Direction of data flow:** HL pair → Memory

### Key Difference

| | `LHLD FA00H` | `SHLD FA00H` |
|---|---|---|
| Data flow direction | Memory → HL pair | HL pair → Memory |
| First memory location (`FA00H`) | Its content loads into **L** | Receives content **from L** |
| Consecutive memory location (`FA01H`) | Its content loads into **H** | Receives content **from H** |

---

## Session Summary

This session reinforced key distinctions across four commonly-confused instruction pairs:

1. **`LXI H, d16` vs `LHLD a16`** — one loads **data directly** into HL; the other loads HL's content **from memory**, using the given address as a **pointer**, not as data.
2. **`LDA a16` vs `STA a16`** — opposite data flow directions between memory and the accumulator.
3. **`MVI M, d8` vs `LXI H, d16`** — one writes 8-bit data **into memory** (via HL as a pointer); the other loads 16-bit data **into HL itself**.
4. **`LHLD a16` vs `SHLD a16`** — opposite data flow directions between memory and the HL pair; in both cases, **L corresponds to the first (given) address** and **H corresponds to the next consecutive address**.

**General pattern to remember:**
- Instructions starting with **L** (Load) → data flows **into** a register/pair.
- Instructions starting with **S** (Store) → data flows **out to** memory.
- When an instruction's operand looks like a 16-bit value, always check whether it's being treated as **immediate data** (loaded directly) or as an **address** (used to point to memory) — this is the core distinction across all four problems above.

---

*Next session: More solved problems on Data Transfer instructions.*
