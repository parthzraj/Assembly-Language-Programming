# ARM Assembly Language — Complete Course Notes

> **Source:** Subtitles from an "Introduction to ARM Assembly" course (instructor: Scott Cosentino).
> **Important compatibility note (read first):** These notes are 100% about **ARM assembly** (a 32-bit RISC architecture). You said you want to apply this to **GNUSim8085**, which simulates the **Intel 8085** — an 8-bit CISC processor with a completely different register set, instruction mnemonics, and addressing modes. The two are **not syntax-compatible** — you cannot type `MOV R0, #30` into GNUSim8085 and expect it to work. See the **[Mapping to 8085 / GNUSim8085](#mapping-to-8085--gnusim8085)** section at the end for what *concepts* carry over and what doesn't.

---

## Table of Contents
1. [Course Overview](#1-course-overview)
2. [ARM Architecture Basics — Registers](#2-arm-architecture-basics--registers)
3. [Special-Purpose Registers](#3-special-purpose-registers)
4. [Stack Memory](#4-stack-memory)
5. [The CPSR Register](#5-the-cpsr-register)
6. [Program Structure Basics](#6-program-structure-basics)
7. [Your First Program](#7-your-first-program)
8. [Addressing Modes](#8-addressing-modes)
9. [Arithmetic Instructions](#9-arithmetic-instructions)
10. [Logical Operators](#10-logical-operators)
11. [Shifts and Rotations (Theory)](#11-shifts-and-rotations-theory)
12. [Shifts and Rotations (In Practice)](#12-shifts-and-rotations-in-practice)
13. [Conditionals — Compare and Branch](#13-conditionals--compare-and-branch)
14. [Looping](#14-looping)
15. [Conditional Execution (Conditional Instructions)](#15-conditional-execution-conditional-instructions)
16. [Functions / Subroutines](#16-functions--subroutines)
17. [Preserving Registers — Push/Pop](#17-preserving-registers--pushpop)
18. [Hardware Interaction (Switches & LEDs)](#18-hardware-interaction-switches--leds)
19. [Setting Up ARM Linux (Raspberry Pi Emulation)](#19-setting-up-arm-linux-raspberry-pi-emulation)
20. [Writing "Hello World" with System Calls](#20-writing-hello-world-with-system-calls)
21. [Debugging with GDB](#21-debugging-with-gdb)
22. [Mapping to 8085 / GNUSim8085](#mapping-to-8085--gnusim8085)

---

## 1. Course Overview

- **Assembly language** = a low-level programming language closest to machine language; usually specific to one CPU architecture (hence "multiple assembly languages" — ARM, x86, MIPS, 8085, etc.).
- **ARM** is an increasingly popular assembly language/architecture. Over an estimated **200 billion devices** contain an ARM chip (phones — Android and some iPhones, embedded devices, newer MacBooks, etc.), which is why it's valuable to learn.
- **Why learn low-level/assembly at all:** it lets you work closer to the hardware, write more efficient hardware/software interactions, and understand what higher-level languages are doing "under the hood" when compiled.
- **Course goals:**
  - Understand the ARM processor and general assembly programming principles.
  - Be able to write basic ARM assembly programs.
  - Cover: basic instructions, arithmetic, logical operators, shift/rotate operators, branching, looping, hardware interaction, and troubleshooting ARM assembly on Linux.
- **Tooling used:**
  - Primarily an online emulator called **CPUlator** (works in browser, no installation needed) — architecture used: **ARMv7**, specifically the **ARMv7 DE1-SoC** configuration (this variant has simulated hardware devices like switches/LEDs).
  - The final few videos move to a **real/emulated Linux environment** (Raspberry Pi OS via QEMU) since some things (system calls, GDB) need an OS.
- **Teaching philosophy:** concepts taught here (registers, stack, flags, branching, functions) are general assembly-language concepts — only the *syntax* is ARM-specific. The ideas transfer to other assembly languages (x86, MIPS, 8085, etc.), even though exact instructions differ.

---

## 2. ARM Architecture Basics — Registers

- **Registers** = small storage locations physically very close to the CPU → very fast to read/write, but limited in number/size.
- In the emulator, each register is shown as **8 hexadecimal digits** (e.g., `00000000`).
  - Each hex digit = **4 bits**.
  - 8 hex digits × 4 bits = **32 bits total** per register.
  - This confirms the processor being emulated is a **32-bit processor**.
- **Word / Half-word / Byte / Bit terminology** (very important, general concept for *any* architecture):
  | Term | Size on a 32-bit processor | Size on a 64-bit processor |
  |---|---|---|
  | Word | 32 bits (max register size) | 64 bits |
  | Half-word | 16 bits (half of a word) | 32 bits |
  | Byte | 8 bits (always) | 8 bits (always) |
  | Bit | 1 bit | 1 bit |
  - A "word" always means *the full register width* for that architecture — this changes between architectures, but byte and bit are always fixed sizes.
- **General-purpose registers:** `R0`–`R6` — freely usable for any storage/computation.
- Some registers (`R7`, `SP`, `LR`, `PC`, and the CPSR) have **special reserved purposes** (detailed next).

---

## 3. Special-Purpose Registers

### R7 — System Call Number Register
- Used to communicate with the **operating system**.
- To ask the OS to do something (e.g., terminate the program, write output), you:
  1. Store a specific numeric code in `R7` (this number maps to a specific OS action in a lookup table).
  2. Trigger a **software interrupt** (`SWI`) to hand control to the OS.
  3. The OS reads the value in `R7`, looks it up, and performs the corresponding action.
- Example: storing `1` in `R7` then interrupting → OS terminates ("exits") the program.

### SP — Stack Pointer
- Points to the **address of the next available memory slot on the stack**.
- Introduces the concept of **stack memory** (see Section 4).

### LR — Link Register
- Stores the **return address** — i.e., where execution should jump back to after a function/subroutine finishes.
- Directly analogous to how a function `return` works in high-level languages.
- Gets automatically set when you use a **branch-and-link** instruction (`BL`) — see Section 16.

### PC — Program Counter
- Keeps track of the **address of the next instruction to execute**.
- All instructions live in memory; PC steps through them one at a time (or jumps, on a branch).

---

## 4. Stack Memory

- Stack memory is (physically) stored in **RAM** — slower to access than registers, but much larger capacity.
- Used for storing **more complex data** than fits comfortably in registers — e.g., a list/array of numbers.
- **Addressing:** each memory location has an address; consecutive addresses increase by **4** (because each "slot" holds one 32-bit word = 4 bytes).
  - Address `0`, then `4`, then `8`, then `12`, then `16`, etc.
- Remember: values shown in the emulator memory view are in **hexadecimal** (e.g., `0x10` = 16 in decimal) — easy to misread if you forget this.
- `SP` always reflects the address of the next free stack slot.

---

## 5. The CPSR Register

**CPSR = Current Program Status Register** (there's also an **SPSR** — Saved Program Status Register — mentioned but not detailed).

- Stores **status flags** about the result of the most recent (flag-setting) operation. Key flags:
  - **N** — Negative: set if the last result was negative.
  - **Z** — Zero: set if the last result was zero.
  - **C** — Carry: set if there was a carry out of the operation (e.g., addition overflowed available bits).
  - **V** — Overflow: set if there was a signed overflow.
- **Why this matters:** In binary, negative numbers are stored using **two's complement**. A given bit pattern could represent either a large positive number or a negative number — you can't tell just by looking at the raw bits. The CPSR's **N flag** disambiguates this: if set, interpret the result as negative.
- **Important quirk — flags are NOT set automatically:** Regular instructions (e.g., `SUB`, `ADD`) do **not** update CPSR by default, because updating CPSR is itself extra work/overhead that costs efficiency.
  - To set flags, use the **"S" suffix** version of an instruction, e.g. `SUBS` instead of `SUB`, `ADDS` instead of `ADD`.
  - **Rule of thumb:** Use the `S`-suffixed version when (a) you expect negative numbers could result, or (b) you don't control/know the input values in advance (e.g., loaded from memory/user input). If you're just combining two known constants and you're sure of the sign of the result, the non-`S` version is fine and slightly more efficient.
- **Flag display format:** the CPSR flags are stored as bits in a nibble; e.g., if only N is set, you'd see it displayed as **`8`** (binary `1000`) in raw form outside the emulator — the emulator visually bolds/highlights the letter (N/Z/C/V) for you, but a plain terminal/hardware view will just show the raw hex digit.

---

## 6. Program Structure Basics

Every ARM assembly program (in this emulator) starts with two boilerplate lines:

```asm
.global _start
_start:
```

- **`_start:`** — a **label**. A label is conceptually similar to a function name in high-level languages: it marks a point in the code you can jump ("branch") to.
  - Labels are **local by default** — not visible outside your own program unless exported.
- **`.global _start`** — exports/declares the `_start` label so that external things (the OS/loader) can find your program's entry point. Without this declaration, nothing outside your file would know where execution should begin.
- All of your program's code goes underneath the `_start:` label.
- The emulator inserts these two lines automatically; when writing raw assembly outside the emulator (e.g., on Linux), you must add them yourself.

---

## 7. Your First Program

Goal: move a constant into a register, then cleanly terminate the program.

```asm
.global _start
_start:
    MOV R0, #30      @ move decimal 30 into R0
    MOV R7, #1        @ 1 = "exit" system call code
    SWI 0              @ software interrupt -> hand control to OS
```

### Breaking down `MOV`
- `MOV` = the **move** opcode/instruction/mnemonic (all three terms — opcode, operation, mnemonic — are used interchangeably by the instructor).
- Case-insensitive; instructor prefers writing operations and register names in uppercase by convention (not required).
- Syntax: `MOV <destination>, <source>`
  - **Destination is always the first argument.**
  - **Source is the second argument.**
- **Immediate values** (literal constants) are written with a `#` prefix: `#30` = decimal 30.
- **Hex immediate values** use `0x` prefix: `MOV R0, #0x0A` moves hex `0A` (decimal 10) into R0.
  - The instructor defaults to **decimal** throughout the course unless there's a specific reason to use hex.

### Ending the program with a system call
- `MOV R7, #1` — loads the system call code for **"terminate/exit program"** into R7.
- `SWI 0` — **Software Interrupt**. Hands control to the operating system.
  - The OS reads R7, sees `1`, looks it up in its internal table, and terminates the program.
- **Emulator quirk:** CPUlator doesn't fully/properly execute `SWI` the way a real OS would — the instructor teaches it the "real hardware/real ARM" way, so it will work correctly on an actual ARM Linux device (e.g., Raspberry Pi), even though the online emulator doesn't visibly complete the interrupt.

### Stepping through execution (in the emulator)
- **Compile and load**, then use **Step Into** to execute one instruction at a time.
- After `MOV R0, #30`:
  - R0 shows raw hex `1E`.
  - Switching the register **display format** to "decimal unsigned" (via Settings) shows `30` — confirms `1E` (hex) = `30` (decimal).
- **Endianness:**
  - The way bits/bytes are stored and displayed relates to **Little Endian** vs **Big Endian**.
  - **Little Endian** = the most significant byte/bit conceptually appears in a particular arrangement (as demonstrated visually in the emulator) — ARM's default in this course's context.
  - ARM is actually a **bi-endian** processor — it can operate in **either** Little Endian or Big Endian mode depending on configuration.
- **Program Counter (PC) in action:** after executing the first instruction, PC updates to point at the *address* of the next instruction (e.g., becomes `4`, pointing at the second `MOV` instruction) — demonstrating how PC tracks "where we are" in the instruction stream.

---

## 8. Addressing Modes

Addressing modes = the different ways you can move data to/from registers and memory.

### 1. Immediate Addressing
- Moving a **constant/literal** value directly into a register.
- Example: `MOV R0, #5`

### 2. Register Direct Addressing
- Moving a value **from one register directly into another**.
- Example: `MOV R1, R0` (copies R0's value into R1).

### Setting up data in memory (the `.data` section)
To work with stack/memory data, declare a **data section**:

```asm
.data
list:
    .word 4, 5, -9, 10, 2, -3
```

- `.data` — marks the start of a data declaration section (placed after your code, e.g. below `.global _start`... block).
- `list:` — a **label** acting like a variable name for this block of data.
- `.word` — declares that each following value should be treated as a **word-sized** (32-bit) entry. (Other type declarators exist too, e.g. for ASCII text, half-words, bytes — introduced later.)
- Values are stored **sequentially** in memory, one after another, starting at the address of the label.

### 3. Direct Addressing (loading an address into a register)
```asm
LDR R0, =list
```
- `LDR` = **Load Register** — loads data **from stack/memory into a register**.
- `=list` — the `=` here means "load the **address** of the label `list`" into R0 (not the value stored there — the *location*).
- This is called **direct addressing**.

### 4. Register Indirect Addressing
```asm
LDR R1, [R0]
```
- Square brackets `[...]` tell the assembler: "go to the **address held inside** this register, and get the value stored **at that address**."
- So this loads **the value found at the memory address currently stored in R0** into R1.
- Analogy to high-level languages: like `list[i]` where `i = R0`, but here `R0` directly *is* the address (i.e., more like `list_value_at(R0)`).

### 5. Register Indirect with Offset
```asm
LDR R2, [R0, #4]
```
- Takes the address in R0, **adds an offset** (here, `4`), and retrieves the value at that computed address — **without modifying R0 itself**.
- Since each stack slot is 4 bytes, adding `4` moves you to the *next* element in a word-array; adding `8` moves two elements over, `12` moves three, etc.
- Conceptually like `list[i + 1]` in a high-level language (offset `4` = "+1 index" for word-sized elements).
- **R0's value is unchanged** after this instruction.

### 6. Pre-Increment Addressing
```asm
LDR R1, [R0, #4]!
```
- The trailing `!` marks this as **pre-increment**: R0 is **incremented first** (by the offset), *then* the value at the new address is fetched.
- **Key difference from plain offset addressing:** here, **R0's value itself is permanently updated** (changed) to the new address after this instruction runs. Plain offset addressing (#5 above) does *not* change R0.

### 7. Post-Increment Addressing
```asm
LDR R2, [R0], #4
```
- Fetches the value at the **current** address in R0 **first**, *then* increments R0 by the offset **afterward**.
- Equivalent conceptually to: "access `list[R0]`, **then** do `R0 += 1` (in element terms)."
- Contrast with pre-increment, which increments *before* accessing.

### Summary Table of Addressing Modes
| Mode | Syntax example | Effect | Changes source register? |
|---|---|---|---|
| Immediate | `MOV R0, #5` | Loads literal constant | N/A |
| Register direct | `MOV R1, R0` | Copies register to register | No |
| Direct (address-of) | `LDR R0, =list` | Loads address of a label | N/A |
| Register indirect | `LDR R1, [R0]` | Loads value at address in R0 | No |
| Indirect + offset | `LDR R2, [R0, #4]` | Loads value at (R0 + offset) | No |
| Pre-increment | `LDR R1, [R0, #4]!` | R0 += offset, then load at new R0 | **Yes** |
| Post-increment | `LDR R2, [R0], #4` | Load at current R0, then R0 += offset | **Yes** |

---

## 9. Arithmetic Instructions

ARM supports **add**, **subtract**, and **multiply** directly. **Division is NOT a built-in instruction** (it's significantly more complex to implement at this level, so it's excluded from basic instruction coverage).

### Basic syntax (3-operand form)
```asm
ADD R2, R0, R1     @ R2 = R0 + R1
SUB R2, R0, R1     @ R2 = R0 - R1
MUL R2, R0, R1     @ R2 = R0 * R1
```
- Destination is always first; the two source operands follow.
- For `ADD`/`MUL`, operand order doesn't matter (commutative).
- For `SUB`, **order matters**: `R0` is always the first (minuend) operand, `R1` is the second (subtrahend) operand — `SUB R2, R0, R1` means `R2 = R0 - R1`, **not** the reverse.

### The ambiguity problem with subtraction results
- If you subtract and get a "big-looking" hex number, you can't immediately tell if that's a **large positive number** or a **negative number** stored in two's complement — the raw bit pattern can look identical either way depending on context.
- **Solution: use the CPSR / flag-setting instructions.**

### Flag-setting variants (the "S" suffix)
```asm
SUBS R2, R0, R1   @ same as SUB, but also updates CPSR flags
ADDS R2, R0, R1
```
- Appending `S` to almost any arithmetic (or logical) instruction makes it **also update the CPSR flags** (N, Z, C, V).
- **Why isn't this always on by default?** Setting the CPSR register requires *extra work/an additional operation*, adding overhead. Since efficiency matters at this low level, ARM lets you opt in only when needed.
- **When to use the `S` version:**
  - When you expect the result *could* be negative, **or**
  - When you don't control/know the operand values in advance (e.g., they come from memory, user input, or elsewhere unpredictable).
- **When it's safe to skip `S`:** when you're combining known constants and you're already certain about the sign of the result.
- After running `SUBS` with a negative result, the CPSR's **N flag** becomes set (shown bolded/highlighted in the emulator). In the raw CPSR value, this corresponds to it reading as **`8`** (binary `1000`, i.e., only the N bit is on) rather than `0`.

### Carry and the ADC (Add with Carry) instruction
- When adding two numbers whose sum is **too large to fit in a single 32-bit register**, the result **overflows** — this sets the **Carry (C) flag** (when using an `S`-suffixed instruction).
- To incorporate a previous carry into a *new* addition (useful for multi-word/extended-precision arithmetic), use:
```asm
ADC R2, R0, R1    @ R2 = R0 + R1 + (Carry flag ? 1 : 0)
```
- `ADC` = **Add with Carry**. It adds R0 + R1 **plus** 1 if the Carry flag was previously set, or plus 0 otherwise.
- Similar carry-aware variants exist for other operations (e.g., subtract-with-carry), but `ADC` is the most common one covered.

---

## 10. Logical Operators

Bitwise logical operations performed directly on register contents.

### AND
```asm
AND R2, R0, R1    @ R2 = R0 AND R1 (bitwise)
ANDS R2, R0, R1   @ same, but also sets CPSR flags
```
- Standard bitwise AND: result bit is `1` only where **both** input bits are `1`.
- Example demonstrated: `0xFF AND 0x16` = `0x16` (since ANDing with all-1s bits preserves the other operand exactly).

### OR
```asm
ORR R2, R0, R1    @ R2 = R0 OR R1 (bitwise)
```
- Note the mnemonic is **`ORR`** (two R's) — not `OR`.
- Result bit is `1` if **either** input bit is `1`.
- Example: `0xFF OR 0x16` = `0xFF` (ORing with all-1s bits always yields all 1s).

### Exclusive OR (XOR)
```asm
EOR R2, R0, R1    @ R2 = R0 XOR R1 (bitwise)
```
- Mnemonic is **`EOR`** (Exclusive OR), not `XOR`.
- Result bit is `1` only if the input bits **differ** (one is 1, the other 0); if both bits match, result is `0`.
- Example: `0xFF EOR 0x16` = `0xE9`.

### Negation — `MVN` (Move NOT)
```asm
MVN R0, R0        @ R0 = bitwise NOT of R0
```
- ARM has **no simple standalone "NOT" instruction**. Instead there's **`MVN`** = **Move Negative/Move NOT**: it takes the source, **negates (bitwise-inverts) every bit of it**, and moves the negated result into the destination — all in one instruction.
- It inverts the **entire register**, not just selected bits.

### Practical use of AND — clearing/masking specific bits
- You can use `AND` to **selectively clear specific bits** of a register while preserving others — a common technique called **masking**.
- Example idea: after negating a value with `MVN` (which flips *all* bits), you can `AND` the result with a mask like `0x000000FF` (zeros in the upper bits, ones in the lower byte) to **clear** the upper bits back to zero while **keeping** the lower byte's bits intact.
  - Because `X AND 0 = 0` (clears those bits) and `X AND 1 = X` (preserves those bits).
- This is a foundational technique for **bit manipulation / masking**, useful whenever you need to isolate or zero out specific bit ranges.

---

## 11. Shifts and Rotations (Theory)

Covered first purely at the **binary/bit level** (conceptually), before the ARM-specific instruction syntax.

### Logical Shift Left (LSL)
- Every bit moves **one position to the left**; a `0` is shifted in on the right; the leftmost bit that "falls off" is discarded (in the simple binary illustration).
- **Key property:** shifting left by 1 is mathematically **equivalent to multiplying the value by 2**.
  - Example: `1010` (decimal 10) shifted left once → `10100` (decimal 20) = 10 × 2.
  - This holds true for **every** logical left shift — it's not a coincidence, it's a general property of binary place-value.
- **Practical use:** LSL is a **fast way to multiply by 2** (or by powers of 2, if shifted multiple times).

### Logical Shift Right (LSR)
- Every bit moves **one position to the right**; the rightmost bit that "falls off" is discarded; a `0` is shifted in on the left.
- **Key property:** shifting right by 1 is equivalent to **dividing the value by 2** (integer division).
  - Example: `1010` (10) shifted right once → `0101` (5) = 10 ÷ 2.
- **Practical use:** since ARM has **no native division instruction**, repeated logical right shifts by powers of 2 provide a way to implement division by 2 (and, combined with other techniques, general division).

### Rotation (ROR — Rotate Right)
- Similar to a shift, but instead of **discarding** the bit that falls off the end, it **wraps around** to the opposite end.
- Example: rotating `10000001` right by 1 → the rightmost `1` doesn't disappear; it wraps around to become the new leftmost bit → `11000000` (whereas a plain logical shift right would have just produced `01000000` and lost that bit).
- **Uses of rotation:** less common in everyday arithmetic; mostly seen in **hashing, cryptography, and graphics** algorithms. Considered somewhat "historical"/niche compared to shifts.
- **Important ARM quirk:** ARM only provides **rotate right (ROR)** — there is **no rotate-left instruction**.
  - To achieve a "rotate left by n," you instead perform a **rotate right by (32 − n)** — mathematically equivalent, since rotating all the way around by 32 total returns you to the start.

---

## 12. Shifts and Rotations (In Practice)

### Basic shift syntax
```asm
LSL R0, R0, #1    @ shift R0 left by 1 (R0 = R0 * 2)
LSR R0, R0, #1    @ shift R0 right by 1 (R0 = R0 / 2)
```
- Syntax: `LSL <dest>, <source>, <shift-amount>`
- Shifting left by 1 doubles the value; shifting right by 1 halves it (confirmed step-by-step in the emulator: 10 → shift left → 20 → shift right → back to 10).
- You can shift by **more than once**, e.g. shifting left by 2 = multiply by 4 (2 shifts of "×2" = ×2×2); shifting left by 3 = multiply by 8; and so on.
- **The shift amount can be a register instead of a literal**, e.g. `LSL R0, R0, R1` — shifts by however many bits are currently stored in R1. This allows a **variable/dynamic** shift amount.

### Combined move + shift (single instruction)
```asm
MOV R1, R0, LSL #1   @ R1 = R0, then shifted left by 1 — R0 itself is unchanged
```
- Instead of needing two separate instructions (one `MOV` to copy the value, one `LSL` to shift it), ARM lets you **append the shift directly onto a `MOV` instruction**.
- This copies R0 into R1 **and** applies the shift **in the same instruction**, while **leaving the original R0 unmodified**.
- Demonstrated: R0 = 10 → `MOV R1, R0, LSL #1` → R1 becomes 20 immediately, R0 stays 10.

### Rotation syntax
```asm
ROR R0, R0, #1    @ rotate R0 right by 1 bit
```
- Example demonstrated: rotating the value `15` (binary `...1111`, i.e., all 1s in the low nibble) right by 1 causes the rightmost `1` bit to wrap around to the **most significant bit position**, producing a large resulting hex value where the top hex digit becomes `8` (binary `1000`) and the rest becomes `7` (binary `0111`) — visually confirming the "wrap-around" behavior.
- Just like `MOV ..., LSL #n`, you can also combine rotation directly with `MOV`: e.g. `MOV R1, R0, ROR #1`.

---

## 13. Conditionals — Compare and Branch

Two building blocks: **comparators** (to evaluate conditions) and **branches** (to jump based on the result).

### `CMP` — Compare
```asm
CMP R0, R1
```
- Internally computes `R0 - R1` **and always sets the CPSR flags** (equivalent to a flag-setting subtraction), **without storing the numeric result anywhere** — its only purpose is to set flags for a subsequent branch/conditional instruction.
- Result interpretation:
  - If `R0 > R1` → subtraction result is **positive** (no Negative flag; likely sets Carry).
  - If `R0 < R1` → subtraction result is **negative** → **N flag set**.
  - If `R0 == R1` → subtraction result is **zero** → **Z flag set**.

### Branch instructions
```asm
BGT greater      @ Branch if Greater Than -> jump to label "greater"
BAL default       @ Branch ALways -> unconditional jump
```
- General form: `B<condition> <label>`
- `B` = branch, followed by a condition code, followed by the target **label**.
- Common condition suffixes:
  | Suffix | Meaning |
  |---|---|
  | `GT` | Greater Than |
  | `GE` | Greater Than or Equal |
  | `LT` | Less Than |
  | `LE` | Less Than or Equal |
  | `EQ` | Equal |
  | `NE` | Not Equal |
  - (Many more exist in the full ARM documentation; these are the most common/basic ones.)
- **`BAL` — Branch ALways**: unconditionally jumps to the given label regardless of any comparison — useful for skipping over blocks of code you don't want to "fall into."

### ⚠️ Critical concept: sequential fall-through
- **Instructions always execute sequentially unless a branch redirects flow.**
- If you define a conditional branch (e.g., `BGT greater`) and the condition is **false**, execution does **not** skip the *entire* rest of the file — it simply continues to the **very next instruction in memory order**, which might be the start of a label block you didn't intend to "fall into" (e.g., accidentally running into a `greater:` block anyway from below, if not properly bypassed).
- **This is why `BAL` (branch always) is often needed** — to explicitly skip over a block of code (like a `greater:` label's body) that you don't want to execute in the non-matching case, redirecting instead to something like a `default:` label positioned after it.
- **Practical takeaway:** always trace through your labels and make sure that after a conditional branch's target block finishes, execution doesn't unintentionally fall into a block you didn't want — insert additional branches as needed to control flow precisely.

---

## 14. Looping

Loops are built entirely from the **compare + branch** primitives already covered — there is no dedicated "for"/"while" instruction; you construct the loop manually using labels and conditional branches.

### Example walkthrough — summing a list until a sentinel value
```asm
.data
list:
    .word 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
END_LIST_VAL:
    .eq END_LIST_VAL, 0xAAAAAAAA   @ example constant declaration syntax (see note below)

.global _start
_start:
    LDR R0, =list        @ R0 = address of list
    LDR R3, =END_LIST_VAL @ R3 = the "end of list" sentinel value (loaded via constant, see below)
    LDR R1, [R0]           @ load first element into R1
    ADD R2, R2, R1         @ R2 (running total) += first element  (initialization step)

loop:
    LDR R1, [R0, #4]!      @ pre-increment: move to next element, load it into R1
    CMP R1, R3               @ compare current element to the sentinel
    BEQ exit                  @ if equal -> we've hit the end -> exit loop
    ADD R2, R2, R1           @ otherwise, add this element to the running total
    BAL loop                  @ unconditionally jump back to top of loop

exit:
    @ R2 now holds the sum of the list
```

### Key concepts introduced here

**Finding the end of a list (sentinel values):**
- Memory in the emulator, when uninitialized, may show up filled with a repeating pattern (e.g., all `A`s) — this is used as a convenient (if artificial) "end of list" marker in the demo.
- **Real-world practice:** you would deliberately place a **known, predictable sentinel value** at the end of your data, rather than relying on whatever the uninitialized memory happens to contain (which is unpredictable in real systems).
  - **C-language parallel:** C strings use a **null terminator** (`\0`) at the end to mark where the string ends — same underlying concept as a sentinel value marking "end of data."
- **Alternative approach (not deeply covered):** track/store the **length of the list** separately (e.g., in a constant/register) instead of scanning for a sentinel — this avoids relying on uninitialized memory content altogether and is generally considered easier/more robust.

**Declaring constants — `.eq`:**
```asm
.eq NAME, value
```
- ARM's **immediate values (literals) have a limited size** — you cannot directly `LDR` an arbitrarily large literal constant into a register in one step (there's a maximum literal size, roughly "two hex digits" as described, though the exact real limit depends on the ARM encoding).
- For larger constant values, declare them with `.eq` at the top of the program (giving a name and a value), then load that named constant using `LDR Rx, =NAME` — this works around the immediate-value size restriction.

**Loop structure summary:**
1. **Initialize** — set up starting values before entering the loop (e.g., load the first element, seed the running total).
2. **Loop label** — marks the top of the repeating block.
3. **Load/advance** — get the next piece of data (often via pre/post-increment addressing).
4. **Compare** — check the loop's exit condition.
5. **Conditional branch to exit** — if the condition for stopping is met, branch out of the loop.
6. **Do loop body work** — the actual repeated computation (e.g., accumulate a sum).
7. **Unconditional branch back to the loop label** — repeat.
8. **Exit label** — marks where execution continues after the loop ends.

This general shape (init → label → check → conditional exit → body → jump back) is exactly how both **for-loops** and **while-loops** are implemented at the assembly level — the difference between them is really just *where* the check happens and how the loop variable is managed, not a different set of instructions.

---

## 15. Conditional Execution (Conditional Instructions)

A more compact alternative to writing full branch-based if-statements for simple single-instruction conditional actions.

### The problem this solves
Doing a simple "if condition then do one instruction" using pure branching requires several lines:
```asm
CMP R0, R1
BLT do_add
BAL skip
do_add:
    ADD R2, R2, #1
skip:
    @ continue...
```
This is a lot of code for a very small conditional action.

### The compact solution — condition-suffixed instructions
Almost any ARM instruction (not just branches) can have a **condition code suffix** appended, causing it to execute **only if** the condition (based on the most recent flag-setting comparison) holds true — no separate branch needed:

```asm
CMP R0, R1
ADDLT R2, R2, #1    @ only executes if the previous CMP found R0 < R1
```

- Same idea applies to other instructions:
```asm
CMP R0, R1
MOVGE R2, #5        @ only moves 5 into R2 if R0 >= R1
```

- **How it works internally:** the instruction is fetched and decoded regardless, but it only actually **commits/executes its effect** if the condition (derived from the last flags-setting comparison) matches; otherwise it behaves like a no-op for that cycle.
- **Benefit:** drastically reduces code size/complexity for simple conditional single-instruction actions, avoiding the need for extra branch labels and unconditional "skip" branches everywhere.
- Works with common conditions: `EQ`, `NE`, `GT`, `GE`, `LT`, `LE`, etc. — same suffix set as used with branches.
- You can flip which operand is "first" in the `CMP` to effectively flip the comparison direction (e.g., `CMP R1, R0` instead of `CMP R0, R1` reverses the greater/less relationship being tested).

---

## 16. Functions / Subroutines

### Basic call via plain branch (no return)
```asm
_start:
    MOV R0, #1
    MOV R1, #2
    B add_two      @ jump into the function
    @ (execution does NOT naturally come back here with plain B)

add_two:
    ADD R2, R0, R1
```
- A basic `B` (branch) to a label works as a simple "function call," but it does **not** automatically return to where you branched from — execution just continues on from within the function block, sequentially, into whatever comes next in memory.

### The problem: returning to where you left off
- If you want execution to **come back** to the instruction right after the function call (so you can keep doing other things afterward), you need a way to remember "where to return to."
- A crude, hacky solution (implied but discouraged) would be manually chaining more branches around — the instructor explicitly says this creates unwanted complexity.

### The clean solution — `BL` (Branch and Link) + `BX LR`
```asm
_start:
    MOV R0, #1
    MOV R1, #2
    BL add_two      @ Branch AND LINK: jumps to add_two, AND stores the return address in LR
    MOV R3, #4        @ <-- execution resumes HERE after the function returns

add_two:
    ADD R2, R0, R1
    BX LR             @ Branch to eXchange -> jump back to the address stored in LR
```
- **`BL` (Branch with Link):** functions like a normal branch, **but additionally stores the address of the instruction immediately following the `BL`** into the **Link Register (LR)**.
  - Demonstrated: after executing `BL add_two`, `LR` gets set to the exact address of the `MOV R3, #4` instruction that follows the branch.
- **`BX LR` (Branch and eXchange to the address in LR):** at the end of the subroutine, this jumps **back** to whatever address is currently stored in `LR` — i.e., back to right after the original call.
- **This exactly mirrors how "return" works in high-level language functions** — and is, in fact, essentially what your compiler generates under the hood when it compiles a function call in C, Python, etc.
- **Caveat mentioned by instructor:** this is the simplified core idea; real-world, more complex programs need **additional safeguards** (see next section — preserving registers) to be fully robust, especially with nested/recursive function calls.

---

## 17. Preserving Registers — Push/Pop

### The problem
- ARM has a **limited number of registers**. A subroutine may need to use registers (e.g., R0, R1) that are **already holding important values** from the calling code.
- If the subroutine simply overwrites R0/R1, those original values are **lost** once you return to the caller — analogous to needing local variable scoping in high-level languages, but here you must manage it manually.

### The solution — `PUSH` and `POP`
```asm
_start:
    MOV R0, #1
    MOV R1, #3
    PUSH {R0, R1}      @ save current R0 and R1 onto the stack
    BL get_value          @ call the function (which will overwrite R0 and R1 internally)
    POP {R0, R1}         @ restore original R0 and R1 from the stack
    BAL end

get_value:
    MOV R0, #5          @ function freely reuses R0...
    MOV R1, #7           @ ...and R1 for its own purposes
    ADD R2, R0, R1       @ does its calculation
    BX LR                  @ return to caller

end:
    @ R0 is back to 1, R1 is back to 3 here — successfully preserved!
```
- **`PUSH {R0, R1}`** — places the current values of R0 and R1 onto the **stack** (in the order listed).
- **`POP {R0, R1}`** — pops values off the stack **back into** R0 and R1, **in stack order** (first thing popped goes into the first register listed, i.e., R0 gets the value that was on "top" of the stack at pop time, and so on) — this correctly restores the original values, because push/pop operate in matching (LIFO) order.
- **Order matters:** the stack pointer (`SP`) points at the most recently pushed value; popping retrieves values starting from the top of the stack downward, filling registers in the order listed in the `POP` instruction.
- This is directly analogous to how **local variables** work inside a function in a high-level language: allocated for the function's duration, then cleaned up/restored afterward.

### Returning values via the stack (alternative technique, less common)
- You *can* also push a computed result (e.g., R2, holding a "return value") onto the stack and pop it into an unused register (e.g., R9) after returning, as an alternative way of "returning" a value.
- **However**, the instructor notes this is **not the typical approach** — in practice, return values are usually just left directly in a register (like R0 or R2) for the caller to use immediately, since the value is typically needed again very soon after the call anyway. Using the stack for a return value is shown mainly for completeness/awareness, not as the recommended default.

---

## 18. Hardware Interaction (Switches & LEDs)

> **Note:** This section requires the specific **ARM DE1-SoC** emulator configuration in CPUlator, which simulates on-screen hardware devices (switches, LEDs, push buttons, seven-segment displays, JTAG, etc.) at fixed memory-mapped addresses. Plain ARMv7 emulator configs do not include these.

### Declaring hardware memory-mapped addresses
```asm
.eq switch, 0xFF200040
.eq led, 0xFF200000
```
- Hardware devices are accessed via **memory-mapped I/O** — each device corresponds to a fixed memory address.
- Because these addresses are **too large to load directly as an immediate value** (same size restriction discussed in Section 14), they must be declared as named constants via `.eq`, then loaded using `LDR Rx, =name`.

### Reading input — Switches
```asm
LDR R0, =switch     @ R0 = address of the switch device
LDR R1, [R0]           @ R1 = current value AT that address (i.e., the switch states)
```
- Each switch corresponds to one **bit** of the value read: the rightmost switch = bit 0 (value 2⁰ = 1), next switch = bit 1 (value 2¹ = 2), then bit 2 (value 4), etc. — standard binary place-value, same as any binary encoding.
- If a switch is **on**, its bit reads as `1`; if **off**, it reads as `0`.
- Example: only the "value-2" switch on → reads as decimal `2`. All of the first three switches on → binary `111` → decimal `7` (4+2+1).
- This is a basic, general technique for reading **arbitrary binary/digital input** from hardware in embedded systems.

### Writing output — LEDs
```asm
LDR R0, =led            @ R0 = address of the LED device
STR R1, [R0]              @ store the value in R1 OUT to that address -> lights up LEDs accordingly
```
- **`STR`** = **Store Register** — the mirror-image of `LDR`: instead of *reading* data *from* a memory address into a register, it *writes* data *from* a register *out to* a memory address.
- Same rightmost-bit-is-LSB convention as the switches: bit 0 controls the rightmost LED, bit 1 the next, etc.
- Demonstrated end-to-end: read the current switch states into R1, then `STR` that value out to the LED address → the LEDs visually mirror whatever the switches are currently set to.
- **General takeaway (applies to embedded/hardware programming broadly):**
  - **Input devices** → typically read via `LDR` from a fixed hardware address.
  - **Output devices** → typically written via `STR` to a fixed hardware address.
  - Other devices (push buttons, seven-segment displays, JTAG, etc.) follow the same general memory-mapped pattern — the instructor recommends experimenting with each and consulting documentation, as the specific addresses/bit-layouts differ per device.

---

## 19. Setting Up ARM Linux (Raspberry Pi Emulation)

> This section is about environment setup (QEMU + VirtualBox + Raspberry Pi OS), needed because the final videos require a real OS for system calls and GDB debugging. Summarized here for completeness, but this is **infrastructure**, not an assembly-language concept.

- **Why a real OS is needed:** the online emulator can't fully demonstrate real OS-level system calls (`SWI`), so the course switches to an emulated **Raspberry Pi** (which runs a real ARM chip) using **Raspbian ("Jessie")** as the OS image, run inside **QEMU**, inside a **VirtualBox Ubuntu 20.04** host VM.
- **Files needed:**
  1. A Raspbian Jessie disk image (`.zip`, from the Raspberry Pi Foundation's older downloads).
  2. A compatible QEMU **kernel** file for that Raspbian version (sourced from a GitHub repository hosting various QEMU-compatible kernels).
- **Install QEMU:** `sudo apt-get install qemu-system` (ARM support specifically).
- **Launch command (conceptual structure, exact command shown in video):**
  - `qemu-system-arm` with flags for: `-kernel` (path to kernel), `-cpu arm1176` (specific CPU model that works reliably with this Raspbian version), `-m 256` (RAM in MB), `-M versatilepb` (emulated machine/board type), `-serial stdio`, an `-append` string configuring the root filesystem (`root=/dev/sda2 rootfstype=ext4 ...`), `-hda` (path to the disk image), and a `-net`/`-nic` **port-forwarding** setup mapping a host port (e.g., `5022`) to the guest's SSH port (`22`), plus `-no-reboot`.
  - Port forwarding is used because it's easier to **SSH into** the emulated Pi (for a proper full-screen terminal experience) than to interact directly through QEMU's own window.
- **Inside the emulated Pi:** once booted, run `sudo service ssh start` to enable SSH, then SSH in from the host (`ssh pi@127.0.0.1 -p 5022`), default password typically `raspberry` (default username `pi`).
- **Outcome:** a working ARM Linux terminal, ready for writing/assembling/running real ARM assembly programs and using GDB.

---

## 20. Writing "Hello World" with System Calls

Written and assembled on the (emulated) Linux/Raspberry Pi system, using a `.s` file (the standard file extension for assembly source).

### String declaration — `.asciz` / `.string`
```asm
.data
message:
    .asciz "Hello World\n"
len = . - message
```
- **`.ascii`** — declares a raw string of characters **without** any terminator. Fine only for cases where you don't need to know where the string ends automatically.
- **`.asciz`** — declares a string **with an automatic null terminator** (`\0`) appended at the end. This is the "safe default" you should almost always use, because it lets code reliably determine where the string ends (mirrors the **null-terminated string** concept from C).
- **`.string`** — an **alias/synonym** for `.asciz` — functionally identical, just a different name you might see used interchangeably.
- **`\n`** — the escape sequence for a newline character, interpreted by the terminal/OS when printed.
- **Calculating string length:** `len = . - message` — the `.` represents "current address," so subtracting the start address of `message` gives the total byte length of the string (this technique scans from the string's start up through its null terminator to determine length).

### Making the `write` system call
To print to the screen, three pieces of information must be placed into specific registers before triggering the interrupt:

```asm
.global _start
_start:
    MOV R0, #1          @ file descriptor: 1 = standard output (stdout)
    LDR R1, =message  @ R1 = address of the data to write (the "buffer")
    LDR R2, =len        @ R2 = length of the data to write
    MOV R7, #4          @ system call number 4 = "write"
    SWI 0                  @ trigger the interrupt -> OS performs the write

    MOV R7, #1          @ system call number 1 = "exit"
    SWI 0                  @ trigger the interrupt -> OS terminates the program
```

- **R0 — file descriptor:** tells the OS *where* to write.
  - `0` = **standard input** (stdin)
  - `1` = **standard output** (stdout) — e.g., your terminal
  - `2` = **standard error** (stderr)
  - Any other valid integer = a specific **file descriptor** for an actually-open file on the system (Linux assigns integer file descriptors to open files; you could write directly to a file this way too, if it's already open with that descriptor).
- **R1 — buffer address:** the memory address of the data you want to output (loaded via `LDR ..., =message`).
- **R2 — length:** how many bytes to write (loaded via `LDR ..., =len`, using the `.eq`/label-based length calculation from above).
- **R7 — system call number:** tells the OS *which* system call to perform.
  - `4` = **write**
  - `1` = **exit**
  - (Different numbers correspond to different OS-level services — this mirrors the **C standard library's `write()`** function; the instructor explicitly draws this parallel — under the hood, this system call *is* the same underlying OS mechanism that C's `write()` uses.)
- **`SWI 0`** — the actual software interrupt that hands control to the OS, which reads R7 to determine the action, then reads R0/R1/R2 as needed for that action's specific parameters, performs the work, then returns control back to your program to continue executing the next instruction.
- **Critical: you must explicitly exit.** If you don't perform the "exit" system call (R7 = 1, then `SWI 0`) at the end, execution will "fall through" past your intended code into the data section / undefined memory, causing undefined/garbage behavior. This wasn't an issue in the pure emulator exercises (which just stopped), but **is** an issue in a real OS context.

### Assembling and running (Linux command line)
```bash
as HelloWorld.s -o hello_world.o     # Step 1: Assemble into an object file
ld hello_world.o -o hello_world       # Step 2: Link into an executable binary
./hello_world                            # Step 3: Run it
```
- **`as`** — the **assembler**; converts your `.s` source into an intermediate **object file** (`.o`).
- **`ld`** — the **linker**; combines object file(s) (potentially multiple, if a bigger project) into a final executable **binary**.
- The resulting binary is what actually runs on the system; execute it with `./binary_name`.
- Expected output: `Hello World` printed to the terminal.

---

## 21. Debugging with GDB

GDB (**GNU Debugger**) — a standard Linux debugger, useful for both assembly and higher-level languages like C.

### Starting a debug session
```bash
gdb hello_world
```
- Launches GDB loaded with your compiled binary.

### Setting a breakpoint
```
break _start
```
- **Breakpoint** = a point at which execution will automatically pause, letting you inspect the current program state before continuing.
- Breaking at `_start` (your program's entry label) is the typical starting point.

### Running the program
```
run
```
- Starts execution; it will pause at your breakpoint.

### Viewing instructions
```
layout asm
```
- Switches GDB's display to show the **assembly instruction listing**, highlighting the current instruction.

### Viewing registers
Two methods:
1. **On-demand, one at a time:**
   ```
   info register r0
   ```
   (Note: register name given in **lowercase** here, unlike the uppercase convention used in the actual `.s` source.)
2. **Persistent live view of all registers:**
   ```
   layout regs
   ```
   - Shows all registers (including SP, LR, PC, CPSR) continuously alongside the instruction view.

### Switching between views
```
Ctrl+X, then O
```
- Toggles/cycles between the assembly-instruction layout and the registers layout, letting you keep both accessible and switch focus with the arrow keys.

### Stepping through instructions
```
stepi
```
(shown in the video as "step i" — steps **one single machine instruction** at a time, highlighting the next line to be executed and updating the register view live as you go.)

### Examining memory
```
x/10xw $r1
```
- **`x`** = **examine memory**.
- General syntax: `x/<count><format><size> <address>`
  - `<count>` — how many units of memory to display (e.g., `10`).
  - `<format>` — how to interpret/display each unit:
    - `x` = hexadecimal
    - `d` = signed decimal
    - `u` = unsigned decimal
    - `c` = character (useful for viewing strings!)
  - `<address>` — the starting memory address; can use a register directly, e.g. `$r1` (the `$` prefix refers to a register's current value as an address).
- **Practical demonstrated example:** examining memory at the address held in R1 (which points to your `"Hello World\n"` string) using the **character format** (`x/15c $r1`) visually reveals each individual character of the string **plus** the `\n` (newline) **and** the terminating **null character (`\0`)** at the end — concretely confirming how `.asciz` strings are laid out in memory, byte by byte.
- This is described as paralleling the exact same memory-inspection ideas available in the CPUlator emulator (viewing memory in hex/decimal/character format) — just accessed via GDB commands instead of a GUI panel.

### General debugging workflow
1. Set a breakpoint at `_start` (or another point of interest).
2. `run` to reach that breakpoint.
3. Use `layout asm` / `layout regs` (toggle with `Ctrl+X, O`) to watch instructions and registers simultaneously.
4. `stepi` repeatedly to advance one instruction at a time, observing how each instruction changes register/memory state.
5. Use `x/...` to inspect specific memory regions (e.g., strings, stack data, lists) as needed.
6. `run` again to restart the program from the beginning if needed (can remove/reset breakpoints along the way).

---

## Mapping to 8085 / GNUSim8085

You said the end goal is applying this to **GNUSim8085**. Here's an honest breakdown of what transfers and what doesn't, so you don't try to type ARM syntax into an 8085 simulator:

### Concepts that DO carry over (general assembly-language ideas, not ARM-specific)
| ARM concept | 8085 equivalent idea |
|---|---|
| Registers (R0–R6, general purpose) | 8085 has A (accumulator), B, C, D, E, H, L — fewer registers, and the **accumulator (A)** is special (most arithmetic must go through it, unlike ARM where any general register can be an operand). |
| CPSR flags (N, Z, C, V) | 8085 has its **own flag register** (Sign, Zero, Auxiliary Carry, Parity, Carry) — same *concept* (status bits after an operation) but different flags and different instructions set them. |
| Stack (SP, PUSH/POP) | 8085 also has a **Stack Pointer (SP)** and its own `PUSH`/`POP` instructions (operating on register **pairs**, e.g. `PUSH B`, `POP H`) — concept is the same, syntax differs. |
| Program Counter (PC) | 8085 also has a PC — same concept. |
| Branching / conditional jumps | 8085 uses `JMP`, `JZ`, `JNZ`, `JC`, `JNC`, `JP`, `JM`, etc. instead of ARM's `B`, `BEQ`, `BNE`... — same *idea* (jump based on flags), totally different mnemonics. |
| Subroutines / call-return | 8085 uses `CALL` and `RET` (which automatically use the stack for the return address — 8085 has **no Link Register**; it's stack-based, not register-based, unlike ARM's `BL`/`LR`/`BX LR`). |
| Looping via compare+branch | Same general pattern in 8085 (`CMP`, then conditional jump back to a label), though 8085 typically uses register-pair-based counters and `DCR`/`INR` (decrement/increment) rather than ARM-style pointer arithmetic. |
| Arithmetic (add/sub/mul-ish) | 8085 has `ADD`, `SUB`, `ADC`, `SBB` (subtract with borrow) — **but 8085 has no native multiply/divide at all** (even more limited than ARM, which at least has `MUL`). |
| Bitwise logic | 8085 has `ANA`, `ORA`, `XRA`, `CMA` (complement) — same concepts as ARM's `AND`/`ORR`/`EOR`/`MVN`. |
| Shifts/rotates | 8085 has `RLC`, `RRC`, `RAL`, `RAR` (rotate accumulator left/right, with/without carry) — conceptually parallel to ARM's `LSL`/`LSR`/`ROR`, but 8085 rotates only affect the **accumulator**, not arbitrary registers, and there's no separate "logical shift" vs "rotate" distinction the same way. |
| Memory addressing | 8085 uses direct addressing (`LDA`, `STA` with a 16-bit address) and indirect addressing via register pairs (`LDAX`, `STAX`, or via `H-L` pair with `MOV M, ...`) — same *concept* as ARM's `LDR`/`STR`/`[R0]`, different syntax and far more limited (8085 is 16-bit addressing on an 8-bit data bus). |

### Concepts that do NOT carry over directly (ARM/OS-specific, not applicable to 8085 or GNUSim8085)
- `SWI`/software interrupts as a way to call OS services (write/exit) — **GNUSim8085 has no operating system underneath it**; there's no `write()` syscall equivalent. Output in GNUSim8085 is typically done by just writing to memory/registers and inspecting them in the simulator, not through OS calls.
- The Linux toolchain steps (`as`, `ld`, GDB, QEMU, Raspberry Pi setup) — **not applicable at all** to GNUSim8085, which is a self-contained simulator with its own assembler/runner built in.
- ARM-specific addressing-mode syntax (`[R0, #4]!`, `[R0], #4`, `=label`) — 8085 doesn't have pre/post-increment addressing or this bracket syntax; its addressing modes are far simpler (direct, register, register-indirect via H-L, and immediate).
- Hardware memory-mapped switches/LEDs example — specific to the ARM DE1-SoC emulator's simulated board; GNUSim8085 has no equivalent peripheral simulation.
- 32-bit word size / CPSR's V (overflow) flag — 8085 is 8-bit, so "word" size and flag behavior differ numerically even where the concept (e.g., a Carry flag) is similar.

### Practical suggestion
Since the **underlying ideas** (registers, flags, stack, conditional jumps, subroutines with call/return, looping, bit operations) are universal to assembly programming, use this ARM document as a **conceptual reference** for *why* these mechanisms exist and *how* they behave logically — then learn the **actual 8085 instruction set and its specific mnemonics/syntax** separately (ideally straight from GNUSim8085's own instruction reference or an 8085 textbook/datasheet) before writing 8085 programs. I'm happy to build you a **separate, dedicated 8085/GNUSim8085 notes file** (with correct 8085 mnemonics, register set, flag register layout, and GNUSim8085-specific usage) if that would help — just say the word.
