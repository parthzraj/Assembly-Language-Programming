
# 8085 Experiment 2: 8-Bit Arithmetic Operations — Complete Lab & Exam Guide

Your experiment covers four operations on the 8085 microprocessor: addition, subtraction, multiplication, and division. The first two use built-in arithmetic instructions, while multiplication and division are implemented using loops.

I’ll explain the concepts in the order that makes the programs easiest to understand: registers and memory, arithmetic instructions, flags, loop logic, GNUSim8085 programs, and finally a general method you can use to solve unfamiliar assembly questions in your lab exam.

The examples follow your uploaded experiment sheet.&#x20;

Experiment-2-8-Bit-Arithmetic-Operations-Using-8085-Microprocessor.pptx

## 1. What are you actually learning in this experiment?

| Operation      | How the 8085 performs it | Main concept                       |
| -------------- | ------------------------ | ---------------------------------- |
| Addition       | `ADD` or `ADI`       | Accumulator and carry              |
| Subtraction    | `SUB` or `SUI`       | Borrow and two's complement        |
| Multiplication | Repeated addition        | Counter and loop                   |
| Division       | Repeated subtraction     | Comparison, quotient and remainder |

The 8085 is an 8-bit microprocessor. Its arithmetic logic unit (ALU) directly performs arithmetic on 8-bit operands. It does not have dedicated `MUL` and `DIV` instructions, so you must construct multiplication and division algorithms from simpler instructions.

## 2. Concepts you must know before writing any code

### 2.1 Registers

A register is a small, fast storage location inside the processor.

## A

Accumulator

Holds one operand and receives the result of most arithmetic operations.

## B, C

General-purpose registers

Store operands, counters, and intermediate values. In our multiplication example, `B` is the counter.

## D, E, H, L

Other general-purpose registers

Can store operands and intermediate data. `H` and `L` can also be combined as the `HL` register pair to point to a memory address.

Each individual register holds 8 bits, meaning a value from `00H` to `FFH` (0–255).

For example:

```
MVI A, 45H
MVI B, 39H
ADD B
```

The processor executes:

1. `MVI A, 45H`: put `45H` in A.
2. `MVI B, 39H`: put `39H` in B.
3. `ADD B`: add B to A and store the result in A.

Therefore:

\\[ 45H+39H=7EH \\]

After execution, `A = 7EH`.

Important: `ADD B` means `A ← A + B`. It does not overwrite B.

### 2.2 Hexadecimal and 8-bit limits

You will see hexadecimal numbers throughout your lab.

| Hexadecimal | Decimal |
| ----------- | ------- |
| `05H`     | 5       |
| `0AH`     | 10      |
| `10H`     | 16      |
| `45H`     | 69      |
| `7EH`     | 126     |
| `FFH`     | 255     |

The `H` indicates hexadecimal notation. Each hexadecimal digit represents four bits.

An 8-bit result can represent values from `00H` to `FFH`. If an addition produces a result greater than `FFH`, the extra ninth bit appears as the Carry flag, rather than remaining in the 8-bit accumulator.

For example:

\\[ \begin{aligned} &\texttt{FFH}+\texttt{01H}=\texttt{100H}\\\ & A=\texttt{00H},\quad CY=1 \end{aligned} \\]

This distinction is central to the experiment: the accumulator stores only eight result bits, while the flags record additional information.

### 2.3 Memory and addresses

Memory stores data and instructions. Each memory location in the 8085 stores one byte.

- `3000H` is a memory address.
- `7EH` is a data value.
- `STA 3000H` stores the accumulator's value at address `3000H`.
- `LDA 3000H` loads the value at `3000H` into the accumulator.

Do not confuse an address with the data stored at that address.

For example:

```
MVI A, 7EH
STA 3000H
```

After these instructions:

- A contains `7EH`.
- Memory location `3000H` contains `7EH`.

The two locations are different storage locations holding the same value.

## 3. The five 8085 flags — what changes and why?

The flag register contains five condition flags. Arithmetic and logical instructions update them according to their defined behavior. These flags help you make decisions, especially inside loops.

## CY

Carry flag

Set to 1 when an addition produces a carry out of bit 7, or when a subtraction requires a borrow. For subtraction, `CY = 1` means a borrow was required in unsigned arithmetic.

## Z

Zero flag

Set to 1 when the result of an operation is zero; otherwise cleared. This is particularly useful when a loop counter becomes zero.

## S

Sign flag

Copies bit 7 of the 8-bit result. A set sign flag indicates that the most significant bit is 1. It does not, by itself, tell you whether an unsigned value is negative.

## P

Parity flag

Set to 1 when the result contains an even number of 1 bits; cleared when the count is odd.

## AC

Auxiliary Carry flag

Set when an arithmetic operation generates a carry from bit 3 into bit 4. It is mainly relevant to BCD arithmetic and instructions such as `DAA`.

### 3.1 Example: addition flags

Consider:

```
MVI A, 45H
ADI 39H
```

The result is `7EH`, or binary `01111110`.

- `CY = 0`: no carry beyond bit 7.
- `Z = 0`: the result is not zero.
- `S = 0`: bit 7 is zero.
- `P = 1`: `01111110` contains six set bits, which is even parity.
- `AC = 1`: the low nibble adds `5H + 9H = EH`; this does not generate a carry from bit 3 to bit 4. Therefore, AC is actually 0.

The correct final flags are CY = 0, Z = 0, S = 0, P = 1, AC = 0.

### 3.2 Example: subtraction flags

```
MVI A, 85H
SUI 42H
```

\\[ 85H-42H=43H \\]

- `CY = 0`: no borrow was needed.
- `Z = 0`: result is nonzero.
- `S = 0`: bit 7 is zero.
- `P = 0`: `43H` is `01000011`, containing three 1 bits, which is odd parity.
- `AC = 0`: no borrow from bit 4 was needed in the low-nibble subtraction.

### 3.3 Which instructions affect flags?

This is an important exam distinction.

| Instruction                        | Flags affected                                       |
| ---------------------------------- | ---------------------------------------------------- |
| `ADD`, `ADI`, `SUB`, `SUI` | CY, Z, S, P, AC                                      |
| `INR`, `DCR`                   | Z, S, P, AC; CY is unchanged                         |
| `CMP`, `CPI`                   | Flags updated as for subtraction, but A is unchanged |
| `MOV`, `MVI`, `LDA`, `STA` | Flags unchanged                                      |
| `JZ`, `JNZ`, `JC`, `JNC`   | Flags unchanged; they test existing flags            |

Memory aid: arithmetic updates flags; data movement generally does not; conditional jumps read flags without changing them.

## 4. Essential instructions to memorize

These are the instructions you will use repeatedly in this experiment and many other 8085 programs.

| Instruction        | Meaning                                    | Example          |
| ------------------ | ------------------------------------------ | ---------------- |
| `MVI r, data`    | Load immediate data into a register        | `MVI B, 05H`   |
| `MOV r1, r2`     | Copy one register into another             | `MOV C, A`     |
| `LXI rp, data16` | Load a 16-bit value into a register pair   | `LXI H, 3000H` |
| `ADD r`          | Add register to A                          | `ADD B`        |
| `ADI data`       | Add immediate data to A                    | `ADI 05H`      |
| `SUB r`          | Subtract register from A                   | `SUB B`        |
| `SUI data`       | Subtract immediate data from A             | `SUI 02H`      |
| `INR r`          | Increment register by 1                    | `INR C`        |
| `DCR r`          | Decrement register by 1                    | `DCR B`        |
| `CMP r`          | Compare A with register without changing A | `CMP B`        |
| `JMP label`      | Jump unconditionally                       | `JMP LOOP`     |
| `JZ label`       | Jump if zero flag is 1                     | `JZ DONE`      |
| `JNZ label`      | Jump if zero flag is 0                     | `JNZ LOOP`     |
| `JC label`       | Jump if carry flag is 1                    | `JC DONE`      |
| `JNC label`      | Jump if carry flag is 0                    | `JNC LOOP`     |
| `STA address`    | Store A directly in memory                 | `STA 3000H`    |
| `LDA address`    | Load A directly from memory                | `LDA 2500H`    |
| `HLT`            | Halt execution                             | `HLT`          |

A register operand is usually a register name, while an immediate operand is a literal value encoded in the instruction.

For example:

```
ADD B       ; A = A + B
ADI 05H     ; A = A + 05H
SUB B       ; A = A - B
SUI 05H     ; A = A - 05H
```

`ADD B` and `ADI 05H` are not interchangeable unless B already contains `05H`.

## 5. The most important concept: how loops work

Once you understand loops, you can solve multiplication, division, summation, repeated subtraction, counting, and many array-processing problems.

A loop repeatedly executes a block of instructions until a condition is satisfied.

### 5.1 The standard 8085 loop pattern

```
MVI B, 05H       ; Initialize counter

LOOP: 
; Perform the required operation here

DCR B            ; Decrease counter
JNZ LOOP         ; Repeat if counter is not zero

HLT
```

This pattern repeats the body five times, provided the body does not alter B or overwrite the flags between `DCR B` and `JNZ LOOP`.

1\. Initialize

B = 05H

2\. Execute the loop body

Perform one operation

3\. Decrement the counter

DCR B

4\. Test the Zero flag

JNZ LOOP: repeat if B is not zero

Z = 0

Go back to LOOP

Z = 1

Continue to HLT

### 5.2 Trace a loop manually

Suppose B starts at `03H`.

| Iteration | B before `DCR B` | B after `DCR B` | Z flag | `JNZ LOOP`  |
| --------- | ------------------ | ----------------- | ------ | ------------- |
| 1         | `03H`            | `02H`           | 0      | Jumps         |
| 2         | `02H`            | `01H`           | 0      | Jumps         |
| 3         | `01H`            | `00H`           | 1      | Does not jump |

The loop body executes three times.

The key idea: `JNZ` does not directly check B. It checks the Zero flag set by the preceding instruction. Since `DCR B` updates Z, the jump effectively repeats until B reaches zero.

This is a general pattern for 8085 assembly programming.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.syedengg.ac.in\&sz=32)

Syed Engineering College+1

### 5.3 Rules that prevent loop mistakes

1. Initialize the counter before entering the loop. Otherwise, it may contain an unpredictable value.
2. Decrement or increment the counter at the appropriate point. If you forget to update it, the loop may run forever.
3. Keep the flag-producing instruction next to the conditional jump. If you intend to use the Zero flag from `DCR B`, avoid putting another flag-changing instruction between `DCR B` and `JNZ LOOP`.
4. Use a counter register that the loop body does not overwrite.
5. Check whether the loop runs at least once. A counter initialized to zero and then decremented wraps around to `FFH`, causing 256 iterations in an 8-bit counter.
6. Keep track of which register stores the answer. The counter is not necessarily the result.

### 5.4 Test your understanding

B starts at `04H`. The loop body executes, then `DCR B` and `JNZ LOOP` execute. How many times does the loop body execute?

3 times

4 times

5 times

256 times

## 6. Program 1 — 8-bit addition

Objective: Add two 8-bit operands and store the result at memory location `3000H`.

Example: `45H + 39H = 7EH`.

### GNUSim8085 code

```
MVI A, 45H
MVI B, 39H

ADD B
STA 3000H

HLT
```

### Execution trace

| Instruction    | A       | B       | Explanation            |
| -------------- | ------- | ------- | ---------------------- |
| `MVI A, 45H` | `45H` | —      | Load first operand     |
| `MVI B, 39H` | `45H` | `39H` | Load second operand    |
| `ADD B`      | `7EH` | `39H` | Add B to A             |
| `STA 3000H`  | `7EH` | `39H` | Store result in memory |

Expected result:

- Memory `3000H` = `7EH`
- `CY = 0`
- `Z = 0`
- `S = 0`
- `P = 1`
- `AC = 0`

### What if the result overflows?

Try:

```
MVI A, 0F0H
ADI 20H
STA 3000H
HLT
```

The mathematical result is `110H`, but the accumulator holds only `10H`. The Carry flag is set to 1.

If your question asks for an 8-bit sum and carry, record both the accumulator result and CY. If it asks for the complete 9-bit sum, you must preserve the carry separately or use a suitable wider-result algorithm.

## 7. Program 2 — 8-bit subtraction

Objective: Subtract one operand from another and store the 8-bit result.

Example: `85H − 42H = 43H`.

### GNUSim8085 code

```
MVI A, 85H
MVI B, 42H

SUB B
STA 3001H

HLT
```

Expected result:

- A = `43H`
- Memory `3001H` = `43H`
- CY = 0 (no borrow)
- Z = 0
- S = 0
- P = 0
- AC = 0

### What happens when subtraction is negative?

Consider:

```
MVI A, 05H
MVI B, 09H
SUB B
STA 3001H
HLT
```

The mathematical result is `−4`. In 8-bit arithmetic, the accumulator holds the two's-complement representation:

\\[ 256-4=252=\texttt{FCH} \\]

So:

- A = `FCH`
- CY = 1, indicating a borrow
- S = 1, because bit 7 is 1
- Z = 0
- P = 1, because `FCH = 11111100` contains six set bits
- AC = 1, because the low-nibble subtraction `5H − 9H` requires a borrow from bit 4

Exam distinction: CY is the unsigned borrow indicator for subtraction. S indicates the most significant bit of the stored result; it is not a universal signed-overflow detector.

## 8. Program 3 — multiplication by repeated addition

The 8085 has no dedicated multiplication instruction. To calculate \\(5 \times 4\\), add 5 to a running total four times:

\\[ 5+5+5+5=20=\texttt{14H} \\]

### Algorithm

1. Initialize A to zero to hold the running total.
2. Load the multiplicand into C.
3. Load the multiplier into B as the loop counter.
4. Add C to A.
5. Decrement B.
6. Repeat until B becomes zero.
7. Store A as the result.

### GNUSim8085 code

```
MVI A, 00H       ; Running total = 0
MVI C, 05H       ; Multiplicand = 5
MVI B, 04H       ; Multiplier = 4

LOOP: ADD C      ; Total = total + multiplicand
      DCR B      ; One addition completed
      JNZ LOOP   ; Repeat if counter is not zero

STA 3002H
HLT
```

Expected result: `A = 14H`, and memory `3002H = 14H`.

### Step-by-step dry run

| Iteration | A before addition | A after `ADD C` | B after `DCR B` |
| --------- | ----------------- | ----------------- | ----------------- |
| 1         | `00H`           | `05H`           | `03H`           |
| 2         | `05H`           | `0AH`           | `02H`           |
| 3         | `0AH`           | `0FH`           | `01H`           |
| 4         | `0FH`           | `14H`           | `00H`           |

At the end, `B = 00H`, so Z = 1 and the loop exits.

### Which flags are affected?

- `ADD C` updates CY, Z, S, P and AC.
- `DCR B` updates Z, S, P and AC but preserves CY.
- `JNZ LOOP` checks Z without changing it.
- `STA 3002H` does not affect the flags.

The final flags reflect the most recent flag-changing instruction, `DCR B`, rather than necessarily describing the multiplication result.

Important limitation: this simple program is for a nonzero multiplier and an 8-bit result. A multiplier of zero would cause 256 iterations, and a product greater than `FFH` would overflow the accumulator. Those cases require additional handling.

## 9. Program 4 — division by repeated subtraction

The 8085 does not have a dedicated division instruction. We can calculate the quotient by subtracting the divisor repeatedly from the dividend.

For example:

\\[ 16 \div 4 = 4 \text{ remainder } 0 \\]

In hexadecimal:

\\[ \texttt{10H} \div \texttt{04H} \\]

Subtract `04H` four times:

`10H → 0CH → 08H → 04H → 00H`

The number of successful subtractions is the quotient. The final leftover value is the remainder.

### The crucial difference from multiplication

For multiplication, you know the number of repetitions in advance, so a counter works well.

For division, you generally do not know how many repetitions will be needed. You must test whether another subtraction is possible.

That requires `CMP`, `JC`, `SUB` and `JMP`.

### GNUSim8085 code

This program divides `10H` by `04H`, stores the quotient at `3003H` and the remainder at `3004H`.

```
MVI A, 10H       ; Dividend = 16
MVI B, 04H       ; Divisor = 4
MVI C, 00H       ; Quotient = 0

LOOP: CMP B      ; Compare A with B
      JC DONE    ; If A < B, stop
      SUB B      ; A = A - B
      INR C      ; Quotient = quotient + 1
      JMP LOOP   ; Try another subtraction

DONE: STA 3004H  ; Store remainder
      MOV A, C   ; Move quotient into A
      STA 3003H  ; Store quotient
      HLT
```

Expected output:

| Memory location | Meaning   | Value   |
| --------------- | --------- | ------- |
| `3003H`       | Quotient  | `04H` |
| `3004H`       | Remainder | `00H` |

### How does `CMP B` work?

`CMP B` internally performs the equivalent of \\(A-B\\) for flag-setting purposes, but it does not change A.

- If A is smaller than B, CY = 1.
- If A equals B, Z = 1.
- If A is greater than B, CY = 0 and Z = 0.

The instruction `JC DONE` jumps when CY = 1. This prevents the program from subtracting the divisor after the dividend has become smaller than it.

### Trace the division program

| Step | A before comparison | Comparison     | Action                | Quotient C |
| ---- | ------------------- | -------------- | --------------------- | ---------- |
| 1    | `10H`             | `10H ≥ 04H` | Subtract; A = `0CH` | `01H`    |
| 2    | `0CH`             | `0CH ≥ 04H` | Subtract; A = `08H` | `02H`    |
| 3    | `08H`             | `08H ≥ 04H` | Subtract; A = `04H` | `03H`    |
| 4    | `04H`             | `04H ≥ 04H` | Subtract; A = `00H` | `04H`    |
| 5    | `00H`             | `00H < 04H`  | CY = 1; jump to DONE  | `04H`    |

At the end, A contains the remainder `00H`, and C contains the quotient `04H`. The program stores the remainder before reusing A for the quotient.

### Flags to understand

- `CMP B` changes the arithmetic flags without modifying A.
- `JC DONE` tests CY and does not change flags.
- `SUB B` changes all five arithmetic flags.
- `INR C` updates Z, S, P and AC but preserves CY.
- `JMP LOOP` does not affect flags.

The division algorithm is only valid when the divisor is nonzero. If the divisor is `00H`, repeated subtraction never terminates normally. A robust program must explicitly check for division by zero before entering the loop.

## 10. How to use GNUSim8085 in your practical

The exact layout can differ by simulator version, but the workflow is generally the following.

[GNUSim8085 - Introduction](https://images.openai.com/static-rsc-4/FGbKBrgMHEgkpg3Jxwo2bDnNOcuVAQa1Vex9oYw_v260I8PGbu7kAvPxPXK9ieqHbAQ8jmTpZ00yD-T-IF55g5Zuj-5Ty5k47xrsNVFs9Bi-cYc9WN1iyv0jz3VM6IipZrtKwxR062NU4toW4X_F4HN_yzI7ld48jlRYB1USo08?purpose=inline)

Step 1 — Open the simulator

Open GNUSim8085 and locate the assembly editor, registers, flags and memory view.

[GNUSim8085 - Screenshots](https://images.openai.com/static-rsc-4/lCT0GEuWs5dEauBR77yAhFwiewNQytUx5tDqKd7o5TJ9IDd0e-6ffSoi27fyX3eNPs-KVC4VjFclLbThjlasUJqGeAaM-AYIxFbeffgi_LTT4lr9RjDuKnDzhTJKN0x_YWLZQw0RF_giuA1xB8aK-f-LrOTBAz8jBVrDZFwEE8c?purpose=inline)

Step 2 — Enter the program

Type the instructions, labels and `HLT`. Use valid 8085 mnemonics and hexadecimal operands.

[GNU 8085 Simulator download | SourceForge.net](https://images.openai.com/static-rsc-4/mb-QjDUOCZwRyGqkPeW1IPUdMLwKL_hwz3qGTUJRZzS2SB1AFEJVYH9yn4Z4nD4BESq0BYz4JtYluavSiS1F5a5LzqoQpnaksEYmqBdSsIpXt4tl9k25CXknXGVW7_x-ySMnOIUW110Aq3jN5eaqOgX-HkMBH0c3vmRZRWWtlxM?purpose=inline)

Step 3 — Assemble and execute

Assemble the source code, correct any syntax errors, then execute or step through the program. If your version uses a separate load step, load the assembled program into memory.

[GNUSim8085 - Screenshots](https://images.openai.com/static-rsc-4/BXdUEEeJoGC5x61MctaM-qG-bRBOy1UZJ3OsvmQqfGGsWDnEKiC6Vp68rj8ZRinJcLcnAiU_AhtVHLlom4-1yAu-g_7NstmVjeLzUCiL4P9h0hJIyxSExsqXkIkPs37MuF13q0bdQFsTgq7hULEtUO3PN02pak48EuHKg6PW4uI?purpose=inline)

Step 4 — Verify the answer

Inspect A, B, C, the flag register and the specified memory addresses. Check that the output agrees with your hand calculation.

### A practical verification checklist

- Is the program assembled without errors?
- Are the input values loaded into the intended registers?
- Does the program terminate at `HLT`?
- Is the answer stored at the correct memory location?
- Are you interpreting the displayed values as hexadecimal?
- Have you checked the relevant flags after the correct instruction?
- For loops, have you verified the number of iterations?

If your simulator displays memory as address–data pairs, navigate to the actual address, such as `3002H`, rather than assuming the output is automatically displayed in the accumulator.

## 11. A general method for solving almost any 8085 assembly question

Use the following procedure when you are given a new problem in an exam. It applies beyond arithmetic, including data transfer, counting, comparisons and block operations.

### 1

Identify the inputs and outputs

Write down what is given, where it is stored, and where the result must go.

### 2

Choose registers

Assign registers to operands, counters, intermediate results and final answers. Never assign two simultaneously needed values to the same register.

### 3

Write the algorithm in plain English

Example: load the multiplicand, initialize the total, add repeatedly, decrement the counter, and stop when the counter is zero.

### 4

Translate each step into instructions

Use `MVI` to initialize, `ADD` or `SUB` for arithmetic, `INR` or `DCR` for counters, and conditional jumps for decisions.

### 5

Trace the program manually

Record the register values after every instruction in a few sample iterations. Check that the loop condition changes as intended.

### 6

Store and verify the result

Check output addresses, carry or borrow, loop termination and any special cases such as zero operands or overflow.

### The coding memory aid: LOAD → PROCESS → TEST → REPEAT → STORE → STOP

This is the main sequence to remember when you are stuck.

- LOAD: initialize operands, counters and totals.
- PROCESS: perform the operation.
- TEST: check the appropriate flag or condition.
- REPEAT: jump to the loop label if required.
- STORE: save the final answer in the required location.
- STOP: execute `HLT`.

Not every program needs a loop, but the same reasoning structure helps you organize one.

## 12. The most useful memory aids for your exam

### A. Remember the accumulator rule

For most arithmetic questions, think:

- `ADD` → add to A.
- `SUB` → subtract from A.
- `CMP` → compare with A without changing A.
- `STA` → save A to memory.
- `LDA` → load memory into A.

This gives you the core of many arithmetic programs.

### B. Choose your jump using the condition you need

| Your requirement                     | Instruction  | Condition    |
| ------------------------------------ | ------------ | ------------ |
| Repeat while counter is nonzero      | `JNZ LOOP` | Z = 0        |
| Exit when counter becomes zero       | `JZ DONE`  | Z = 1        |
| Jump if a subtraction needs a borrow | `JC DONE`  | CY = 1       |
| Jump if there is no borrow           | `JNC LOOP` | CY = 0       |
| Always repeat or branch              | `JMP LOOP` | No condition |

The most important habit is to ask yourself: Which instruction last set the flag, and what condition do I want to test?

### C. Recognize the algorithm from the wording

| Question says…                              | Think of…                             |
| -------------------------------------------- | -------------------------------------- |
| Add two numbers                              | `ADD` / `ADI`                      |
| Subtract two numbers                         | `SUB` / `SUI`                      |
| Multiply two numbers                         | Repeated addition                      |
| Divide two numbers                           | Repeated subtraction                   |
| Repeat N times                               | Counter loop using `DCR` and `JNZ` |
| Count how many times a condition holds       | Counter plus conditional branch        |
| Find the largest or smallest number          | Compare using `CMP` and branch       |
| Move a block of data                         | Memory pointers and a counter          |
| Check whether two values are equal           | `CMP`, then `JZ`                   |
| Check whether A is smaller than B (unsigned) | `CMP B`, then `JC`                 |

For maximum/minimum and array questions, you will also need to manage memory addresses and preserve the current best value.

## 13. Common mistakes that cost marks

Mistake 1: Forgetting that arithmetic results go into A

`SUB B` changes A, not B. If B must be preserved, do not expect the subtraction to update it.

Mistake 2: Using the wrong flag

After `DCR B`, use Z to determine whether B reached zero. After `CMP B`, use CY to detect whether unsigned A is smaller than B.

Mistake 3: Overwriting the result

In division, A contains the remainder at the end of the loop. Store it before moving the quotient from C into A.

Mistake 4: Infinite or unexpectedly long loops

Check the initial counter, the instruction that changes it, and the conditional jump. A zero-initialized 8-bit counter followed by `DCR` wraps to `FFH`.

Mistake 5: Ignoring overflow

The accumulator is only 8 bits. For multiplication, the mathematical product may need 16 bits. For addition, CY may hold the ninth bit of the result.

Mistake 6: Forgetting the required memory address

The program may calculate the correct result and still lose marks if it stores the output somewhere other than the address specified in the question.

## 14. What to do if this experiment comes in your practical exam

Use this sequence during the exam.

1. Read the question carefully. Identify the operation, operands, required output address and whether the carry, quotient or remainder must also be stored.
2. Write the algorithm first. Even if you know the code, a short algorithm helps prevent mistakes.
3. Assign registers on paper. For example, in multiplication: A = total, C = multiplicand, B = counter.
4. Write the program with labels. Ensure every jump points to a valid label.
5. Dry-run the loop. Test the first iteration and the last iteration before running it in the simulator.
6. Assemble and execute. Fix syntax errors, inspect registers and check memory.
7. Record the result and flags. Report only flags that you have correctly derived or observed after the relevant instruction.
8. Explain the logic in your viva. You should be able to explain why `CMP` is used before `JC` in division, and why `DCR` is used before `JNZ` in a counter loop.

### Likely viva questions

1\. Why does the 8085 not multiply directly in this experiment?

2\. Why is A used for arithmetic?

3\. Why does DCR B precede JNZ LOOP?

4\. Does JNZ check register B directly?

5\. What is the purpose of CMP B in division?

6\. What happens if a divisor is zero?

7\. What is the difference between CY and Z?

8\. Why store the remainder before the quotient?

9\. What is the role of HLT?

## 15. Final revision sheet

Before the exam, make sure you can recall these five things without looking at your notes:

- Arithmetic: `ADD B` means A = A + B; `SUB B` means A = A − B.
- Flags: CY records carry/borrow; Z records a zero result; S records bit 7; P records even parity; AC records carry between bits 3 and 4.
- Counter loop: initialize → process → decrement → `JNZ`.
- Division loop: compare → `JC` to exit → subtract → increment quotient → jump back.
- Output: store the answer at the required memory location before overwriting the register holding it.

The best preparation is to write all four programs from memory, trace every loop by hand, and then verify the answers in GNUSim8085. Once you can explain the purpose of each instruction, you will be able to adapt the same techniques to unfamiliar 8085 assembly questions.
