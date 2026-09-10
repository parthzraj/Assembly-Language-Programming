# 22 - Arithmetic Instructions: Solved Problems (8085 Microprocessor)

> Practical problem-solving session applying the arithmetic instructions learned so far: `ADD`, `ADC`, `SUI`, `SBB`, `SUB`.
> One question, **five parts**, each executed **sequentially** on a shared initial register/memory/flag state, computing the resulting **accumulator value** and **flags register** after each instruction.

---

## 1. Initial State (Given, Before Any Instruction Executes)

| Register / Location | Value |
|---|:---:|
| Accumulator (A) | `65H` |
| Register B | `B2H` |
| Register H | `F9H` |
| Register L | `50H` |
| Carry Flag (CY) | **1 (Set)** |
| Memory location `F950H` (pointed by HL pair) | `38H` |

> **Important:** Each of the 5 instructions below is evaluated **independently**, starting fresh from this same initial state (not chained one after another) — i.e., the accumulator is reset to `65H` and carry back to `1` before each new instruction.

### Flags Register Recap (rules used throughout):
- **Carry (CY):** Set if addition produces a carry out of the MSB. For subtraction (done via 2's complement addition), CY = 1 means **no borrow needed** (positive result) → carry is fed through a **NOT gate**, so it appears as **reset**. CY = 0 in the raw addition means **borrow was needed** → after the NOT gate, the carry flag shows as **set**.
- **Parity (P):** Set to 1 if the accumulator has an **even number of 1s** in binary; reset if odd.
- **Auxiliary Carry (AC):** Set if a carry is generated out of the **least significant nibble (4 bits)** during addition.
- **Zero (Z):** Set if the accumulator is **all zeros**.
- **Sign (S):** Set if the **most significant bit (MSB)** of the accumulator is 1 (i.e., value looks "negative" in signed interpretation).

---

## 2. Instruction 1: `ADD L`

**Operation:** Accumulator ← Accumulator + Register L

- Accumulator = `65H`, Register L = `50H`
- 65H + 50H:
  - Least significant digit: 5 + 0 = **5**
  - Most significant digit: 6 + 5 = 11 decimal = **B** (hex)
- **Result: B5H**

### Binary of B5H:
`1011 0101`

### Flags:
| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | No carry generated out of MSB during this addition |
| Parity (P) | **0 (Reset)** | B5H = `1011 0101` → five 1s → odd → parity reset |
| Auxiliary Carry (AC) | **0 (Reset)** | 5 + 0 = 5, no carry out of lower nibble |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **1 (Set)** | MSB of B5H (`1011...`) = 1 |

### ✅ Result: **A = B5H**, Flags → only **Sign (S)** is set.

---

## 3. Instruction 2: `ADC B`

**Operation:** Accumulator ← Accumulator + Register B + Carry

- Accumulator = `65H`, Register B = `B2H`, Carry = `1`
- First, B2H + 1 (carry) = **B3H**
- Now add 65H + B3H:
  - Least significant digit: 5 + 3 = **8**
  - Most significant digit: 6 + B (11) = 17 decimal = **11H** → write **1**, carry **1** out of the accumulator (3-digit result: 118, but accumulator only holds 2 hex digits)
- **Result stored in accumulator: 18H** (with the overflow bit going into the Carry flag)

### Binary of 18H:
`0001 1000`

### Flags:
| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **1 (Set)** | Addition overflowed past 2 hex digits (result was 3 digits: "118") |
| Parity (P) | **1 (Set)** | 18H = `0001 1000` → two 1s → even → parity set |
| Auxiliary Carry (AC) | **0 (Reset)** | 5 + 3 = 8, no carry out of lower nibble |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **0 (Reset)** | MSB of 18H (`0001...`) = 0 |

### ✅ Result: **A = 18H**, Flags → **Carry (CY)** and **Parity (P)** are set.

---

## 4. Instruction 3: `SUI 56H`

**Operation:** Accumulator ← Accumulator − 56H (immediate data)

- Accumulator (minuend) = `65H`, Immediate data (subtrahend) = `56H`
- 8085 performs subtraction via **16's complement (2's complement in hex) addition**.

### Step 1: Find 16's complement of 56H
- Least significant digit: 10H (16 decimal) − 6 = **A**
- Most significant digit: F (15 decimal) − 5 = **A**
- 16's complement of 56H = **AAH**

### Step 2: Add 65H + AAH
- Least significant digit: 5 + A(10) = 15 decimal = **F**
- Most significant digit: 6 + A(10) = 16 decimal = **10H** → write **0**, carry **1**

**Raw addition result: 0FH** (carry generated out of MSB)

### Interpreting the Carry:
- Carry **generated** during this 2's-complement addition → result is a **positive value**, and the carry is **ignored** for the numeric result.
- However, for subtraction operations, the carry is passed through a **NOT gate** before updating the Carry flag: raw carry = 1 → NOT(1) = **0** → **Carry flag reset**.

### Binary of 0FH:
`0000 1111`

### Flags:
| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | Raw carry was 1 (positive result) → passed through NOT gate → flag reset |
| Parity (P) | **1 (Set)** | 0FH = `0000 1111` → four 1s → even → parity set |
| Auxiliary Carry (AC) | **0 (Reset)** | 5 + A = F, no carry out of lower nibble |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **0 (Reset)** | MSB of 0FH (`0000...`) = 0 |

### ✅ Result: **A = 0FH**, Flags → only **Parity (P)** is set.

---

## 5. Instruction 4: `SBB M`

**Operation:** Accumulator ← Accumulator − Memory[HL] − Carry (borrow)

- Accumulator (minuend) = `65H`
- HL register pair points to memory address `F950H`, which contains `38H` (subtrahend)
- Carry (borrow) = `1` initially set

### Step 1: Incorporate the borrow into the subtrahend
- Effective subtraction: 65H − 38H − 1 = 65H − (38H + 1) = 65H − **39H**

### Step 2: Find 16's complement of 39H
- Least significant digit: 10H (16) − 9 = **7**
- Most significant digit: F (15) − 3 = **C**
- 16's complement of 39H = **C7H**

### Step 3: Add 65H + C7H
- Least significant digit: 5 + 7 = 12 decimal = **C**
- Most significant digit: 6 + C(12) = 18 decimal = **12H** → write **2**, carry **1**

**Raw addition result: 2CH** (carry generated out of MSB — only the last two digits `2C` are kept in the accumulator)

### Interpreting the Carry:
- Carry generated → result is **positive** → carry ignored for the numeric value.
- Passed through NOT gate: raw carry = 1 → NOT(1) = **0** → **Carry flag reset**.

### Binary of 2CH:
`0010 1100`

### Flags:
| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **0 (Reset)** | Raw carry was 1 (positive result) → NOT gate → flag reset |
| Parity (P) | **0 (Reset)** | 2CH = `0010 1100` → three 1s → odd → parity reset |
| Auxiliary Carry (AC) | **0 (Reset)** | 5 + 7 = C, no carry out of lower nibble |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **0 (Reset)** | MSB of 2CH (`0010...`) = 0 |

### ✅ Result: **A = 2CH**, Flags → **all flags reset to 0**.

---

## 6. Instruction 5: `SUB H`

**Operation:** Accumulator ← Accumulator − Register H

- Accumulator (minuend) = `65H`, Register H (subtrahend) = `F9H`

### Step 1: Find 16's complement of F9H
- Least significant digit: 10H (16) − 9 = **7**
- Most significant digit: F(15) − F(15) = **0**
- 16's complement of F9H = **07H**

### Step 2: Add 65H + 07H
- Least significant digit: 5 + 7 = 12 decimal = **C**
- Most significant digit: 6 + 0 = **6**

**Raw addition result: 6CH** (no carry generated out of MSB this time)

### Interpreting the (Lack of) Carry:
- **No carry generated** during the 2's-complement addition → this indicates a **borrow was needed** to perform the subtraction → result represents a valid subtraction requiring borrow.
- Passed through NOT gate: raw carry = 0 → NOT(0) = **1** → **Carry flag SET** (signifies borrow occurred).

### Binary of 6CH:
`0110 1100`

### Flags:
| Flag | Value | Reasoning |
|---|:---:|---|
| Carry (CY) | **1 (Set)** | Raw carry was 0 (borrow needed) → NOT gate → flag set |
| Parity (P) | **1 (Set)** | 6CH = `0110 1100` → four 1s → even → parity set |
| Auxiliary Carry (AC) | **0 (Reset)** | 5 + 7 = C, no carry out of lower nibble |
| Zero (Z) | **0 (Reset)** | Accumulator ≠ 0 |
| Sign (S) | **0 (Reset)** | MSB of 6CH (`0110...`) = 0 |

### ✅ Result: **A = 6CH**, Flags → **Carry (CY)** and **Parity (P)** are set.

---

## 7. Summary Table of All 5 Instructions

| # | Instruction | Operation | Result in Accumulator | Flags Set |
|:---:|---|---|:---:|---|
| 1 | `ADD L` | 65H + 50H | **B5H** | Sign (S) |
| 2 | `ADC B` | 65H + B2H + Carry(1) | **18H** | Carry (CY), Parity (P) |
| 3 | `SUI 56H` | 65H − 56H | **0FH** | Parity (P) |
| 4 | `SBB M` | 65H − 38H − Carry(1) | **2CH** | *(none — all reset)* |
| 5 | `SUB H` | 65H − F9H | **6CH** | Carry (CY), Parity (P) |

---

## 8. Key Concepts Reinforced in This Session

1. **Subtraction in 8085 is always performed via addition**, using the **16's (2's) complement** of the subtrahend — the ALU hardware only knows how to add.
2. **How to compute 16's complement of a hex number quickly** (digit-by-digit):
   - Least significant digit: subtract from **10H (16 decimal)**
   - All other digits: subtract from **F (15 decimal)**
3. **Interpreting carry after a subtraction-via-addition:**
   - Carry **generated** (1) during the raw addition → result is **positive** → carry is **ignored** → after passing through the **NOT gate**, the Carry flag shows **reset (0)**.
   - Carry **NOT generated** (0) during the raw addition → a **borrow** was required → after passing through the **NOT gate**, the Carry flag shows **set (1)**, signaling borrow occurred.
   - This NOT-gate relationship is why, in subtraction, the "Carry flag" is often interpreted as a **"borrow flag."**
4. **`ADC` and `SBB`** incorporate the carry/borrow from a previous operation — essential for multi-byte arithmetic chains.
5. Flags (Parity, Auxiliary Carry, Zero, Sign) follow the **same rules regardless of whether the operation was addition or subtraction** — they're evaluated purely based on the **final accumulator content** (and, for AC, the lower-nibble addition during the process).

---

## 9. What's Next

- Next session: Start of a **new topic — The Logical Group of Instructions**.

---
*Notes prepared from video lecture transcript — "Arithmetic Instructions: Solved Problems"*
