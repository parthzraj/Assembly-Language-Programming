# 8085 Microprocessor — Addressing Modes

> This session covers the **need for addressing modes** and the **five different addressing modes** of the 8085 microprocessor.

---

## Table of Contents
1. [What Are Addressing Modes?](#what-are-addressing-modes)
2. [Analogy: Identifying a Person in a Group](#analogy-identifying-a-person-in-a-group)
3. [Illustration: Multiple Ways to Access the Accumulator](#illustration-multiple-ways-to-access-the-accumulator)
4. [The Five Addressing Modes of 8085](#the-five-addressing-modes-of-8085)
   - [1. Immediate Addressing Mode](#1-immediate-addressing-mode)
   - [2. Register Addressing Mode](#2-register-addressing-mode)
   - [3. Absolute / Direct Addressing Mode](#3-absolute--direct-addressing-mode)
   - [4. Register Indirect Addressing Mode](#4-register-indirect-addressing-mode)
   - [5. Implied Addressing Mode](#5-implied-addressing-mode)
5. [Summary Table with Examples](#summary-table-with-examples)
6. [Session Summary](#session-summary)

---

## What Are Addressing Modes?

**Addressing modes** = the **different ways of accessing data** in the microprocessor.

The same end result (e.g., loading a value into the accumulator) can be achieved through **multiple different instruction styles** — each representing a different "mode" of specifying where the data comes from.

---

## Analogy: Identifying a Person in a Group

To build intuition, consider identifying a specific person within a group of people. There are multiple ways to do this:

1. **By name** — if you know the person's name, you can directly specify them.
2. **By position** — e.g., "the person second from the right." (Note: positions can change if the group rearranges.)
3. **By features/characteristics** — e.g., "wearing a blue sweatshirt, has brown hair, carrying a side bag."

Each of these is a **different way of specifying the same target** — this is conceptually what addressing modes do for data in a microprocessor.

---

## Illustration: Multiple Ways to Access the Accumulator

Suppose the goal is simply: **get the value `12H` into the accumulator register.** This can be done in several different ways:

### Way 1 — `MVI A, 12H`
- Sends the data (`12H`) **immediately/directly** within the instruction to the accumulator.
- Result: Accumulator = `12H`

### Way 2 — `MOV A, B`
- Assume register B already contains `12H`.
- Copies the data from register B into the accumulator.
- Result: Accumulator = `12H`

### Way 3 — `LDA F820H`
- Assume memory location `F820H` contains `12H`.
- Loads the accumulator directly from that memory address.
- Result: Accumulator = `12H`

### Way 4 — `MOV A, M`
- Assume the **HL register pair** contains the address `F820H` (pointing to that memory location).
- The instruction uses HL to indirectly point to the memory location, then loads its content into the accumulator.
- Result: Accumulator = `12H`

**Key takeaway:** All four instructions achieve the *same result* (loading `12H` into the accumulator), but they do so using **different methods of specifying where the data comes from**. This is exactly what addressing modes describe.

---

## The Five Addressing Modes of 8085

The 8085 microprocessor has **5 addressing modes**:

### 1. Immediate Addressing Mode
- The data is provided **directly within the instruction itself**.
- Example: `MVI A, 12H` — the value `12H` is immediately moved into the accumulator.
- **Identifying tip:** Mnemonics ending in **`I`** (for "Immediate") typically belong to this group — e.g., `MVI`, `LXI`.

### 2. Register Addressing Mode
- The instruction specifies **registers** (not raw data or addresses); the microprocessor knows the data resides within those registers.
- Example: `MOV A, B` — data isn't given directly; instead, the source and destination *registers* are specified.
- General type: `MOV R1, R2` (R2 = source register, R1 = destination register — as studied in the op-code formation session).

### 3. Absolute / Direct Addressing Mode
- The instruction specifies the **actual (absolute) address** in memory where the data resides — not a register, not immediate data.
- Example: `LDA F820H` — the actual memory address is given directly in the instruction; the data at that address is loaded into the accumulator.
- Also called **direct addressing mode** because the address is sent directly through the instruction.
- Other instruction types in this category: `STA`, `LHLD`, `SHLD` (LHLD and SHLD to be studied in later sessions).

### 4. Register Indirect Addressing Mode
- The instruction does **not** mention a source register or a direct address; instead, it references a **memory location via a register pair** (e.g., using `M`, which refers to the memory location pointed to by HL).
- Example: `MOV A, M` — the instruction never explicitly mentions the HL register pair, but `M` implicitly refers to the memory location that HL is pointing to. This is "indirect" because the actual register pair holding the address is never named directly.
- Other instruction types in this category (to be studied in upcoming sessions): `LDAX RP`, `STAX RP`.

### 5. Implied Addressing Mode
- The instruction **does not mention any register, register pair, address, or data at all** — but the microprocessor inherently knows what operation to perform based on the instruction alone.
- Example: `XCHG` — a 1-byte instruction that exchanges the contents of the DE and HL register pairs, without ever explicitly naming DE or HL within the instruction itself. The microprocessor "implies" the operands from the instruction's fixed meaning.

---

## Summary Table with Examples

| # | Addressing Mode | How Data Is Specified | Example Instruction Types |
|---|---|---|---|
| 1 | **Immediate** | Data given directly in the instruction | `MVI`, `LXI` (mnemonics often end in "I") |
| 2 | **Register** | Registers specified; data resides in them | `MOV R1, R2` |
| 3 | **Absolute / Direct** | Actual memory address given directly | `LDA`, `STA`, `LHLD`, `SHLD` |
| 4 | **Register Indirect** | Memory location referenced indirectly via a register pair (e.g., `M` via HL) | `MOV R, M`, `LDAX RP`, `STAX RP` |
| 5 | **Implied** | No operand mentioned at all; microprocessor infers the operation | `XCHG` |

---

## Session Summary

- **Addressing modes** = different ways of accessing/specifying data for an instruction to operate on.
- The 8085 microprocessor has **5 addressing modes**:
  1. **Immediate** — data given directly in the instruction (e.g., `MVI A, 12H`)
  2. **Register** — data accessed via named registers (e.g., `MOV A, B`)
  3. **Absolute/Direct** — data accessed via an explicit memory address (e.g., `LDA F820H`)
  4. **Register Indirect** — data accessed via a memory location pointed to by a register pair, referenced indirectly (e.g., `MOV A, M`)
  5. **Implied** — no operand mentioned; the operation and its operands are inherent to the instruction (e.g., `XCHG`)
- The **same result** can often be achieved via different addressing modes, but each has trade-offs in instruction length, flexibility, and use case.

---

*Next session: Continuing the study of different instruction types.*
