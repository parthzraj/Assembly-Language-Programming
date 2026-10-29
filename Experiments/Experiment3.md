
# 8085 Microprocessor Lab: Block Data Operations — Complete Exam Guide

Your uploaded experiment covers five operations on the Intel 8085 microprocessor: Block Move, Block Compare, Block Exchange, Block Insert, and Block Delete.&#x20;

Experiment-3-8085_Block_Operations_Lab.pptx

The goal is not just to memorize five programs. It is to understand the common logic behind them so that, in your lab exam, you can construct a program even when the examiner changes the memory addresses, block size, or insertion/deletion position.

We'll build this from the fundamentals to the actual GNUSim8085 code, flag behavior, loops, debugging, and exam strategy.

## 1. The basic idea: what is a block operation?

A block is simply a group of consecutive bytes stored in memory.

For example, suppose memory contains this data:

Source block beginning at 2100H

2100H

### 11H

2101H

### 22H

2102H

### 33H

2103H

### 44H

2104H

### 55H

Five consecutive memory locations, each storing one 8-bit value.

Each memory location stores one byte, or 8 bits. Since an 8085 has a 16-bit address bus, it can address \\(2^{16}=65,536\\) memory locations, from `0000H` through `FFFFH`, assuming the full address space is available.

A block operation processes these bytes one at a time.

| Operation | What it does                                   |
| --------- | ---------------------------------------------- |
| Move      | Copies bytes to another memory region          |
| Compare   | Checks whether two blocks contain equal bytes  |
| Exchange  | Swaps corresponding bytes between blocks       |
| Insert    | Shifts bytes right to make room for a new byte |
| Delete    | Shifts bytes left to close a gap               |

The common pattern is:

Set up pointers → set up count → process a byte → update pointers → update count → repeat or stop.

Once you understand this pattern, most block-processing programs become variations of the same template.

## 2. Understand the registers before writing code

The 8085 has several registers, but these are the important ones for this experiment.

## A

8-bit

Accumulator

Holds the byte being processed. Memory data is often loaded into A before being stored somewhere else or compared.

## HL

16-bit pair

Source pointer (common convention)

H and L combine to form a 16-bit address. `LXI H,2100H` makes HL point to `2100H`.

## DE

16-bit pair

Destination pointer (common convention)

D and E combine to form a 16-bit address. `LXI D,2200H` makes DE point to `2200H`.

## B / C

8-bit each

Counter and temporary storage

C is often used as the loop counter. B can temporarily save a byte or hold the final comparison result.

## F

Flags

Condition flags

Record results from certain arithmetic, logical, and comparison instructions. Conditional jumps use these flags to decide where execution goes next.

### 2.1 How HL and DE actually work

The 8085 does not have a dedicated instruction such as `MOV A,[2100H]` for register-indirect access.

Instead, it uses the special symbol `M`, meaning the memory location whose address is held in HL.

Suppose:

```
LXI H,2100H
MOV A,M
```

Execution:

1. `LXI H,2100H` loads the address `2100H` into HL.
2. `MOV A,M` reads the byte stored at memory address `2100H` into A.

If `[2100H] = 3AH`, then after `MOV A,M`, `A = 3AH`.

Likewise:

```
LXI D,2200H
STAX D
```

`STAX D` stores the contents of A at the memory address held in DE. If `A = 3AH`, then `[2200H] = 3AH`.

Remember the distinction:

- `HL` contains an address.
- `M` means the byte stored at that address.
- `A` contains a byte, not a memory address.

This distinction is fundamental to nearly every program in your experiment.

### 2.2 The most important instructions

| Instruction     | Meaning                         |
| --------------- | ------------------------------- |
| `LXI H,2100H` | Load the 16-bit address into HL |
| `MVI C,05H`   | Load the count 5 into C         |
| `MOV A,M`     | Read `[HL]` into A            |
| `MOV M,A`     | Write A into `[HL]`           |
| `LDAX D`      | Read `[DE]` into A            |
| `STAX D`      | Write A into `[DE]`           |
| `INX H`       | HL = HL + 1                     |
| `DCX H`       | HL = HL − 1                    |
| `INR C`       | C = C + 1                       |
| `DCR C`       | C = C − 1                      |
| `CMP M`       | Compare A with `[HL]`         |
| `JZ label`    | Jump if the Zero flag is 1      |
| `JNZ label`   | Jump if the Zero flag is 0      |
| `JMP label`   | Jump unconditionally            |
| `HLT`         | Stop program execution          |

Notice that `INX H` and `INR C` are different:

- `INX H` changes a 16-bit register pair.
- `INR C` changes an 8-bit register.

Also, `INX H` does not change the condition flags, whereas `DCR C` changes the Zero, Sign, and Parity flags.

## 3. Flags: which ones change and why?

Flags are a common source of mistakes in assembly exams. The key is to know which instruction sets the flag and which instruction consumes it.

The 8085 has five condition flags:

| Flag | Name            | Meaning when set to 1                                 |
| ---- | --------------- | ----------------------------------------------------- |
| S    | Sign            | Result's most significant bit is 1                    |
| Z    | Zero            | Result is zero                                        |
| AC   | Auxiliary Carry | Carry from bit 3 to bit 4                             |
| P    | Parity          | Result has even parity                                |
| CY   | Carry           | Carry or borrow condition, depending on the operation |

For this experiment, Z and CY are especially important.

### 3.1 Zero flag (Z)

Z becomes 1 when an instruction that updates flags produces a zero result.

For example:

```
MVI C,01H
DCR C
JNZ LOOP
```

Execution:

| Instruction   | C   | Z         | What happens                 |
| ------------- | --- | --------- | ---------------------------- |
| `MVI C,01H` | 01H | Unchanged | Counter starts at 1          |
| `DCR C`     | 00H | 1         | Counter reaches zero         |
| `JNZ LOOP`  | 00H | 1         | No jump; execution continues |

If C had been `02H`, then `DCR C` would make C `01H`, and Z would become 0. `JNZ LOOP` would jump back to the loop.

The important point is that `JNZ` does not itself change Z. It reads the existing Zero flag.

### 3.2 Carry flag (CY) and the `CMP` instruction

`CMP` compares two values by internally performing a subtraction without saving the subtraction result.

For `CMP M`, the operation is conceptually:

\\[ A-[HL] \\]

A and the memory contents remain unchanged by the comparison, but the flags are updated.

Consider:

```
MVI A,20H
MVI B,30H
CMP B
```

The conceptual subtraction is:

\\[ 20H-30H \\]

Because A is smaller than B, CY becomes 1 and Z becomes 0.

| Condition                 | Z | CY |
| ------------------------- | - | -- |
| A equals operand          | 1 | 0  |
| A is greater than operand | 0 | 0  |
| A is less than operand    | 0 | 1  |

This is the normal unsigned comparison interpretation of `CMP`.

For the block-compare program, the most important test is:

```
CMP M
JNZ NOMATCH
```

If the bytes are equal, Z = 1 and the jump is not taken. If they differ, Z = 0 and the jump is taken.

### 3.3 Which instructions affect flags in this experiment?

This table is worth memorizing.

| Instruction                          | Flags affected           |
| ------------------------------------ | ------------------------ |
| `MOV`, `MVI`, `LXI`            | None                     |
| `LDAX`, `STAX`, `LDA`, `STA` | None                     |
| `INX`, `DCX`                     | None                     |
| `INR`, `DCR`                     | S, Z, P; CY is unchanged |
| `CMP`, `CPI`                     | S, Z, AC, P, CY          |
| `JZ`, `JNZ`, `JMP`             | None                     |
| `XCHG`                             | None                     |
| `HLT`                              | None                     |

Exam rule: when you see a conditional jump, identify the most recent instruction that changed the relevant flag.

For example:

```
CMP M
INX H
JNZ LOOP
```

This is valid because `INX H` does not affect flags. `JNZ` still uses the Z flag produced by `CMP M`.

But consider:

```
CMP M
DCR C
JNZ LOOP
```

Here, `DCR C` overwrites Z before `JNZ` checks it. The jump now depends on the counter, not the comparison. That may be intentional in a loop, but it is wrong if you intended to branch based on the comparison result.

This is one of the most useful debugging principles in 8085 programming.

## 4. Loops: a method you can apply to almost any program

A loop means repeating a group of instructions until a condition is met.

In 8085 assembly, a common loop is built from three parts:

1. Initialize a counter.
2. Perform one iteration and decrement the counter.
3. Use a conditional jump to repeat or stop.

The standard pattern is:

```
MVI C,05H

LOOP: ; process one item here
      DCR C
      JNZ LOOP

      HLT
```

This processes five items, provided the processing instructions are placed before `DCR C`.

### 4.1 How the loop executes

Suppose C begins at `05H`.

Initialize C = 05H

Process one byte

DCR C

JNZ LOOP

C becomes 04H, 03H, 02H, 01H, and finally 00H. The final `DCR C` sets Z = 1, so `JNZ` stops repeating.

### 4.2 The two pointer rules

Every block loop usually has one or two pointers. After processing one byte, decide where each pointer must go.

| Required movement                | Instruction                   |
| -------------------------------- | ----------------------------- |
| Move to next memory location     | `INX H` or `INX D`        |
| Move to previous memory location | `DCX H` or `DCX D`        |
| Keep pointer unchanged           | Do not increment/decrement it |

For example, copying a block from `2100H` to `2200H`:

- Source pointer HL moves forward.
- Destination pointer DE moves forward.
- Counter decreases.

For insertion, shifting right:

- HL moves backward.
- DE moves backward.

For deletion, shifting left:

- HL moves forward.
- DE moves forward.

Memory aid: the pointers must follow the direction in which the data is moving.

### 4.3 What you must know about counters

The 8085 registers B and C are 8-bit registers. Their values range from `00H` to `FFH`.

- `05H` means five iterations for the usual decrement-and-test loop.
- `01H` means one iteration.
- `00H` is dangerous as an initial count in this loop pattern: decrementing it wraps to `FFH`, so the loop can execute 256 times.

Therefore, if the block length might be zero, handle that case before entering a do-first loop.

For block sizes greater than 255 bytes, a single 8-bit register is insufficient as a straightforward counter. You need a larger-count technique, such as a register-pair counter or nested loops.

### 4.4 A general loop checklist

Before running any loop, answer these five questions:

1. Start: What is the initial counter value?
2. Work: What exactly happens during one iteration?
3. Move: Which pointers change, and in which direction?
4. Count: Which instruction changes the counter?
5. Exit: Which flag is tested, and which jump uses it?

If you can answer these five questions, you can construct and debug most basic 8085 loops.

## 5. Program 1 — Block Move

### Objective

Copy five bytes from the source block at `2100H` to the destination block at `2200H`.

For example:

| Source address | Initial value | Destination address | Expected value |
| -------------- | ------------- | ------------------- | -------------- |
| 2100H          | 11H           | 2200H               | 11H            |
| 2101H          | 22H           | 2201H               | 22H            |
| 2102H          | 33H           | 2202H               | 33H            |
| 2103H          | 44H           | 2203H               | 44H            |
| 2104H          | 55H           | 2204H               | 55H            |

### GNUSim8085 code

```

ORG 2000H

MVI C,05H
LXI H,2100H
LXI D,2200H

LOOP: MOV A,M
      STAX D
      INX H
      INX D
      DCR C
      JNZ LOOP

HLT

```

### Understand each instruction

| Instruction     | Purpose                         |
| --------------- | ------------------------------- |
| `MVI C,05H`   | Set the number of bytes to five |
| `LXI H,2100H` | Point HL at the source          |
| `LXI D,2200H` | Point DE at the destination     |
| `MOV A,M`     | Read source byte into A         |
| `STAX D`      | Write A to destination          |
| `INX H`       | Advance source pointer          |
| `INX D`       | Advance destination pointer     |
| `DCR C`       | Decrease remaining count        |
| `JNZ LOOP`    | Repeat if the count is nonzero  |
| `HLT`         | Finish                          |

### Trace the first iteration

Initially:

- HL = `2100H`
- DE = `2200H`
- C = `05H`

Then:

1. `MOV A,M` makes A equal to `[2100H]`, or `11H`.
2. `STAX D` writes `11H` to `[2200H]`.
3. `INX H` makes HL = `2101H`.
4. `INX D` makes DE = `2201H`.
5. `DCR C` makes C = `04H`.
6. `JNZ LOOP` jumps back because Z = 0.

The next iteration processes the second byte.

Exam point: `MOV A,M` and `STAX D` are used because the 8085 cannot directly copy an arbitrary byte from the memory location pointed to by HL to the one pointed to by DE with a single `MOV` instruction.

## 6. Program 2 — Block Compare

### Objective

Compare two blocks byte by byte. Store:

- `00H` at `2300H` if every corresponding byte matches.
- `FFH` at `2300H` if any byte differs.

### GNUSim8085 code

```

ORG 2000H

LXI H,2100H
LXI D,2200H
MVI C,05H
MVI B,00H

LOOP: LDAX D
      CMP M
      JNZ NOMATCH

      INX H
      INX D
      DCR C
      JNZ LOOP

      JMP STORE

NOMATCH: MVI B,0FFH

STORE: MOV A,B
       STA 2300H

HLT

```

### How the logic works

The two pointers begin at `2100H` and `2200H`.

```
LDAX D
CMP M
JNZ NOMATCH
```

- `LDAX D` loads the byte from `[DE]` into A.
- `CMP M` compares A with `[HL]`.
- `JNZ NOMATCH` branches to the mismatch handler if the bytes differ.

If the bytes match, the program advances both pointers, decreases C and checks whether more bytes remain.

If the entire loop finishes without finding a mismatch, B remains `00H`. If a mismatch is found, B becomes `FFH`.

Finally, `STA 2300H` stores the result.

### Example

Suppose:

| Address | Value |
| ------- | ----- |
| 2100H   | 11H   |
| 2200H   | 11H   |
| 2101H   | 22H   |
| 2201H   | 22H   |
| 2102H   | 33H   |
| 2202H   | 99H   |

The first two pairs match. The third pair does not. Execution jumps to `NOMATCH`, and the final result at `2300H` is `FFH`.

Flag concept: `CMP M` changes flags, particularly Z and CY. The immediate `JNZ` checks the Zero flag before any instruction can overwrite it.

## 7. Program 3 — Block Exchange

### Objective

Exchange the corresponding bytes of two memory blocks.

Before:

| Address | Value |
| ------- | ----- |
| 2100H   | 11H   |
| 2200H   | AAH   |

After:

| Address | Value |
| ------- | ----- |
| 2100H   | AAH   |
| 2200H   | 11H   |

### GNUSim8085 code

```

ORG 2000H

LXI H,2100H
LXI D,2200H
MVI C,05H

LOOP: LDAX D
      MOV B,A
      MOV A,M
      STAX D
      MOV A,B
      MOV M,A

      INX H
      INX D
      DCR C
      JNZ LOOP

HLT

```

### Why do we need register B?

The 8085 does not provide a general instruction that swaps the contents of two arbitrary memory locations.

So we use B as temporary storage.

Suppose `[2100H] = 11H` and `[2200H] = AAH`.

| Step | Instruction | A   | B   | `[2100H]` | `[2200H]` |
| ---- | ----------- | --- | --- | ----------- | ----------- |
| 1    | `LDAX D`  | AAH | —  | 11H         | AAH         |
| 2    | `MOV B,A` | AAH | AAH | 11H         | AAH         |
| 3    | `MOV A,M` | 11H | AAH | 11H         | AAH         |
| 4    | `STAX D`  | 11H | AAH | 11H         | 11H         |
| 5    | `MOV A,B` | AAH | AAH | 11H         | 11H         |
| 6    | `MOV M,A` | AAH | AAH | AAH         | 11H         |

The two bytes have been exchanged.

### What about `XCHG`?

`XCHG` swaps the contents of the HL and DE register pairs.

It does not exchange the bytes stored at `[HL]` and `[DE]`.

That distinction is important in viva questions. The instruction cannot directly replace the temporary-register sequence above.

## 8. Program 4 — Block Insert

### Objective

Insert `99H` at the third position in a five-byte block starting at `2100H`.

We count positions from 1 when describing the third position. In programming, its zero-based index is 2.

Before:

| Address | Value |
| ------- | ----- |
| 2100H   | 11H   |
| 2101H   | 22H   |
| 2102H   | 33H   |
| 2103H   | 44H   |
| 2104H   | 55H   |

After:

| Address | Value |
| ------- | ----- |
| 2100H   | 11H   |
| 2101H   | 22H   |
| 2102H   | 99H   |
| 2103H   | 33H   |
| 2104H   | 44H   |
| 2105H   | 55H   |

Important: there must be an available memory slot at `2105H`. The operation shifts data; it does not create additional memory.

### GNUSim8085 code

```

ORG 2000H

LXI H,2104H
LXI D,2105H
MVI C,03H

SHIFT: MOV A,M
       STAX D
       DCX H
       DCX D
       DCR C
       JNZ SHIFT

MVI A,99H
STAX D

HLT

```

### Why do we move backwards?

The last existing byte is at `2104H`, and the new slot is `2105H`.

First, we copy `[2104H]` to `[2105H]`. Then we copy `[2103H]` to `[2104H]`, and finally `[2102H]` to `[2103H]`.

Only after shifting do we write `99H` at `2102H`.

If we instead started at the beginning and shifted forward, we could overwrite a byte before copying its original value. That is why a right-shift normally starts at the end and proceeds backwards.

### Calculate the shift count

Let:

- \\(N\\) = original number of bytes
- \\(p\\) = zero-based insertion index

Then:

\\[ \text{Shift count}=N-p \\]

For five bytes and index 2:

\\[ 5-2=3 \\]

Therefore, `MVI C,03H`.

### A general insertion formula

For a block beginning at address \\(S\\):

- Last existing byte: \\(S+N-1\\)
- New empty slot: \\(S+N\\)
- Insertion address: \\(S+p\\)
- Number of bytes to shift: \\(N-p\\)

These formulas assume the insertion index is valid and the new slot is available.

If insertion is at the end, \\(p=N\\), the shift count is zero. In that special case, skip the shift loop and directly write the new byte at \\(S+N\\); the usual do-first loop cannot handle a zero count safely.

## 9. Program 5 — Block Delete

### Objective

Delete the third byte from the same five-byte block.

Before:

`11H, 22H, 33H, 44H, 55H`

After the shift:

`11H, 22H, 44H, 55H, ...`

The value at `2104H` is still physically present unless you explicitly overwrite it. The logical block length, however, has decreased from five bytes to four.

### GNUSim8085 code

```

ORG 2000H

LXI D,2102H
LXI H,2103H
MVI C,02H

SHIFT: MOV A,M
       STAX D
       INX H
       INX D
       DCR C
       JNZ SHIFT

HLT

```

### Why does deletion move forward?

The byte at `2102H` is the one to remove. We begin by copying the next byte, `[2103H]`, over it. Then the byte at `2104H` is copied over `2103H`.

The source pointer HL and destination pointer DE both advance.

The count is:

\\[ \text{Shift count}=N-p-1 \\]

For \\(N=5\\) and \\(p=2\\):

\\[ 5-2-1=2 \\]

So C begins at `02H`.

### General deletion formula

For a block beginning at \\(S\\), with zero-based deletion index \\(p\\):

- Destination starts at \\(S+p\\).
- Source starts at \\(S+p+1\\).
- Shift count is \\(N-p-1\\).

If the last element is deleted, the shift count is zero. No bytes need to move; simply reduce the logical length. As with insertion, explicitly handle this case instead of entering a do-first loop with C = `00H`.

## 10. One master template for solving block-operation questions

Instead of memorizing five separate programs, memorize the skeleton below and change the processing instructions and pointer movement.

```

ORG 2000H

; 1. Initialize pointers
LXI H,2100H
LXI D,2200H

; 2. Initialize count
MVI C,05H

; 3. Process one byte
LOOP: ; insert operation here

; 4. Update pointers
      INX H
      INX D

; 5. Update counter and repeat
      DCR C
      JNZ LOOP

HLT

```

The skeleton above is for a forward-processing loop. It is not meant to be copied unchanged into every program.

Use the following decision table to adapt it:

| Problem asks you to… | Processing logic                            | Pointer movement |
| --------------------- | ------------------------------------------- | ---------------- |
| Copy                  | Read source, write destination              | Forward          |
| Compare               | Read one block, compare against other       | Forward          |
| Exchange              | Save one byte, swap the two values          | Usually forward  |
| Insert                | Shift existing bytes right, write new value | Backward         |
| Delete                | Shift subsequent bytes left                 | Forward          |

### The five operation-specific memory aids

### MOVE

Read → Write → Advance

`MOV A,M` → `STAX D` → `INX H` and `INX D`

### COMPARE

Read → Compare → Branch

`LDAX D` → `CMP M` → `JNZ NOMATCH`

### EXCHANGE

Save → Overwrite → Restore → Overwrite

Use B to preserve one byte while exchanging the two memory values.

### INSERT

Shift right → Insert

Start at the last byte, move backwards, then write the new value.

### DELETE

Shift left → Reduce logical length

Start from the byte after the deleted element, move forwards, and close the gap.

## 11. GNUSim8085: how to execute and verify the experiment

The precise menu labels can differ between simulator versions, but the workflow is the same.

1. Enter the program. Start the code at `2000H`, using `ORG 2000H` if your assembler supports it. Use the simulator's own assembly syntax and label rules.
2. Assemble. Resolve syntax errors before running the program. Check labels, hexadecimal suffixes, and instruction operands.
3. Load input data. Open the memory view and enter test values at `2100H` and, where required, `2200H`.
4. Set a breakpoint or single-step. Watch HL, DE, A, B, C and the flags. In a loop, check the counter after every `DCR C`.
5. Run to HLT. Open the destination memory or result address and confirm that the output matches the expected values.

### Suggested test data

Use these values when checking the programs.

| Address      | Value       | Purpose                     |
| ------------ | ----------- | --------------------------- |
| 2100H        | 11H         | First byte                  |
| 2101H        | 22H         | Second byte                 |
| 2102H        | 33H         | Third byte                  |
| 2103H        | 44H         | Fourth byte                 |
| 2104H        | 55H         | Fifth byte                  |
| 2200H–2204H | Your choice | Destination or second block |

For the compare program, first make both blocks identical and confirm that `[2300H] = 00H`. Then change one byte in the second block and confirm that `[2300H] = FFH`.

For insert and delete, inspect the memory locations before and after the operation, not just the final counter.

### Common errors and how to diagnose them

| Symptom                         | Likely cause                                                         | What to check                                                    |
| ------------------------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Only one byte copied            | Incorrect loop or jump address                                       | Confirm that `JNZ` points to the first instruction of the loop |
| Program runs too long           | Counter began at `00H`, or loop count is wrong                     | Inspect C before the loop                                        |
| Incorrect byte overwritten      | Wrong starting pointer                                               | Check the initial HL and DE values                               |
| Compare always reports mismatch | `CMP` operands are reversed or input blocks differ                 | Check A, `[HL]`, and `[DE]`                                  |
| Compare result is wrong         | Another flag-changing instruction occurs before the conditional jump | Put `JZ`/`JNZ` immediately after `CMP`                     |
| Insertion corrupts data         | Shift direction is wrong or the new slot is unavailable              | Shift backward from the end                                      |
| Delete leaves the old last byte | Expected behavior if logical length is reduced only                  | Track the new block length separately                            |
| Assembler rejects the program   | Syntax or unsupported directive                                      | Follow the installed simulator's syntax                          |

### An important point about overlapping memory

If source and destination blocks overlap, the copy direction matters.

For example, copying a block one location to the right can overwrite source bytes if you copy from low addresses to high addresses. In that case, a backward copy is generally required. Copying to a completely separate destination block, such as `2100H` to `2200H`, does not have this particular problem.

## 12. How to approach unfamiliar assembly questions

In an exam, the examiner might change the values, addresses, count, or requested operation. Use this sequence rather than guessing instructions.

1. Translate the question into a memory diagram. Identify the source, destination, input values, output address, and block length.
2. Decide what one iteration must accomplish. For example, read one byte and copy it, compare two bytes, or shift one byte.
3. Choose the registers. Use HL and DE as pointers, C as the counter, A for data and B for temporary storage when needed.
4. Determine the direction. Ask whether the data should move forward or backward. In particular, insertion and deletion use opposite directions.
5. Write the loop condition. Decide whether the counter reaches zero or whether a data condition terminates the loop.
6. Trace the first and last iterations by hand. Check the addresses, values and count to catch off-by-one errors.
7. Check special cases. Consider a one-byte block, insertion at the end, deletion of the last element, or an initial count of zero.

### Address calculations to memorize

For a block starting at address \\(S\\), containing \\(N\\) bytes:

\\[ \text{Last address}=S+N-1 \\]

\\[ \text{Next free address}=S+N \\]

For an element at zero-based index \\(p\\):

\\[ \text{Element address}=S+p \\]

These equations prevent most pointer-initialization mistakes.

Example: A six-byte block begins at `2500H`.

- Last byte: `2500H + 6 − 1 = 2505H`
- Next free location: `2500H + 6 = 2506H`
- Fourth element (index 3): `2500H + 3 = 2503H`

Hexadecimal address arithmetic follows the same rules as ordinary addition, except the base is 16.

## 13. What to keep in mind during the practical exam

### Before writing the program

- Identify the starting address of the program.
- Identify the source and destination memory addresses.
- Convert the block length into hexadecimal.
- Calculate the correct initial pointer values.
- Decide which register is the counter.
- Determine which flags control the loop or comparison.
- Write the algorithm before the assembly instructions.

### Before executing the program

- Ensure input data is loaded at the correct memory addresses.
- Check the loop label and jump target.
- Verify the initial count and pointer values.
- Check that a conditional jump follows the instruction whose flags it needs.
- Make sure the program terminates at `HLT`.

### When explaining the program to the examiner

Explain it in this order:

1. Aim: What operation is performed?
2. Algorithm: What steps are followed?
3. Register usage: What do HL, DE, A, B and C represent?
4. Instructions: Explain the main data-transfer and control instructions.
5. Loop: Explain pointer movement, counter decrement and the jump condition.
6. Flags: Explain how Z or CY is used.
7. Result: State which memory locations should contain the output.

This order makes your explanation logical and easier to follow.

## 14. Viva questions with model answers

Q1. What is a block data operation?

An operation that processes a sequence of bytes stored in consecutive memory locations, such as copying, comparing, exchanging, inserting or deleting data.

Q2. Why is `MOV A,M` used?

The instruction transfers the byte stored at the memory location addressed by HL into the accumulator.

Q3. What does M represent?

M is not a physical register. It refers to the memory location whose 16-bit address is contained in the HL register pair.

Q4. Why is `STAX D` used?

It stores the accumulator's contents at the memory location addressed by DE.

Q5. Why does `DCR C` affect the Zero flag?

DCR decrements C and sets Z if the result is zero. `JNZ` uses that flag to decide whether to repeat the loop.

Q6. Does `INX H` affect the flags?

No. INX increments the register pair but does not change the condition flags.

Q7. What does `CMP M` do?

It compares A with the byte at `[HL]` by performing an internal subtraction. It updates flags without changing A or the memory byte.

Q8. Why does insertion shift data backwards?

To avoid overwriting bytes that have not yet been copied. Moving from the end toward the insertion position preserves the original data.

Q9. Why does deletion shift data forwards?

Each subsequent byte is copied into the preceding location, closing the gap left by the deleted element.

Q10. What is the purpose of `XCHG`?

It exchanges the contents of the HL and DE register pairs, not the data stored at the memory addresses they point to.

Q11. What happens if the loop counter starts at zero?

In a loop that processes first and decrements afterwards, decrementing `00H` wraps the counter to `FFH`. The loop can therefore run 256 iterations instead of zero.

Q12. Does deleting a byte automatically erase the last memory location?

No. The program shifts the remaining bytes left. The old last location may still contain a duplicate of the previous final byte; the logical block length must be reduced separately.

## 15. Final revision: test yourself

Try these without looking back at the explanations. Select an answer to each question and check your score.

1\. Which instruction reads the byte addressed by HL into A?

MOV M,A

MOV A,M

STAX D

LXI H,2100H

2\. Which flag does JNZ examine?

Carry

Parity

Zero

Sign

3\. What is the last address of a 5-byte block starting at 2100H?

2104H

2105H

2106H

2100H

4\. Which direction is normally used to shift data during insertion?

Forward from the beginning

Backward from the end

No pointer movement

Alternate directions

5\. After CMP, what does Z = 1 indicate?

A is smaller

A is greater

The values are equal

The counter is zero in all cases

6\. Which instruction pair can form a basic counter-controlled loop?

LXI H; HLT

DCR C; JNZ LOOP

MOV A,M; STAX D

XCHG; NOP

7\. What does XCHG swap?

Two memory bytes

A and B

HL and DE register pairs

The contents of A and memory

Check answers (0/7 answered)

## The key principle to remember

For any new 8085 assembly problem, first identify the data, pointers, counter, operation and exit condition. Then determine the direction of movement, write one correct iteration, and add the counter-controlled loop.

You do not need to memorize every assembly program independently. Learn what each instruction does, how flags are affected, and how the memory addresses change. That is what allows you to adapt a known program to a new exam question.
