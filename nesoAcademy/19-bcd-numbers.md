# 01 - BCD Numbers (Binary Coded Decimal)

> Part of the **DAA (Decimal Adjust Accumulator) instruction** series for 8085 Microprocessor.
> This is the foundational session — before understanding the `DAA` instruction, we must first understand BCD numbers.

---

## 1. Why This Topic Matters

The `DAA` instruction helps perform **decimal addition** in the 8085 microprocessor.
Before learning `DAA` itself, we need to understand the concept of **BCD (Binary Coded Decimal) numbers**, since `DAA` operates on BCD data.

---

## 2. Background: How Data is Represented in a Computer

- In a digital computer, **everything** (instructions + data) is represented using sequences of `1`s and `0`s.
- A **program** = a group of instructions, and each instruction is a sequence of 1s and 0s.
- **Data** can also be represented as sequences of 1s and 0s, and data itself can be of multiple types:
  - Unsigned numbers
  - Signed numbers
  - Floating point numbers (used for fractions)
  - **Binary Coded Decimal (BCD) numbers** ← today's topic

> **Key idea:** The *same* sequence of 1s and 0s can mean different things depending on how we **interpret** it (as an instruction, unsigned number, signed number, float, or BCD).

---

## 3. Why Do We Need BCD Numbers?

**Motivating example:** Suppose we want to total a student's marks from an exam.

- Marks are naturally represented in **decimal**.
- But computers/microprocessors only understand **binary**.
- So, when a microprocessor needs to add decimal values directly (like adding marks) without converting to a completely different binary representation, we use **BCD numbers**.

**BCD = Binary Coded Decimal** → we are literally *coding decimal digits in binary form*.

---

## 4. How BCD Numbers Are Coded

- BCD uses **4 bits** to represent each single decimal digit (0–9).
- The 4-bit positions have **place values**: `8 4 2 1`
  - Rightmost bit → place value **1** (2⁰)
  - Next bit → place value **2** (2¹)
  - Next bit → place value **4** (2²)
  - Leftmost bit → place value **8** (2³)

### Examples of single-digit BCD encoding:

| Decimal Digit | BCD (4-bit) |
|:---:|:---:|
| 0 | 0000 |
| 1 | 0001 |
| 5 | 0101 (4+1=5) |

With 4 bits, **16 combinations** are possible in total (this is the same range used for hexadecimal digits 0–F).
But BCD only needs **10** of those 16 combinations (for digits 0–9).

---

## 5. Valid vs Invalid BCD Sequences

- Since only digits `0`–`9` need encoding, the 4-bit sequences from **`1010` to `1111`** (i.e., decimal 10–15 in plain binary) are **NOT valid BCD codes** for a single digit.
- These are called **invalid BCD sequences** — there are exactly **6** of them (1010, 1011, 1100, 1101, 1110, 1111).

### Why "10" and above are treated differently

A two-digit decimal number (like 10, 11, 12...) is **not** encoded as a single 4-bit group. Instead, **each digit gets its own 4-bit group.**

### Two-digit decimal → BCD examples:

| Decimal | BCD Encoding (MSD + LSD) |
|:---:|:---:|
| 10 | 0001 0000 |
| 11 | 0001 0001 |
| 12 | 0001 0010 |
| 13 | 0001 0011 |
| 14 | 0001 0100 |
| 15 | 0001 0101 |

- **Single-digit decimal numbers** → 4-bit BCD code.
- **Double-digit decimal numbers** → 8-bit (1-byte) BCD code (4 bits per digit).

---

## 6. The Problem: Binary Addition Doesn't Respect BCD Rules

- The microprocessor's hardware naturally performs **binary addition**, not decimal-aware addition.
- If we add two BCD digits using plain binary addition, the result can fall into the **invalid BCD range** (1010–1111).

**Example:** Adding 9 + 1 in binary does **not** directly give the correct two-digit BCD result (`0001 0000`). Instead it gives `1010` (an invalid BCD sequence), because binary counting continues past 9 without "rolling over" into a new decimal digit the way BCD needs it to.

### The Fix: Correction (Adding 6)

Whenever a binary addition of BCD digits produces an **invalid BCD sequence** (or generates a carry), we apply a **correction** by adding **`0110` (6 in binary)** to that result.

**Why 6 specifically?**
- There are exactly **6 invalid BCD sequences** (1010 through 1111).
- Adding 6 "skips over" these 6 invalid codes, pushing the result into a valid BCD range and correctly generating a carry into the next decimal digit position.

---

## 7. Worked Example 1: Correcting Invalid BCD `1010` (represents result of 9+1)

Add `0110` (6) to `1010`:

```
   1010
 + 0110
 ------
 1 0000
```

Step-by-step bit addition (from LSB to MSB):
- Bit 0: 0 + 0 = 0
- Bit 1: 1 + 1 = 10 → write 0, carry 1
- Bit 2: 0 + 1 (carry) + 1 = 10 → write 0, carry 1
- Bit 3: 1 + 1 (carry) + 0 = 10 → write 0, carry 1
- Final carry out = 1

**Result:** `1 0000` (a 5-bit number)

- Pad with leading zeros to make it a clean 4-bit group + carry: `0001 0000`
- This is exactly the valid BCD encoding of **10** (`0001 0000` = digit "1" then digit "0"). ✅

---

## 8. Worked Example 2: Correcting Invalid BCD `1110` (represents 14)

Add `0110` (6) to `1110`:

```
   1110
 + 0110
 ------
 1 0100
```

Step-by-step bit addition:
- Bit 0: 0 + 0 = 0
- Bit 1: 1 + 1 = 10 → write 0, carry 1
- Bit 2: 1 + 1 (carry) + 1 = 11 → write 1, carry 1
- Bit 3: 1 + 1 (carry) + 0 = 10 → write 0, carry 1
- Final carry out = 1

**Result:** `1 0100` (5-bit number)

- Pad appropriately: `0001 0100`
- This equals decimal **14** in valid BCD (digit "1" then digit "4"). ✅

**Conclusion:** Adding the correction value **6** to any invalid BCD sequence converts it into the correct valid BCD representation (with proper carry generation into the next digit).

---

## 9. Multi-Digit BCD Example: Decimal 1024

Let's encode **1024** in BCD vs plain Binary to see the difference.

### BCD Encoding of 1024 (digit by digit):

| Digit | BCD (4-bit) |
|:---:|:---:|
| 1 | 0001 |
| 0 | 0000 |
| 2 | 0010 |
| 4 | 0100 |

**Full BCD for 1024:** `0001 0000 0010 0100` (16 bits total — 4 bits per digit × 4 digits)

### Plain Binary Encoding of 1024:

Using place values (each place double the previous): `1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024...`

**1024 in plain binary:** `10000000000` (just `1` at the 1024 place value, rest zeros)

### Key Takeaway

- Both representations use only 1s and 0s.
- **BCD** = decimal digits individually encoded in binary (easier to convert back to decimal digit-by-digit, but takes more bits).
- **Plain Binary** = the actual binary equivalent of the whole decimal value (more compact, but not easily separable into decimal digits).
- The correct interpretation of a bit sequence depends entirely on **which encoding scheme** is being used.

---

## 10. Summary / Key Points to Remember

1. **BCD (Binary Coded Decimal)** encodes each decimal digit (0–9) using 4 bits.
2. Place values in a 4-bit BCD group: **8, 4, 2, 1**.
3. Only **10 out of 16** possible 4-bit combinations are valid BCD codes (0000–1001). The remaining 6 (1010–1111) are **invalid**.
4. Multi-digit decimal numbers are encoded **digit by digit** — each digit gets its own 4-bit group (e.g., a 2-digit number needs 8 bits / 1 byte).
5. Since microprocessor hardware adds in plain binary, adding two BCD digits can produce an **invalid BCD result**.
6. The fix is a **correction**: add **6 (0110)** to the invalid result — this generates the correct carry and brings the result back into valid BCD range.
7. This concept of correction is the **foundation for understanding the `DAA` (Decimal Adjust Accumulator) instruction** in the 8085 microprocessor, which will be covered in the next session.

---

## 11. What's Next

- Next session: Deeper theoretical discussion of the **`DAA` instruction**.
- Session after that: **Practical usage** of the `DAA` instruction with actual 8085 examples.

---
*Notes prepared from video lecture transcript — "Instruction Type DAA - Part 1 (BCD Numbers)"*
