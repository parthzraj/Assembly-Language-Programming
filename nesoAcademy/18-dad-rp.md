# 8085 Microprocessor — Arithmetic Group: DAD RP

> This session covers `DAD RP` — the **"double add"** instruction, used for performing **16-bit addition** between register pairs.

---

## Table of Contents
1. [DAD RP — Double Add Register Pair with HL](#dad-rp--double-add-register-pair-with-hl)
2. [Why It's Called "Double Add"](#why-its-called-double-add)
3. [Worked Example: DAD B](#worked-example-dad-b)
4. [Practice Question: DAD H](#practice-question-dad-h)
5. [Session Summary](#session-summary)

---

## DAD RP — Double Add Register Pair with HL

### Meaning
- **DAD** = **D**ouble **A**dd register pair with **D** (HL pair, treated as the fixed destination/second operand).
- **DAD RP** = Add a 16-bit value from a register pair (`RP`) to the **HL register pair**.

### The rule
- **One 16-bit operand** can reside in **any** register pair: BC, DE, or HL itself.
- **The other 16-bit operand must always reside in the HL register pair.**
- **The result is always stored back into the HL register pair.**

> This mirrors the earlier pattern seen with the accumulator (in `ADD R`, `SUB R`, etc.) — except here, **HL plays the role of the accumulator** for 16-bit operations.

### Instruction variations
Only **3 specific instructions** exist for this type:
- `DAD B` → adds the **BC** pair to HL
- `DAD D` → adds the **DE** pair to HL
- `DAD H` → adds the **HL** pair to itself (doubles HL's content)

> Note: Even though the mnemonic doesn't include the letter "X" like `INX`/`DCX`, the same "extension" convention applies — only the first register's name (`B`, `D`, `H`) is mentioned, and the second register (`C`, `E`, `L`) is implied as its extension.

### Instruction length
- `DAD RP` is a **1-byte long instruction**.

### Flags affected
- **Only the Carry (CY) flag is affected** — no other flag changes, regardless of the result.
- Reasoning: this is a genuine **addition** operation (so a carry can meaningfully occur, unlike `INX`/`DCX`), but since the result affects a 16-bit register pair (not the 8-bit accumulator), the other flags (which are typically evaluated based on the accumulator's 8-bit content) are not applicable here.

---

## Why It's Called "Double Add"

- All previous addition instructions (`ADD R`, `ADI d8`, `ADC R`, `ACI d8`) operated on **8-bit values**, since the accumulator is only 8 bits wide. These are considered **"single" additions**.
- `DAD RP` operates on **16-bit values** (register pairs) — which can be thought of as performing **two 8-bit additions "at once"**: the least-significant-byte addition and the most-significant-byte addition, combined into a single 16-bit operation.
- Hence the name: **double add** — because it's effectively double the width of a single 8-bit addition.

---

## Worked Example: DAD B

This example revisits the multi-byte addition seen in an earlier session (when studying GPRs and the accumulator), but this time performed as a **single `DAD B` instruction** instead of doing it digit-by-digit manually.

**Setup:**
- HL register pair loaded with `1234H` (first 16-bit operand)
- BC register pair loaded with `5678H` (second 16-bit operand)

**Execution of `DAD B`:** Adds BC to HL, treating both as full 16-bit values.

**Digit-by-digit addition (for illustration):**
- `4 + 8 = 12` (decimal) = **`C`** (hex)
- `3 + 7 = 10` (decimal) = **`A`** (hex)
- `2 + 6 = 8`
- `1 + 5 = 6`

**Result:** `1234H + 5678H = 68ACH`

After execution, this result (`68ACH`) is stored back into the **HL register pair**.

> **Key parallel:** Just as the accumulator holds one operand and receives the result in 8-bit operations, the **HL register pair** plays that same role for 16-bit operations in `DAD RP`.

---

## Practice Question: DAD H

**Question posed in the lecture:** If the HL register pair holds `1234H`, what will be the result of executing `DAD H`?

**Think it through:**
- `DAD H` adds the HL register pair **to itself**.
- This is mathematically equivalent to **doubling** the value in HL.
- `1234H + 1234H = 2468H`

> This follows the same logic as `ADD A` (adding the accumulator to itself) from earlier sessions — except scaled up to 16-bit operands via HL.

---

## Session Summary

| Instruction Type | Meaning | Length | Op-Codes | Flags Affected |
|---|---|---|---|---|
| `DAD RP` | Add a 16-bit register pair to HL | 1 byte | 3 (`DAD B`, `DAD D`, `DAD H`) | **Only CY** |

**Key points to remember:**
- `DAD RP` performs **16-bit ("double") addition**, unlike the 8-bit additions seen in `ADD R`/`ADI d8`/`ADC R`/`ACI d8`.
- **HL register pair** always holds one operand and receives the result — functioning like a "16-bit accumulator."
- The other operand can come from **BC, DE, or HL itself** — giving 3 total instructions: `DAD B`, `DAD D`, `DAD H`.
- **Only the Carry (CY) flag is affected** — no other flags change.
- Follows the same "first-register-implies-pair" naming convention as `INX`/`DCX`/`LDAX`/`STAX`, even without an "X" in the mnemonic.

---

*Next session: Another arithmetic group instruction.*
