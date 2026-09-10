# 25 - Logical Group of Instructions: XRA R and XRI D8 (Exclusive OR Operation)

> Continuation of **The Logical Group of Instructions** for the 8085 microprocessor.
> Covers: basics of the **Exclusive OR (XOR) operation**, and instruction types **`XRA R`** and **`XRI D8`**.

---

## 1. Recap

- Previous sessions: **AND** (`ANA R`, `ANI D8`) and **OR** (`ORA R`, `ORI D8`).
- This session: **Exclusive OR / XOR** (`XRA R`, `XRI D8`).
- The 8085 Logical Group supports: **AND, OR, Exclusive OR, NOT**.

---

## 2. Basics of the Exclusive OR (XOR) Operation

- Just like AND and OR, **XOR is a binary operation** — needs two operands.
  - Operand 1: **always in the accumulator**
  - Operand 2: can come from
    1. Any **GPR** (B, C, D, E, H, L)
    2. **Memory location** pointed by HL pair (denoted `M`)
    3. **8-bit immediate data** in the instruction

### Instruction mapping:
- **`XRA R`** → handles cases 1 and 2 (register or memory)
- **`XRI D8`** → handles case 3 (immediate data)

### Truth Table for XOR

| X | Y | X XOR Y |
|:---:|:---:|:---:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

**Output = 1 only when the inputs are DIFFERENT (alternating bits). Output = 0 when inputs are the SAME.**

### Why "Exclusive" OR?
- Recall: OR is called **Inclusive OR** because when both inputs are 1, the output is still 1 (it "includes" the AND case).
- **XOR excludes that case** — when both inputs are 1, the output becomes **0**. Hence, "Exclusive OR": it excludes the AND logic that inclusive OR keeps.

### Two Important Special Cases:

1. **X XOR 0 = X**
   - When Y is fixed at 0, the output always equals X (unchanged) — whatever X is, it passes through.
2. **X XOR 1 = X̄ (complement of X)**
   - When Y is fixed at 1, the output is always the **inverted/complemented** value of X.
     - If X = 0 → output = 1
     - If X = 1 → output = 0

### 🔑 Key Insight: XOR as a "Controlled Inverter"
- If the second operand bit = **0** → output = same as input (no change)
- If the second operand bit = **1** → output = **complement** of input (inverted)
- This is why XOR is also known as a **controlled inverter** — you control whether inversion happens by choosing the second operand's bits.

### Application: Selectively Complementing Bits

Using AND we could **selectively reset** bits (via 0), and using OR we could **selectively set** bits (via 1). With **XOR**, we can **selectively complement (invert)** specific bits.

**Example concept:** To complement only the **least significant bit (LSB)** of the accumulator while leaving all other bits unchanged:
- Choose the second operand as: **seven 0s followed by a single 1** (i.e., `0000 0001`)
- XORing: the LSB gets inverted (since it's XORed with 1), and all other bits remain unchanged (since they're XORed with 0).

---

## 3. Special Trick: `XRA A` (XOR-ing Accumulator with Itself)

- Since XOR of **identical bits always gives 0**, executing `XRA A` (i.e., XOR-ing the accumulator with its own content) will make **every bit 0**.
- **Result: `XRA A` resets/clears the entire accumulator to `00`.**
- This is a common, efficient technique used in 8085 programming to zero out the accumulator.

---

## 4. Flags Behavior for XOR Operations (Fixed by Intel)

| Flag | Behavior |
|---|---|
| **Carry (CY)** | Always **RESET** (0) |
| **Auxiliary Carry (AC)** | Always **RESET** (0) |
| Parity (P), Zero (Z), Sign (S) | Determined normally, based on the **resulting accumulator content** |

> ⚠️ XOR is treated as a **variation of OR**, so it follows the same fixed flag rule as OR: **both CY and AC are always reset.**

---

## 5. Instruction Type 1: `XRA R` (Exclusive OR Accumulator with R)

- **Mnemonic breakdown:** `X` = Exclusive, `R` = OR, `A` = Accumulator → "Exclusive OR accumulator with R"
- **`R`** represents: Accumulator, all 6 GPRs (B, C, D, E, H, L), or **M** (memory location pointed by HL pair)
- **Category:** 1-byte long instruction
- **Opcodes:** 8 (XRA A, XRA B, XRA C, XRA D, XRA E, XRA H, XRA L, XRA M)

### Flags for `XRA R`:
- Carry → always reset
- Auxiliary Carry → always reset
- Parity, Zero, Sign → based on accumulator result

---

### Worked Example: `XRA E`

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

### Step 2: Perform XOR bit-by-bit
```
  0001 0010   (12)
XOR
  1010 1011   (AB)
-----------
  1011 1001
```

- Least significant nibble: `0010 XOR 1011` = `1001`
  - bit0: 0⊕1=1, bit1: 1⊕1=0, bit2: 0⊕0=0, bit3: 0⊕1=1 → `1001`
- Most significant nibble: `0001 XOR 1010` = `1011`
  - bit0: 1⊕0=1, bit1: 0⊕1=1, bit2: 0⊕0=0, bit3: 1⊕1=0 → `1011`
- **Result: `1011 1001` = `B9` in hex**

### Step 3: Determine Flags

| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | Always reset for XOR |
| Auxiliary Carry (AC) | **0 (Reset)** | Always reset for XOR |
| Parity (P) | **0 (Reset)** | `10111001` has **five** 1s → odd → parity reset |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **1 (Set)** | MSB of `B9` (`1011...`) = 1 |

### ✅ Result: **A = B9H**, Flags → Sign (S) set; Carry, Auxiliary Carry, Parity, Zero all reset.

---

## 6. Instruction Type 2: `XRI D8` (Exclusive OR Immediate with Accumulator)

- **Mnemonic breakdown:** `X` = Exclusive, `R` = OR, `I` = Immediate → "Exclusive OR immediate with accumulator"
- Operand 1 = accumulator content
- Operand 2 = **8-bit immediate data**, sent directly in the instruction
- **Category:** 2-byte long instruction
  - Byte 1: opcode for `XRI`
  - Byte 2: the 8-bit immediate data
- **Opcodes:** Only **1** (single opcode since data is immediate)

### Flags for `XRI D8`:
- Carry → always reset
- Auxiliary Carry → always reset
- Parity, Zero, Sign → based on accumulator result

---

### Worked Example: `XRI F3`

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

### Step 2: Perform XOR bit-by-bit
```
  0100 0101   (45)
XOR
  1111 0011   (F3)
-----------
  1011 0110
```

- Least significant nibble: `0101 XOR 0011` = `0110`
  - bit0: 1⊕1=0, bit1: 0⊕1=1, bit2: 1⊕0=1, bit3: 0⊕0=0 → `0110`
- Most significant nibble: `0100 XOR 1111` = `1011`
  - bit0: 0⊕1=1, bit1: 0⊕1=1, bit2: 1⊕1=0, bit3: 0⊕1=1 → `1011`
- **Result: `1011 0110` = `B6` in hex**

### Step 3: Determine Flags

| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | Always reset for XOR |
| Auxiliary Carry (AC) | **0 (Reset)** | Always reset for XOR |
| Parity (P) | **0 (Reset)** | `10110110` has **five** 1s → odd → parity reset |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **1 (Set)** | MSB of `B6` (`1011...`) = 1 |

### ✅ Result: **A = B6H**, Flags → Sign (S) set; Carry, Auxiliary Carry, Parity, Zero all reset.

---

## 7. Summary Comparison Table

| Feature | `XRA R` | `XRI D8` |
|---|---|---|
| Full Meaning | Exclusive OR accumulator with R | Exclusive OR immediate with accumulator |
| Second Operand Source | Register (B,C,D,E,H,L,A) or Memory (M via HL) | Immediate 8-bit data in instruction |
| Instruction Length | 1 byte | 2 bytes |
| Number of Opcodes | 8 | 1 |
| Carry Flag | Always reset | Always reset |
| Auxiliary Carry Flag | Always reset | Always reset |
| Other Flags (P, Z, S) | Based on result | Based on result |

---

## 8. AND vs OR vs XOR — Full Comparison

| Aspect | AND | OR | XOR |
|---|---|---|---|
| Output = 1 when... | **Both** inputs = 1 | **Either/both** inputs = 1 | Inputs are **different** |
| Identity operand | `X AND 1 = X` | `X OR 0 = X` | `X XOR 0 = X` |
| Override operand | `X AND 0 = 0` | `X OR 1 = 1` | `X XOR 1 = X̄` (complement) |
| Bit manipulation use | **Selectively RESET** bits | **Selectively SET** bits | **Selectively COMPLEMENT** bits |
| Carry Flag | Always reset | Always reset | Always reset |
| Auxiliary Carry Flag | Always **set** | Always reset | Always reset |
| Special trick | — | — | `XRA A` clears accumulator to 0 |

---

## 9. Key Points to Remember

1. **XOR (Exclusive OR):** output = 1 only for alternating (different) input bits; output = 0 for identical input bits.
2. XOR is called "exclusive" because it **excludes** the case where both inputs are 1 (unlike inclusive OR).
3. **`X XOR 0 = X`** (bit preserved) and **`X XOR 1 = X̄`** (bit inverted) — this makes XOR a **controlled inverter**, enabling **selective bit complementing**.
4. **Special programming trick:** `XRA A` (XOR accumulator with itself) always results in `00` — a fast way to **clear/reset the accumulator**.
5. **Fixed flag behavior for ALL XOR operations:** Both **Carry** and **Auxiliary Carry** flags are **always reset** (same as OR, since XOR is treated as a variation of OR).
6. **`XRA R`**: 1-byte instruction, 8 opcodes (covers register and memory operand).
7. **`XRI D8`**: 2-byte instruction, only 1 opcode (covers immediate data operand).
8. Together, AND / OR / XOR give three complementary bit-manipulation tools: **reset**, **set**, and **complement**.

---

## 10. What's Next

- Next session: Instructions based on the **NOT logic**.

---
*Notes prepared from video lecture transcript — "Logical Group of Instructions - XRA R and XRI D8"*
