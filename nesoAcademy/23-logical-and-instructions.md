# 23 - Logical Group of Instructions: ANA R and ANI D8 (AND Operation)

> First session on **The Logical Group of Instructions** for the 8085 microprocessor.
> Covers: overview of the logical group, basics of the **AND operation**, and the first two instruction types — **`ANA R`** and **`ANI D8`**.

---

## 1. Recap: Instruction Groups Covered So Far

| Group | Instruction Types | Opcodes |
|---|:---:|:---:|
| Data Transfer | 13 | 83 |
| Arithmetic | 14 | 62 |
| **Logical** *(starting now)* | **15** | **43** |

- The Logical Group has **15 instruction types**, totaling **43 opcodes**.
- This session covers **2 of the 15** types: `ANA R` and `ANI D8`.

---

## 2. Overview of the Logical Group (8085)

The 8085 microprocessor natively provides these logical operations via instructions:
- **AND**
- **OR**
- **Exclusive OR (XOR)**
- **NOT**

It does **NOT** directly provide:
- **NAND** — can be built as: AND → then NOT
- **NOR** — can be built as: OR → then NOT

---

## 3. General Structure of Logical Operations in 8085

- Logical operations like AND are **binary operations** — they need **two operands** (Op1, Op2).
- **Rule:** One operand **must always** reside in the **accumulator**.
- The **second operand** can be supplied in **three possible ways**:
  1. Content of any of the **6 GPRs** (B, C, D, E, H, L)
  2. Content of the **memory location** pointed to by the **HL register pair**
  3. **8-bit immediate data** sent directly within the instruction

### Mapping to today's instructions:
- **`ANA R`** → covers ways 1 and 2 (register or memory as second operand)
- **`ANI D8`** → covers way 3 (immediate data as second operand)

---

## 4. Basics of the AND Operation

AND is a binary operation on two bits, X and Y. Truth table:

| X | Y | X AND Y |
|:---:|:---:|:---:|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

**Output = 1 only when BOTH inputs are 1.**

### Two Important Special Cases:

1. **X AND 1 = X**
   - When Y is fixed at 1, whatever X is, it passes through unchanged to the output.
2. **X AND 0 = 0**
   - When Y is fixed at 0, the output is always 0, regardless of X.

### 🔑 Key Application: Selectively Resetting Bits

Since one operand of AND always sits in the accumulator, and the result also goes back into the accumulator, we can **carefully choose the second operand** to:
- **Keep bits unchanged** → AND that bit position with **1**
- **Force bits to 0** → AND that bit position with **0**

**Example concept:** If the accumulator holds `FF` (all 1s), and we want to reset just the **MSB and LSB** to 0 while keeping the middle bits unchanged:
- Choose the second operand as a byte with **0 at MSB and LSB positions**, and **1s everywhere else**.
- After ANDing, the MSB and LSB of the accumulator become 0, while the middle bits (ANDed with 1) remain unchanged.

This is a core technique: **AND is used to selectively clear (reset) specific bits of the accumulator while preserving others.**

---

## 5. Flags Behavior for AND Operations (Important — Fixed by Intel)

For **any AND operation** on the 8085:

| Flag | Behavior |
|---|---|
| **Carry (CY)** | Always **RESET** (0) |
| **Auxiliary Carry (AC)** | Always **SET** (1) |
| Parity (P), Zero (Z), Sign (S) | Determined normally, based on the **resulting accumulator content** |

> ⚠️ Note: This CY=0, AC=1 behavior is **not derived from any logical principle** — it is simply a **predefined design choice by Intel** for the 8085. Even though AND is a logical (not arithmetic) operation, these two flags still get fixed values.

---

## 6. Instruction Type 1: `ANA R` (AND Accumulator with R)

- **Mnemonic breakdown:** `AN` = AND, `A` = Accumulator → "AND accumulator with R"
- **`R`** represents: Accumulator itself, all 6 GPRs (B, C, D, E, H, L), or **M** (memory location pointed by HL pair)
- **Category:** 1-byte long instruction
- **Opcodes:** 8 (ANA A, ANA B, ANA C, ANA D, ANA E, ANA H, ANA L, ANA M)

### Flags for `ANA R`:
- Carry → always reset
- Auxiliary Carry → always set
- Parity, Zero, Sign → based on accumulator result

---

### Worked Example: `ANA E`

**Given:**
- Accumulator = `AB` (hex)
- Register E = `12` (hex)

### Step 1: Convert to binary
- A = 10 decimal → `1010`
- B = 11 decimal → `1011`
- **AB in binary = `1010 1011`**

- 1 → `0001`
- 2 → `0010`
- **12 in binary = `0001 0010`**

### Step 2: Perform AND bit-by-bit
```
  1010 1011   (AB)
AND
  0001 0010   (12)
-----------
  0000 0010
```

- Only positions where **both** bits are 1 remain 1; everything else becomes 0.
- **Result: `0000 0010` = `02` in hex**

### Step 3: Determine Flags

| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | Always reset for AND |
| Auxiliary Carry (AC) | **1 (Set)** | Always set for AND |
| Parity (P) | **0 (Reset)** | `00000010` has only **one** 1 → odd → parity reset |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **0 (Reset)** | MSB of `02` = 0 |

### ✅ Result: **A = 02H**, Flags → AC set (CY reset), all others reset (P, Z, S all reset since only 1 one and MSB=0).

> **Note on illustration purpose:** In this example, the bits weren't chosen carefully for a "selective reset" — the input bits at the reset positions happened to already be 0, so the AND simply reflects the natural result of `AB AND 12`. If you specifically want to preserve certain bits regardless of their value, you'd deliberately set those positions to `1` in the second operand.

---

## 7. Instruction Type 2: `ANI D8` (AND Immediate with Accumulator)

- **Mnemonic breakdown:** `ANI` = AND Immediate → "AND immediate with accumulator"
- Operand 1 = accumulator content
- Operand 2 = **8-bit immediate data**, provided directly in the instruction (immediate addressing mode)
- **Category:** 2-byte long instruction
  - Byte 1: opcode for `ANI`
  - Byte 2: the 8-bit immediate data
- **Opcodes:** Only **1** (since data is sent immediately, no register/memory variant needed)

### Flags for `ANI D8`:
- Carry → always reset
- Auxiliary Carry → always set
- Parity, Zero, Sign → based on accumulator result

---

### Worked Example: `ANI F3`

**Given:**
- Accumulator = `45` (hex)
- Immediate data = `F3` (hex)

### Step 1: Convert to binary
- 4 → `0100`
- 5 → `0101`
- **45 in binary = `0100 0101`**

- F → `1111`
- 3 → `0011`
- **F3 in binary = `1111 0011`**

### Step 2: Perform AND bit-by-bit
```
  0100 0101   (45)
AND
  1111 0011   (F3)
-----------
  0100 0001
```

- Least significant nibble: only the last bit position has 1 in both → result `0001`
- Most significant nibble: only the second bit position (from left, value 4) has 1 in both → result `0100`
- **Result: `0100 0001` = `41` in hex**

### Step 3: Determine Flags

| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | Always reset for AND |
| Auxiliary Carry (AC) | **1 (Set)** | Always set for AND |
| Parity (P) | **1 (Set)** | `01000001` has **two** 1s → even → parity set |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **0 (Reset)** | MSB of `41` = 0 |

### ✅ Result: **A = 41H**, Flags → Carry reset, Auxiliary Carry set, Parity set, Zero reset, Sign reset.

---

## 8. Summary Comparison Table

| Feature | `ANA R` | `ANI D8` |
|---|---|---|
| Full Meaning | AND accumulator with R | AND immediate with accumulator |
| Second Operand Source | Register (B,C,D,E,H,L,A) or Memory (M via HL) | Immediate 8-bit data in instruction |
| Instruction Length | 1 byte | 2 bytes |
| Number of Opcodes | 8 | 1 |
| Carry Flag | Always reset | Always reset |
| Auxiliary Carry Flag | Always set | Always set |
| Other Flags (P, Z, S) | Based on result | Based on result |

---

## 9. Key Points to Remember

1. **Logical Group of Instructions** has 15 types and 43 opcodes total (this session covers 2 of them).
2. 8085 natively supports: **AND, OR, XOR, NOT**. NAND and NOR must be built using AND/OR followed by NOT.
3. **AND is a binary operation** — one operand is always in the accumulator; the second comes from a register, memory (M), or immediate data.
4. **Truth table shortcuts:**
   - `X AND 1 = X` (bit preserved)
   - `X AND 0 = 0` (bit forced to zero)
5. This property allows **selective bit resetting** of the accumulator — a very useful technique in low-level programming.
6. **Fixed flag behavior for ALL AND operations:** Carry flag is **always reset**, Auxiliary Carry flag is **always set** — this is an Intel-defined convention, not derived from operation logic.
7. **`ANA R`**: 1-byte instruction, 8 opcodes (covers register and memory operand).
8. **`ANI D8`**: 2-byte instruction, only 1 opcode (covers immediate data operand).

---

## 10. What's Next

- Next session: Instructions based on the **OR logic**.

---
*Notes prepared from video lecture transcript — "Logical Group of Instructions - ANA R and ANI D8"*
