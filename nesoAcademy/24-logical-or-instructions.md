# 24 - Logical Group of Instructions: ORA R and ORI D8 (OR Operation)

> Continuation of **The Logical Group of Instructions** for the 8085 microprocessor.
> Covers: basics of the **OR (inclusive OR) operation**, and instruction types **`ORA R`** and **`ORI D8`**.

---

## 1. Recap

- Previous session covered the **AND** operation (`ANA R`, `ANI D8`).
- This session covers the **OR** operation (`ORA R`, `ORI D8`).
- The Logical Group of 8085 supports: **AND, OR, Exclusive OR (XOR), NOT**.

---

## 2. Basics of the OR Operation

- Just like AND, **OR is a binary operation** — needs two operands.
  - Operand 1: **always in the accumulator**
  - Operand 2: can come from
    1. Any **GPR** (B, C, D, E, H, L)
    2. **Memory location** pointed by HL pair
    3. **8-bit immediate data** in the instruction

### Instruction mapping:
- **`ORA R`** → handles cases 1 and 2 (register or memory)
- **`ORI D8`** → handles case 3 (immediate data)

### Truth Table for OR

| X | Y | X OR Y |
|:---:|:---:|:---:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

**Output = 1 if EITHER (or both) input is 1.** Output = 0 only when both inputs are 0.

### Why "Inclusive OR"?
- OR is called **Inclusive OR** because it **includes** the AND case — when **both** inputs are 1, the output is still 1 (unlike Exclusive OR, which will be covered next session).

### Two Important Special Cases:

1. **X OR 0 = X**
   - When Y is fixed at 0, whatever X is, passes through unchanged.
2. **X OR 1 = 1**
   - When Y is fixed at 1, the output is always 1, regardless of X.

### 🔑 Key Application: Selectively Setting Bits

- In the AND session, we learned `X AND 0 = 0` lets us **selectively reset** bits.
- Now, with OR: `X OR 1 = 1` lets us **selectively set** bits.

**Example concept:** To set the **most significant bit** of the accumulator to 1 (while leaving other bits unchanged):
- Choose the second operand with **MSB = 1** and **all other bits = 0**.
- ORing: the MSB becomes 1 (forced), and the rest of the bits remain unchanged (since `X OR 0 = X`).

---

## 3. Flags Behavior for OR Operations (Fixed by Intel)

| Flag | Behavior |
|---|---|
| **Carry (CY)** | Always **RESET** (0) |
| **Auxiliary Carry (AC)** | Always **RESET** (0) |
| Parity (P), Zero (Z), Sign (S) | Determined normally, based on the **resulting accumulator content** |

> ⚠️ **Difference from AND:** For AND, CY was reset but **AC was set**. For OR, **both CY and AC are reset**. This is an Intel-defined convention for the 8085, not derived from the logic operation itself.

---

## 4. Instruction Type 1: `ORA R` (OR Accumulator with R)

- **Mnemonic breakdown:** `OR` (from "OR") + `A` (Accumulator) → "OR accumulator with R"
- **`R`** represents: Accumulator, all 6 GPRs (B, C, D, E, H, L), or **M** (memory location pointed by HL pair)
- **Category:** 1-byte long instruction
- **Opcodes:** 8 (ORA A, ORA B, ORA C, ORA D, ORA E, ORA H, ORA L, ORA M)

### Flags for `ORA R`:
- Carry → always reset
- Auxiliary Carry → always reset
- Parity, Zero, Sign → based on accumulator result

---

### Worked Example: `ORA E`

**Given:**
- Accumulator = `12` (hex)
- Register E = `AB` (hex)

### Step 1: Convert to binary
- 1 → `0001`
- 2 → `0010`
- **12 in binary = `0001 0010`**

- A → `1010`
- B → `1011`
- **AB in binary = `1010 1011`**

### Step 2: Perform OR bit-by-bit
```
  0001 0010   (12)
OR
  1010 1011   (AB)
-----------
  1011 1011
```

- Least significant nibble: `0010 OR 1011` = `1011` (output is 1 wherever at least one input is 1)
- Most significant nibble: `0001 OR 1010` = `1011`
- **Result: `1011 1011` = `BB` in hex**

### Step 3: Determine Flags

| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | Always reset for OR |
| Auxiliary Carry (AC) | **0 (Reset)** | Always reset for OR |
| Parity (P) | **1 (Set)** | `10111011` has **six** 1s → even → parity set |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 (has ones present) |
| Sign (S) | **1 (Set)** | MSB of `BB` (`1011...`) = 1 |

### ✅ Result: **A = BBH**, Flags → Parity (P) set, Sign (S) set; Carry, Auxiliary Carry, Zero all reset.

---

## 5. Instruction Type 2: `ORI D8` (OR Immediate with Accumulator)

- **Mnemonic breakdown:** `ORI` = OR Immediate → "OR immediate with accumulator"
- Operand 1 = accumulator content
- Operand 2 = **8-bit immediate data**, sent directly in the instruction
- **Category:** 2-byte long instruction
  - Byte 1: opcode for `ORI`
  - Byte 2: the 8-bit immediate data
- **Opcodes:** Only **1** (single opcode since data is immediate, no register/memory variant)

### Flags for `ORI D8`:
- Carry → always reset
- Auxiliary Carry → always reset
- Parity, Zero, Sign → based on accumulator result

---

### Worked Example: `ORI F3`

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

### Step 2: Perform OR bit-by-bit
```
  0100 0101   (45)
OR
  1111 0011   (F3)
-----------
  1111 0111
```

- Least significant nibble: `0101 OR 0011` = `0111`
- Most significant nibble: `0100 OR 1111` = `1111`
- **Result: `1111 0111` = `F7` in hex**

### Step 3: Determine Flags

| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | Always reset for OR |
| Auxiliary Carry (AC) | **0 (Reset)** | Always reset for OR |
| Parity (P) | **0 (Reset)** | `11110111` has **seven** 1s → odd → parity reset |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **1 (Set)** | MSB of `F7` (`1111...`) = 1 |

### ✅ Result: **A = F7H**, Flags → Sign (S) set; Carry, Auxiliary Carry, Parity, Zero all reset.

---

## 6. Summary Comparison Table

| Feature | `ORA R` | `ORI D8` |
|---|---|---|
| Full Meaning | OR accumulator with R | OR immediate with accumulator |
| Second Operand Source | Register (B,C,D,E,H,L,A) or Memory (M via HL) | Immediate 8-bit data in instruction |
| Instruction Length | 1 byte | 2 bytes |
| Number of Opcodes | 8 | 1 |
| Carry Flag | Always reset | Always reset |
| Auxiliary Carry Flag | Always reset | Always reset |
| Other Flags (P, Z, S) | Based on result | Based on result |

---

## 7. AND vs OR — Quick Comparison

| Aspect | AND | OR |
|---|---|---|
| Truth logic | Output = 1 only if **both** inputs = 1 | Output = 1 if **at least one** input = 1 |
| Special identity | `X AND 1 = X` | `X OR 0 = X` |
| Special override | `X AND 0 = 0` | `X OR 1 = 1` |
| Bit manipulation use | **Selectively RESET** bits (using 0) | **Selectively SET** bits (using 1) |
| Carry Flag | Always reset | Always reset |
| Auxiliary Carry Flag | Always **set** | Always **reset** |

---

## 8. Key Points to Remember

1. **OR (inclusive OR)** operation: output is 1 if either or both inputs are 1; output is 0 only when both inputs are 0.
2. Called "**inclusive**" OR because it includes the case where both inputs are 1 (unlike Exclusive OR, covered next).
3. **`X OR 0 = X`** (bit preserved) and **`X OR 1 = 1`** (bit forced to 1) — these enable **selective bit setting** in the accumulator.
4. **Fixed flag behavior for ALL OR operations:** Both **Carry** and **Auxiliary Carry** flags are **always reset** — an Intel-defined convention.
5. **`ORA R`**: 1-byte instruction, 8 opcodes (covers register and memory operand — accumulator, B, C, D, E, H, L, M).
6. **`ORI D8`**: 2-byte instruction, only 1 opcode (covers immediate data operand).
7. AND lets you **reset** bits selectively; OR lets you **set** bits selectively — these are complementary bit-manipulation tools.

---

## 9. What's Next

- Next session: Instructions based on **Exclusive OR (XOR) logic**.

---
*Notes prepared from video lecture transcript — "Logical Group of Instructions - ORA R and ORI D8"*
