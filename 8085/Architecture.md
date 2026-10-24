![What is 8085 Microprocessor? Definition, Architecture, Block Diagram,  Working and Applications of 8085 Microprocessor - Electronics Desk](https://electronicsdesk.com/wp-content/uploads/2019/03/architecture-of-8085-microprocessor.jpg)

# Intel 8085 Microprocessor — Complete Architecture and Registers Notes

> Detailed, exam-oriented reference covering the 8085 CPU architecture, every major register and register pair, flags, buses, pins, instruction execution, interrupts, timing, interfacing, and important low-level details.

---

## Table of Contents

1. [Introduction to the 8085](#1-introduction-to-the-8085)
2. [8085 Specifications](#2-8085-specifications)
3. [Complete Architecture Overview](#3-complete-architecture-overview)
4. [Register Classification](#4-register-classification)
5. [Accumulator (A)](#5-accumulator-a)
6. [General-Purpose Registers (B, C, D, E, H, L)](#6-general-purpose-registers-b-c-d-e-h-l)
7. [Register Pairs: BC, DE, HL](#7-register-pairs-bc-de-hl)
8. [Program Counter (PC)](#8-program-counter-pc)
9. [Stack Pointer (SP)](#9-stack-pointer-sp)
10. [Flag Register and Condition Flags](#10-flag-register-and-condition-flags)
11. [Internal Registers and Temporary Storage](#11-internal-registers-and-temporary-storage)
12. [Instruction Register and Instruction Decoder](#12-instruction-register-and-instruction-decoder)
13. [ALU: Arithmetic and Logic Unit](#13-alu-arithmetic-and-logic-unit)
14. [Incrementer/Decrementer and Address Generation](#14-incrementerdecrementer-and-address-generation)
15. [Timing and Control Unit](#15-timing-and-control-unit)
16. [Address Bus, Data Bus, and Control Signals](#16-address-bus-data-bus-and-control-signals)
17. [8085 Pin Diagram: Pin Functions](#17-8085-pin-diagram-pin-functions)
18. [Instruction Fetch, Decode, and Execute](#18-instruction-fetch-decode-and-execute)
19. [Machine Cycles and T-States](#19-machine-cycles-and-t-states)
20. [Stack, CALL, RET, PUSH, and POP](#20-stack-call-ret-push-and-pop)
21. [Interrupt System](#21-interrupt-system)
22. [Serial I/O and SID/SOD](#22-serial-io-and-sidsod)
23. [Reset and Startup Behavior](#23-reset-and-startup-behavior)
24. [Memory and I/O Addressing](#24-memory-and-io-addressing)
25. [Register Effects of Important Instructions](#25-register-effects-of-important-instructions)
26. [Common Misconceptions and Subtle Details](#26-common-misconceptions-and-subtle-details)
27. [Quick Reference Tables](#27-quick-reference-tables)
28. [Worked Examples](#28-worked-examples)
29. [Final Summary](#29-final-summary)
30. [Glossary](#30-glossary)

---

## 1. Introduction to the 8085

The **Intel 8085** is an 8-bit microprocessor introduced by Intel in 1976. It is a general-purpose CPU used historically in educational systems, control systems, and early microcomputer designs.

A microprocessor is the CPU implemented on a single integrated circuit. It fetches instructions from memory, decodes them, performs operations, and communicates with memory and peripheral devices.

### What does “8-bit” mean?

The 8085 is called an 8-bit microprocessor mainly because its **ALU and principal data-processing path operate on 8-bit values**.

- An 8-bit value can represent an unsigned number from `0` to `255`.
- A 16-bit address can identify `2^16 = 65,536` byte locations.
- Therefore, the 8085 can address **64 KiB of memory** in its full 16-bit memory-address space.
- The processor can perform 16-bit operations in certain cases, such as register-pair increment/decrement and 16-bit addition, even though it is an 8-bit CPU.

### Why is it called 8085?

The “8085” is the processor model number. The device uses a single +5 V supply in its standard form, unlike the earlier 8080 design that required multiple supply voltages. The final digit is commonly associated with this +5 V supply characteristic, though the model number itself is Intel's product designation.

### Basic working cycle

At a high level, the CPU repeatedly:

1. Uses the Program Counter (PC) to identify the next instruction byte.
2. Fetches the byte from memory.
3. Decodes the instruction.
4. Fetches any required operand bytes or data.
5. Executes the operation in the ALU, registers, or memory interface.
6. Updates registers, flags, and the PC as required.
7. Responds to eligible interrupts at defined instruction boundaries.

---

## 2. 8085 Specifications

| Feature                        | 8085 detail                                                                  |
| ------------------------------ | ---------------------------------------------------------------------------- |
| Data width                     | 8 bits                                                                       |
| Address width                  | 16 bits                                                                      |
| Maximum memory address space   | 64 KiB (`0000H`–`FFFFH`)                                                |
| I/O port address width         | 8 bits for direct `IN`/`OUT` addressing                                  |
| Directly addressable I/O ports | 256 port addresses (`00H`–`FFH`)                                        |
| Main ALU width                 | 8 bits                                                                       |
| General-purpose registers      | Six 8-bit registers: B, C, D, E, H, L                                        |
| Accumulator                    | One 8-bit register, A                                                        |
| Flag register                  | 5 active condition flags in an 8-bit flag register                           |
| Program Counter                | 16 bits                                                                      |
| Stack Pointer                  | 16 bits                                                                      |
| Address/data multiplexing      | AD0–AD7 are multiplexed                                                     |
| Interrupt inputs               | TRAP, RST 7.5, RST 6.5, RST 5.5, INTR                                        |
| Serial pins                    | SID input and SOD output                                                     |
| Clock                          | On-chip clock-generation circuitry; external crystal/clock input             |
| Typical clock rates            | Depends on 8085 variant; commonly taught versions run around 3 MHz           |
| Instruction set                | Commonly described as 74 basic instruction types, with many opcode encodings |
| Supply                         | Standard 8085 uses +5 V                                                      |

**Notation:** `H` after a number indicates hexadecimal. For example, `3AH` is hexadecimal 3A. `B` after a number may indicate binary in some textbooks; context matters. `M` in 8085 assembly generally means the memory byte addressed by the HL pair.

---

## 3. Complete Architecture Overview

The architecture is the arrangement of functional blocks and the way they exchange data and control signals.

### Main functional blocks

1. **Accumulator (A)** — primary 8-bit operand/result register.
2. **General-purpose registers (B, C, D, E, H, L)** — temporary data storage.
3. **Register-pair paths** — BC, DE, and HL can hold 16-bit values.
4. **Program Counter (PC)** — address of the next instruction byte to fetch.
5. **Stack Pointer (SP)** — address used to manage the stack in memory.
6. **Flag register (F)** — records selected outcomes of ALU operations.
7. **Arithmetic and Logic Unit (ALU)** — arithmetic, logic, comparison, and related operations.
8. **Instruction register and decoder** — hold and interpret the current instruction.
9. **Incrementer/decrementer and address-generation logic** — updates addresses and supports 16-bit operations.
10. **Timing and control unit** — generates internal timing and external control signals.
11. **Address/data buffers** — interface the CPU with external buses.
12. **Interrupt control** — handles interrupt requests, priorities, masks, and acknowledgements.
13. **Serial I/O control** — reads SID and drives SOD.
14. **Clock-generation circuit** — provides timing for internal operation.

### Conceptual block diagram

```text
                    +-----------------------------------+
                    |             8085 CPU              |
                    |                                   |
 Address/data pins  |  +-----------+   +-------------+  |
 <----------------->|  | Bus       |<->| Internal    |  |
                    |  | buffers   |   | data paths  |  |
                    |  +-----------+   +------+------+  |
                    |                         |         |
                    |  +-----------+   +------v------+  |
                    |  | Program   |   | Registers   |  |
                    |  | Counter   |   | A,B,C,D,E,  |  |
                    |  | (PC)      |   | H,L and F   |  |
                    |  +-----------+   +------+------+  |
                    |                         |         |
                    |  +-----------+   +------v------+  |
                    |  | Stack     |   | ALU         |  |
                    |  | Pointer   |   | arithmetic  |  |
                    |  | (SP)      |   | and logic   |  |
                    |  +-----------+   +------+------+  |
                    |                         |         |
                    |  +-----------+   +------v------+  |
                    |  | Instruction|  | Flag        |  |
                    |  | register & |  | generation  |  |
                    |  | decoder    |  | / register  |  |
                    |  +-----------+   +-------------+  |
                    |                                   |
                    | Timing/control, interrupt control,|
                    | serial I/O and clock circuitry    |
                    +-----------------------------------+
```

This is a **conceptual** diagram, not a transistor-level or exact physical layout. Different textbooks draw the internal blocks somewhat differently.

### How the blocks cooperate

Suppose the instruction is:

```asm
MVI A, 25H
```

Conceptually:

1. PC identifies the address of the opcode.
2. The CPU fetches the opcode for `MVI A, data`.
3. PC advances to the next byte.
4. The immediate data byte `25H` is fetched.
5. The data is loaded into A.
6. PC advances past the two-byte instruction.
7. No arithmetic result is produced, so the condition flags are not modified by this instruction.

For an arithmetic example:

```asm
ADD B
```

1. A and B supply the operands to the ALU.
2. The ALU computes `A + B`.
3. The result is written back to A.
4. The condition flags are updated according to the result and carry/auxiliary-carry conditions.
5. B remains unchanged.

### Data flow versus control flow

- **Data flow**: movement of values among registers, ALU, memory, and I/O.
- **Control flow**: selection of the next instruction address, including sequential execution, jumps, calls, returns, resets, and interrupt servicing.

The PC is central to control flow; the registers and ALU are central to data processing.

---

## 4. Register Classification

Registers are very fast storage locations inside the CPU. They hold operands, addresses, results, status information, and temporary values.

They are made up of Flip flops

8bit registers = 8 Flipflops (stores 1 bit each)

### 4.1 Programmer-visible registers

These are registers that the instruction set allows a programmer to use directly or indirectly.

| Register             |                            Width | Primary role                                       |
| -------------------- | -------------------------------: | -------------------------------------------------- |
| A (Accumulator)      |                           8 bits | Main ALU operand/result                            |
| B                    |                           8 bits | General-purpose data                               |
| C                    |                           8 bits | General-purpose data                               |
| D                    |                           8 bits | General-purpose data                               |
| E                    |                           8 bits | General-purpose data                               |
| H                    |                           8 bits | General-purpose data; high byte of HL address pair |
| L                    |                           8 bits | General-purpose data; low byte of HL address pair  |
| Flag register (F)    | 8 bits, 5 active condition flags | Status after selected operations                   |
| Program Counter (PC) |                          16 bits | Next instruction-byte address                      |
| Stack Pointer (SP)   |                          16 bits | Stack address pointer                              |

A and F are often discussed together as the **PSW (Program Status Word)** for `PUSH PSW` and `POP PSW`.

### 4.2 Internal or non-programmer-visible storage

The 8085 also contains internal storage used during instruction execution. Educational block diagrams commonly show:

- **Instruction Register (IR)** — holds the instruction opcode being decoded.
- **Temporary Register(s)** — hold intermediate operands or values for ALU operations.
- **W and Z temporary registers** — often shown in textbook architectures as a temporary 16-bit pair, WZ. They are not directly accessible through ordinary assembly mnemonics.
- **Internal address/data latches and buffers** — preserve values while buses are used for different purposes.
- **ALU-related carry/temporary logic** — supports arithmetic and flag generation.

**Important:** Internal register naming and drawing details can vary by textbook. W and Z are commonly shown in 8085 teaching diagrams, but they are not part of the programmer-visible register set. Do not assume that an assembly instruction can directly read or write IR, W, or Z.

### 4.3 Register, memory location, and I/O port are different

- A **register** is storage inside the CPU.
- A **memory location** is a byte stored in external memory and selected by a memory address.
- An **I/O port** is a device register/address selected through I/O instructions or memory-mapped I/O logic.

For example, `M` is not a physical CPU register. It is an assembly-language shorthand for the memory byte at the address contained in HL.

---

## 5. Accumulator (A)

The **Accumulator** is an 8-bit register used by many arithmetic and logical instructions. It is the primary operand and destination register for a large part of the 8085 instruction set.

### Main functions

1. Holds one operand for arithmetic and logical operations.
2. Receives the result of many ALU operations.
3. Transfers data to and from memory and I/O.
4. Participates in compare, rotate, decimal-adjust, and complement operations.
5. Forms the upper byte of the PSW representation used by `PUSH PSW`/`POP PSW` conventions.

### Example: addition

```asm
MVI A, 25H
MVI B, 17H
ADD B
```

Calculation:

```text
  25H = 0010 0101
+ 17H = 0001 0111
-----------------
  3CH = 0011 1100
```

After `ADD B`:

- `A = 3CH`
- `B = 17H` (unchanged)
- `Z = 0` because the result is not zero
- `S = 0` because bit 7 is zero
- `P = 1` because `3CH` has four 1-bits (even parity)
- `CY = 0` because there is no carry out of bit 7
- `AC = 0` because there is no carry from bit 3 to bit 4

### Instructions that use A

Examples include:

- Arithmetic: `ADD`, `ADC`, `SUB`, `SBB`, `ADI`, `ACI`, `SUI`, `SBI`, `INR A`, `DCR A`, `DAD` indirectly uses pair values, `DAA`
- Logic: `ANA`, `XRA`, `ORA`, `ANI`, `XRI`, `ORI`, `CMP`, `CPI`
- Data transfer: `MVI A`, `LDA`, `STA`, `LDAX`, `STAX`, `MOV A,r`, `MOV r,A`
- Rotates: `RLC`, `RRC`, `RAL`, `RAR`
- Complement: `CMA`
- I/O: `IN`, `OUT`
- Exchange-related instructions: `XCHG` exchanges DE and HL, not A

### Does every instruction change A?

No. `MOV B,C` does not change A. `CMP B` updates flags as if subtraction were performed but **does not store the subtraction result in A**. `STA address` writes A to memory but leaves A unchanged.

---

## 6. General-Purpose Registers (B, C, D, E, H, L)

The 8085 contains six 8-bit general-purpose registers:

- B
- C
- D
- E
- H
- L

Each can hold an 8-bit value (`00H`–`FFH`). They are used to store operands, counters, intermediate values, and address bytes.

### 6.1 B and C

B and C are general-purpose registers. Together they form the **BC register pair**, which can hold a 16-bit value.

Common uses:

- Store data or loop counters.
- Hold a 16-bit address in BC for instructions such as `LDAX B` and `STAX B`.
- Use B or C as a source/destination in `MOV`, arithmetic, and increment/decrement instructions.

Example:

```asm
MVI B, 12H
MVI C, 34H
```

Now `B = 12H`, `C = 34H`, and `BC = 1234H` when interpreted as a register pair.

### 6.2 D and E

D and E are general-purpose registers. Together they form the **DE register pair**.

Common uses:

- Hold data and intermediate results.
- Hold a 16-bit memory address for `LDAX D` and `STAX D`.
- Exchange values with HL using `XCHG`.

Example:

```asm
LXI D, 2050H
```

After execution:

- `D = 20H`
- `E = 50H`
- `DE = 2050H`

### 6.3 H and L

H and L are general-purpose registers, but they have a particularly important role: together they form the **HL register pair**, frequently used as a memory pointer.

- H contains the high-order byte of HL.
- L contains the low-order byte of HL.
- `M` refers to the memory location whose address is held in HL.

Example:

```asm
LXI H, 2050H
MVI M, 7AH
```

After `LXI H,2050H`, `HL = 2050H`. The second instruction stores `7AH` in memory location `2050H`; it does **not** place `7AH` in H or L.

### 6.4 Registers are not all interchangeable in every instruction

Most register-to-register operations support B, C, D, E, H, L, and A. However:

- `M` is the memory operand addressed by HL, not an independent register.
- `LDAX` and `STAX` accept BC or DE as the address pair, not HL.
- `LXI`, `INX`, `DCX`, and `DAD` use specific 16-bit register-pair forms.
- `PUSH`/`POP` support BC, DE, HL, and PSW, not arbitrary register pairs like SP or PC.
- Some operations explicitly use A or a particular pair.

---

## 7. Register Pairs: BC, DE, HL

A **register pair** combines two 8-bit registers into a 16-bit value. The first named register is the high byte; the second is the low byte.

| Pair | High byte | Low byte | 16-bit interpretation   |
| ---- | --------- | -------- | ----------------------- |
| BC   | B         | C        | `BC = (B × 256) + C` |
| DE   | D         | E        | `DE = (D × 256) + E` |
| HL   | H         | L        | `HL = (H × 256) + L` |

Example:

```text
B = 12H
C = 34H
BC = 1234H
```

The pair is a way of interpreting two registers as a 16-bit quantity; it is not a separate independent storage location.

### 7.1 `LXI` — load immediate 16-bit value

```asm
LXI B, 1234H
LXI D, 4567H
LXI H, 2050H
LXI SP, 3000H
```

Each instruction loads a 16-bit immediate value into the specified pair or SP.

For `LXI H,2050H`:

- H receives `20H`
- L receives `50H`

`LXI` does not alter condition flags.

### 7.2 `INX` and `DCX`

- `INX rp`: increments a register pair or SP by one.
- `DCX rp`: decrements a register pair or SP by one.

Example:

```asm
LXI H, 20FFH
INX H
```

Result: `HL = 2100H`. The low byte wraps from `FFH` to `00H`, and the high byte increments.

**Flag detail:** `INX` and `DCX` do not modify the five condition flags.

### 7.3 `DAD` — 16-bit addition to HL

`DAD rp` performs:

```text
HL ← HL + rp
```

The source can be BC, DE, HL, or SP.

Example:

```asm
LXI H, 2000H
LXI D, 1234H
DAD D
```

Result: `HL = 3234H`.

**Flag detail:** `DAD` affects the Carry flag only. It does not update Sign, Zero, Parity, or Auxiliary Carry in the standard 8085 instruction model.

### 7.4 `XCHG`

`XCHG` exchanges the contents of DE and HL:

```text
DE ↔ HL
```

If `DE = 1234H` and `HL = 5678H`, after `XCHG`:

- `DE = 5678H`
- `HL = 1234H`

Flags are not affected.

### 7.5 Why HL is special

HL is used by many instructions to access memory through the symbolic operand `M`:

```asm
LXI H, 2500H
MOV A, M     ; A ← memory[2500H]
MOV M, B     ; memory[2500H] ← B
INR M        ; memory[HL] ← memory[HL] + 1
```

The address in HL is 16-bit, so HL can point anywhere in the 64 KiB memory space.

---

## 8. Program Counter (PC)

The **Program Counter (PC)** is a 16-bit register that holds the memory address from which the CPU will fetch the next instruction byte.

contains the adress of next instruction

### Main roles

- Identifies the next instruction byte to fetch.
- Advances as instruction bytes are fetched.
- Is replaced by a target address during jumps, calls, returns, resets, and interrupt servicing.
- Supports sequential execution and control-flow changes.

### PC is a byte address, not an instruction number

8085 instructions can be one, two, or three bytes long. PC counts **byte addresses**, not instructions.

Examples:

- `MOV A,B` is one byte.
- `MVI A,32H` is two bytes.
- `LXI H,2050H` is three bytes.

If a three-byte instruction starts at `2000H`, its bytes are at `2000H`, `2001H`, and `2002H`; the sequential next address is `2003H`.

### Sequential execution

If an instruction is one byte long, the PC normally advances by one byte during fetching. Two-byte and three-byte instructions require additional byte fetches. Exact internal update timing depends on the instruction's execution sequence.

### Branching

```asm
JMP 2500H
```

The PC is loaded with `2500H`, so the next instruction fetch is from that address.

```asm
JZ 2500H
```

If Zero flag `Z = 1`, PC becomes `2500H`; otherwise execution continues at the next sequential instruction.

### PC width and range

- Width: 16 bits.
- Range: `0000H` to `FFFFH`.
- Address-space size: 65,536 byte addresses.

PC is not normally available as a general-purpose source or destination in `MOV` instructions.

---

## 9. Stack Pointer (SP)

The **Stack Pointer (SP)** is a 16-bit register that points to the stack area in memory. The stack is used for temporary storage, subroutine return addresses, and saving register contents.

### The stack is in memory

The 8085 does not contain a separate hardware stack memory. The stack occupies ordinary memory locations chosen by the program. SP identifies the current top according to the processor's stack convention.

### Stack direction

The 8085 stack grows **toward lower memory addresses**:

- `PUSH` decrements SP as data is placed on the stack.
- `POP` reads data and increments SP as the stack is unwound.
- `CALL` saves a return address on the stack.
- `RET` retrieves a return address from the stack.

For a common 8085 convention, `SP` is initialized to the address one byte above the highest intended stack location, for example:

```asm
LXI SP, 3000H
```

The exact addresses used by a push depend on the instruction and the initial SP; see the stack section below.

### Why initialize SP?

If SP is not set to a safe region of RAM, a `PUSH`, `CALL`, interrupt, or other stack operation may overwrite program data or use invalid memory.

Example:

```asm
LXI SP, 3FFFH
```

This initializes SP to `3FFFH`. It is only a valid choice if the system's memory map and program layout make that region appropriate.

### SP can be directly loaded or incremented/decremented

```asm
LXI SP, 4000H
INX SP
DCX SP
```

`INX SP` and `DCX SP` do not affect condition flags.

SP is not the same as HL. Both are 16-bit registers, but HL is a general-purpose/address pair and SP is dedicated to stack management.

---

## 10. Flag Register and Condition Flags

The **flag register** is 8 bits wide. Five bits are the condition flags used by the 8085 instruction set:

- S — Sign
- Z — Zero
- AC — Auxiliary Carry
- P — Parity
- CY — Carry

A commonly taught bit layout is:

```text
Bit:   7   6   5   4   3   2   1   0
       S   Z   -  AC   -   P   -  CY
```

The dashes are unused/reserved positions in the conventional programmer-visible flag format. Do not assume these positions are independently usable condition flags.

### 10.1 Sign flag (S)

S is a copy of bit 7 of the 8-bit result for instructions that update the flags.

- `S = 1` when result bit 7 is 1.
- `S = 0` when result bit 7 is 0.

For signed two's-complement interpretation, bit 7 is commonly interpreted as the sign bit:

- `00H`–`7FH`: non-negative
- `80H`–`FFH`: negative

But the processor does not enforce signed arithmetic. S simply reflects result bit 7.

Example:

- Result `7FH` → S = 0.
- Result `80H` → S = 1.

### 10.2 Zero flag (Z)

Z is set if the 8-bit result is exactly zero.

- `Z = 1` if result = `00H`.
- `Z = 0` otherwise.

Example:

```asm
MVI A, 05H
SUI 05H
```

Result A = `00H`; Z = 1.

### 10.3 Auxiliary Carry flag (AC)

AC indicates a carry from bit 3 to bit 4 in addition, or the corresponding borrow condition in subtraction. It is primarily useful for BCD-related operations, especially `DAA`.

For addition, consider the lower nibble:

- `0FH + 01H = 10H`
- A carry moves from bit 3 to bit 4, so AC = 1.

AC is not the same as CY:

- AC concerns the boundary between the low and high nibbles (bit 3 → bit 4).
- CY concerns carry out of bit 7.

For subtraction, AC behavior reflects the borrow across the low-nibble boundary according to 8085 arithmetic rules.

### 10.4 Parity flag (P)

P reports the parity of the 8-bit result.

- `P = 1` for **even parity**: an even number of 1-bits.
- `P = 0` for **odd parity**: an odd number of 1-bits.

Examples:

- `03H = 00000011` has two 1-bits → P = 1.
- `07H = 00000111` has three 1-bits → P = 0.
- `00H` has zero 1-bits, which is even → P = 1.

Parity is based on the bit pattern, not whether the numeric value is mathematically even or odd. For example, `03H` is numerically odd but has even bit parity.

### 10.5 Carry flag (CY)

CY is set when an operation produces a carry out of bit 7, or a borrow in subtraction as defined by the 8085 subtraction operation.

For unsigned 8-bit addition:

```text
FFH + 01H = 100H
```

Only the low 8 bits are retained in A (`00H`), and CY = 1.

For subtraction, CY = 1 indicates a borrow was needed. For example, `00H - 01H` produces low byte `FFH` and sets CY = 1.

### 10.6 Which instructions update flags?

Flags are not updated by every instruction.

Typical flag-updating groups:

- Arithmetic instructions (`ADD`, `ADC`, `SUB`, `SBB`, immediate arithmetic, `INR`, `DCR`, `CMP`, etc.)
- Logical instructions (`ANA`, `XRA`, `ORA`, immediate variants)
- Rotate instructions update CY, but do not update the other arithmetic flags in the same way as an ALU arithmetic operation.
- `DAD` updates CY only.

Common instructions that **do not change condition flags** include:

- `MOV`, `MVI`, `LXI`
- `LDA`, `STA`, `LHLD`, `SHLD`
- `INX`, `DCX`
- `PUSH`, `POP` (with the important exception that `POP PSW` restores the flag register from the stack)
- `JMP`, `CALL`, `RET`
- `XCHG`
- `CMA` (complements A but does not update flags)
- `STC` and `CMC` specifically set/complement CY; they do not recompute all flags.

Always check the individual instruction's definition rather than assuming that a register changed means flags changed.

### 10.7 `INR` and `DCR` flag nuance

`INR r` and `DCR r` update S, Z, P, and AC but **do not modify CY**.

Example:

- If CY was 1 before `INR A`, it remains 1 afterward.
- This is a major exam and debugging detail.

### 10.8 `CMP` does not store a subtraction result

`CMP r` compares A with a register by performing flag-setting subtraction logic equivalent to `A - r`, but it does not replace A with the difference.

- Z = 1 if A equals the operand.
- CY = 1 if A is less than the operand in unsigned comparison.
- A remains unchanged.

### 10.9 PSW: Accumulator + Flags

PSW means **Program Status Word** in the 8085 context and is used by the `PUSH PSW` and `POP PSW` instructions. It comprises A and the flag register F.

- `PUSH PSW` saves the accumulator and flags on the stack.
- `POP PSW` restores them.
- It is not a separate general-purpose 16-bit register that can be used in `LXI PSW, ...`.

The exact stack byte ordering follows the processor's push/pop convention; conceptually both A and F are saved/restored together.

---

## 11. Internal Registers and Temporary Storage

Not every storage element inside a CPU is directly addressable by assembly code. Some exist only to support the internal execution sequence.

### 11.1 Instruction Register (IR)

The IR holds the opcode byte currently being decoded/executed. The CPU's decoder interprets opcode fields to determine the operation and operands.

- It is internal.
- It is not normally accessible through `MOV`, `MVI`, or similar programmer instructions.
- It is distinct from the PC: PC identifies an instruction's memory address; IR holds the instruction byte itself.

### 11.2 Temporary registers

Temporary registers preserve operands or intermediate results while the ALU, internal buses, and external buses are used in sequence.

Textbook diagrams may show one or more temporary registers. Their exact naming and number can vary according to the abstraction level of the diagram.

Examples of their conceptual purpose:

- Hold an operand while another operand is fetched.
- Hold an intermediate result before it is written back.
- Support arithmetic and memory-transfer micro-operations.

These are not part of the documented programmer-visible register set.

### 11.3 W and Z registers (WZ pair in many teaching diagrams)

Many 8085 architecture diagrams show two internal 8-bit temporary registers called W and Z, sometimes treated as a 16-bit WZ pair.

- Used internally for temporary address or data handling in common educational descriptions.
- Not accessible as ordinary assembly operands.
- Not the same as BC, DE, or HL.
- Their use should not be relied upon by application code.

Because many textbook block diagrams simplify or vary internal implementation details, use W/Z as an explanation of internal temporary storage, not as a programmer-controlled register pair.

### 11.4 Address and data latches/buffers

These are not general-purpose registers. They temporarily hold address/data signals while the processor communicates with memory or I/O.

For example:

- AD0–AD7 first carry the low-order address during the beginning of a bus cycle.
- The address is externally latched using ALE.
- The same pins then carry the data byte.

This multiplexing requires latching because the address must remain available while the pins are reused for data.

### 11.5 Internal carry and flag logic

Carry propagation and flag-generation circuits are internal hardware. They support arithmetic but are not independently programmable registers.

---

## 12. Instruction Register and Instruction Decoder

An instruction is encoded as one or more bytes. The first byte is generally the **opcode**, which identifies the operation and often encodes register selections. Additional bytes may provide immediate data or a 16-bit address.

### Instruction register

The opcode is placed in internal instruction storage while the decoder interprets it. This register is internal, not a programmer-visible register.

### Instruction decoder

The decoder determines which operation the CPU must carry out and coordinates the required internal transfers and control signals.

For example:

```asm
MOV A, B
```

The opcode identifies a register-to-register transfer, with B as source and A as destination.

```asm
MVI A, 32H
```

The opcode identifies an immediate load into A, and the following byte contains `32H`.

```asm
JMP 2050H
```

The opcode is followed by two address bytes. The target address is assembled according to the 8085's little-endian instruction encoding: the low-order address byte appears first, then the high-order byte.

### Little-endian order

For:

```asm
JMP 2050H
```

The instruction bytes are conceptually:

```text
Opcode, 50H, 20H
```

The address is reconstructed as `2050H`. This is called little-endian order because the least significant byte is stored first.

---

## 13. ALU: Arithmetic and Logic Unit

The **ALU** is the main computation block. It performs arithmetic and logical operations on 8-bit data and helps generate condition flags.

### Arithmetic operations

- Addition: `ADD`, `ADC`, `ADI`, `ACI`
- Subtraction: `SUB`, `SBB`, `SUI`, `SBI`
- Increment: `INR`
- Decrement: `DCR`
- Compare: `CMP`, `CPI`
- Decimal adjust: `DAA`
- 16-bit addition into HL: `DAD`

### Logical operations

- AND: `ANA`, `ANI`
- XOR: `XRA`, `XRI`
- OR: `ORA`, `ORI`
- Complement accumulator: `CMA`
- Rotates: `RLC`, `RRC`, `RAL`, `RAR`

### Example: AND

```asm
MVI A, 5AH       ; 01011010
ANI 0FH          ; 00001111
```

Bitwise AND:

```text
  01011010
AND 00001111
------------
  00001010 = 0AH
```

A becomes `0AH`. The logical operation updates the appropriate flags according to the 8085 rules; for `ANA`/`ANI`, CY is cleared and AC is set in the standard 8085 behavior, while S, Z, and P reflect the result.

### Example: compare

```asm
MVI A, 20H
CPI 20H
```

The subtraction result is zero, so Z = 1. A remains `20H`.

### ALU is not a storage register

The ALU computes values; registers retain them. An arithmetic result often goes to A, while flags record selected properties of that result.

---

## 14. Incrementer/Decrementer and Address Generation

The processor needs to update addresses, especially PC, SP, and register pairs. Internal increment/decrement and address-generation logic supports this.

Examples of programmer-visible operations:

- `INX B`, `INX D`, `INX H`, `INX SP`
- `DCX B`, `DCX D`, `DCX H`, `DCX SP`
- PC progression during instruction-byte fetching
- Stack pointer updates during push/pop/call/return
- Address calculation for jumps and calls

This does not mean every address change is performed by one standalone, programmer-visible “incrementer register.” It is internal logic that supports the required micro-operations.

---

## 15. Timing and Control Unit

The timing and control unit coordinates the internal sequence of each instruction and produces external control signals.

Does the execution or atleast start it, tell everyone to do the process

Release the control signal everywhere

It coordinates:

- Instruction fetch and decode
- Memory reads and writes
- I/O reads and writes
- Address/data bus usage
- Interrupt acknowledgement and response
- Machine-cycle timing and T-states
- Status signals such as IO/M, S0, and S1

### Common control outputs

- **ALE** — Address Latch Enable; indicates that AD0–AD7 contain the low-order address to be latched.
- **RD̅** — active-low read control.
- **WR̅** — active-low write control.
- **IO/M** — distinguishes I/O-related cycles from memory cycles (interpret together with status signals).
- **S1, S0** — status outputs that help identify the current machine cycle.
- **INTA̅** — interrupt acknowledge output for INTR.

The overbar (or trailing `#` in some notation) means the signal is **active low**: its asserted state is logic 0.

---

## 16. Address Bus, Data Bus, and Control Signals

### 16.1 Address bus

The 8085 has a 16-bit address path:

- `A15–A8`: high-order address lines.
- `AD7–AD0`: multiplexed low-order address/data lines.

Together they identify a 16-bit memory address.

### 16.2 Data bus

The data path is 8 bits wide:

- Data is transferred on `AD7–AD0` during the data portion of the bus cycle.
- The same pins carry the low-order address earlier in the cycle.
- Fetch program and data

### 16.3 Why multiplex address and data?

Multiplexing reduces the number of package pins. The trade-off is that external hardware must latch the low-order address using ALE before those pins are reused for data.

### 16.4 Memory read, conceptually

1. CPU places the address on address pins.
2. ALE indicates the low-order address is valid on AD0–AD7.
3. External latch holds the low-order address.
4. CPU asserts RD̅.
5. Memory places the selected byte on the data bus.
6. CPU reads the data.

### 16.5 Memory write, conceptually

1. CPU presents the target address.
2. ALE allows the low-order address to be latched.
3. CPU places the data byte on AD0–AD7.
4. CPU asserts WR̅.
5. Memory stores the byte at the selected address.

### 16.6 Memory versus I/O cycles

The 8085 has separate I/O instructions (`IN` and `OUT`) and a distinct I/O addressing mode. Direct `IN`/`OUT` instructions use an 8-bit port address. Memory-mapped I/O is also possible, where peripheral registers occupy locations in the memory address space.

---

## 17. 8085 Pin Diagram: Pin Functions

The standard 8085 is commonly supplied in a 40-pin package. Pin names and active-low conventions should be checked against the specific manufacturer's datasheet when designing hardware.

### 17.1 Address and multiplexed data pins

| Pin group | Function                                                                              |
| --------- | ------------------------------------------------------------------------------------- |
| A15–A8   | High-order 8 address bits                                                             |
| AD7–AD0  | Low-order address bits during the address phase; 8-bit data bus during the data phase |
| ALE       | Pulses to allow the low-order address on AD7–AD0 to be latched externally            |

### 17.2 Read/write and status signals

| Signal | Direction / nature | Function                                                    |
| ------ | ------------------ | ----------------------------------------------------------- |
| RD̅   | Output, active low | Enables a memory or I/O device to place data on the bus     |
| WR̅   | Output, active low | Commands a memory or I/O device to accept data from the bus |
| IO/M   | Output             | Helps distinguish I/O and memory machine cycles             |
| S1, S0 | Outputs            | Machine-cycle status signals                                |
| INTA̅ | Output, active low | Acknowledges an INTR interrupt request                      |

`IO/M`, `S1`, and `S0` are interpreted together for detailed machine-cycle identification. Do not infer every cycle from a single status pin in isolation.

### 17.3 Interrupt pins

| Pin     | Type   | Function                                                                         |
| ------- | ------ | -------------------------------------------------------------------------------- |
| TRAP    | Input  | Highest-priority, non-maskable, vectored interrupt; designed for critical events |
| RST 7.5 | Input  | Maskable, vectored interrupt; edge-triggered and latched                         |
| RST 6.5 | Input  | Maskable, vectored interrupt; level-sensitive                                    |
| RST 5.5 | Input  | Maskable, vectored interrupt; level-sensitive                                    |
| INTR    | Input  | General maskable interrupt request; non-vectored                                 |
| INTA̅  | Output | Interrupt acknowledgement for INTR                                               |

### 17.4 Clock pins

| Pin     | Function                                                                                                                                                                          |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| X1, X2  | Connect to a crystal or external clock network, according to the device circuit                                                                                                   |
| CLK OUT | Clock output for synchronizing external devices; its frequency relationship to the internal clock is device-specific and commonly taught as half the internal operating frequency |

### 17.5 Reset pins

| Signal     | Function                                                                                            |
| ---------- | --------------------------------------------------------------------------------------------------- |
| RESET IN̅ | Active-low reset input; resets the processor's control state, including setting PC to `0000H`     |
| RESET OUT  | Indicates reset status for external system components; consult the datasheet for exact timing/usage |

On reset, the PC is cleared to `0000H`; interrupt enable is disabled and the serial output state is reset according to standard 8085 descriptions. Do not assume all external memory or peripheral devices are automatically reset unless the hardware connects them to the reset circuitry.

### 17.6 DMA/bus control pins

| Signal | Function                                                                      |
| ------ | ----------------------------------------------------------------------------- |
| READY  | Allows the CPU to insert wait states when a memory or I/O device is not ready |
| HOLD   | External device requests control of the buses, commonly for DMA               |
| HLDA   | Hold Acknowledge; CPU indicates that it has granted the bus request           |

HOLD/HLDA are used to coordinate bus ownership. The exact transfer sequence should be understood from the hardware timing diagram.

### 17.7 Serial pins

| Signal | Function                                                        |
| ------ | --------------------------------------------------------------- |
| SID    | Serial Input Data; one-bit serial input sampled by `RIM`      |
| SOD    | Serial Output Data; one-bit serial output controlled by `SIM` |

### 17.8 Power pins

- `VCC`: +5 V supply for standard 8085.
- `VSS`/GND: ground reference.

Pin names, bars, and package pin numbers should be verified using the datasheet for the exact 8085 variant used.

---

## 18. Instruction Fetch, Decode, and Execute

The 8085 executes instructions through a sequence of machine cycles, each composed of T-states. The precise sequence depends on the instruction.

### Generic flow

1. **Fetch opcode** from the memory location indicated by PC.
2. **Decode** the opcode.
3. **Fetch additional bytes**, if the instruction requires immediate data or an address.
4. **Fetch operand(s)** from registers, memory, or I/O as needed.
5. **Execute** the operation.
6. **Write back** the result and update flags where applicable.
7. **Update control flow**, including any branch/call/return target.
8. **Check eligible interrupts** at the appropriate instruction boundary.

### Example 1: register transfer

```asm
MOV A, B
```

- One-byte instruction.
- Copies B into A.
- B is unchanged.
- Flags are unchanged.
- PC advances past the opcode.

### Example 2: immediate load

```asm
MVI C, 45H
```

- Two-byte instruction.
- Opcode is fetched, then immediate byte `45H`.
- C becomes `45H`.
- Flags are unchanged.

### Example 3: direct memory load

```asm
LDA 2050H
```

- Three-byte instruction.
- The address is encoded low byte first (`50H`, then `20H`).
- The CPU reads memory at `2050H` and loads that byte into A.
- Flags are unchanged.

### Example 4: conditional branch

```asm
JNZ 2500H
```

- If Z = 0, PC is loaded with `2500H`.
- If Z = 1, execution continues sequentially.
- The instruction itself does not calculate a new arithmetic result and does not change the condition flags.

---

## 19. Machine Cycles and T-States

### T-state

A **T-state** is one clock period in the processor's timing sequence. It is the basic unit used to describe instruction timing.

### Machine cycle

A **machine cycle** is a group of T-states used for a basic bus operation, such as:

- Opcode fetch
- Memory read
- Memory write
- I/O read
- I/O write
- Interrupt acknowledge

An instruction can consist of one or more machine cycles.

### Instruction cycle

An **instruction cycle** is the total sequence required to complete one instruction. It may contain multiple machine cycles.

### Example timing concepts

- A register-to-register instruction generally needs fewer machine cycles than a direct memory instruction.
- A direct 16-bit address instruction needs extra byte fetches.
- READY can insert wait states for slower devices.
- Interrupt handling and stack operations require additional bus activity.

Exact T-state counts depend on the instruction and sometimes on whether a conditional branch is taken. Use the relevant instruction timing table or datasheet for exact cycle counts; do not infer them solely from the number of source-code operands.

---

## 20. Stack, CALL, RET, PUSH, and POP

### 20.1 Stack principle

A stack follows **Last In, First Out (LIFO)**: the most recently saved item is retrieved first.

The 8085 stack is stored in ordinary memory. SP tracks the stack top and normally moves toward lower addresses during pushes.

### 20.2 `PUSH rp`

`PUSH` saves a 16-bit register pair on the stack. Supported operands are BC, DE, HL, and PSW.

Conceptually, two bytes are written to the stack and SP is decremented for the stack growth direction. The precise byte order is important when examining memory dumps; the 8085 convention leaves the high-order byte at the lower address after the push sequence and the low-order byte at the next higher address, when viewed from the final SP. This matches the pair's later restoration by `POP`.

### 20.3 `POP rp`

`POP` restores a pair from the stack and increments SP to remove the saved 16-bit value from the stack.

`POP PSW` restores both A and the flag register. Since the flag bits are restored from memory, flags can change as a result of `POP PSW`.

### 20.4 `CALL address`

`CALL` transfers control to a subroutine and saves the return address on the stack. The return address is the address of the instruction byte that would have been fetched next after the CALL instruction.

Conceptually:

1. Determine the return address.
2. Save it on the stack.
3. Load PC with the subroutine target address.

### 20.5 `RET`

`RET` retrieves the saved return address from the stack and loads it into PC.

### 20.6 Stack safety

- Initialize SP before using `CALL`, `RET`, `PUSH`, `POP`, or interrupts.
- Reserve enough stack memory for nested calls and interrupt service routines.
- Avoid overlapping the stack with code, data, or memory-mapped peripherals.
- An incorrect SP can cause apparently unrelated program corruption.

---

## 21. Interrupt System

An interrupt allows an external or internal event to request CPU service outside the normal sequential flow.

### 21.1 8085 interrupt inputs

The standard interrupt hierarchy is commonly taught as:

1. **TRAP** — highest priority
2. **RST 7.5**
3. **RST 6.5**
4. **RST 5.5**
5. **INTR** — lowest priority

TRAP is non-maskable in normal 8085 interrupt control. The RST interrupts and INTR are maskable in the standard interrupt system.

### 21.2 Vectored versus non-vectored

- **Vectored interrupt:** hardware associates the interrupt with a fixed service address.
- **Non-vectored interrupt:** the external hardware must supply or arrange the instruction/address information needed to redirect execution.

The RST interrupts have fixed vector addresses:

| Interrupt | Vector address |
| --------- | -------------: |
| TRAP      |      `0024H` |
| RST 5.5   |      `002CH` |
| RST 6.5   |      `0034H` |
| RST 7.5   |      `003CH` |

INTR does not have one fixed vector address in the same way; it is acknowledged by INTA̅ and normally requires external hardware to supply an instruction, often an RST opcode.

### 21.3 Masking and enabling

The 8085 provides instructions for interrupt control:

- `EI` — enable maskable interrupts, with enable taking effect after the instruction following EI.
- `DI` — disable maskable interrupts.
- `SIM` — Set Interrupt Mask; can set masks for RST 7.5/6.5/5.5, reset the RST 7.5 latch, and control serial output.
- `RIM` — Read Interrupt Mask/status; reads mask and pending status information and samples SID.

TRAP is not masked by the normal SIM masks. The exact mask/status bit layout should be learned from the 8085 SIM/RIM format.

### 21.4 Edge and level sensitivity

- RST 7.5 is edge-triggered and internally latched.
- RST 6.5 and RST 5.5 are level-sensitive.
- TRAP has both edge- and level-sensitive qualification in the standard 8085 description.
- INTR is level-sensitive.

This distinction affects whether a short pulse can be remembered and how long a request must remain asserted.

### 21.5 Interrupt service sequence

At a permitted instruction boundary, if a valid enabled interrupt is accepted:

1. The CPU recognizes the request according to priority and mask state.
2. The return address is saved on the stack.
3. The CPU transfers control to the appropriate vector or follows the INTR acknowledge sequence.
4. The interrupt service routine handles the event.
5. `RET` returns to the interrupted program, assuming the service routine preserves the stack correctly.

An interrupt service routine should save and restore any registers it changes if the interrupted program expects those values to remain intact.

---

## 22. Serial I/O and SID/SOD

The 8085 provides simple one-bit serial input and output pins.

### SID — Serial Input Data

- Input pin for serial data.
- The `RIM` instruction can read the SID state into the accumulator's designated bit position as specified by the RIM format.
- It is a simple serial interface, not a full UART.

### SOD — Serial Output Data

- Output pin for serial data.
- `SIM` can set or reset the SOD output through its serial-output control bits.
- It does not automatically provide a complete asynchronous serial protocol with start bits, stop bits, baud-rate generation, and buffering.

### SIM and RIM

These instructions are special control/status instructions:

- `SIM`: sets interrupt masks and can control serial output and the RST 7.5 latch.
- `RIM`: reads interrupt masks, pending status, and SID into A.

For exams and hardware programming, memorize the exact bit layouts of the accumulator input/output formats for SIM and RIM from the course reference or datasheet.

---

## 23. Reset and Startup Behavior

Reset initializes the processor's control state so execution can begin from a known point.

Standard 8085 behavior includes:

- PC is set to `0000H`.
- Maskable interrupts are disabled.
- The serial output state is reset according to the processor's reset behavior.
- The processor begins fetching instructions from address `0000H` after reset is released and timing conditions permit.

Reset does not automatically initialize every general-purpose register or every RAM location to a known value. Software should initialize registers, SP, and any required memory before relying on their contents.

**Practical rule:** The first startup code should establish a safe stack pointer and initialize required state before using subroutines or interrupts.

---

## 24. Memory and I/O Addressing

### 24.1 Memory space

With 16 address bits:

```text
2^16 = 65,536 bytes = 64 KiB
```

The full address range is:

```text
0000H through FFFFH
```

The processor does not inherently know which addresses are RAM, ROM, or device registers. External decoding and hardware determine the actual map.

### 24.2 Immediate addressing

The operand is included in the instruction bytes.

```asm
MVI A, 2AH
```

The data byte `2AH` follows the opcode.

### 24.3 Register addressing

The operand is in a CPU register.

```asm
MOV A, B
ADD C
```

### 24.4 Register-indirect addressing

A register pair contains the memory address.

```asm
MOV A, M
```

Here M means memory at address HL.

`LDAX B` and `LDAX D` load A from the memory address in BC or DE, respectively.

### 24.5 Direct addressing

The instruction includes the 16-bit memory address.

```asm
LDA 2050H
STA 2051H
LHLD 2050H
SHLD 2050H
```

### 24.6 I/O addressing

```asm
IN 80H
OUT 81H
```

`IN` reads from an 8-bit I/O port into A. `OUT` writes A to an 8-bit I/O port.

### 24.7 Memory-mapped I/O

A peripheral can also be assigned an address within the memory address space. It is then accessed using memory instructions, provided external hardware decodes the address and implements the required bus behavior.

---

## 25. Register Effects of Important Instructions

This section summarizes the usual register and flag effects. It is a reference, not a substitute for the complete instruction-set table.

### 25.1 Data transfer

| Instruction       | Effect                                             | Flags     |
| ----------------- | -------------------------------------------------- | --------- |
| `MOV r1,r2`     | Copies source register to destination              | Unchanged |
| `MOV r,M`       | Loads r from memory at HL                          | Unchanged |
| `MOV M,r`       | Stores r at memory address HL                      | Unchanged |
| `MVI r,data`    | Loads immediate byte into r                        | Unchanged |
| `MVI M,data`    | Stores immediate byte at memory[HL]                | Unchanged |
| `LXI rp,data16` | Loads 16-bit pair or SP                            | Unchanged |
| `LDA addr`      | A ← memory[addr]                                  | Unchanged |
| `STA addr`      | memory[addr] ← A                                  | Unchanged |
| `LHLD addr`     | L ← memory[addr], H ← memory[addr+1]             | Unchanged |
| `SHLD addr`     | memory[addr] ← L, memory[addr+1] ← H             | Unchanged |
| `LDAX B/D`      | A ← memory[BC/DE]                                 | Unchanged |
| `STAX B/D`      | memory[BC/DE] ← A                                 | Unchanged |
| `XCHG`          | Exchange DE and HL                                 | Unchanged |
| `XTHL`          | Exchange HL with the two bytes at the top of stack | Unchanged |
| `SPHL`          | SP ← HL                                           | Unchanged |
| `PCHL`          | PC ← HL                                           | Unchanged |

### 25.2 Arithmetic

| Instruction  | Effect                                | Important flag behavior                    |
| ------------ | ------------------------------------- | ------------------------------------------ |
| `ADD r/M`  | A ← A + operand                      | Updates S, Z, AC, P, CY                    |
| `ADC r/M`  | A ← A + operand + CY                 | Updates arithmetic flags                   |
| `ADI data` | A ← A + immediate                    | Updates arithmetic flags                   |
| `ACI data` | A ← A + immediate + CY               | Updates arithmetic flags                   |
| `SUB r/M`  | A ← A − operand                     | Updates arithmetic flags                   |
| `SBB r/M`  | A ← A − operand − CY               | Updates arithmetic flags                   |
| `SUI data` | A ← A − immediate                   | Updates arithmetic flags                   |
| `SBI data` | A ← A − immediate − CY             | Updates arithmetic flags                   |
| `INR r/M`  | Operand ← operand + 1                | Updates S, Z, AC, P; CY unchanged          |
| `DCR r/M`  | Operand ← operand − 1               | Updates S, Z, AC, P; CY unchanged          |
| `INX rp`   | Pair/SP ← pair/SP + 1                | Unchanged                                  |
| `DCX rp`   | Pair/SP ← pair/SP − 1               | Unchanged                                  |
| `DAD rp`   | HL ← HL + pair/SP                    | CY only                                    |
| `DAA`      | Decimal-adjust A after BCD addition   | Updates flags according to adjusted result |
| `CMP r/M`  | Compare A with operand; A unchanged   | Flags as subtraction                       |
| `CPI data` | Compare A with immediate; A unchanged | Flags as subtraction                       |

### 25.3 Logic and rotate

| Instruction  | Effect                                      | Important flag behavior                            |
| ------------ | ------------------------------------------- | -------------------------------------------------- |
| `ANA r/M`  | A ← A AND operand                          | S, Z, P reflect result; CY cleared; AC set in 8085 |
| `ANI data` | A ← A AND immediate                        | Same general behavior as ANA                       |
| `XRA r/M`  | A ← A XOR operand                          | S, Z, P reflect result; CY and AC cleared          |
| `XRI data` | A ← A XOR immediate                        | Same general behavior as XRA                       |
| `ORA r/M`  | A ← A OR operand                           | S, Z, P reflect result; CY and AC cleared          |
| `ORI data` | A ← A OR immediate                         | Same general behavior as ORA                       |
| `CMA`      | A ← bitwise complement of A                | Flags unchanged                                    |
| `RLC`      | Rotate A left; bit 7 wraps to bit 0 and CY  | CY updated                                         |
| `RRC`      | Rotate A right; bit 0 wraps to bit 7 and CY | CY updated                                         |
| `RAL`      | Rotate A left through CY                    | CY updated                                         |
| `RAR`      | Rotate A right through CY                   | CY updated                                         |
| `STC`      | CY ← 1                                     | Only CY intentionally set                          |
| `CMC`      | CY ← complement of CY                      | Only CY intentionally complemented                 |

### 25.4 Control flow, stack, and I/O

| Instruction     | Effect                                                          | Flags                               |
| --------------- | --------------------------------------------------------------- | ----------------------------------- |
| `JMP addr`    | PC ← address                                                   | Unchanged                           |
| `Jcc addr`    | Conditional PC change                                           | Unchanged                           |
| `CALL addr`   | Save return address; PC ← address                              | Unchanged                           |
| `Ccc addr`    | Conditional call                                                | Unchanged                           |
| `RET`         | PC ← saved return address                                      | Unchanged                           |
| `Rcc`         | Conditional return                                              | Unchanged                           |
| `RST n`       | Call fixed restart vector                                       | Unchanged                           |
| `PUSH rp/PSW` | Save 16 bits on stack                                           | Unchanged                           |
| `POP rp`      | Restore 16-bit pair                                             | Unchanged                           |
| `POP PSW`     | Restore A and flags                                             | Flags are restored                  |
| `IN port`     | A ← input port data                                            | Unchanged                           |
| `OUT port`    | Output A to port                                                | Unchanged                           |
| `EI`          | Enable maskable interrupts after the defined delay              | Not an ALU flag operation           |
| `DI`          | Disable maskable interrupts                                     | Not an ALU flag operation           |
| `SIM`         | Set interrupt masks/serial output controls                      | Not a normal arithmetic operation   |
| `RIM`         | Read interrupt status and SID into A                            | Does not calculate arithmetic flags |
| `HLT`         | Halt normal instruction execution until an accepted event/reset | Flags unchanged                     |

**Notation:** `r` means a supported 8-bit register; `M` means memory at address HL; `rp` means an allowed register pair. Conditional mnemonics include conditions such as Z, NZ, C, NC, P, M, PE, and PO.

---

## 26. Common Misconceptions and Subtle Details

1. **“8085 has a 16-bit ALU.”**Incorrect in the usual architectural description. Its main ALU is 8-bit. Certain 16-bit operations are supported through multi-step internal operations.
2. **“HL is a separate 16-bit register independent of H and L.”**HL is the combined interpretation of H (high byte) and L (low byte).
3. **“M is a register.”**M is assembly notation for memory at the address stored in HL.
4. **“PC counts instructions.”**PC counts byte addresses. Instructions can occupy one, two, or three bytes.
5. **“SP points to a dedicated stack chip.”**The stack is held in ordinary memory; SP points into it.
6. **“All instructions update flags.”**False. Many data-transfer and control-flow instructions leave flags unchanged.
7. **“INR and DCR change carry.”**They do not change CY. This is one of the most frequently tested details.
8. **“DAD updates all flags.”**Standard 8085 behavior: DAD updates CY only.
9. **“CMP stores the difference in A.”**It does not. It updates flags as a subtraction would, while A remains unchanged.
10. **“Parity means the number itself is even.”**No. Parity counts the number of set bits in the result.
11. **“CY and AC are the same.”**CY relates to carry/borrow at the 8-bit boundary; AC relates to the low-nibble boundary.
12. **“All interrupt requests behave identically.”**They differ in priority, masking, vectoring, and edge/level sensitivity.
13. **“INTR is vectored like RST 7.5.”**INTR is non-vectored; the external interrupting hardware must provide the required instruction/addressing mechanism.
14. **“Reset initializes all registers and RAM to zero.”**Reset establishes defined control state, including PC = `0000H`; it does not generally clear all RAM or guarantee all general-purpose register contents.
15. **“A 16-bit register pair has a different capacity from its two bytes.”**It represents the same 16 bits. For example, `BC = 1234H` means B = `12H` and C = `34H`.
16. **“The 8085 can address 64 KB of RAM in every system.”**It has a 64 KiB memory address space. Actual installed RAM may be smaller, shared with ROM, or partly assigned to I/O devices.
17. **“Every address on AD0–AD7 remains present throughout a bus cycle.”**No. These pins are multiplexed: low address first, data later. External latching is required.
18. **“A conditional jump always takes the same time.”**Timing can depend on whether the condition is satisfied. Check the timing table for the exact instruction.
19. **“PSW is an ordinary 16-bit register pair.”**PSW is a special stack-transfer combination of A and the flag register, supported by `PUSH PSW` and `POP PSW`.
20. **“Internal W/Z registers can be used in assembly.”**
    They are internal temporary storage in common architecture diagrams, not ordinary programmer-visible registers.

---

## 27. Quick Reference Tables

### 27.1 Register widths

| Register                    |                       Width | Address/data/status role                   |
| --------------------------- | --------------------------: | ------------------------------------------ |
| A                           |                      8 bits | Accumulator                                |
| B, C, D, E, H, L            |                 8 bits each | General-purpose registers                  |
| F                           |                      8 bits | Flag register; five active condition flags |
| BC, DE, HL                  |       16 bits each as pairs | Data/address values                        |
| PC                          |                     16 bits | Instruction-byte address                   |
| SP                          |                     16 bits | Stack address                              |
| IR                          |                    Internal | Holds current opcode during decoding       |
| W, Z                        | Internal in common diagrams | Temporary storage; not directly accessible |
| Temporary registers/latches |                    Internal | Intermediate data/address storage          |

### 27.2 Main register pairs and related instructions

| Pair / register | Load        | Increment  | Decrement  | Add to HL  | Push/Pop                                |
| --------------- | ----------- | ---------- | ---------- | ---------- | --------------------------------------- |
| BC              | `LXI B`   | `INX B`  | `DCX B`  | `DAD B`  | `PUSH B`, `POP B`                   |
| DE              | `LXI D`   | `INX D`  | `DCX D`  | `DAD D`  | `PUSH D`, `POP D`                   |
| HL              | `LXI H`   | `INX H`  | `DCX H`  | `DAD H`  | `PUSH H`, `POP H`                   |
| SP              | `LXI SP`  | `INX SP` | `DCX SP` | `DAD SP` | Not supported as `PUSH SP`/`POP SP` |
| PSW             | Not via LXI | —         | —         | —         | `PUSH PSW`, `POP PSW`               |

### 27.3 Flag summary

| Flag | Meaning when set                     | Typical use                                 |
| ---- | ------------------------------------ | ------------------------------------------- |
| S    | Result bit 7 = 1                     | Sign-oriented conditional branching         |
| Z    | Result is zero                       | Equality/zero checks                        |
| AC   | Carry/borrow across bit 3/4 boundary | BCD adjustment support                      |
| P    | Even number of 1-bits in result      | Parity checks                               |
| CY   | Carry out or subtraction borrow      | Unsigned arithmetic and carry/borrow chains |

### 27.4 Conditional branch conditions

| Condition | Meaning                          | Condition |
| --------- | -------------------------------- | --------- |
| NZ        | Not zero                         | Z = 0     |
| Z         | Zero                             | Z = 1     |
| NC        | No carry                         | CY = 0    |
| C         | Carry                            | CY = 1    |
| PO        | Parity odd                       | P = 0     |
| PE        | Parity even                      | P = 1     |
| P         | Plus / non-negative bit-7 result | S = 0     |
| M         | Minus / bit 7 set                | S = 1     |

`P` in the condition mnemonic means “plus,” while `P` in the flag register means “parity.” Context distinguishes them.

### 27.5 Important fixed addresses

| Purpose                |   Address |
| ---------------------- | --------: |
| Reset start address    | `0000H` |
| TRAP vector            | `0024H` |
| RST 5.5 vector         | `002CH` |
| RST 6.5 vector         | `0034H` |
| RST 7.5 vector         | `003CH` |
| Highest 16-bit address | `FFFFH` |

### 27.6 Memory-size conversions

| Quantity              |                Value |
| --------------------- | -------------------: |
| 1 bit                 |     One binary digit |
| 1 byte                |               8 bits |
| 1 KiB                 |          1,024 bytes |
| 1 KiB in address bits |              10 bits |
| 64 KiB                |         65,536 bytes |
| 64 KiB address range  | `0000H`–`FFFFH` |

---

## 28. Worked Examples

### Example 1: Register-pair construction

Given:

```asm
MVI H, 2AH
MVI L, 80H
```

Result:

- H = `2AH`
- L = `80H`
- HL = `2A80H`

If the next instruction is `MOV A,M`, A receives the byte stored at memory address `2A80H`.

### Example 2: Carry versus zero

```asm
MVI A, FFH
ADI 01H
```

Calculation:

```text
FFH + 01H = 100H
```

Only 8 bits are stored:

- A = `00H`
- Z = 1
- CY = 1
- P = 1 because `00H` contains zero set bits (even parity)
- S = 0
- AC = 1 because `FH + 1` carries from bit 3 to bit 4

This example shows that zero result and carry can both be true at the same time.

### Example 3: `INR` preserves carry

Assume CY = 1 before the instructions:

```asm
MVI A, 0FH
INR A
```

After `INR A`:

- A = `10H`
- AC = 1 because of the low-nibble carry
- CY remains 1 because INR does not alter carry
- Other updated flags reflect `10H`

### Example 4: Compare without destroying A

```asm
MVI A, 25H
CPI 30H
```

The CPU compares `25H` against `30H`:

- A remains `25H`.
- Since `25H` is less than `30H` as an unsigned value, CY = 1.
- Z = 0 because they are not equal.

### Example 5: Memory pointer and increment

```asm
LXI H, 3000H
MVI M, 55H
INX H
MVI M, AAH
```

After execution:

- memory[`3000H`] = `55H`
- HL = `3001H`
- memory[`3001H`] = `AAH`

The `INX H` instruction changes HL, not the memory byte at the old address, and it does not change flags.

### Example 6: 16-bit addition

```asm
LXI H, FFF0H
LXI B, 0020H
DAD B
```

Mathematical sum:

```text
FFF0H + 0020H = 10010H
```

HL retains the low 16 bits:

- HL = `0010H`
- CY = 1 because the sum exceeded `FFFFH`
- S, Z, P, and AC are not updated by DAD.

### Example 7: Direct memory addressing and byte order

```asm
LDA 2050H
```

The instruction contains a 16-bit address, encoded low byte first:

- Address low byte = `50H`
- Address high byte = `20H`
- CPU reads memory at `2050H`
- A receives the byte stored there
- Flags remain unchanged

---

## 29. Final Summary

The 8085 architecture becomes easier to understand when its components are grouped by purpose.

### Data processing

- **A** is the main accumulator.
- **B, C, D, E, H, L** are six 8-bit general-purpose registers.
- **BC, DE, HL** are 16-bit interpretations of register pairs.
- The **ALU** performs arithmetic and logic.
- The **flag register** records selected outcomes of operations.

### Program control

- **PC** identifies the next instruction byte.
- **SP** manages the stack in memory.
- The instruction register and decoder interpret opcodes.
- Jumps, calls, returns, resets, and interrupts alter the normal PC sequence.

### External communication

- The address path is 16 bits, allowing 64 KiB of memory address space.
- The data path is 8 bits.
- AD0–AD7 are multiplexed between low address and data.
- ALE, RD̅, WR̅, IO/M, and status pins coordinate bus transactions.
- READY supports wait states; HOLD/HLDA coordinate bus ownership.
- Interrupt pins and SID/SOD provide event and serial-I/O interfaces.

### Details worth memorizing

1. A, B, C, D, E, H, and L are 8-bit registers.
2. PC and SP are 16-bit registers.
3. HL is used by the `M` memory operand.
4. `INR`/`DCR` do not modify CY.
5. `DAD` modifies CY only.
6. `CMP` changes flags but leaves A unchanged.
7. P = 1 means even bit parity.
8. `LXI`, `MOV`, and most data-transfer instructions leave flags unchanged.
9. The stack grows toward lower addresses.
10. The 8085 has 64 KiB of memory address space and 256 directly addressed I/O ports.
11. TRAP has the highest interrupt priority; INTR is non-vectored.
12. Reset sets PC to `0000H`; it does not clear all RAM.
13. The opcode is followed by immediate bytes; 16-bit addresses are encoded low byte first.
14. `M` means memory at address HL, not a physical register.
15. W/Z and the instruction register are internal implementation storage, not ordinary assembly-visible registers.

**Core idea:** The 8085 is an 8-bit CPU with a 16-bit address system. Registers store values and addresses, the ALU transforms data, flags summarize selected results, the PC controls instruction sequencing, the SP manages subroutine/interrupt storage, and the control unit coordinates communication with memory and peripherals.

---

## 30. Glossary

| Term                   | Meaning                                                                      |
| ---------------------- | ---------------------------------------------------------------------------- |
| ALU                    | Arithmetic and Logic Unit                                                    |
| Accumulator            | Main 8-bit register used by many ALU operations                              |
| Opcode                 | Byte identifying the instruction operation                                   |
| Operand                | Data or address used by an instruction                                       |
| PC                     | Program Counter                                                              |
| SP                     | Stack Pointer                                                                |
| PSW                    | Program Status Word: accumulator and flags for stack operations              |
| Flag                   | Status bit describing a property of an operation's result                    |
| Register pair          | Two 8-bit registers interpreted as one 16-bit value                          |
| `M`                  | Memory byte at address HL                                                    |
| Bus                    | Shared set of signal lines for data, addresses, or control                   |
| Multiplexing           | Using the same pins for different signals at different times                 |
| ALE                    | Address Latch Enable                                                         |
| Machine cycle          | Timing sequence for a basic bus operation                                    |
| T-state                | One clock period in a processor timing sequence                              |
| Interrupt vector       | Fixed address where an interrupt service routine begins                      |
| Maskable interrupt     | Interrupt that can be disabled by interrupt control                          |
| Non-vectored interrupt | Interrupt whose service destination is not fixed solely by a built-in vector |
| LIFO                   | Last In, First Out                                                           |
| BCD                    | Binary-Coded Decimal                                                         |
| Little-endian          | Multi-byte value stored least-significant byte first                         |

---

**Study tip:** Learn the register widths and roles first, then the five flags, then the effect of each instruction group on registers and flags. Finally, practice tracing PC, SP, HL, and the flags instruction by instruction.
