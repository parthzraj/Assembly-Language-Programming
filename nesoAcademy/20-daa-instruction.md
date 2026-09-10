# 20 - DAA Instruction (Decimal Adjust Accumulator) - Part 2

> Part of the **DAA (Decimal Adjust Accumulator) instruction** series for 8085 Microprocessor — Final part.
> Builds directly on the previous session's concept of **BCD numbers** and the **+6 correction**.

---

## 1. Recap from Previous Session

- We studied **BCD (Binary Coded Decimal)** numbers.
- We learned that certain 4-bit sequences are **invalid BCD** (10–15 in binary form).
- To convert an invalid BCD sequence into a valid one, we apply a **correction of 6**.
- Reason for using 6: there are exactly **6 invalid BCD sequences**, and adding 6 skips over them, generating the correct carry while landing on a valid BCD digit.

This session builds on that idea to explain the actual **`DAA` instruction** used in the 8085 microprocessor.

---

## 2. What is the DAA Instruction?

- **DAA** = **D**ecimal **A**djust **A**ccumulator.
- After a binary addition is performed, the result is stored in the **accumulator**.
- The `DAA` instruction adjusts the contents of the accumulator so the result is a **valid BCD number**.
- What `DAA` actually does depends on three things:
  1. Contents of the **accumulator**
  2. The **Auxiliary Carry (AC) flag**
  3. The **Carry (CY) flag**

### Key properties of DAA:
- It is a **1-byte long instruction**.
- It uses **implied / inherent addressing mode** — no operand is mentioned in the instruction itself (similar to `XCHG`, which implicitly operates on the DE and HL register pairs without naming them).
- The microprocessor "knows" that a BCD operation was just performed and that the accumulator's content needs to be validated/corrected — this intent is **implied** by the instruction itself.
- `DAA` is always used **after an addition** has already been performed and stored in the accumulator.

---

## 3. Motivating Example: Adding 38 + 45

Let's understand *why* DAA is needed using decimal 38 + 45.

### BCD representation:
| Decimal Digit | BCD |
|:---:|:---:|
| 3 | 0011 |
| 8 | 1000 |
| 4 | 0100 |
| 5 | 0101 |

### Binary addition:
```
  0011 1000   (38)
+ 0100 0101   (45)
-----------
  0111 1101
```

- Result: `0111 1101`
- In hexadecimal: **7D**
- Breaking it down:
  - Least significant nibble `1101` = 13 decimal = **D** in hex → **this is an invalid BCD digit** (anything above 9 is invalid).
  - This happened because 8 + 5 = 13 in decimal, which should produce **sum digit 3, carry 1** — but plain binary addition doesn't do that "rollover" automatically.
- Since `D` is not a valid decimal symbol, this result is **not valid BCD**, and the `DAA` instruction is needed to fix it.

---

## 4. How DAA Decides What Correction to Apply

The 8085 performs 8-bit (2 hex digit) BCD additions, and DAA examines **both the least significant and most significant hexadecimal digits** separately.

### 4.1 Rule for the Least Significant Digit (uses Auxiliary Carry)

The **addition of the least significant two BCD digits** is what generates the **Auxiliary Carry (AC)**.

| Condition | Action |
|---|---|
| Least significant hex digit **≤ 9** AND **AC = 0** | Result is valid BCD → **no change** to least significant digit |
| Least significant hex digit **> 9** OR **AC = 1** | Result is invalid BCD → **add 6** to the least significant digit |

**Why check AC even when digit ≤ 9?**
Example: adding `8 + 8` in the least significant digits = 16 decimal = `10` in hex.
- Hex digit obtained = `0` (≤ 9 ✅) — but the **carry (AC) was generated** (0 became the sum, 1 was carried).
- So even though the digit alone looks "valid," the AC flag being set signals the digit is actually incorrect and needs correction.

### 4.2 Rule for the Most Significant Digit (uses Carry Flag)

After correcting the least significant digit, that correction might **generate a carry into the most significant digit** — so DAA must also check the most significant digit, this time using the **Carry (CY) flag**.

| Condition | Action |
|---|---|
| Most significant hex digit **≤ 9** AND **CY = 0** | Result is valid BCD → **no change** to most significant digit |
| Most significant hex digit **> 9** OR **CY = 1** | Result is invalid BCD → **add 6** to the most significant digit |

---

## 5. The Four Possible Corrections DAA Can Apply

Based on the above rules, DAA will add exactly one of these four values to the accumulator:

| Correction | When Applied |
|:---:|---|
| **00** | Both digits valid (≤9), no AC, no CY → no change needed |
| **06** | Only least significant digit needs correction |
| **60** | Only most significant digit needs correction |
| **66** | Both digits need correction |

---

## 6. Worked Example 1: 53 + 36 (Correction = 00)

### Hex/BCD addition:
```
  5 3
+ 3 6
-----
```
- Least significant: 3 + 6 = **9**
- Most significant: 5 + 3 = **8**

**Result: 89**

- Least significant digit (9) ≤ 9, no AC generated → valid
- Most significant digit (8) ≤ 9, no CY generated → valid
- **DAA adds `00`** — no correction needed. Result `89` is already valid BCD (matches decimal 53+36=89). ✅

---

## 7. Worked Example 2: 45 + 38 (Correction = 06)

### Hex/BCD addition:
```
  4 5
+ 3 8
-----
```
- Least significant: 5 + 8 = 13 decimal = **D** (hex)
- Most significant: 4 + 3 = **7**

**Result: 7D**

- Least significant digit `D` (13) > 9 → **invalid**, needs correction
- Most significant digit `7` ≤ 9, no CY → valid, no change

**DAA adds `06`:**
```
   7D
 + 06
 ----
```
- D (13) + 6 = 19 decimal
- 19 decimal = **13 in hexadecimal** → write digit **3**, carry **1** into most significant position
- Most significant: 7 + 1 (carry) = **8**

**Final Result: 83**

✅ Check: 45 + 38 = 83 in decimal. Correct!

---

## 8. Worked Example 3: 63 + 42 (Correction = 60)

### Hex/BCD addition:
```
  6 3
+ 4 2
-----
```
- Least significant: 3 + 2 = **5**
- Most significant: 6 + 4 = **10** decimal = **A** (hex)

**Result: A5**

- Least significant digit `5` ≤ 9, no AC → valid, no change
- Most significant digit `A` (10) > 9 → **invalid**, needs correction

**DAA adds `60`:**
```
   A5
 + 60
 ----
```
- Least significant: 5 + 0 = **5** (unchanged)
- Most significant: A (10) + 6 = 16 decimal = **10 in hex** → write digit **0**, carry set (CY flag = 1)

**Final Result: 05** (with Carry flag set, indicating the true result is a 3-digit number: **105**)

✅ Check: 63 + 42 = 105 in decimal. Correct! (05 in accumulator + carry flag = 1 represents the leading "1" of 105)

---

## 9. Worked Example 4: 63 + 88 (Correction = 66)

### Hex/BCD addition:
```
  6 3
+ 8 8
-----
```
- Least significant: 3 + 8 = 11 decimal = **B** (hex)
- Most significant: 6 + 8 = 14 decimal = **E** (hex)

**Result: EB**

- Least significant digit `B` (11) > 9 → **invalid**, needs correction
- Most significant digit `E` (14) > 9 → **invalid**, needs correction

**DAA adds `66`:**
```
   EB
 + 66
 ----
```
- Least significant: B (11) + 6 = 17 decimal = **11 in hex** → write digit **1**, carry **1**
- Most significant: E (14) + 1 (carry) = **F** (15) → then + 6 (correction) = 15 + 6 = 21 decimal = **15 in hex** → write digit **5**, carry set (CY = 1)

**Final Result: 51** (with Carry flag set, representing **151**)

✅ Check: 63 + 88 = 151 in decimal. Correct!

---

## 10. Summary Table of All Four Examples

| Example | Addition | Raw Hex Result | Correction Applied | Final Result | Decimal Check |
|---|:---:|:---:|:---:|:---:|:---:|
| 1 | 53 + 36 | 89 | 00 | 89 | 89 ✅ |
| 2 | 45 + 38 | 7D | 06 | 83 | 83 ✅ |
| 3 | 63 + 42 | A5 | 60 | 05 (CY=1) | 105 ✅ |
| 4 | 63 + 88 | EB | 66 | 51 (CY=1) | 151 ✅ |

---

## 11. Key Points to Remember

1. **DAA (Decimal Adjust Accumulator)** adjusts the result of a binary addition so it becomes valid BCD.
2. It is used **only after an addition** has already placed a result in the accumulator.
3. It is a **1-byte instruction** using **implied/inherent addressing mode** (no operand specified).
4. Decision logic:
   - **Least significant digit** correction depends on: digit > 9 **OR** Auxiliary Carry (AC) = 1 → add **6**
   - **Most significant digit** correction depends on: digit > 9 **OR** Carry (CY) = 1 → add **6**
5. There are exactly **4 possible corrections**: `00`, `06`, `60`, `66` — depending on which digit(s) are invalid.
6. This concludes the **arithmetic instruction group** of the 8085 microprocessor — **14 instruction types**, covering **62 different opcodes** in total.

---

## 12. What's Next

- Next session: **Summary of all 14 arithmetic instruction types** covered so far in the series.

---
*Notes prepared from video lecture transcript — "Instruction Type DAA - Part 2 (Final)"*
