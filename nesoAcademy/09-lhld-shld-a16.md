# 8085 Microprocessor — LHLD a16 and SHLD a16 Instructions

> This session covers `LHLD a16` and `SHLD a16` — the final two instruction types in the **Data Transfer Group**, completing all **13 instruction types** and **83 op-codes** of this group.

---

## Table of Contents
1. [LHLD a16 — Load HL Pair Using Direct Addressing](#lhld-a16--load-hl-pair-using-direct-addressing)
2. [Why LHLD Involves Two Consecutive Memory Locations](#why-lhld-involves-two-consecutive-memory-locations)
3. [Why There's No LBCD or LDED](#why-theres-no-lbcd-or-lded)
4. [SHLD a16 — Store HL Pair Using Direct Addressing](#shld-a16--store-hl-pair-using-direct-addressing)
5. [Why There's No SBCD or SDED](#why-theres-no-sbcd-or-sded)
6. [Session Summary](#session-summary)

---

## LHLD a16 — Load HL Pair Using Direct Addressing

### Meaning
- **LHLD** = **L**oad **H**L pair using **D**irect addressing from a memory location.
- **a16** = a 16-bit address (same meaning as seen previously in `LDA a16` and `STA a16`) — it refers to any location within the memory.

### Addressing mode
- The 16-bit address is provided **directly within the instruction itself**.
- This makes `LHLD a16` a **direct (absolute) addressing mode** instruction — as covered in the Addressing Modes session.

### Instruction length
- Opcode `LHLD` → 8 bits (1 byte)
- Address `a16` → 16 bits (2 bytes)
- **Total = 3 bytes** → `LHLD a16` is a **3-byte long instruction**.

### Why Two Consecutive Memory Locations Are Involved

The **HL register pair** consists of **two 8-bit registers** (H and L). Since each memory location can only store **8 bits**, loading a full 16-bit register pair requires reading from **two consecutive memory locations**.

### Worked Example: `LHLD F820H`

Suppose:
- Memory location `F820H` contains the value `34`
- The next (consecutive) memory location `F821H` contains the value `12`

**Execution steps:**
1. The instruction sends the address `F820H` directly to the microprocessor.
2. The microprocessor points to memory location `F820H` first.
   - The value there (`34`) is loaded into the **L register** (the lower-order part of the HL pair).
3. The microprocessor then points to the **next consecutive** memory location, `F821H`.
   - The value there (`12`) is loaded into the **H register** (the higher-order part of the HL pair).

**Result:**
| Register | Value | Source Memory Location |
|---|---|---|
| L (lower order) | `34` | `F820H` (the address given in the instruction) |
| H (higher order) | `12` | `F821H` (the next/consecutive location) |

> **Pattern to remember:** The address given in the instruction points to the **first** location, whose content goes into **L**. The **next consecutive** location's content goes into **H**.

---

## Why There's No LBCD or LDED

A natural question: if `LHLD` exists for the HL pair, why isn't there an equivalent `LBCD` (for BC) or `LDED` (for DE)?

**Answer:** The **HL register pair is special** — it can be addressed in multiple unique ways (as seen repeatedly throughout this series, e.g., via the `M` reference in register indirect addressing). This special treatment does not extend to the other register pairs (BC, DE), so no equivalent direct-load instruction exists for them.

---

## SHLD a16 — Store HL Pair Using Direct Addressing

### Meaning
- **SHLD** = **S**tore **H**L pair using **D**irect addressing into a memory location.
- This is the **reverse** of `LHLD a16`: instead of loading HL from memory, it **stores** the content of the HL pair **into** memory.

### Addressing mode
- Just like `LHLD`, the 16-bit address is given directly within the instruction → **direct (absolute) addressing mode**.

### Instruction length
- Opcode `SHLD` → 8 bits (1 byte)
- Address `a16` → 16 bits (2 bytes)
- **Total = 3 bytes** → `SHLD a16` is also a **3-byte long instruction**.

### Why Two Consecutive Memory Locations Are Involved

Just like `LHLD`, storing the full 16-bit HL pair requires **two consecutive memory locations**, since each location holds only 8 bits.

### Worked Example: `SHLD F820H`

Suppose the HL register pair currently holds:
- H = `AB`
- L = `CD`

**Execution steps:**
1. The instruction sends the address `F820H` directly to the microprocessor.
2. The microprocessor points to memory location `F820H` first.
   - The content of the **L register** (`CD`) is stored there **first**.
3. The microprocessor then points to the **next consecutive** memory location, `F821H`.
   - The content of the **H register** (`AB`) is stored there.

**Result:**
| Memory Location | Value Stored | Source Register |
|---|---|---|
| `F820H` (the address given in the instruction) | `CD` | L (lower order) |
| `F821H` (the next/consecutive location) | `AB` | H (higher order) |

> **Pattern to remember:** Just like with `LHLD`, the **L register's content goes to the first (given) address**, and the **H register's content goes to the next consecutive address**.

---

## Why There's No SBCD or SDED

For the same reason there's no `LBCD`/`LDED`, there is also **no `SBCD` or `SDED`** — this direct-store-to-memory capability is **specific to the HL register pair only**, due to its special addressing status.

---

## Session Summary

| Instruction Type | Meaning | Length | Addressing Mode | Memory Locations Used |
|---|---|---|---|---|
| `LHLD a16` | Load HL pair from memory (direct addressing) | 3 bytes | Direct / Absolute | 2 consecutive locations: L ← first address, H ← next address |
| `SHLD a16` | Store HL pair into memory (direct addressing) | 3 bytes | Direct / Absolute | 2 consecutive locations: L → first address, H → next address |

**Key points to remember:**
- Both `LHLD a16` and `SHLD a16` are **3-byte long instructions** (1 byte opcode + 2 bytes address).
- Both use **direct/absolute addressing mode**, since the 16-bit address is given directly in the instruction.
- Both involve **two consecutive memory locations**, because the HL pair is 16 bits (2 × 8-bit registers), while each memory location only holds 8 bits.
- In both instructions: the **address given in the instruction corresponds to L** (for LHLD, source; for SHLD, destination), and **the next consecutive address corresponds to H**.
- Neither instruction has an equivalent for BC or DE (no `LBCD`, `LDED`, `SBCD`, `SDED`) — this functionality is **exclusive to the HL register pair**.

### 🎉 Data Transfer Group Complete!

With `LHLD a16` and `SHLD a16` covered, **all 13 instruction types** of the Data Transfer Group have now been studied, covering a total of **83 op-codes**. The next session will **summarize all Data Transfer Group instructions** covered so far.

---

*Next session: Summary of the Data Transfer Group of Instructions.*
