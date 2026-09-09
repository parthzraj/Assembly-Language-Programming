# 8085 Microprocessor — Formation of Op-Codes

> This session builds directly on the previous one (**Register Codes**). We now use register codes to derive the actual **op-codes (hexadecimal machine codes)** for specific instructions — without needing to memorize them.

---

## Table of Contents
1. [Why This Matters](#why-this-matters)
2. [Instruction Types Used in This Session](#instruction-types-used-in-this-session)
3. [Type 1: MOV R1, R2 (1-byte)](#type-1-mov-r1-r2-1-byte)
4. [Type 2: MOV R, M (1-byte)](#type-2-mov-r-m-1-byte)
5. [Type 3: MVI R, d8 (2-byte)](#type-3-mvi-r-d8-2-byte)
6. [General Formulas Derived](#general-formulas-derived)
7. [Session Summary](#session-summary)

---

## Why This Matters

Previously, register codes were introduced — 3-bit binary patterns that identify each register (and the memory reference `M`). This session shows that **op-codes are not arbitrary** — they are built systematically from:
- A **fixed code assigned by Intel** to the instruction's mnemonic (e.g., `MOV`, `MVI`).
- The **register code(s)** of the specific registers involved (source and/or destination).

Once you know the mnemonic's fixed code and the register code chart, you can **derive** the op-code for any specific instruction of that type — no memorization required. (Intel's official documentation also provides these mnemonic codes for every instruction.)

### Register Code Recap (from previous session)

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

## Instruction Types Used in This Session

Three instruction **types** (not specific instructions) are used to demonstrate op-code formation, chosen because they differ in size:

| Type | Description | Length |
|---|---|---|
| `MOV R1, R2` | Move between two registers | 1 byte |
| `MOV R, M` | Move between a register and memory (via HL) | 1 byte |
| `MVI R, d8` | Move immediate 8-bit data into a register | 2 bytes |

> Note: These are **types**, not specific instructions — op-codes can only be formed for **actual, specific instructions** (e.g., `MOV A, B`), not for a general type.

---

## Type 1: MOV R1, R2 (1-byte)

### Rule recap
- In `MOV R1, R2`: **R2 is the source**, **R1 is the destination**.
- Intel's fixed code for the `MOV` mnemonic (register-to-register) is `01`.

### Worked Example 1: `MOV A, B`
- Source = B → register code `000`
- Destination = A → register code `111`
- Mnemonic code for MOV = `01`

Assembling all 8 bits (mnemonic + destination + source):
```
01   111   000
MOV   A     B
```
Full byte: `01111000`

**Convert to hex** — group into nibbles from LSB to MSB:
- Most significant nibble: `0111` = **7**
- Least significant nibble: `1000` = **8**

**Op-code: `78H`**

### Worked Example 2: `MOV E, H`
- Source = H → register code `100`
- Destination = E → register code `011`
- Mnemonic code for MOV = `01`

Assembling:
```
01   011   100
MOV   E     H
```
Full byte: `01011100`

**Convert to hex:**
- Most significant nibble: `0101` = **5**
- Least significant nibble: `1100` = **C**

**Op-code: `5CH`**

---

## Type 2: MOV R, M (1-byte)

Same overall pattern as Type 1, except one operand is **M** (the memory location pointed to by the HL register pair), which uses register code `110`.

### Worked Example 1: `MOV A, M`
- Source = M → register code `110`
- Destination = A → register code `111`
- Mnemonic code for MOV = `01`

Assembling:
```
01   111   110
MOV   A     M
```
Full byte: `01111110`

**Convert to hex:**
- Most significant nibble: `0111` = **7**
- Least significant nibble: `1110` = **E**

**Op-code: `7EH`**

### Worked Example 2: `MOV B, M`
- Source = M → register code `110`
- Destination = B → register code `000`
- Mnemonic code for MOV = `01`

Assembling:
```
01   000   110
MOV   B     M
```
Full byte: `01000110`

**Convert to hex:**
- Most significant nibble: `0100` = **4**
- Least significant nibble: `0110` = **6**

**Op-code: `46H`**

> Both `MOV R1, R2` and `MOV R, M` types produce **1-byte op-codes**, matching their classification as 1-byte long instructions.

---

## Type 3: MVI R, d8 (2-byte)

This type is different: it has **both a prefix and a suffix** fixed code, with the register code sandwiched in between.

### Rule recap
- Intel's fixed code for `MVI` is: `00` (prefix) + **register code (3 bits)** + `110` (suffix).
- Since this instruction also carries an 8-bit immediate data value (`d8`), the **total instruction length is 2 bytes**:
  - Byte 1: the op-code itself (8 bits: mnemonic prefix + register code + suffix)
  - Byte 2: the 8-bit immediate data (`d8`)

### Worked Example 1: `MVI A, EAH`
- Destination register = A → register code `111`
- MVI format: `00` + `111` + `110`

Assembling:
```
00   111   110
     A
```
Full byte: `00111110`

**Convert to hex:**
- Most significant nibble: `0011` = **3**
- Least significant nibble: `1110` = **E**

**Op-code: `3EH`** (followed by the data byte `EAH`)

Total instruction size: 8 bits (op-code) + 8 bits (data) = **2 bytes**.

### Worked Example 2: `MVI M, AFH`
- Destination = M → register code `110`
- MVI format: `00` + `110` + `110`

Assembling:
```
00   110   110
     M
```
Full byte: `00110110`

**Convert to hex:**
- Most significant nibble: `0011` = **3**
- Least significant nibble: `0110` = **6**

**Op-code: `36H`** (followed by the data byte `AFH`)

Again, total instruction size = 2 bytes (op-code byte + data byte).

---

## General Formulas Derived

From the worked examples, the following general bit-patterns emerge for these instruction types:

| Instruction Type | Bit Pattern (8 bits) | Notes |
|---|---|---|
| `MOV R1, R2` | `01 DDD SSS` | DDD = destination register code, SSS = source register code |
| `MOV R, M` | `01 DDD SSS` | Same pattern; SSS = `110` when source is M, or DDD = `110` when destination is M |
| `MVI R, d8` | `00 DDD 110` | DDD = destination register code; followed by a separate 8-bit data byte |

> These formulas let you **derive** the op-code for *any* specific instruction of these types just by plugging in the correct register code(s) — without memorizing a full op-code table.

---

## Session Summary

- Op-codes are **not arbitrary** — they are systematically built from a **fixed mnemonic code** (assigned by Intel) combined with **register codes**.
- Three instruction types were used to illustrate this:
  - `MOV R1, R2` and `MOV R, M` → both 1-byte instructions, pattern `01 DDD SSS`.
  - `MVI R, d8` → 2-byte instruction, pattern `00 DDD 110` + 1 data byte.
- To convert the assembled binary op-code into hexadecimal: **group bits into nibbles (4 bits) starting from the least significant bit**, then convert each nibble to its hex digit.
- Knowing the **mnemonic's fixed code** + the **register code chart** is enough to derive op-codes for any instruction of these types — Intel's documentation also provides these codes directly.

### Quick Reference Table (Examples Covered)

| Instruction | Op-Code |
|---|---|
| `MOV A, B` | `78H` |
| `MOV E, H` | `5CH` |
| `MOV A, M` | `7EH` |
| `MOV B, M` | `46H` |
| `MVI A, EAH` | `3EH` (+ data `EAH`) |
| `MVI M, AFH` | `36H` (+ data `AFH`) |

---

*Next session topic: Addressing Modes.*
