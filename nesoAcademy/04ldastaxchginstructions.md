# 8085 Microprocessor — LDA a16, STA a16, and XCHG Instructions

> Continuing the **Data Transfer Group of Instructions** series.
> This session covers three instruction types: `LDA a16`, `STA a16`, and `XCHG`.

---

## Table of Contents
1. [Why Do We Need LDA a16?](#why-do-we-need-lda-a16)
2. [LDA a16 — Load Accumulator from Memory](#lda-a16--load-accumulator-from-memory)
3. [STA a16 — Store Accumulator into Memory](#sta-a16--store-accumulator-into-memory)
4. [Why Only LDA/STA and No LDB, LDC, STB, STC?](#why-only-ldasta-and-no-ldb-ldc-stb-stc)
5. [XCHG — Exchange HL and DE Register Pairs](#xchg--exchange-hl-and-de-register-pairs)
6. [Session Summary](#session-summary)

---

## Why Do We Need LDA a16?

To understand why `LDA a16` is useful, recall the **previous session's** approach using `LXI RP, D16` combined with `MOV`.

### The old (indirect) approach
Suppose the goal is: *load the accumulator with the content of a specific memory location.*

Steps taken previously:
1. `LXI H, <address>` → load the 16-bit address into the **HL register pair**.
   - This instruction alone takes **3 bytes** of memory (1 byte opcode + 2 bytes address).
2. `MOV A, M` → move the content of the memory location (pointed to by HL) into the accumulator.
   - This instruction takes **1 byte**.

**Total cost:** 2 instructions, **4 bytes** of memory, and it required involving an *extra register pair* (HL) just to hold an address temporarily.

### The question
Could the same result — loading the accumulator from a specific memory address — be achieved:
- without using an extra register/register pair, and
- using a single instruction?

**Yes** — this is exactly what `LDA a16` does.

---

## LDA a16 — Load Accumulator from Memory

### Meaning
- **LDA** = **L**oad **A**ccumulator contents from memory.
- **a16** = a 16-bit address (since the 8085 has a 16-bit address bus and can access **64K (65,536)** distinct memory locations).

Instead of loading the address into HL and then doing a separate `MOV A, M`, `LDA a16` lets you send the memory address **directly within the instruction itself** to the microprocessor.

### Instruction length
- Opcode `LDA` → occupies **8 bits (1 byte)**.
- Address `a16` → occupies **16 bits (2 bytes)**.
- **Total = 3 bytes** → `LDA a16` is a **3-byte long instruction**.

### Example
Suppose memory location `F821H` contains the hexadecimal value `EA`.

> Note: Hexadecimal is just used for human readability. Internally, the data is actually stored in binary.

Executing:
```
LDA F821H
```
- The microprocessor receives the address `F821H` directly from the instruction.
- It points directly to that memory location.
- The content of that location (`EA`) is loaded into the **accumulator register**.

### Benefit
- No need to involve the HL register pair.
- Only **one instruction** is needed instead of two.
- Memory savings: **3 bytes** used instead of **4 bytes** (compared to the `LXI H` + `MOV A, M` approach) — saving **1 byte**.

### Key takeaway
> `LDA a16` is a 3-byte instruction that loads the accumulator directly with the contents of the memory location specified by `a16`.

---

## STA a16 — Store Accumulator into Memory

### Meaning
- **STA** = **St**ore **A**ccumulator contents in memory.
- This is essentially the **reverse operation** of `LDA a16`.

### Instruction length
- Opcode `STA` → occupies **8 bits (1 byte)**.
- Address `a16` → occupies **16 bits (2 bytes)** (since all memory addresses are 16 bits).
- **Total = 3 bytes** → `STA a16` is also a **3-byte long instruction**.

### Example
Suppose the accumulator register currently holds the value `AF`.
Goal: store this value into memory location `F821H`.

Executing:
```
STA F821H
```
- The microprocessor receives the address `F821H` directly from the instruction.
- It points directly to that memory location.
- The content of the accumulator (`AF`) is stored into that memory location.

### Key takeaway
> `STA a16` stores the content of the accumulator into the memory location specified by `a16`. It is a 3-byte instruction, just like `LDA a16`.

---

## Why Only LDA/STA and No LDB, LDC, STB, STC?

- There is **no** `LDB`, `LDC`, `STB`, or `STC` instruction — `LDA` and `STA` work **only with the accumulator**.
- This is because the **accumulator is a special-purpose register**.
- Unlike general-purpose registers (B, C, D, E, H, L), the accumulator can be addressed and accessed in **multiple different ways**, making it uniquely suited for direct memory addressing instructions like these.

---

## XCHG — Exchange HL and DE Register Pairs

### What it does
`XCHG` stands for: **Exchange the contents of the HL register pair with the DE register pair.**

> ⚠️ Important: `XCHG` works **only** between **HL** and **DE**.
> - It does **not** work between BC and DE.
> - It does **not** work between BC and HL.
> - Only **DE ↔ HL**.

### Example Walkthrough

**Step 1:** Load data into the DE register pair.
```
LXI D, ABCDH
```
After execution, DE pair contains: `AB CD`

**Step 2:** Load data into the HL register pair.
```
LXI H, 1234H
```
After execution, HL pair contains: `12 34`

**Step 3:** Execute the exchange.
```
XCHG
```
After execution:
- DE pair now contains: `12 34` (previously in HL)
- HL pair now contains: `AB CD` (previously in DE)

The contents of the two register pairs are **swapped**.

### Instruction length
- `XCHG` is a **1-byte long instruction**.

### Why is XCHG useful?
Without `XCHG`, swapping the contents of two register pairs manually would require:
1. Using a **third temporary register** to hold the value from one register (to make space).
2. Moving data from the second register into the first.
3. Moving the temporary-stored data into the second register.
4. Repeating this process for **both bytes** of each register pair (since register pairs are 16 bits = 2 bytes each).

This would require **multiple instructions** and **more memory space**.

`XCHG` accomplishes the entire swap in **just 1 byte** and a **single instruction** — a major saving in both instruction count and memory space.

---

## Session Summary

| Instruction | Full Meaning | Length | Works With | Function |
|---|---|---|---|---|
| `LDA a16` | Load Accumulator from memory | 3 bytes | Accumulator only | Loads accumulator with content from address `a16` |
| `STA a16` | Store Accumulator into memory | 3 bytes | Accumulator only | Stores accumulator content into address `a16` |
| `XCHG` | Exchange register pairs | 1 byte | HL ↔ DE only | Swaps contents of HL and DE register pairs |

**Key concepts to remember:**
- `LDA a16` and `STA a16` are **3-byte instructions**, specific to the **accumulator register** only.
- `XCHG` is a **1-byte instruction**, and works exclusively between the **HL** and **DE** register pairs.
- These instructions exist to **save memory space** and **reduce instruction count** compared to older, more indirect approaches (e.g., using `LXI` + `MOV`, or manual register-by-register exchanges).

---

*Next session topic: Register Codes.*