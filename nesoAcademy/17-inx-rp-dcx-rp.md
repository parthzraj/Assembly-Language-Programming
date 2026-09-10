# 8085 Microprocessor — Arithmetic Group: INX RP and DCX RP

> Continuing the **Arithmetic Group of Instructions**. This session covers `INX RP` and `DCX RP` — instructions that increment/decrement an entire **register pair** at once, rather than a single 8-bit register.

---

## Table of Contents
1. [INX RP — Increment Extended Register](#inx-rp--increment-extended-register)
2. [Why INX B Is Needed (vs. Just Using INR C)](#why-inx-b-is-needed-vs-just-using-inr-c)
3. [DCX RP — Decrement Extended Register](#dcx-rp--decrement-extended-register)
4. [Session Summary](#session-summary)

---

## INX RP — Increment Extended Register

### Meaning
- **INX** = **In**crement e**x**tended register.
- **RP** = short for **R**egister **P**air.
- **INX RP** = Increment the entire contents of a **register pair** (treated as a single 16-bit value) by 1.

### Which register pairs?
Just like other "extended register" instructions (`LXI`, `LDAX`, `STAX`), only the **first register's name** of each pair is mentioned — the second register is implied as its "extension":
- `INX B` → operates on the **BC** pair (C is the extension of B)
- `INX D` → operates on the **DE** pair (E is the extension of D)
- `INX H` → operates on the **HL** pair (L is the extension of H)

### Instruction length
- `INX RP` is a **1-byte long instruction**.

### ⚠️ Important: No flags are affected
Unlike most arithmetic instructions, **executing `INX RP` affects NONE of the flags** in the Flags register.

### Example: `INX B`
Suppose the BC register pair currently holds `F212H` (B = `F2`, C = `12`).

**Execution of `INX B`:**
- The entire 16-bit value is incremented as one unit: `F212H → F213H`.

---

## Why INX B Is Needed (vs. Just Using INR C)

A natural question: since `INX B` here just changed `F212H → F213H`, couldn't the same result be achieved with `INR C` (incrementing only the C register)?

**In this specific example — yes, they'd give the same result.** But this is **not always true**, and here's why:

### The problem case: incrementing across a digit boundary

Suppose the BC register pair holds the **address** `F2FFH` — meaning the BC pair is currently pointing to memory location `F2FFH`. The goal: move to the **next** memory address.

**If we (incorrectly) use `INR C`:**
- `INR C` only affects the **C register** (8 bits), **not** the full 16-bit pair.
- C currently holds `FF`. Incrementing `FF` by 1 would logically produce `100H` — but this is a **3-digit (12-bit)** value, and the 8-bit C register can only hold **8 bits (2 digits)**.
- The overflow (most significant hex digit) is **discarded**, so C simply **wraps around to `00`**.
- **Result:** BC becomes `F200H` — but this is **not** the next address! We *wanted* `F300H`, but instead moved **backward** to `F200H`. ❌

> This happens because `INR C` only operates on the 8-bit C register in isolation — it has no way to "carry" the overflow into the B register.

**If we (correctly) use `INX B`:**
- `INX B` treats **BC as a single 16-bit unit**.
- `F2FFH + 1 = F300H` — correctly rolls over into the B register when C overflows.
- **Result:** BC correctly becomes `F300H` — the true "next address." ✅

### Key takeaway
> `INX RP` is essential whenever you need to increment a **register pair used as a memory address pointer**, because it correctly handles the carry from the lower register (e.g., C) into the upper register (e.g., B) — something a single-register instruction like `INR C` cannot do.

---

## DCX RP — Decrement Extended Register

### Meaning
- **DCX** = **D**e**c**rement e**x**tended register.
- **DCX RP** = Decrement the entire contents of a register pair (as a single 16-bit value) by 1.
- Performs the **opposite** operation of `INX RP`.

### Which register pairs?
Same as `INX RP`: `DCX B` (BC pair), `DCX D` (DE pair), `DCX H` (HL pair) — only the first register's name is mentioned, with the second treated as its extension.

### Instruction length
- `DCX RP` is also a **1-byte long instruction**.

### ⚠️ Important: No flags are affected
Just like `INX RP`, **executing `DCX RP` affects NONE of the flags.**

### Example: `DCX H`
Suppose the HL register pair currently holds `F800H`.

**Execution of `DCX H`:**
- The entire 16-bit value is decremented as one unit.
- Since `F800H` is the number that comes **right after** `F7FFH`, decrementing `F800H` by 1 gives: `F800H → F7FFH`.

**Why this makes sense:** Consider the reverse — if you add 1 to `F7FFH`: the least significant two digits (`FF`) are already at their maximum, so incrementing resets them to `00` and carries into the next digit (`7 → 8`), giving `F800H`. Decrementing `F800H` simply reverses this process, correctly rolling back into `F7FFH` (not just resetting the last two digits to some incorrect value).

---

## Session Summary

| Instruction Type | Meaning | Length | Register Pairs | Flags Affected |
|---|---|---|---|---|
| `INX RP` | Increment the register pair (as a 16-bit unit) by 1 | 1 byte | BC, DE, HL (`INX B`, `INX D`, `INX H`) | **None** |
| `DCX RP` | Decrement the register pair (as a 16-bit unit) by 1 | 1 byte | BC, DE, HL (`DCX B`, `DCX D`, `DCX H`) | **None** |

**Key points to remember:**
- Both `INX RP` and `DCX RP` operate on the **entire 16-bit register pair** as a single unit — unlike `INR R`/`DCR R`, which operate on individual 8-bit registers.
- **Neither instruction affects any flags** — a key distinguishing feature from `INR R`/`DCR R` (which affect all flags except CY).
- `INX RP` correctly handles **carry propagation** from the lower register to the upper register — critical when the register pair is being used as a **memory address pointer**, where a single-register increment (like `INR C`) would incorrectly wrap around and produce the wrong address.
- Both instructions only mention the **first register's name** of each pair (`B`, `D`, `H`) — the second register (`C`, `E`, `L`) is always treated as its implied extension.

---

*Next session: A couple more arithmetic instructions.*
