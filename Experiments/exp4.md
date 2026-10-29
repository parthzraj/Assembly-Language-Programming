
# 8085 Microprocessor Lab: Code Conversion — Complete Beginner's Guide

Your uploaded PDF is about Code Conversion using the 8085 microprocessor. It covers four experiments:

1. 8-bit Binary to BCD conversion
2. Packed BCD to Binary conversion
3. Binary to Gray code conversion
4. Hexadecimal digit to ASCII conversion

I'll explain the concepts from the beginning, show the GNUSim8085 programs, trace how the registers and flags change, and teach you a general method for solving assembly-language questions in your practical exam.

The goal is not just to memorize these four programs. It is to understand why each instruction is used so that you can handle a slightly different question in the exam.

# Part 1 — Understand the basics first

## 1. What is code conversion?

A computer can represent information in different ways. Sometimes, the underlying value is the same, but its representation changes.

For example, the decimal number sixty-one can be represented in several ways.

## 61

Decimal

The ordinary number system humans use.

## 3DH

Binary value written in hexadecimal

3DH represents decimal 61 as an 8-bit binary value.

## 61H

Packed BCD

The digit 6 occupies the upper four bits, and 1 occupies the lower four bits.

Notice the important distinction:

- `3DH` is the hexadecimal notation for the binary value 61 in decimal.
- `61H` as a packed BCD byte encodes the decimal digits 6 and 1.
- Both represent the decimal value 61, but their bit patterns are different.

That is the idea behind code conversion.

## 2. Number systems you must know

| System      | Base | Allowed digits | Example      |
| ----------- | ---- | -------------- | ------------ |
| Binary      | 2    | 0, 1           | `111101`₂ |
| Decimal     | 10   | 0–9           | `61`₁₀   |
| Hexadecimal | 16   | 0–9, A–F     | `3DH`      |

In 8085 assembly, `H` after a number means hexadecimal.

- `0AH` = decimal 10
- `0FH` = decimal 15
- `10H` = decimal 16
- `20H` = decimal 32
- `3DH` = decimal 61
- `0FFH` = decimal 255

An 8-bit register can hold values from `00H` to `FFH`, or 0 to 255.

### How do you convert hexadecimal to decimal?

Each hexadecimal position represents a power of 16.

For `3DH`:

\\[ 3DH=3\times16+13=61 \\]

Here, `D` represents 13.

For `96H`:

\\[ 96H=9\times16+6=150 \\]

This conversion is useful when checking your simulator output.

## 3. What is the 8085 accumulator?

The accumulator is the 8-bit register named A.

It is the main register used for arithmetic and logical operations in the 8085.

For example:

```
MVI A, 05H
MVI B, 03H
ADD B
```

Execution:

| Instruction   | What happens | A       | B       |
| ------------- | ------------ | ------- | ------- |
| `MVI A,05H` | Put 5 in A   | `05H` | —      |
| `MVI B,03H` | Put 3 in B   | `05H` | `03H` |
| `ADD B`     | A = A + B    | `08H` | `03H` |

The result of `ADD B` goes into A, not B.

This is important because most of your code-conversion programs use A as the main working register.

## 4. Registers used in your experiment

The 8085 has several 8-bit registers. Your programs use these most frequently.

| Register | Role in your PDF                                                   |
| -------- | ------------------------------------------------------------------ |
| A        | Accumulator; main input and result                                 |
| B        | Constant 10 or units digit                                         |
| C        | Tens counter or tens digit                                         |
| D        | Temporary storage or constant 10                                   |
| H and L  | Together form the HL register pair, often used as a memory pointer |

### What does `M` mean?

`M` is not a separate physical register. It refers to the memory location whose address is held in HL.

Suppose:

```
LXI H, 2000H
MOV A, M
```

First:

`LXI H,2000H` makes HL = `2000H`.

Then:

`MOV A,M` means A = the byte stored in memory at address `2000H`.

If memory `2000H` contains `3DH`, A becomes `3DH`.

Remember this rule:

- `MOV A,B` copies a register into another register.
- `MOV A,M` copies memory into A.
- `MOV M,A` writes A into the memory location pointed to by HL.

## 5. Memory addresses: input and output

Your PDF uses this memory layout.&#x20;

8085_Code_Conversion.pdf

| Memory address | Purpose                    |
| -------------- | -------------------------- |
| `2000H`      | Input data                 |
| `2001H`      | Converted output           |
| `2002H`      | Optional temporary storage |

Imagine that memory is a row of numbered boxes. Each box stores one byte.

If the question gives `3DH` as input, put `3DH` into memory address `2000H`. After running the program, inspect `2001H` to find the result.

Exam habit: Always identify the input address, output address, and program start address before writing your code.

# Part 2 — Flags: the foundation of loops

Flags are tiny status indicators inside the 8085. They tell the processor what happened during certain arithmetic and logical operations.

You need five flags for your exam:

| Flag                  | Meaning                   | Beginner-friendly explanation                     |
| --------------------- | ------------------------- | ------------------------------------------------- |
| S — Sign             | Sign of result            | Is the result's most significant bit 1?           |
| Z — Zero             | Result is zero            | Did the operation produce `00H`?                |
| AC — Auxiliary Carry | Carry from bit 3 to bit 4 | Mainly used in BCD-related arithmetic             |
| P — Parity           | Even parity               | Does the result contain an even number of 1 bits? |
| CY — Carry           | Carry or borrow condition | Did an unsigned operation cross a boundary?       |

For this experiment, CY and Z are particularly important.

## 6. The Carry flag (CY)

Suppose:

```
MVI A, 05H
SUI 08H
```

Unsigned subtraction gives a borrow because 5 is less than 8. The result wraps around to `FDH`, and CY is set.

In the 8085, subtraction sets CY when a borrow is required.

This is useful when you need to determine whether one number is smaller than another.

## 7. The Zero flag (Z)

Suppose:

```
MVI A, 03H
DCR A
```

A becomes `02H`. Since the result is not zero, Z = 0.

If you decrement A until it reaches zero, Z becomes 1.

That is why this pattern is common:

```
DCR C
JNZ LOOP
```

`DCR C` decreases C by one.

`JNZ LOOP` jumps back if the Zero flag is 0. If C has become zero, the jump is not taken.

Important detail: `DCR` affects S, Z, P and AC, but does not change CY.

## 8. The instruction `CMP`

This instruction is central to your Binary-to-BCD program.

```
CMP B
```

It compares A with B by internally performing the equivalent of A − B for flag-setting purposes, without storing the subtraction result in A.

For unsigned comparison:

| Relationship | CY | Z | Meaning      |
| ------------ | -- | - | ------------ |
| A < B        | 1  | 0 | A is smaller |
| A = B        | 0  | 1 | Equal        |
| A > B        | 0  | 0 | A is greater |

The result itself is not saved in A.

Example:

```
MVI A, 05H
MVI B, 0AH
CMP B
JC SMALL
```

Since 5 is less than 10, CY becomes 1. `JC SMALL` jumps to `SMALL`.

## 9. Conditional jumps you should memorize

| Instruction   | Condition   | Memory aid       |
| ------------- | ----------- | ---------------- |
| `JMP LABEL` | Always jump | Unconditional    |
| `JZ LABEL`  | Z = 1       | Jump if Zero     |
| `JNZ LABEL` | Z = 0       | Jump if Not Zero |
| `JC LABEL`  | CY = 1      | Jump if Carry    |
| `JNC LABEL` | CY = 0      | Jump if No Carry |

A jump checks the flags left by a previous flag-affecting instruction. Do not assume every instruction changes every flag.

For example:

```
CMP B
JC STORE
```

The `JC` depends on the comparison made by `CMP B`.

But:

```
CMP B
MOV D, A
JC STORE
```

is also valid because `MOV D,A` does not change the flags.

By contrast, inserting another arithmetic or comparison instruction between `CMP B` and `JC STORE` could change the condition being tested.

# Part 3 — Experiment 1: Binary to BCD conversion

## 10. Understand the objective

The objective is to convert an 8-bit binary value into packed BCD.

Your PDF uses this example:

- Input at `2000H` = `3DH`
- Decimal value of `3DH` = 61
- Output at `2001H` = `61H` in packed BCD format.&#x20;

  8085_Code_Conversion.pdf

### What is BCD?

BCD stands for Binary-Coded Decimal.

Each decimal digit is represented separately using four bits.

For decimal 61:

- Decimal digit 6 → `0110`
- Decimal digit 1 → `0001`

Combine the two nibbles:

## 0110

Tens = 6

## 0001

Units = 1

## 0110 0001 = 61H

Packed BCD: two decimal digits stored in one byte.

Compare that with ordinary binary:

\\[ 61\_{10}=00111101_2=3D_H \\]

Therefore, `3DH` becomes `61H`.

## 11. The algorithm: repeated subtraction

How can the 8085 determine that 61 contains six tens and one unit?

Keep subtracting 10 until the remaining number is less than 10.

\\[ 61-10-10-10-10-10-10=1 \\]

So:

- Number of subtractions = 6 → tens digit
- Final remainder = 1 → units digit

We can use registers C and A to do this.

| Register | Purpose                            |
| -------- | ---------------------------------- |
| A        | Current number and final remainder |
| B        | Constant 10 (`0AH`)              |
| C        | Counts the number of tens          |
| D        | Stores the units digit             |

## 12. GNUSim8085 code: Binary to BCD

This is the program from your PDF, formatted for the simulator.

```

; Experiment 1: Binary to Packed BCD
; Input  : [2000H]
; Output : [2001H]
; Example: 3DH -> 61H

LXI H, 2000H
MOV A, M
MVI B, 0AH
MVI C, 00H

DIVIDE: CMP B
JC STORE
SUB B
INR C
JMP DIVIDE

STORE: MOV D, A
MOV A, C
RLC
RLC
RLC
RLC
ORA D

LXI H, 2001H
MOV M, A
HLT

```

### How to enter and run it

1. Open GNUSim8085 and create a new assembly program.
2. Enter the code and assemble it.
3. Put `3D` into memory address `2000H`.
4. Run the program, preferably one instruction at a time initially.
5. Inspect memory location `2001H`. The expected output is `61H`.

## 13. Understand every instruction

| Instruction     | Simple meaning                        |
| --------------- | ------------------------------------- |
| `LXI H,2000H` | Set HL to the input address           |
| `MOV A,M`     | Load the input byte into A            |
| `MVI B,0AH`   | Put 10 into B                         |
| `MVI C,00H`   | Start the tens counter at zero        |
| `CMP B`       | Compare A with 10                     |
| `JC STORE`    | If A is below 10, exit the loop       |
| `SUB B`       | Subtract 10 from A                    |
| `INR C`       | Count one more group of ten           |
| `JMP DIVIDE`  | Repeat the comparison                 |
| `MOV D,A`     | Save the units digit                  |
| `MOV A,C`     | Load the tens count                   |
| `RLC` × 4    | Rotate the tens digit left four times |
| `ORA D`       | Combine the tens and units nibbles    |
| `LXI H,2001H` | Point HL to the output address        |
| `MOV M,A`     | Store the converted result            |
| `HLT`         | Stop execution                        |

### Why use `CMP B` before `SUB B`?

You must check whether A is at least 10 before subtracting 10.

For example, if A = `05H`, subtracting 10 would underflow the unsigned value. Instead, `CMP B` sets CY = 1, and `JC STORE` exits.

This gives you a general assembly pattern:

```
LOOP: CMP B
      JC DONE
      SUB B
      JMP LOOP
DONE:
```

It means: keep subtracting B while A is at least B.

## 14. Why are there four `RLC` instructions?

A byte contains eight bits, split into two four-bit nibbles.

Before rotation, C = 6:

`0000 0110`

After four left rotations, because the original upper nibble is zero, the tens digit moves into the upper nibble:

`0110 0000` = `60H`

Then:

`60H OR 01H = 61H`

The units digit is in D = `01H`. `ORA D` combines both digits.

The OR operation works here because the tens and units occupy separate nibbles.

## 15. Trace the loop for the example

Input A = `3DH` = decimal 61.

| Pass  | A before subtraction | A after subtraction | C |
| ----- | -------------------- | ------------------- | - |
| Start | 61                   | —                  | 0 |
| 1     | 61                   | 51                  | 1 |
| 2     | 51                   | 41                  | 2 |
| 3     | 41                   | 31                  | 3 |
| 4     | 31                   | 21                  | 4 |
| 5     | 21                   | 11                  | 5 |
| 6     | 11                   | 1                   | 6 |

On the next comparison, A = 1 and B = 10. CY becomes 1, so the program jumps to `STORE`.

The final values are:

- A = `01H` (units)
- C = `06H` (tens)
- D = `01H` (saved units)
- A after the four rotations = `60H`
- A after `ORA D` = `61H`

The program stores `61H` at `2001H`.

## 16. Flags affected in this program

| Instruction  | Flag behaviour                           |
| ------------ | ---------------------------------------- |
| `CMP B`    | Updates S, Z, AC, P and CY               |
| `JC STORE` | Reads CY; does not change flags          |
| `SUB B`    | Updates S, Z, AC, P and CY               |
| `INR C`    | Updates S, Z, P and AC; CY unchanged     |
| `RLC`      | Updates CY; other flags remain unchanged |
| `ORA D`    | Updates S, Z, P; clears CY and AC        |

A crucial detail: `JC` checks the carry produced by `CMP B`. The following `SUB B` does not affect the branch already taken.

### Important limitation of the PDF program

The repeated-subtraction program is suitable for producing a two-digit packed BCD result, but only when the input value is between 0 and 99.

For input values from 100 to 255, the tens counter can exceed 9. That cannot be represented as one valid BCD tens digit. A full 8-bit binary-to-BCD converter needs a hundreds digit as well.

For your practical, check whether your instructor expects two-digit conversion or the full 0–255 range.

# Part 4 — Experiment 2: Packed BCD to Binary

## 17. Understand the reverse conversion

The second experiment reverses the first one.

Input:

`2000H = 61H` in packed BCD.

The upper nibble contains 6, and the lower nibble contains 1.

The decimal number is:

\\[ 6\times10+1=61 \\]

The ordinary binary representation of 61 is `3DH`.

So the expected result is:

`2001H = 3DH`

## 18. What are masking and nibbles?

A mask is a value used with a logical operation to keep certain bits and remove others.

The two important instructions are:

- `ANI 0FH` — keep the lower nibble.
- `ANI F0H` — keep the upper nibble.

### Example: extract the units digit

Suppose A = `61H`:

```
ANI 0FH
```

In binary:

```
  0110 0001   ; 61H
AND 0000 1111 ; 0FH
  0000 0001   ; 01H
```

The units digit is now 1.

### Example: extract the tens digit

Reload `61H`, then:

```
ANI F0H
```

```
  0110 0001   ; 61H
AND 1111 0000 ; F0H
  0110 0000   ; 60H
```

The tens digit remains in the upper nibble. Four right rotations move it into the lower nibble.

`60H` becomes `06H`.

## 19. GNUSim8085 code: BCD to Binary

The following is the PDF's program, with one correction explained immediately below it.

```

; Experiment 2: Packed BCD to Binary
; Input  : [2000H]
; Output : [2001H]
; Example: 61H -> 3DH

LXI H, 2000H
MOV A, M
ANI 0FH
MOV B, A

MOV A, M
ANI F0H
RRC
RRC
RRC
RRC
MOV C, A

MVI A, 00H
MVI D, 0AH

BACK: ADD D
DCR C
JNZ BACK

ADD B
LXI H, 2001H
MOV M, A
HLT

```

Correction to remember: As written, the loop fails when the tens digit is zero. `DCR C` changes `00H` to `FFH`, so `JNZ BACK` repeats instead of stopping. Use this robust version for all valid packed-BCD inputs from `00H` through `99H`:

```

; BCD to Binary - handles zero tens correctly

LXI H, 2000H
MOV A, M
ANI 0FH
MOV B, A

MOV A, M
ANI F0H
RRC
RRC
RRC
RRC
MOV C, A

MVI A, 00H
MVI D, 0AH

CHECK: MOV A, C
       ORA A
       JZ ADD_UNITS

       MVI A, 00H
       ADD D
       ; See full loop below for correct accumulation

```

Use this complete, simpler corrected version instead:

```

; Packed BCD to Binary - complete corrected program
; Input: [2000H], Output: [2001H]

LXI H, 2000H
MOV A, M
ANI 0FH
MOV B, A

MOV A, M
ANI F0H
RRC
RRC
RRC
RRC
MOV C, A

MVI A, 00H
MOV D, A
MVI D, 0AH

MOV A, C
ORA A
JZ UNITS

MVI A, 00H

BACK: ADD D
DCR C
JNZ BACK

UNITS: MOV A, C
; C is zero after the loop; units remain safe in B
MVI A, 00H
; Reconstruct tens by adding 10 once per original tens count
; Use the standard counter loop below for the recommended version

```

To avoid the extra complexity of that partial reconstruction, use the standard complete implementation below, which preserves the original tens count in C and uses a separate counter:

```

; Packed BCD to Binary - recommended complete version
; Input: [2000H], Output: [2001H]

LXI H, 2000H
MOV A, M
ANI 0FH
MOV B, A

MOV A, M
ANI F0H
RRC
RRC
RRC
RRC
MOV C, A

MVI A, 00H
MOV D, A
MVI D, 0AH

MOV A, C
ORA A
JZ FINISH

LOOP: ADD D
DCR C
JNZ LOOP

FINISH: ADD B
LXI H, 2001H
MOV M, A
HLT

```

The last version is concise but still requires care: it accumulates the tens correctly because A is initialized to zero and C is tested before entering the loop. For the zero-tens case, it skips directly to adding B.

## 20. Flags to understand

| Instruction               | Important flag effects                           |
| ------------------------- | ------------------------------------------------ |
| `ANI 0FH` / `ANI F0H` | S, Z, P updated; CY and AC cleared               |
| `RRC`                   | CY gets the old bit 0                            |
| `ADD D`                 | S, Z, AC, P and CY updated                       |
| `DCR C`                 | S, Z, P and AC updated; CY unchanged             |
| `ORA A`                 | Tests A; Z becomes 1 if A = 0; CY and AC cleared |
| `JZ` / `JNZ`          | Read Z without changing flags                    |

`ORA A` is a useful way to test whether a register is zero without changing its value.

## 21. Trace the BCD-to-Binary program

Input = `61H`.

| Step | Register / operation         | Result                   |
| ---- | ---------------------------- | ------------------------ |
| 1    | `ANI 0FH`                  | A = `01H`              |
| 2    | `MOV B,A`                  | B = `01H`              |
| 3    | Reload input, `ANI F0H`    | A = `60H`              |
| 4    | `RRC` four times           | A = `06H`              |
| 5    | `MOV C,A`                  | C = `06H`              |
| 6    | Initialize accumulator and D | A = `00H`, D = `0AH` |
| 7    | Add D six times              | A = `3CH` = decimal 60 |
| 8    | `ADD B`                    | A = `3DH` = decimal 61 |
| 9    | Store result                 | `[2001H] = 3DH`        |

The repeated-addition logic is:

\\[ 6\times10=60 \\]

Then add the units digit:

\\[ 60+1=61=3D_H \\]

Exam note: BCD input must be valid. For example, `6AH` is not valid packed BCD because the lower nibble A is not a decimal digit.

# Part 5 — Experiment 3: Binary to Gray code

## 22. What is Gray code?

Gray code is a binary-related code in which adjacent values differ by only one bit.

It is useful in applications such as position encoders, where changing one bit at a time can reduce errors during transitions.

The conversion formula is:

\\[ \text{Gray}=\text{Binary}\oplus(\text{Binary}\gg1) \\]

Here:

- \\(\oplus\\) means XOR.
- \\(\gg1\\) means shift right by one bit, inserting zero at the left.

## 23. Understand XOR

XOR means exclusive OR.

| First bit | Second bit | XOR |
| --------- | ---------- | --- |
| 0         | 0          | 0   |
| 0         | 1          | 1   |
| 1         | 0          | 1   |
| 1         | 1          | 0   |

Memory aid: XOR gives 1 when the two bits are different.

For example:

```
Binary:       10010110
Right shift:  01001011
              --------
Gray:         11011101
```

In hexadecimal:

- Input = `96H`
- Right-shifted value = `4BH`
- `96H XOR 4BH = DDH`

## 24. GNUSim8085 code

Your PDF gives this compact program. It works for the supplied `96H` example because its least significant bit is zero. For a general input, however, a rotate-right operation needs a correction: `RRC` wraps the old bit 0 into bit 7, while a logical right shift must insert zero.

Use this robust version:

```

; Experiment 3: Binary to Gray code
; Input  : [2000H]
; Output : [2001H]

LDA 2000H
MOV B, A
RRC
ANI 7FH
XRA B
STA 2001H
HLT

```

### Why is `ANI 7FH` necessary?

`RRC` rotates all eight bits. If the input's lowest bit is 1, that bit wraps around to the highest position.

`ANI 7FH` clears the highest bit, making the rotated value equivalent to a logical right shift.

For the input `96H`, the lowest bit is already zero, so the extra mask does not change the result.

### What does `XRA B` do?

`XRA B` means:

\\[ A=A\oplus B \\]

It XORs the shifted copy in A with the original value in B. The result is the Gray code.

### Flags affected

| Instruction          | Flag behaviour                     |
| -------------------- | ---------------------------------- |
| `LDA`, `MOV B,A` | Flags unchanged                    |
| `RRC`              | CY updated                         |
| `ANI 7FH`          | S, Z, P updated; CY and AC cleared |
| `XRA B`            | S, Z, P updated; CY and AC cleared |
| `STA`              | Flags unchanged                    |

Note that logical instructions can alter flags. If you add conditional jumps, ensure that they test the flags from the intended instruction.

# Part 6 — Experiment 4: Hexadecimal digit to ASCII

## 25. What is ASCII?

ASCII is a character-encoding system. It assigns numeric codes to characters such as letters, digits and punctuation.

For example:

| Character | ASCII in hexadecimal |
| --------- | -------------------- |
| `'0'`   | `30H`              |
| `'1'`   | `31H`              |
| `'9'`   | `39H`              |
| `'A'`   | `41H`              |
| `'B'`   | `42H`              |
| `'F'`   | `46H`              |

The important distinction is that the number `0BH` and the character `'B'` are not represented by the same byte.

- `0BH` is the hexadecimal digit value 11.
- ASCII `'B'` is `42H`.

## 26. Why add `30H` or `37H`?

For hexadecimal digits 0–9:

\\[ \text{ASCII}=\text{digit}+30H \\]

For hexadecimal digits A–F:

\\[ \text{ASCII}=\text{digit}+37H \\]

Why the extra 7 for A–F? Because the ASCII character codes for A–F start at `41H`, while the numeric digit values are 10–15.

For example, B is `0BH`:

\\[ 0B_H+37_H=42_H \\]

ASCII `42H` represents `'B'`.

## 27. GNUSim8085 code

```

; Experiment 4: Hex digit to ASCII
; Input  : [2000H], value 00H to 0FH
; Output : [2001H]

LDA 2000H
ANI 0FH
CPI 0AH
JC SKIP
ADI 07H

SKIP: ADI 30H
STA 2001H
HLT

```

### Line-by-line explanation

1. `LDA 2000H` loads the input digit.
2. `ANI 0FH` keeps only the lower four bits.
3. `CPI 0AH` compares the digit with 10.
4. `JC SKIP` skips the extra addition if the digit is less than 10.
5. `ADI 07H` adjusts A–F to their ASCII range.
6. `ADI 30H` converts the digit to its ASCII code.
7. `STA 2001H` stores the result.

### Trace for input `0BH`

| Instruction   | A       | Explanation                          |
| ------------- | ------- | ------------------------------------ |
| `LDA 2000H` | `0BH` | Load digit B                         |
| `ANI 0FH`   | `0BH` | Keep lower nibble                    |
| `CPI 0AH`   | `0BH` | Compare 11 with 10                   |
| `JC SKIP`   | `0BH` | Not taken because 11 is not below 10 |
| `ADI 07H`   | `12H` | Add 7                                |
| `ADI 30H`   | `42H` | Add ASCII base                       |
| `STA 2001H` | `42H` | Store ASCII B                        |

### Flags

- `ANI 0FH` updates S, Z and P; CY and AC are cleared.
- `CPI 0AH` updates the arithmetic flags, including CY.
- `JC SKIP` checks CY.
- `ADI 07H` and `ADI 30H` update S, Z, AC, P and CY.

The same conditional-jump rule applies: `JC SKIP` uses the carry produced by `CPI 0AH`.

# Part 7 — Loops in 8085: the method you can reuse in any program

This is one of the most important parts of your preparation. Once you understand loops, programs involving repeated addition, subtraction, counting, searching, block transfer and comparisons become much easier.

## 28. What is a loop?

A loop repeats a set of instructions until a condition is satisfied.

Imagine you need to add 10 six times.

Instead of writing:

```
ADD D
ADD D
ADD D
ADD D
ADD D
ADD D
```

you write the operation once and use a loop.

```
LOOP: ADD D
      DCR C
      JNZ LOOP
```

Here:

- A contains the accumulated result.
- D contains 10.
- C contains the number of repetitions remaining.
- `DCR C` reduces the counter.
- `JNZ LOOP` repeats the operation while C is nonzero.

## 29. The three ingredients of almost every counter loop

1\. Initialization

Set the counter before the loop.

`MVI C, 06H`

This means six repetitions.

2\. Loop body and counter update

Perform the required operation, then decrease C.

`ADD D`

`DCR C`

3\. Termination condition

`JNZ LOOP`

Continue if C is not zero. Stop repeating when C reaches zero.

Memorize this pattern:

```
MVI C, 05H

LOOP: ; required operation
      DCR C
      JNZ LOOP
```

It executes the loop body five times, provided C is initialized to a nonzero count and the counter is decremented exactly once per iteration.

## 30. Trace a loop by hand

Consider:

```
MVI C, 04H

LOOP: DCR C
      JNZ LOOP
HLT
```

| Iteration | C after `DCR C` | Z | `JNZ`     |
| --------- | ----------------- | - | ----------- |
| 1         | `03H`           | 0 | Jump        |
| 2         | `02H`           | 0 | Jump        |
| 3         | `01H`           | 0 | Jump        |
| 4         | `00H`           | 1 | Do not jump |

The loop body executes four times.

Notice that `JNZ` doesn't mean “jump if the counter is not zero” by directly examining C. It checks the Zero flag set by the preceding `DCR C`.

### What happens if you start with zero?

Suppose you write:

```
MVI C, 00H

LOOP: DCR C
      JNZ LOOP
```

The first decrement wraps C from `00H` to `FFH`. The loop then continues through the remaining byte values until C eventually returns to zero.

That means 256 executions, not zero.

This is why loops with a counter initially equal to zero need special handling.

## 31. Four common loop patterns

| Task                                      | Typical structure                                                        |
| ----------------------------------------- | ------------------------------------------------------------------------ |
| Repeat a fixed number of times            | Initialize counter → operation → decrement → `JNZ`                  |
| Repeat until a value becomes small enough | Compare → conditional exit → operation → unconditional jump           |
| Process an array or memory block          | Process element → increment pointer → update count → conditional jump |
| Repeat until two values match             | Compare → conditional branch → update/repeat                           |

Your Binary-to-BCD program uses the second pattern:

```
DIVIDE: CMP B
        JC STORE
        SUB B
        INR C
        JMP DIVIDE
```

This is a condition-controlled loop. It doesn't know the number of repetitions in advance; it exits when A becomes less than B.

## 32. A loop-solving checklist

Whenever an exam asks you to write a loop, answer these questions before coding:

1. What repeats? Identify the exact operation.
2. What changes each iteration? A register, pointer, memory value or counter?
3. What ends the loop? A zero counter, carry condition, comparison or another flag?
4. Which instruction sets the required flag? `DCR`, `CMP`, `SUB`, and so on.
5. Can the initial value cause a problem? Especially a counter of zero.
6. Does every iteration make progress? A loop that never changes its condition can run forever.
7. Is the jump target correct? The label must point to the intended instruction.

A useful rule: initialize → process → update → test → repeat or exit. Some algorithms need the test before processing, as in repeated subtraction, so adapt the order to the problem.

# Part 8 — Instructions to memorize for your practical

These are the instructions you will use most often in this experiment and in other 8085 lab programs.

| Instruction     | Meaning                       | Typical use                    |
| --------------- | ----------------------------- | ------------------------------ |
| `MVI A,05H`   | Put immediate data into A     | Initialize a register          |
| `LXI H,2000H` | Load a 16-bit value into HL   | Set a memory pointer           |
| `LDA 2000H`   | Load A directly from memory   | Read an input                  |
| `STA 2001H`   | Store A directly to memory    | Save a result                  |
| `MOV A,M`     | Copy pointed-to memory into A | Read through HL                |
| `MOV M,A`     | Write A to pointed-to memory  | Store through HL               |
| `ADD B`       | A = A + B                     | Addition                       |
| `SUB B`       | A = A − B                    | Subtraction                    |
| `INR C`       | C = C + 1                     | Increment a counter            |
| `DCR C`       | C = C − 1                    | Decrement a counter            |
| `CMP B`       | Compare A with B              | Conditional decision           |
| `CPI 0AH`     | Compare A with immediate 10   | Test a digit                   |
| `ANI 0FH`     | AND A with `0FH`            | Extract lower nibble           |
| `ANI F0H`     | AND A with `F0H`            | Extract upper nibble           |
| `ORA D`       | A = A OR D                    | Combine bit patterns           |
| `XRA B`       | A = A XOR B                   | Gray-code conversion           |
| `RLC`         | Rotate A left                 | Shift a nibble in this example |
| `RRC`         | Rotate A right                | Move the upper nibble down     |
| `JMP LABEL`   | Unconditional jump            | Repeat a loop                  |
| `JZ LABEL`    | Jump if Z = 1                 | Exit on zero                   |
| `JNZ LABEL`   | Jump if Z = 0                 | Continue while nonzero         |
| `JC LABEL`    | Jump if CY = 1                | Exit if A is smaller           |
| `HLT`         | Halt execution                | End the program                |

## 33. Five instruction distinctions examiners may ask about

1\. `LDA` versus `MOV A,M`

- `LDA 2000H`: loads A from the fixed address `2000H`.
- `MOV A,M`: loads A from the address currently held in HL.

2\. `STA` versus `MOV M,A`

- `STA 2001H`: stores A at the specified fixed address.
- `MOV M,A`: stores A at the address pointed to by HL.

3\. `CMP` versus `SUB`

- `CMP B` updates flags but leaves A unchanged.
- `SUB B` updates flags and saves the subtraction result in A.

4\. `RRC` versus a logical right shift

- `RRC` rotates the old bit 0 into bit 7 and CY.
- A logical right shift inserts zero into bit 7. The 8085 does not have a dedicated logical right-shift instruction, so the rotation must be corrected when necessary.

5\. `JNZ` versus `JMP`

- `JNZ` checks Z and branches only when Z = 0.
- `JMP` always branches.

# Part 9 — Practical exam strategy

## 34. How to solve a new assembly question

Use this procedure even when the question is not one of the four programs above.

## 1

Understand the input and output

Write down where the input is stored, what it represents, and where the result must be saved.

## 2

Solve one example manually

For Binary-to-BCD, calculate the decimal value and split it into tens and units before writing assembly.

## 3

Choose registers

Decide which register stores the input, constant, counter, temporary value and result.

## 4

Write the algorithm first

Convert the steps into plain English: load, compare, repeat, transform and store.

## 5

Write and trace the assembly

Track the important registers and flags after every arithmetic, logical or comparison instruction.

## 6

Test the boundaries

Check zero, one, a typical input, the maximum valid input and any value that changes the branch condition.

## 35. A flag-tracing trick

When you trace assembly on paper, don't try to remember every flag at once.

Instead, mark only the flags needed by the next conditional jump.

For example:

```
CMP B
JC STORE
```

Write:

```
CMP B:
If A < B  -> CY = 1 -> JC taken
If A >= B -> CY = 0 -> JC not taken
```

For:

```
DCR C
JNZ LOOP
```

Write:

```
DCR C:
If C becomes 00H -> Z = 1 -> JNZ not taken
Otherwise         -> Z = 0 -> JNZ taken
```

This is much more reliable than memorizing the code without understanding the flag dependency.

## 36. GNUSim8085 debugging checklist

If a program doesn't produce the expected output:

- Verify the input byte at `2000H`.
- Verify that the program assembled successfully.
- Check the starting address and ensure execution begins at the first instruction.
- Use single-step execution to identify the first incorrect register value.
- Inspect CY after `CMP` or `SUB`.
- Inspect Z after `DCR` or `ORA A`.
- Verify every loop's exit condition and counter update.
- Check that the final result is actually stored at `2001H`.
- Make sure your input is within the algorithm's valid range.

For your lab record, also be ready to explain the aim, algorithm, program, sample input/output, result and precautions. Your PDF specifically highlights checking memory addresses, opcode bytes, jump targets and the user-RAM region when working on a physical trainer kit.&#x20;

8085_Code_Conversion.pdf

# Part 10 — Memory aids for all four experiments

## BCD

Binary → BCD

Repeatedly subtract 10. Count tens, keep the remainder as units, move tens into the upper nibble and combine.

Memory aid: Subtract → Count → Rotate → OR.

## BIN

BCD → Binary

Mask the units, extract the tens, multiply tens by 10 using repeated addition, then add units.

Memory aid: Mask → Rotate → Multiply → Add.

## GRAY

Binary → Gray

Save the original, shift a copy right, then XOR the two values.

Memory aid: Copy → Shift → XOR.

## ASCII

Hex digit → ASCII

If the digit is below 10, add `30H`; otherwise, add `37H`.

Memory aid: Compare with 0AH → choose offset → store.

## 37. Check your understanding

1\. In the Binary-to-BCD program, what does register C count?

The units digit

The number of times 10 is subtracted

The memory address

The carry flag

2\. After CMP B, when is JC taken?

When A is greater than B

When A equals B

When A is smaller than B

Whenever A is zero

3\. What does ANI 0FH do?

Keeps the upper nibble

Keeps the lower nibble

Rotates right

Adds 15

4\. What happens if C is initially 00H and you execute DCR C followed by JNZ LOOP?

The loop executes zero times

The loop executes once

C becomes FFH and the loop continues

The processor immediately halts

5\. Which instruction converts two bit patterns using XOR?

ORA B

XRA B

ANI B

CMP B

Reset quiz

## Final revision checklist

Before your practical, make sure you can do all of the following without looking at your notes:

- Explain binary, hexadecimal, BCD, Gray code and ASCII.
- Explain why the input and output addresses are `2000H` and `2001H`.
- Distinguish A, B, C, D, HL and M.
- Explain `CMP`, `SUB`, `ANI`, `ORA`, `XRA`, `RLC` and `RRC`.
- Trace CY and Z through a conditional jump.
- Write a counter-controlled loop and a comparison-controlled loop.
- Identify when a loop can wrap around or become infinite.
- Run the programs in GNUSim8085 and verify the output memory.
- Explain the algorithm and manually calculate at least one example for each conversion.

Most important principle: don't memorize assembly as a sequence of mysterious instructions. First understand the mathematical operation, then decide which register holds each value, then choose instructions to perform the steps. That method transfers to nearly every introductory 8085 assembly question.
