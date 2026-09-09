# 8085 Microprocessor — LDAX RP and STAX RP Instructions

> Resuming the **Data Transfer Group of Instructions** series after the detour into Register Codes, Op-Code Formation, and Addressing Modes. This session covers `LDAX RP` and `STAX RP`.

---

## Table of Contents
1. [Recap: Why We Took a Detour](#recap-why-we-took-a-detour)
2. [LDAX RP — Load Accumulator via Extended Register](#ldax-rp--load-accumulator-via-extended-register)
3. [Why There's No LDAX H](#why-theres-no-ldax-h)
4. [Why There's No LBX RP or LCX RP](#why-theres-no-lbx-rp-or-lcx-rp)
5. [STAX RP — Store Accumulator via Extended Register](#stax-rp--store-accumulator-via-extended-register)
6. [Why There's No STAX H](#why-theres-no-stax-h)
7. [Session Summary](#session-summary)

---

## Recap: Why We Took a Detour

Before this session, several data transfer instructions had already been covered, but it was **unclear which instruction types belonged to which addressing modes**. To clarify this, the series took a detour to study:
1. **Register codes**
2. **Formation of op-codes**
3. **Addressing modes**

With that foundation in place, this session resumes the data transfer group — specifically covering `LDAX RP` and `STAX RP`.

---

## LDAX RP — Load Accumulator via Extended Register

### Meaning
- **LDAX** = **L**oa**d** **A**ccumulator from memory pointed to by the e**x**tended register.
- **"Extended register"** = a **register pair** (formed by combining two general-purpose registers).

### Instruction naming convention
- Only the **first (higher) register's name** of the pair is mentioned in the instruction — e.g., for the **BC** pair, only `B` appears, since **C is treated as the "extension" of B**.
- Specific instructions:
  - `LDAX B` → refers to the **BC** register pair
  - `LDAX D` → refers to the **DE** register pair

### Addressing mode
- The instruction does **not** mention the memory location directly — instead, a **register pair** is used to indirectly point to the memory.
- This makes `LDAX RP` a **register indirect addressing mode** instruction (as covered in the previous session).

### Instruction length
- `LDAX RP` is a **1-byte long instruction**.

### Example 1: `LDAX B`
Suppose memory location `F820H` contains the value `12`.

**Steps:**
1. First, load the address `F820H` into the **BC register pair** (so that BC now points to that memory location).
2. Execute `LDAX B`.
3. Result: The value at the memory location pointed to by BC (i.e., `12`) is loaded into the **accumulator**.

### Example 2: `LDAX D`
Suppose memory location `F821H` contains the value `34`.

**Steps:**
1. First, load the address `F821H` into the **DE register pair** (so that DE now points to that memory location).
2. Execute `LDAX D`.
3. Result: The value at the memory location pointed to by DE (i.e., `34`) is loaded into the **accumulator**.

---

## Why There's No LDAX H

A natural question: if `LDAX B` and `LDAX D` exist, why isn't there an `LDAX H`?

**Answer:** The functionality of `LDAX H` would be **redundant** — it already exists in the form of `MOV A, M`:
- `MOV A, M` loads the accumulator with the 8-bit value from the memory location pointed to by the **HL register pair**.
- This is *also* an example of **register indirect addressing**.
- Since this functionality is already covered by an existing instruction (`MOV A, M`), Intel did not create a separate, duplicate instruction (`LDAX H`) for the same purpose.

> Takeaway: `LDAX RP` only has **two specific instructions**: `LDAX B` and `LDAX D`. There is no `LDAX H`.

---

## Why There's No LBX RP or LCX RP

Another natural question: why is there no instruction like `LBX RP` or `LCX RP` (loading register B or C via an extended register, instead of the accumulator)?

**Answer:** This is because the **accumulator is a special-purpose register** with multiple unique ways of being addressed (as established in earlier sessions). General-purpose registers (like B or C) do not have this same special direct-addressing treatment — only the accumulator does.

---

## STAX RP — Store Accumulator via Extended Register

### Meaning
- **STAX** = **St**ore **A**ccumulator contents into memory pointed to by the e**x**tended register (register pair).
- This is the **reverse** of `LDAX RP`: instead of loading data *into* the accumulator, it stores the accumulator's content *into* memory.

### Addressing mode
- Just like `LDAX RP`, the memory location is not mentioned directly — only the register pair is mentioned.
- This is also a **register indirect addressing mode** instruction.

### Instruction length
- `STAX RP` is also a **1-byte long instruction**.

### Specific instructions
- `STAX B` → stores the accumulator's content into the memory location pointed to by the **BC** register pair.
- `STAX D` → stores the accumulator's content into the memory location pointed to by the **DE** register pair.

> In both cases, the register pair must first be loaded with the intended memory address before executing `STAX B` or `STAX D`.

---

## Why There's No STAX H

Just like `LDAX H` doesn't exist, there is also **no `STAX H`** — for a similar reason.

**Answer:** The functionality of `STAX H` is already covered by the existing instruction `MOV M, A`:
- `MOV M, A` stores the content of the accumulator into the memory location pointed to by the **HL register pair**.
- This is the **opposite** of `MOV A, M` (used earlier to explain why `LDAX H` doesn't exist).
- Since `MOV M, A` already performs this exact function, there is no need for a duplicate `STAX H` instruction.

---

## Session Summary

| Instruction Type | Meaning | Specific Instructions | Length | Addressing Mode |
|---|---|---|---|---|
| `LDAX RP` | Load Accumulator from memory pointed to by a register pair | `LDAX B`, `LDAX D` | 1 byte | Register Indirect |
| `STAX RP` | Store Accumulator into memory pointed to by a register pair | `STAX B`, `STAX D` | 1 byte | Register Indirect |

**Key points to remember:**
- `LDAX RP` and `STAX RP` are both **1-byte long** instructions.
- Only the **first register's name** of the pair is used in the instruction (e.g., `B` for BC, `D` for DE) — the second register is implied as the "extension."
- Neither instruction has an **`H` variant** (`LDAX H` / `STAX H`) because that functionality is already covered by `MOV A, M` and `MOV M, A` respectively.
- There's no equivalent for other registers (like `LBX RP`) because **only the accumulator** has this special set of addressing capabilities.
- Before executing either instruction, the relevant register pair (BC or DE) must first be **loaded with the target memory address**.

---

*Next session: Two more instruction types of the Data Transfer Group.*
