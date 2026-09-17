# 8085 Microprocessor — Branching (Jump/Call) & Stack Group Instructions

This document explains the **Branching Group** (unconditional/conditional jumps, call) and the **Stack Group** (PUSH/POP) instructions of the Intel 8085, along with the background concepts needed to understand them.

> Note: The numbering in this image restarts at 50 (a new board/section), so these numbers overlap with the earlier Logic Group notes — that's expected since they were written on a separate board.

---

## 1. Background Concepts

### 1.1 What "Branching" Means
Normally, the 8085 executes instructions **sequentially** — one after another, with the **Program Counter (PC)** automatically incrementing to point to the next instruction. A **branch/jump instruction** breaks this sequence by **loading a new address into the PC**, causing execution to continue from a different part of the program.

### 1.2 Unconditional vs Conditional Jumps
- **Unconditional jump** — always jumps, no matter what. Example: `JMP`.
- **Conditional jump** — jumps **only if** a specific flag condition is true. If the condition is false, execution just continues to the next instruction normally. These conditions are checked against the **flag register** (Sign, Zero, Carry, Parity flags), which gets set by preceding arithmetic/logic/compare instructions.

### 1.3 The Flags Used for Conditions
| Flag | Meaning |
|---|---|
| **Z (Zero)** | Set to 1 if the last result was zero |
| **CY (Carry)** | Set to 1 if the last operation produced a carry/borrow |
| **S (Sign)** | Set to 1 if the last result was negative (MSB = 1) |
| **P (Parity)** | Set to 1 if the last result has an **even** number of 1-bits |

### 1.4 The Stack
The **stack** is a special area of memory used to temporarily store data (especially return addresses and register values), managed by the **Stack Pointer (SP)**. It works in a **LIFO (Last In, First Out)** manner — the last thing pushed in is the first thing popped out. The stack grows **downward** in memory (SP decreases on PUSH, increases on POP).

---

## 2. Unconditional Jump

### 50) `JMP 2000H`
**Operation:** PC ← 2000H  
Unconditionally loads the 16-bit address `2000H` into the Program Counter. Execution jumps straight to that address, no condition checked.

---

## 3. Conditional Jumps

Each of these checks a flag; if the condition is true, PC is loaded with the target address (in real code, these are written like `JC 2000H` — an address always follows). If false, execution simply proceeds to the next line.

### 51) `JC` — Jump on Carry
Jumps **if Carry flag (CY) = 1**. Used after arithmetic operations where a carry/borrow indicates something notable (e.g., overflow, or A < B after a `CMP`).

### 52) `JNC` — Jump on No Carry
Jumps **if Carry flag (CY) = 0**. The opposite of `JC`.

### 53) `JNZ` — Jump on Not Zero
Jumps **if Zero flag (Z) = 0**, i.e., the previous result was **not** zero. Extremely common for loop counters (e.g., decrement a counter, then `JNZ` back to repeat until it hits zero).

### 54) `JZ` — Jump on Zero
Jumps **if Zero flag (Z) = 1**, i.e., the previous result **was** zero. Commonly used to detect when a counter has finished, or when two compared values were equal.

### 55) `JPE` — Jump on Parity Even
Jumps **if Parity flag (P) = 1**, meaning the result had an **even** number of 1-bits.

### 56) `JPO` — Jump on Parity Odd
Jumps **if Parity flag (P) = 0**, meaning the result had an **odd** number of 1-bits.

### 57) `JM` — Jump on Minus
Jumps **if Sign flag (S) = 1**, meaning the result was **negative** (bit 7 of the result = 1, treating it as signed).

### 58) `JP` — Jump on Positive
Jumps **if Sign flag (S) = 0**, meaning the result was **positive** (bit 7 = 0).

---

## 4. Subroutine Call

### 59) `CALL`
**Operation:** Push (PC+3) onto the stack, then PC ← target address  
Used to call a **subroutine**. Unlike `JMP`, `CALL` first **saves the address of the next instruction (the return address) onto the stack**, then jumps to the specified address. This way, when the subroutine finishes (with a `RET` instruction), the processor knows where to resume execution — it pops that saved address back into PC. This is how the 8085 implements reusable functions/procedures.

---

## 5. Stack Instructions

### 60) `PUSH`
**Operation:** Decrements SP, then stores a 16-bit register pair's contents onto the stack (e.g., `PUSH B` stores BC).  
Copies the contents of a register pair onto the top of the stack, and **decrements SP by 2** (since 2 bytes — a full register pair — are stored). Commonly used to **save register values** before they get overwritten (e.g., before calling a subroutine that will reuse those registers).

### 61) `POP`
**Operation:** Retrieves a 16-bit value from the stack into a register pair, then increments SP.  
Removes (pops) the top 2 bytes from the stack and loads them into the specified register pair (e.g., `POP B` restores BC), then **increments SP by 2**. Used to **restore** register values that were previously saved with `PUSH`.

**Important:** PUSH and POP operations must always be balanced — for every PUSH, there should be a corresponding POP — otherwise the stack pointer gets misaligned and can corrupt the return addresses of `CALL`/`RET`, crashing the program.

---

## 6. Quick Reference Table

| # | Mnemonic | Condition / Action | Category |
|---|----------|---------------------|----------|
| 50 | JMP 2000H | Always jump to 2000H | Unconditional jump |
| 51 | JC | Jump if CY = 1 | Conditional jump |
| 52 | JNC | Jump if CY = 0 | Conditional jump |
| 53 | JNZ | Jump if Z = 0 | Conditional jump |
| 54 | JZ | Jump if Z = 1 | Conditional jump |
| 55 | JPE | Jump if P = 1 (even parity) | Conditional jump |
| 56 | JPO | Jump if P = 0 (odd parity) | Conditional jump |
| 57 | JM | Jump if S = 1 (negative) | Conditional jump |
| 58 | JP | Jump if S = 0 (positive) | Conditional jump |
| 59 | CALL | Save return address, jump to subroutine | Subroutine call |
| 60 | PUSH | Save register pair onto stack | Stack |
| 61 | POP | Restore register pair from stack | Stack |

---

## 7. Key Takeaways
- **JMP** always jumps; **conditional jumps** (`JC`, `JNC`, `JZ`, `JNZ`, `JM`, `JP`, `JPE`, `JPO`) only jump if a specific flag matches, otherwise execution just falls through.
- Conditional jumps rely entirely on the **flag register**, which is set by whatever arithmetic/logic/compare instruction ran just before the jump.
- **CALL** is like `JMP`, but it remembers where to come back to (via the stack) — this is the foundation of subroutines/functions.
- **PUSH/POP** manually save and restore data on the stack, and must always be used in balanced pairs.
- The stack is essential not just for `PUSH`/`POP`, but also silently used behind the scenes by `CALL` and `RET` to manage return addresses.
