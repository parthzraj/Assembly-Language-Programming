# 8085 Microprocessor — One's Complement & Two's Complement (Assembly Language Notes)

> Notes compiled from a two-part video tutorial series on 1's Complement and 2's Complement in 8085 Microprocessor Assembly Language Programming (GNUSim8085 simulator used for execution).

---

## Table of Contents

1. [Part 1: One's Complement](#part-1-ones-complement)
   - [1.1 What is One's Complement](#11-what-is-ones-complement)
   - [1.2 The CMA Instruction](#12-the-cma-instruction)
   - [1.3 One's Complement Program (Step-by-Step)](#13-ones-complement-program-step-by-step)
   - [1.4 Application: Representing Signed Binary Numbers](#14-application-representing-signed-binary-numbers)
   - [1.5 Practice Exercise](#15-practice-exercise)
2. [Part 2: Two's Complement](#part-2-twos-complement)
   - [2.1 What is Two's Complement](#21-what-is-twos-complement)
   - [2.2 Instructions Used for Two's Complement](#22-instructions-used-for-twos-complement)
   - [2.3 Two's Complement Program (Step-by-Step)](#23-twos-complement-program-step-by-step)
   - [2.4 Application: Signed Arithmetic (Addition of +6 and -6)](#24-application-signed-arithmetic-addition-of-6-and--6)
   - [2.5 Practice Exercise](#25-practice-exercise)
3. [Quick Reference / Summary Table](#quick-reference--summary-table)

---

## Part 1: One's Complement

### 1.1 What is One's Complement

**Definition:** One's complement means **inverting every bit** of a binary number — every `0` becomes `1`, and every `1` becomes `0`.

**Example: Find the 1's complement of 08H (hexadecimal)**

Step 1 — Convert hex to binary (8-bit representation, since GNUSim8085 registers work with hexadecimal/8-bit values):

```
08 (Hex) = 0000 1000 (Binary)
```

Step 2 — Invert every bit (0→1, 1→0):

```
0000 1000   →   1111 0111
```

Step 3 — Convert the inverted binary back to hexadecimal:

```
1111 0111 (Binary) = F7 (Hex)
```

**Result:** The 1's complement of `08H` is **`F7H`**.

> Note: The example is worked in hexadecimal because GNUSim8085's registers display and store values in hexadecimal format, so it matches what you'll see during simulation.

---

### 1.2 The CMA Instruction

- **Instruction:** `CMA`
- **Full form:** **C**o**M**plement **A**ccumulator
- **Syntax:** Just `CMA` — it is a **single, standalone instruction**. No operand, register, or value needs to be written after it.
- **Key rule:** `CMA` works **only on the Accumulator (register A)**. This means before you can take the 1's complement of any number, that number **must first be loaded into the Accumulator**.

---

### 1.3 One's Complement Program (Step-by-Step)

**Problem Statement:** Write an Assembly Language Program (ALP) to perform the one's complement of a given number.

**Logic:**
1. Load the given data into the Accumulator.
2. Apply the `CMA` instruction to complement it.
3. Stop execution.

**Assembly Code:**

```asm
MVI A, 08H   ; Load the value 08H into the Accumulator
CMA          ; Complement the Accumulator content (1's complement)
HLT          ; Stop program execution
```

**Instruction-by-instruction explanation:**

| Instruction | Meaning | Effect |
|---|---|---|
| `MVI A, 08H` | Move Immediate value 08H into register A | Accumulator (A) = 08H |
| `CMA` | Complement Accumulator | Inverts every bit of A → A = F7H |
| `HLT` | Halt | Stops the program |

**Execution Trace (as observed step-by-step in GNUSim8085):**

1. Program is typed into the editor and **assembled successfully** (no syntax errors).
2. **After Instruction 1 (`MVI A, 08H`):** Accumulator content becomes `08`.
3. **After Instruction 2 (`CMA`):** Accumulator content is complemented and becomes `F7`.
4. **After Instruction 3 (`HLT`):** Program execution stops.

**Final Result:** Accumulator = `F7H`, which is the 1's complement of `08H`.

---

### 1.4 Application: Representing Signed Binary Numbers

The main real-world use of 1's complement is to **represent signed (positive and negative) numbers** in binary.

**Concept:**
- In binary signed representation, the **Most Significant Bit (MSB)** is called the **Sign Bit**.
  - `0` in the sign bit → **Positive** number
  - `1` in the sign bit → **Negative** number
- The remaining bits are called the **Magnitude** bits.

**Worked Example: Represent +6 and −6 using a 5-bit register**

Step 1 — Convert 6 to binary:

```
6 (Decimal) = 0110 (Binary) → occupies 4 bits
```

Step 2 — Since we're using a 5-bit register, and 4 bits are already used for magnitude, 1 bit remains for the **sign bit**.

Step 3 — Represent **+6**:

```
Sign Bit (0) + Magnitude (0110) = 0 0110
```

(Sign bit = 0 → positive)

Step 4 — To get **−6**, take the **1's complement of +6**:

```
+6  = 0 0110
1's complement (invert all bits) → 1 1001
```

**Result:** `−6 = 1 1001` in the 5-bit register.

**Key takeaway:** Notice that `−6` is *exactly* the one's complement of `+6`. This demonstrates that **1's complement is the mechanism used to represent negative numbers in binary signed number systems**.

---

### 1.5 Practice Exercise

**Question posed in the video:**
> How would you represent **+120** and **−120** in binary? How many bits would the register need? Work it out yourself.

*(The video mentions the solution is posted on the presenter's blog, cheerakbaloria.com — solve it yourself first as an exercise.)*

---

## Part 2: Two's Complement

### 2.1 What is Two's Complement

**Definition:**

```
2's Complement = 1's Complement + 1
```

In other words, to find the two's complement of a binary number:
1. First find its 1's complement (invert all bits).
2. Then add `1` to the result.

**Worked Example: Find the 2's complement of 08H**

Step 1 — Convert hex to binary:

```
08 (Hex) = 0000 1000 (Binary)
```

Step 2 — Find the 1's complement (invert all bits):

```
0000 1000  →  1111 0111
```

Step 3 — Add `1` to the 1's complement value:

```
  1111 0111
+ 0000 0001
-----------
```

Bit-by-bit addition (right to left):
- Bit 0: `1 + 1 = 0`, carry `1`
- Bit 1: `1 + 1 = 0`, carry `1`
- Bit 2: `1 + 1 = 0`, carry `1`
- Bit 3: `1 + 0 = 1`, no carry (carry from previous bit consumed here)
- Remaining higher bits stay `1` (unaffected)

```
Result: 1111 1000
```

Step 4 — Convert the result back to hexadecimal:

```
1111 1000 (Binary) = F8 (Hex)
```

**Result:** The 2's complement of `08H` is **`F8H`**.

> As with Part 1, hexadecimal is used throughout because GNUSim8085 stores/displays register values in hex.

---

### 2.2 Instructions Used for Two's Complement

Unlike 1's complement (which has the dedicated `CMA` instruction), **there is no single dedicated instruction for 2's complement** in the 8085 instruction set. Instead, it is implemented by **combining two instructions**:

1. **`CMA`** — Complement the Accumulator (performs the 1's complement step).
2. **`ADI 01H`** (or `ADD` with a register holding 1) — Add 1 to the Accumulator content (the "+1" step of two's complement).

**Summary:**

```
2's Complement = CMA  (1's complement)
                 +
                 ADI 01H  (add 1)
```

---

### 2.3 Two's Complement Program (Step-by-Step)

**Problem Statement:** Write an Assembly Language Program (ALP) to perform the two's complement of a given number.

**Logic:**
1. Load the given data into the Accumulator.
2. Apply `CMA` to get the 1's complement.
3. Add `1` using `ADI 01H` to complete the two's complement.
4. Stop execution.

**Assembly Code:**

```asm
MVI A, 08H   ; Load the value 08H into the Accumulator
CMA          ; Perform 1's complement → A becomes F7H
ADI 01H      ; Add 01H to Accumulator → A becomes F8H (2's complement)
HLT          ; Stop program execution
```

**Instruction-by-instruction explanation:**

| Instruction | Meaning | Accumulator Value After |
|---|---|---|
| `MVI A, 08H` | Load 08H into register A | A = 08H |
| `CMA` | Complement A (1's complement) | A = F7H |
| `ADI 01H` | Add immediate value 01H to A | A = F8H |
| `HLT` | Halt program | — |

**Execution Trace (as observed step-by-step in GNUSim8085):**

1. Code is entered and the program is **assembled successfully**.
2. **After Instruction 1 (`MVI A, 08H`):** Accumulator = `08`.
3. **After Instruction 2 (`CMA`):** Accumulator becomes `F7` (this is the *one's* complement — an intermediate step).
4. **After Instruction 3 (`ADI 01H`):** `01` is added to `F7`, giving Accumulator = `F8`.
5. **After Instruction 4 (`HLT`):** Program stops.

**Final Result:** Accumulator = `F8H`, which is the 2's complement of `08H`.

---

### 2.4 Application: Signed Arithmetic (Addition of +6 and −6)

The primary applications of 2's complement are:
- Representing **signed numbers** in binary.
- Performing **arithmetic operations** (addition, subtraction, etc.) on signed binary numbers.

**Worked Example: Prove that (+6) + (−6) = 0 using binary/2's complement arithmetic**

We already know mathematically that `+6 + (−6) = 0`. The example demonstrates this holds true using binary 2's complement representation.

**Step 1 — Represent +6 in binary:**

```
+6 (Decimal) = 0 0110 (Binary)
```

**Step 2 — Find the 1's complement of this value (as an intermediate step toward −6):**

```
0 0110  →  1 1001
```

**Step 3 — Add 1 to the 1's complement to get the 2's complement (i.e., −6):**

```
  1 1001
+ 0 0001
--------
```

Bit-by-bit addition:
- Bit 0: `1 + 1 = 0`, carry `1`
- Bit 1: `0 + 0 + carry(1) = 1`, no carry
- Remaining bits: `0 1 1` unchanged

```
Result: 1 1010
```

**So, −6 (in 2's complement, 5-bit) = `1 1010`**

**Step 4 — Add (+6) and (−6) in binary:**

```
  +6  =  0 0110
  -6  =  1 1010
  ---------------
```

Bit-by-bit addition (right to left):
- Bit 0: `0 + 0 = 0`, no carry
- Bit 1: `1 + 1 = 0`, carry `1`
- Bit 2: `1 + 1 (+carry 1) = 1`, carry `1` → *(explained in video as: 1+1=0, generate carry 1, again 1+1=0, generate carry, then final 1+1=0, carry ignored)*
- Continue propagating carries through remaining bits
- The final carry-out (overflow beyond the register width) is **ignored/discarded**

```
Final Result: 0 0000
```

**Result:** `(+6) + (−6) = 00000` → **Zero**, exactly as expected mathematically. This confirms that 2's complement correctly represents negative numbers such that ordinary binary addition produces the correct signed arithmetic result.

**Key takeaway:** This is precisely *why* 2's complement is the standard method computers use to perform subtraction — subtraction of B from A can be done as **A + (2's complement of B)**, using only an adder circuit.

---

### 2.5 Practice Exercise

**Question posed in the video:**
> Calculate **(+7) + (−3)** using the two's complement method. (Hint: The expected answer is `4`.) Work through the binary/2's complement steps yourself.

*(Solution reportedly available on the presenter's blog, cheerakbaloria.com.)*

---

## Quick Reference / Summary Table

| Concept | Rule | 8085 Instruction(s) | Example (08H) |
|---|---|---|---|
| **1's Complement** | Invert every bit (0↔1) | `CMA` | `08H → F7H` |
| **2's Complement** | 1's Complement + 1 | `CMA` followed by `ADI 01H` | `08H → F7H → F8H` |

| Sign Bit Value | Meaning |
|---|---|
| `0` | Positive number |
| `1` | Negative number |

**Core Formulas:**

```
1's Complement of N  = Invert all bits of N
2's Complement of N  = (1's Complement of N) + 1
```

**Why it matters:**
- **1's Complement** → Used to represent signed numbers (basic form); has the limitation of "two representations of zero" (+0 and −0), which is why most systems prefer 2's complement.
- **2's Complement** → The standard used in real digital systems (including the 8085 and virtually all modern processors) for representing signed integers and for performing subtraction via addition, because it has a single, unambiguous representation of zero and simplifies arithmetic circuit design.

---

*Notes compiled from a two-part tutorial video series demonstrating 1's Complement and 2's Complement in 8085 Microprocessor Assembly Language, executed and verified using the GNUSim8085 simulator.*