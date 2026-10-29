
# 8085 Microprocessor Lab: Digital Clock — Complete Experiment Guide

Your uploaded experiment is Digital Clock Using the 8085 Microprocessor. The program maintains seconds, minutes, and hours using BCD counters, uses `DAA` to correct decimal arithmetic, uses subroutines to handle time rollovers, and uses nested loops to generate a software delay.&#x20;

8085_Digital_Clock.pdf

The goal is not just to memorize this one program. You should understand how to construct similar 8085 programs yourself during a practical exam, even if the question changes.

We will study it in this order:

1. What the experiment does and how the algorithm works.
2. The 8085 concepts and instructions you need.
3. BCD arithmetic and the `DAA` instruction.
4. The complete GNUSim8085 program.
5. Every flag affected by the instructions.
6. How nested loops work and how to design them for other questions.
7. How to solve rollover and other assembly problems systematically.
8. Exam precautions, viva questions, and coding memory aids.

## 1. Understand the experiment first

[Assembled Digital Clock with Circuit Diagram (HH : MM : SS Display) – Electronix Express](https://images.openai.com/static-rsc-4/yJ6L9VTFUzNIfp4qwoqqaYXwR9_Yx-ZewsJd75t5K_OgpcXLLahjsWfdm6gzYhO5TfAnNc1ovNWXA45VzkQt_S3UM2E3I6FZ22s2nB4zzh7oB5giby7SvNlytS3muAZnYAfzXLQE2uEUF6DFUy1fhYUhIAmIK-Vtx6wq3DTVVTY?purpose=inline)

[elexp.com](https://www.elexp.com/products/digital-clock-asmbl?utm_source=chatgpt.com)

A digital clock must repeatedly perform these operations:

- Display the current time.
- Wait approximately one second.
- Increase seconds by one.
- If seconds reach 60, reset seconds to zero and increase minutes.
- If minutes reach 60, reset minutes and increase hours.
- If hours reach 24, reset hours to zero.
- Repeat forever.

For example:

Before one tick

# 12:30:59

After one tick

# 12:31:00

What happened?

- Seconds changed from 59 to 00.
- That rollover triggered a one-minute increment.
- Minutes changed from 30 to 31.
- Hours remained 12.

The central idea is that each counter increments independently, but a counter that rolls over triggers the next counter.

## 2. The memory map: where is the time stored?

Your experiment assigns the following memory locations to the clock data.&#x20;

8085_Digital_Clock.pdf

| Memory address | Purpose | Valid values            |
| -------------- | ------- | ----------------------- |
| `2070H`      | Seconds | `00H`–`59H` in BCD |
| `2071H`      | Minutes | `00H`–`59H` in BCD |
| `2072H`      | Hours   | `00H`–`23H` in BCD |

The program begins at `2000H`, while the clock values are kept at `2070H`–`2072H`.

Why separate these?

- Program memory: contains instructions such as `LDA`, `ADI`, and `JMP`.
- Data memory: contains the current time values.
- Accumulator A: temporarily holds a value while arithmetic and comparisons are performed.

For example:

```
LDA 2070H
```

means load the seconds value into accumulator A.

```
STA 2070H
```

means save the value currently in A back to the seconds location.

One important distinction: `2070H` is an address, whereas the byte stored at `2070H` is the seconds value.

## 3. The essential 8085 concepts

### 3.1 Registers

The 8085 contains several registers. You will use these most frequently in this experiment.

| Register             | Main purpose here                                              |
| -------------------- | -------------------------------------------------------------- |
| A (accumulator)      | Arithmetic, comparisons and data transfer                      |
| B, C, D, E           | General-purpose registers; C, D and E are used for delay loops |
| PC (Program Counter) | Holds the address of the next instruction                      |
| SP (Stack Pointer)   | Points to the top of the stack                                 |
| Flag register        | Records results of operations such as arithmetic               |

The registers B and C can also form the BC register pair, as can D and E form DE. However, in the delay routine below, C, D and E are separate loop counters.

### 3.2 The instructions you must understand

| Instruction      | Meaning                             | Example                  |
| ---------------- | ----------------------------------- | ------------------------ |
| `LXI SP,27FFH` | Load a 16-bit value into SP         | Initialize stack         |
| `MVI A,00H`    | Put an immediate byte into A        | Clear A                  |
| `LDA 2070H`    | Load A from memory                  | Read seconds             |
| `STA 2070H`    | Store A in memory                   | Save seconds             |
| `ADI 01H`      | Add 1 to A                          | Increment a counter      |
| `DAA`          | Decimal-adjust A after BCD addition | Correct decimal digits   |
| `CPI 60H`      | Compare A with `60H`              | Check seconds rollover   |
| `JNZ label`    | Jump if Zero flag is 0              | Repeat a loop            |
| `RNZ`          | Return if Zero flag is 0            | Return on ordinary ticks |
| `DCR E`        | Decrement E by 1                    | Reduce loop count        |
| `CALL label`   | Call a subroutine                   | Invoke `INCSEC`        |
| `RET`          | Return from a subroutine            | Resume caller            |
| `JMP label`    | Unconditional jump                  | Repeat the main loop     |

You do not need to memorize the instructions as an isolated list. Learn them by function:

- Move data: `MVI`, `LDA`, `STA`
- Calculate: `ADI`, `DCR`, `DAA`
- Make decisions: `CPI`, `JNZ`, `RNZ`
- Control program flow: `CALL`, `RET`, `JMP`

That classification will help with almost any 8085 assembly question.

## 4. BCD arithmetic: the most important concept

BCD stands for Binary-Coded Decimal.

In packed BCD, each decimal digit occupies four bits. One byte stores two decimal digits.

For example:

| Decimal time value | BCD byte | Binary representation |
| ------------------ | -------- | --------------------- |
| 00                 | `00H`  | `0000 0000`         |
| 07                 | `07H`  | `0000 0111`         |
| 09                 | `09H`  | `0000 1001`         |
| 10                 | `10H`  | `0001 0000`         |
| 25                 | `25H`  | `0010 0101`         |
| 59                 | `59H`  | `0101 1001`         |

Notice that `25H` is 37 in ordinary hexadecimal, but 25 in BCD. The meaning depends on how the value is encoded.

### Why is `DAA` required?

Suppose seconds are currently 09.

```
MVI A, 09H
ADI 01H
```

The processor performs ordinary binary addition:

```
    09H = 0000 1001
  + 01H = 0000 0001
  -----------------
    0AH = 0000 1010
```

But `AH` is not a valid packed-BCD representation of a decimal number with two decimal digits. A BCD digit must be between 0 and 9.

Now execute:

```
DAA
```

The accumulator becomes:

```
A = 10H
```

That is the packed-BCD representation of decimal 10.

### Remember this sequence

## ADI 01H

Ordinary binary addition

## DAA

Correct the result to BCD

## STA address

Save the corrected value

Exam rule: When incrementing a packed-BCD counter with `ADI 01H`, execute `DAA` immediately afterward, before storing and checking the result.

### What happens at 59 + 1?

```
  59H
+ 01H
-----
  5AH    Ordinary binary result
  60H    Result after DAA
```

The clock routine then checks whether the accumulator equals `60H`. If it does, the seconds counter must roll over to zero.

### Why compare with `60H`, not `3CH`?

- `60H` represents decimal 60 in BCD.
- `3CH` represents decimal 60 in ordinary hexadecimal.

Since this experiment stores time in BCD, the comparison constant is `60H`.

Similarly, the 24-hour rollover test uses `24H`, and the 12-hour-format boundary would be `13H` if the clock started at `01H` and counted through `12H`.

## 5. Understand the flags before writing the code

The 8085 has five main condition flags:

## S

Sign flag

Set when the result's most significant bit is 1. For an 8-bit result, this is bit 7.

## Z

Zero flag

Set when the result is zero. This is particularly important for `JNZ`, `JZ`, `RNZ` and `RZ`.

## AC

Auxiliary Carry flag

Indicates carry from bit 3 to bit 4 during addition, or the corresponding auxiliary borrow condition during subtraction.

## P

Parity flag

Set if the result contains an even number of 1 bits.

## CY

Carry flag

Set when addition produces a carry out of bit 7, or subtraction requires a borrow.

### Which instructions affect flags in this experiment?

| Instruction                        | Flags affected            | What matters here           |
| ---------------------------------- | ------------------------- | --------------------------- |
| `ADI 01H`                        | S, Z, AC, P, CY           | Arithmetic result           |
| `DAA`                            | S, Z, AC, P, CY           | BCD correction              |
| `CPI 60H`                        | S, Z, AC, P, CY           | Comparison result           |
| `DCR E`                          | S, Z, AC, P; CY unchanged | Loop termination            |
| `DCR D`                          | S, Z, AC, P; CY unchanged | Loop termination            |
| `DCR C`                          | S, Z, AC, P; CY unchanged | Loop termination            |
| `LDA`, `STA`, `MVI`, `LXI` | None                      | Data movement               |
| `CALL`, `RET`, `JMP`         | None                      | Program flow                |
| `JNZ`, `RNZ`                   | None                      | Read Z; do not modify flags |

### How `CPI` works

Consider:

```
LDA 2070H
ADI 01H
DAA
STA 2070H
CPI 60H
RNZ
```

Assume the seconds value was `35H`.

After the addition and BCD correction:

```
A = 36H
```

The instruction:

```
CPI 60H
```

compares A with `60H` by effectively evaluating A − `60H` for flags. It does not change A.

Since `36H` is not equal to `60H`, the Zero flag becomes 0.

Therefore:

```
RNZ
```

returns to the calling routine.

Now consider the case where the previous value was `59H`:

```
59H + 01H → DAA → 60H
CPI 60H  → Z = 1
RNZ      → does not return
```

Execution continues to the rollover code.

Memory aid:

- `CPI` compares and sets flags.
- `JNZ` jumps if Z = 0.
- `JZ` jumps if Z = 1.
- `RNZ` returns if Z = 0.
- `RZ` returns if Z = 1.

Do not confuse a conditional jump with a conditional return. Both can test the same flag, but their actions differ.

### A critical rule about flags

Flags describe the result of a particular instruction. They are not permanent records of the last arithmetic operation.

For example:

```
CPI 60H
DCR B
JNZ LOOP
```

Here, `JNZ` tests the Zero flag produced by `DCR B`, not the comparison from `CPI 60H`.

This matters in almost every assembly-language program.

## 6. Complete GNUSim8085 program

The following listing follows your experiment's design and uses symbolic labels, so you do not have to calculate every `CALL` address manually.&#x20;

8085_Digital_Clock.pdf

The `DISPLAY` routine is intentionally a placeholder. In GNUSim8085, you can inspect the three memory locations directly. A real trainer kit requires an appropriate display routine for that kit.

```

; EXPERIMENT: DIGITAL CLOCK USING 8085
; Seconds: 2070H (BCD 00-59)
; Minutes: 2071H (BCD 00-59)
; Hours:   2072H (BCD 00-23)
; Assemble from 2000H

START:  LXI SP, 27FFH

        MVI A, 00H
        STA 2070H
        STA 2071H
        STA 2072H

MAIN:   CALL DISPLAY
        CALL DELAY
        CALL INCSEC
        JMP MAIN

; DISPLAY PLACEHOLDER
; Inspect memory in GNUSim8085.
DISPLAY:
        RET

; INCREMENT SECONDS
INCSEC: LDA 2070H
        ADI 01H
        DAA
        STA 2070H
        CPI 60H
        RNZ

        MVI A, 00H
        STA 2070H
        CALL INCMIN
        RET

; INCREMENT MINUTES
INCMIN: LDA 2071H
        ADI 01H
        DAA
        STA 2071H
        CPI 60H
        RNZ

        MVI A, 00H
        STA 2071H
        CALL INCHOUR
        RET

; INCREMENT HOURS
INCHOUR: LDA 2072H
         ADI 01H
         DAA
         STA 2072H
         CPI 24H
         RNZ

         MVI A, 00H
         STA 2072H
         RET

; NESTED SOFTWARE DELAY
; Values below are starting values, not guaranteed
; to give exactly one second on every system.
DELAY:  MVI C, 03H

DL1:    MVI D, 0FFH
DL2:    MVI E, 0FFH

DL3:    DCR E
        JNZ DL3

        DCR D
        JNZ DL2

        DCR C
        JNZ DL1

        RET

```

### How to run it

1. Open GNUSim8085 and enter the complete program.
2. Set the assembly origin/start address to `2000H` if your version requires it.
3. Assemble the program and resolve any syntax errors.
4. Run it continuously rather than single-stepping through the full delay.
5. Inspect memory locations `2070H`, `2071H` and `2072H`.
6. For quick testing, temporarily change the delay constants to `C = 01H`, `D = 05H`, and `E = 05H`.
7. Restore and calibrate the delay for actual timing.

The delay constants are only a starting point. A simulator may run at a different effective speed from a physical trainer kit, and this software delay is not a precise real-time clock source.

### One improvement for exam practice

During debugging, you can initialize the counters to selected values rather than zero. For example, to test the `59 → 00` rollover:

```
MVI A, 59H
STA 2070H

MVI A, 30H
STA 2071H

MVI A, 12H
STA 2072H
```

For a quick rollover test, execute the `INCSEC` routine once. The expected result is:

```
2070H = 00H
2071H = 31H
2072H = 12H
```

Remember to remove or change these test initializations when you return to normal clock operation.

## 7. Understand `CALL`, `RET` and the stack

This is important because your clock uses several subroutines.

Consider:

```
MAIN:   CALL INCSEC
        JMP MAIN
```

When `CALL INCSEC` executes, the 8085 saves the return address on the stack and transfers execution to `INCSEC`.

At the end of the subroutine:

```
RET
```

the processor retrieves the return address and resumes execution at the instruction after the original `CALL`.

### How the clock cascades

The path during an ordinary second is:

```
MAIN
  |
  v
CALL INCSEC
  |
  v
Increment seconds
  |
  v
CPI 60H
  |
  v
RNZ: return to MAIN
```

The path when seconds reach 60 is:

```
MAIN
  |
  v
CALL INCSEC
  |
  v
Seconds = 60H?
  |
  +-- No --> Return to MAIN
  |
  +-- Yes
       |
       v
   Reset seconds
       |
       v
   CALL INCMIN
       |
       +-- Minutes not 60 --> Return to INCSEC
       |
       +-- Minutes = 60
                |
                v
           Reset minutes
                |
                v
           CALL INCHOUR
                |
                v
           Hours = 24H?
                |
                v
           Reset hours if equal
```

A useful detail: if `INCSEC` calls `INCMIN`, and `INCMIN` calls `INCHOUR`, the nested return addresses are handled by the stack. Each `RET` returns to the correct caller.

### Why initialize SP?

```
LXI SP, 27FFH
```

This initializes the stack pointer to a RAM location chosen by the experiment. The stack is used by `CALL` and `RET`.

Without a valid stack pointer, return addresses may be stored in an unintended memory area, corrupting program data or causing incorrect execution.

For your exam, remember:

- `CALL` saves a return address and branches to a subroutine.
- `RET` retrieves that return address.
- `JMP` branches without saving a return address.
- Every `CALL` path must eventually reach the appropriate `RET` if it is intended to return.
- Nested calls require a correctly initialized stack.

## 8. Nested loops: learn the pattern once

The delay routine is the best way to understand nested loops in 8085 assembly.

A loop is a sequence of instructions that repeats until a termination condition is satisfied.

The basic pattern is:

```
LOOP:   ; instructions
        DCR B
        JNZ LOOP
```

It means:

1. Execute the loop body.
2. Decrement B.
3. If B is not zero, jump back.
4. Otherwise, continue to the next instruction.

### What does `DCR` do to flags?

`DCR` decrements a register by one. It updates S, Z, P and AC, but does not change CY.

Suppose B contains `03H` before entering the loop:

| Iteration | B before `DCR B` | B after `DCR B` | Z flag | `JNZ`      |
| --------- | ------------------ | ----------------- | ------ | ------------ |
| 1         | `03H`            | `02H`           | 0      | Jump         |
| 2         | `02H`            | `01H`           | 0      | Jump         |
| 3         | `01H`            | `00H`           | 1      | Fall through |

The loop body executes three times.

### The critical loop-counting rule

Initialize a counter to N, then decrement and test it at the end, and the loop body executes N times, provided the counter is not changed elsewhere.

This rule applies to a single loop and to each level of a nested loop.

### Two nested loops

```
        MVI B, 03H

OUTER:  MVI C, 04H

INNER:  ; Inner-loop work
        DCR C
        JNZ INNER

        DCR B
        JNZ OUTER
```

The inner loop executes 4 times for each outer iteration. The outer loop executes 3 times.

Therefore:

\\[ \text{Total inner-loop iterations}=3\times4=12 \\]

The inner counter must be reinitialized every time the outer loop repeats. Otherwise, after the first run, C would already be zero and the intended repetition would fail.

### Three nested loops in your experiment

Your program uses this structure:

```
DELAY:  MVI C, 03H
DL1:    MVI D, 0FFH
DL2:    MVI E, 0FFH
DL3:    DCR E
        JNZ DL3
        DCR D
        JNZ DL2
        DCR C
        JNZ DL1
        RET
```

The register roles are:

| Register | Role           | Reloaded when?             |
| -------- | -------------- | -------------------------- |
| C        | Outer counter  | Once when `DELAY` begins |
| D        | Middle counter | At each `DL1` iteration  |
| E        | Inner counter  | At each `DL2` iteration  |

`E` is reset for every middle-loop pass. `D` is reset for every outer-loop pass. `C` is initialized only once for the entire delay routine.

With the initial values shown, the nominal counter iterations are:

\\[ 3\times255\times255=195{,}075 \\]

This is the number of executions of the `DCR E` instruction, not the exact number of clock cycles or the total duration. The extra instructions and loop-control overhead also consume time.

### Interactive loop practice

Try predicting the number of inner-loop executions.

### Nested-loop counter calculator

Choose the initial values of the outer and inner counters.

Outer counter (decimal)

2

3

4

5

Inner counter (decimal)

3

4

5

6

Check loop count

## 9. How to design loops for any 8085 question

Do not start by writing `DCR` and `JNZ` randomly. First identify what must repeat, how many times it must repeat, and what should happen after repetition.

Use this four-step method.

1. Identify the repeated operation.

   Examples: increment a counter, process every byte in a block, add array elements, or create a delay.
2. Choose a counter register.

   Use B, C, D or E when an 8-bit count is sufficient. For larger counts, consider a register pair and a suitable loop design.
3. Initialize before entering the loop.

   Put the initialization before the loop label if the counter should be initialized only once. Put it inside an outer loop if it must be reset on every outer iteration.
4. Decrement, test, and branch.

   The common structure is `DCR register` followed by `JNZ label`.

### Example A: Add five numbers

Conceptually, the loop must process five values. You would initialize the count to five, process one value, decrement the count, and repeat until the count becomes zero.

### Example B: Copy ten bytes

The loop must copy one byte per iteration, advance the source and destination addresses, decrease the remaining count, and stop when the count reaches zero.

### Example C: Generate a delay

The loop does not need to calculate a mathematical result. It repeats instructions to consume processor time. Nested counters can extend the duration.

The pattern is the same, but the work inside the loop changes.

### Five loop mistakes to avoid

1. Forgetting to initialize the counter. Its initial value may be unpredictable.
2. Forgetting to decrement it. The loop may never terminate.
3. Jumping to the wrong label. This can skip initialization or repeat the wrong block.
4. Failing to reload an inner counter. The outer loop may repeat without doing the intended inner work.
5. Allowing another instruction to overwrite the Zero flag before `JNZ`. The branch must test the flags produced by the intended instruction.

One additional caution: a counter initialized to `00H` and then decremented becomes `FFH`. Therefore, a decrement-and-test loop initialized to zero does not execute zero times; it can execute 256 times before the counter returns to zero.

## 10. The most important coding memory aids

These patterns are reusable across many 8085 practical questions.

### Pattern 1: Read, modify, store

```
LDA 2050H
; Modify A
STA 2050H
```

Use when a value in memory must be changed.

### Pattern 2: Counted loop

```
MVI B, 05H
LOOP: ; Work
      DCR B
      JNZ LOOP
```

Use when a block of work must repeat a fixed number of times.

### Pattern 3: Compare and branch

```
LDA 2050H
CPI 10H
JZ EQUAL
```

Use when the next operation depends on whether a value matches a constant.

### Pattern 4: Call a subroutine

```
CALL PROCESS
; Continue here

PROCESS:
    ; Work
    RET
```

Use when an operation should be reusable or separated into a routine.

### Pattern 5: Packed-BCD increment

```
LDA 2050H
ADI 01H
DAA
STA 2050H
```

Use for a packed-BCD counter, provided the input is valid BCD and the increment is within the intended range.

### Mnemonic associations

- `LDA` = Load Accumulator.
- `STA` = Store Accumulator.
- `MVI` = Move Immediate.
- `ADI` = Add Immediate.
- `CPI` = Compare Immediate.
- `DCR` = Decrement Register.
- `JNZ` = Jump if Not Zero.
- `CALL` = Call a subroutine.
- `RET` = Return from a subroutine.
- `JMP` = Jump unconditionally.
- `DAA` = Decimal Adjust Accumulator.

These associations are often enough to reconstruct a forgotten instruction during a practical exam.

## 11. A systematic method for solving unfamiliar assembly questions

When your examiner gives you a new problem, use this workflow.

### Step 1: Identify the input, processing and output

Write down what data enters the program, what transformation is required, and where the result must go.

For the clock:

- Input: initial time values.
- Processing: BCD increment, comparison and rollover.
- Output: updated time in memory and, optionally, a display.

### Step 2: Decide where the data lives

Choose memory locations and register roles before coding. Make a small memory map if several values are involved.

### Step 3: Write the algorithm in plain language

For example:

1. Read seconds.
2. Increment seconds.
3. Correct BCD.
4. Store seconds.
5. If seconds have not reached 60, return.
6. Otherwise, reset seconds and increment minutes.

If you cannot explain the algorithm in plain language, you are not ready to translate it into assembly.

### Step 4: Translate each action into instructions

| Required action          | Likely instruction pattern |
| ------------------------ | -------------------------- |
| Load a constant          | `MVI`                    |
| Read a memory byte       | `LDA`                    |
| Store a result           | `STA`                    |
| Increment a binary value | `INR` or `ADI 01H`     |
| Increment packed BCD     | `ADI 01H`, `DAA`       |
| Compare with a constant  | `CPI`                    |
| Repeat while not zero    | `DCR`, `JNZ`           |
| Repeat a subroutine      | `CALL`, `RET`          |
| Repeat the main program  | `JMP`                    |

### Step 5: Trace the flags

For every conditional instruction, ask:

- Which instruction set the flags?
- What is the Zero flag?
- Does the jump happen?
- Does execution continue or return?

### Step 6: Dry-run the boundary cases

For this experiment, ordinary values alone are insufficient. Test:

- `08 → 09 → 10`
- `58 → 59 → 00` with a minute increment at rollover
- `12:59:59 → 13:00:00`
- `23:59:59 → 00:00:00`

The `08 → 09 → 10` test verifies BCD correction. The other cases verify the rollover logic.

### Step 7: Verify addresses and control flow

In GNUSim8085, labels are useful because they allow the assembler to resolve branch destinations. If you are hand-assembling on a trainer kit, verify every instruction address and every `CALL`/`JMP` target carefully.

## 12. What to keep in mind during the practical exam

### Practical checklist

0/10

Write the aim and identify the memory locations.

Initialize SP before executing CALL and RET.

Initialize the seconds, minutes and hours counters.

Use ADI followed immediately by DAA for BCD increments.

Compare seconds/minutes against 60H and hours against 24H.

Check that the Zero flag is tested after CPI without intervening flag-changing instructions.

Verify that every CALL has a valid return path.

Check that inner loop counters are reinitialized at the correct level.

Test rollover conditions, not just ordinary increments.

Explain how the delay depends on loop counts and processor timing.

### Timing: an important limitation

Your experiment uses a software delay with three nested loops. Its duration depends on:

- Processor clock frequency.
- The execution time of each instruction.
- Loop counts and branch overhead.
- Simulator behavior or trainer-kit implementation.

Therefore, do not claim that the delay is exactly one second just because the counters are set to `03H`, `0FFH`, and `0FFH`. Measure the actual delay and calibrate it.

Also, a software delay is not a hardware real-time clock. Execution time may change with the system or implementation.

### Display limitation

The uploaded experiment uses `DISPLAY: RET` as a placeholder for GNUSim8085. That routine does not physically display the digits. For a trainer kit, you must use the kit's monitor display routine or implement output for the particular display hardware.&#x20;

8085_Digital_Clock.pdf

## 13. Viva questions with answers

1\. What is the aim of this experiment?

2\. Why is DAA used?

3\. Why do we compare seconds with 60H?

4\. What is the purpose of RNZ?

5\. What is the difference between JNZ and RNZ?

6\. Why is the stack pointer initialized?

7\. Why is DCR used in the delay loop?

8\. Does DCR affect the Carry flag?

9\. Why are there three nested loops?

10\. Why reset seconds and minutes to zero?

11\. What happens at 23:59:59?

12\. What does the program use memory locations 2070H–2072H for?

## 14. Final revision sheet

If you have only a few minutes before the practical, remember these seven points:

1. BCD: Each packed-BCD byte stores two decimal digits.
2. Increment: `ADI 01H → DAA → STA`.
3. Rollover: `CPI 60H` for seconds and minutes; `CPI 24H` for 24-hour time.
4. Flags: `JNZ` and `RNZ` both depend on Z, but one jumps and the other returns.
5. Loops: Initialize, execute, decrement, test, repeat. Reload inner counters when necessary.
6. Subroutines: Initialize SP, use `CALL` to enter and `RET` to return.
7. Testing: Verify normal increments, decimal transitions, minute rollover and midnight rollover.

The most useful general principle is to design your program around data, operations, conditions, and repetition. Once you can identify those four parts, most introductory 8085 assembly questions become a translation of a simple algorithm into instruction patterns.





# 8085 Digital Clock — Beginner's Complete Line-by-Line Tutorial

We will start from the assumption that you have never written an assembly-language program before. The objective is to understand not only what every line of your Digital Clock program does, but also why that line is needed, how the processor executes it, and how you can reuse the same logic in other assembly questions.

Your experiment uses the 8085 microprocessor, packed BCD counters, the `DAA` instruction, subroutines, conditional returns, and nested software-delay loops. We will follow the structure of your lab PDF.&#x20;

8085_Digital_Clock.pdf

## Part 1 — Understand what you are programming

### 1.1 What is a microprocessor?

Think of a microprocessor as a machine that repeatedly performs three basic activities:

1. Reads an instruction from memory.
2. Executes that instruction.
3. Moves to the next instruction or jumps to another address.

For example, consider:

```
MVI A, 05H
ADI 03H
```

The processor first puts `05H` into register A. Then it adds `03H` to A.

The final result is:

```
A = 08H
```

That is the basic idea behind assembly programming: you give the processor small, precise instructions to accomplish a larger task.

### 1.2 What is assembly language?

Assembly language uses short instruction names called mnemonics instead of writing the machine-code bytes directly.

For example:

| Assembly instruction | Meaning                                |
| -------------------- | -------------------------------------- |
| `MVI A, 05H`       | Put the value 05H into register A      |
| `ADI 03H`          | Add 03H to register A                  |
| `STA 2070H`        | Store A at memory address 2070H        |
| `JMP MAIN`         | Continue execution from the label MAIN |

The assembler translates these instructions into machine code that the 8085 can execute.

You write the program in assembly language; the processor executes the corresponding machine-code instructions.

### 1.3 What does the `H` mean?

The suffix `H` indicates that a number is written in hexadecimal, or base 16.

Hexadecimal digits are:

`0 1 2 3 4 5 6 7 8 9 A B C D E F`

One hexadecimal digit represents four bits. Therefore, two hexadecimal digits represent one byte.

| Hexadecimal | Decimal | Binary        |
| ----------- | ------- | ------------- |
| `00H`     | 0       | `0000 0000` |
| `05H`     | 5       | `0000 0101` |
| `09H`     | 9       | `0000 1001` |
| `0AH`     | 10      | `0000 1010` |
| `10H`     | 16      | `0001 0000` |
| `FFH`     | 255     | `1111 1111` |

Notice that `10H` is decimal 16, not decimal 10.

This distinction is especially important when working with BCD, which we will cover shortly.

## Part 2 — Understand the processor's working areas

Before understanding the code, you need to know where the processor keeps instructions, data, and temporary results.

### 2.1 Registers: the processor's working space

A register is a small storage location inside the processor. It lets the processor hold data without repeatedly accessing memory.

### Registers used in your program

## A

Accumulator

The primary register for arithmetic, BCD adjustment, and comparisons. Most of your clock calculations happen here.

## B–E

General-purpose registers

Temporary storage. Your delay routine uses C, D, and E as nested loop counters.

## PC

Program Counter

Holds the address of the next instruction to execute. Branches and subroutine calls change the flow of execution.

## SP

Stack Pointer

Holds the address used to manage the stack, which stores return addresses during subroutine calls.

## F

Flag register

Records conditions such as zero, carry, sign, and parity that conditional instructions use to make decisions.

### 2.2 What is memory?

Memory is where the processor stores both instructions and data.

Your experiment uses these memory locations for the clock values:

| Address   | Data stored |
| --------- | ----------- |
| `2070H` | Seconds     |
| `2071H` | Minutes     |
| `2072H` | Hours       |

For example, suppose memory contains:

```
Address    Value
2070H      45H
2071H      30H
2072H      12H
```

The clock represents:

```
12:30:45
```

The three bytes are stored separately. The program does not need to store the entire time as one number.

Reusable tip: Whenever a question involves several values, create a memory map first. This prevents you from confusing the location of a value with the value itself.

### 2.3 Address versus value

Consider:

```
LDA 2070H
```

There are two different things here:

- `2070H` is the memory address.
- The byte stored at that address is the value being loaded.

If memory location `2070H` contains `45H`, then after the instruction:

```
A = 45H
```

The instruction does not put `2070H` into A. It reads the contents of that address.

Compare this with:

```
MVI A, 45H
```

This instruction puts the literal value `45H` into A directly. It does not access memory address `0045H`.

This difference between immediate data and memory-addressed data is fundamental to assembly programming.

## Part 3 — Learn the instructions before reading the full program

We will first understand each instruction family. Then, when we reach the clock program, every line will have a clear purpose.

### 3.1 `MVI` — Move Immediate

Syntax:

```
MVI destination, data
```

Example:

```
MVI A, 05H
```

Meaning:

> Put the literal hexadecimal value 05H into register A.

Before execution:

```
A = unknown or previous value
```

After execution:

```
A = 05H
```

`MVI` can also initialize other registers:

```
MVI B, 05H
MVI C, 03H
MVI D, 0AH
```

When to use it: When you need to initialize a register with a known constant, such as a loop count, an initial value, or a temporary result.

### 3.2 `LXI` — Load Register Pair Immediate

Syntax:

```
LXI register_pair, 16_bit_data
```

Example:

```
LXI SP, 27FFH
```

This loads a 16-bit value into SP.

Unlike `MVI`, which loads an 8-bit value into a register, `LXI` loads a 16-bit value into a register pair or the stack pointer.

Other examples:

```
LXI H, 2050H
LXI B, 3000H
LXI SP, 27FFH
```

Here, `LXI H,2050H` loads the HL pair with the address `2050H`. It is commonly used when working with memory indirectly.

Reusable tip: If the destination is a single 8-bit register, think of `MVI`. If the destination is a 16-bit register pair or SP, think of `LXI`.

### 3.3 `LDA` — Load Accumulator Direct

Syntax:

```
LDA memory_address
```

Example:

```
LDA 2070H
```

Suppose:

```
[2070H] = 45H
```

After execution:

```
A = 45H
```

The square brackets in `[2070H]` mean “the contents of memory address 2070H.”

This notation is useful when reasoning about memory, even though you do not type the brackets into GNUSim8085.

### 3.4 `STA` — Store Accumulator Direct

Syntax:

```
STA memory_address
```

Example:

```
STA 2070H
```

If A contains `46H`, the instruction performs:

```
[2070H] = 46H
```

It writes the accumulator's contents into memory.

It does not change A.

The common pattern is:

```
LDA 2070H
; Perform calculation on A
STA 2070H
```

This is a read–modify–write pattern. You will use it whenever you need to change a value stored in memory.

### 3.5 `ADI` — Add Immediate to Accumulator

Syntax:

```
ADI data
```

Example:

```
MVI A, 05H
ADI 03H
```

The processor adds the immediate value to A:

\\[ A \leftarrow A+03H \\]

Result:

```
A = 08H
```

The 8085 updates its arithmetic flags after `ADI`.

For an 8-bit result:

- Z is set if the result is zero.
- CY is set if there is a carry out of bit 7.
- S reflects bit 7 of the result.
- P records whether the result has even parity.
- AC records the auxiliary carry condition.

The clock uses `ADI 01H` because it needs to increment the current counter by one.

### 3.6 `DCR` — Decrement Register

Syntax:

```
DCR register
```

Example:

```
MVI B, 03H
DCR B
```

The result is:

```
B = 02H
```

It subtracts one from the register.

More examples:

```
05H → 04H
02H → 01H
01H → 00H
00H → FFH
```

The last transition is important. Registers are 8-bit, so decrementing zero wraps around to `FFH`.

`DCR` updates S, Z, P and AC, but does not change CY.

Reusable tip: `DCR` is particularly useful for loops because it changes the Zero flag when a counter reaches zero.

### 3.7 `CPI` — Compare Immediate

Syntax:

```
CPI data
```

Example:

```
MVI A, 05H
CPI 05H
```

The processor compares A with the immediate value. Conceptually, it subtracts the comparison value from A to set the flags, but it does not save the subtraction result into A.

Since the two values are equal:

```
A remains 05H
Z = 1
```

Now consider:

```
MVI A, 04H
CPI 05H
```

The values are unequal, so:

```
A remains 04H
Z = 0
```

The comparison is useful because the following instruction can make a decision based on the flags.

### 3.8 `JNZ` — Jump if Not Zero

Syntax:

```
JNZ label
```

If Z = 0, execution jumps to the specified label. If Z = 1, execution continues with the next instruction.

Example:

```
MVI B, 03H

LOOP: DCR B
      JNZ LOOP
```

Execution:

```
B = 03H
DCR B → B = 02H, Z = 0 → jump
DCR B → B = 01H, Z = 0 → jump
DCR B → B = 00H, Z = 1 → no jump
```

The program exits the loop.

### 3.9 `JMP` — Unconditional Jump

Syntax:

```
JMP label
```

Unlike `JNZ`, `JMP` does not check any condition. It always transfers execution to the specified label.

Example:

```
MAIN: MVI A, 00H
      JMP MAIN
```

This program repeats the same instruction indefinitely. In practical programming, an unconditional jump is useful for a main loop or for skipping a section of code.

### 3.10 `CALL` and `RET`

A subroutine is a named section of code that performs a particular task.

For example:

```
MAIN: CALL PROCESS
      MVI A, 05H

PROCESS:
      MVI B, 03H
      RET
```

Conceptually, when `CALL PROCESS` executes, the processor:

1. Saves the return address on the stack.
2. Transfers execution to `PROCESS`.
3. Executes the subroutine.
4. Uses `RET` to retrieve the return address.
5. Continues after the original `CALL`.

The `MVI A,05H` instruction is therefore executed after the subroutine returns.

For the clock, separate routines handle seconds, minutes, hours and delay. This makes the program easier to understand and test.

Important: The example above is only a demonstration of subroutine flow. A real program must ensure that the stack is initialized and that the subroutine is placed so the intended control flow is valid.

### 3.11 `RNZ` — Return if Not Zero

Syntax:

```
RNZ
```

This instruction checks the Zero flag.

- If Z = 0, return to the address saved by `CALL`.
- If Z = 1, continue with the next instruction.

This is different from `JNZ`, which branches to a named label.

The clock uses `RNZ` after a rollover comparison because it can return immediately when no rollover is needed.

### 3.12 `DAA` — Decimal Adjust Accumulator

We will examine this instruction in more detail later, but for now remember its intended use:

```
ADI 01H
DAA
```

When incrementing a valid packed-BCD value, `DAA` corrects the result to valid packed BCD.

It is not a general-purpose instruction for converting arbitrary binary values into decimal.

## Part 4 — Read the clock program from beginning to end

Now we can understand the complete program line by line.

### Block A: Initialization

```
START:  LXI SP, 27FFH

        MVI A, 00H
        STA 2070H
        STA 2071H
        STA 2072H
```

#### Line 1: `START:`

`START` is a label.

A label gives a meaningful name to an instruction's location. The assembler associates the label with an address.

You could name the label `BEGIN` instead. The label name does not itself execute an instruction.

Labels are useful because you can write:

```
JMP START
```

instead of calculating and entering a numeric jump address.

#### Line 2: `LXI SP, 27FFH`

This initializes the Stack Pointer.

Your program calls several subroutines, and the 8085 uses the stack to preserve return addresses.

The value `27FFH` is the chosen initial stack location in the experiment. It is not a universal requirement for every 8085 program; it must be appropriate for the memory map of the particular system.

#### Line 3: `MVI A, 00H`

This puts zero into A.

```
A = 00H
```

The accumulator is now ready to supply the initial value for the clock counters.

#### Line 4: `STA 2070H`

Stores `00H` in the seconds memory location.

```
[2070H] = 00H
```

#### Line 5: `STA 2071H`

A is still zero, so this initializes minutes:

```
[2071H] = 00H
```

#### Line 6: `STA 2072H`

This initializes hours:

```
[2072H] = 00H
```

Why is there no new `MVI A,00H` before each `STA`?

Because `STA` does not modify A. After the first initialization, A remains `00H` throughout these three stores.

This is a simple but useful optimization: if the accumulator already contains the value you need, you do not need to load it again.

The initialization block now leaves the clock in this state:

| Location  | Value   | Meaning    |
| --------- | ------- | ---------- |
| `2070H` | `00H` | 00 seconds |
| `2071H` | `00H` | 00 minutes |
| `2072H` | `00H` | 00 hours   |

### Block B: Main loop

```
MAIN:   CALL DISPLAY
        CALL DELAY
        CALL INCSEC
        JMP MAIN
```

This is the main control loop. It keeps the clock running.

#### Line 1: `MAIN:`

This is a label identifying the start of the repeated sequence.

#### Line 2: `CALL DISPLAY`

Calls the display subroutine.

In the simulator version, `DISPLAY` may simply contain `RET`, allowing you to inspect the time in memory. On hardware, it must be replaced by a working display implementation.

#### Line 3: `CALL DELAY`

Calls the delay routine.

The intention is to make the program wait approximately one second before advancing the clock.

The `CALL` instruction saves a return address, so when `DELAY` executes `RET`, execution resumes at `CALL INCSEC`.

#### Line 4: `CALL INCSEC`

Calls the routine that increments seconds.

This routine is responsible for deciding whether the seconds value has reached its rollover boundary.

If it reaches `60H`, it resets seconds and calls the minutes routine.

#### Line 5: `JMP MAIN`

After `INCSEC` returns, execution jumps to the beginning of the main loop.

Without this jump, the processor would continue into whatever instruction follows the main loop in memory. That is not the intended behavior.

The resulting sequence is:

```
DISPLAY
   ↓
DELAY
   ↓
INCSEC
   ↓
JMP MAIN
   ↓
DISPLAY
   ↓
...
```

This repeats indefinitely.

Reusable tip: When a question asks for a continuously running program, first identify the main loop. Then place the repeated operations inside it and use an unconditional jump to return to its start.

## Part 5 — The `DISPLAY` subroutine

```
DISPLAY:
        RET
```

This is a placeholder, not a complete display driver.

### Line 1: `DISPLAY:`

Defines the label used by:

```
CALL DISPLAY
```

### Line 2: `RET`

Returns to the instruction after `CALL DISPLAY`.

Since the routine does not execute any display instructions, it does not show digits on a seven-segment display.

Why is this useful in GNUSim8085?

Because you can first test the clock's counting logic independently of the hardware display. Once the memory values update correctly, you can add the display functionality.

This illustrates an important programming technique: build and test a complex program in smaller modules.

For other assembly questions, you can create separate routines for input, calculation, output, and delay. That makes it easier to identify which part is malfunctioning.

## Part 6 — Incrementing seconds, line by line

This is the central routine of the experiment.

```
INCSEC: LDA 2070H
        ADI 01H
        DAA
        STA 2070H
        CPI 60H
        RNZ

        MVI A, 00H
        STA 2070H
        CALL INCMIN
        RET
```

We will trace the routine in two situations: an ordinary increment and a rollover.

### Line 1: `INCSEC: LDA 2070H`

`INCSEC` is the subroutine label.

`LDA 2070H` reads the current seconds value from memory into A.

Suppose the clock currently shows:

```
12:30:45
```

The relevant memory is:

```
[2070H] = 45H
[2071H] = 30H
[2072H] = 12H
```

After `LDA 2070H`:

```
A = 45H
```

The instruction reads the old value; it does not increment it.

### Line 2: `ADI 01H`

Adds one to the accumulator.

```
A = 45H + 01H
A = 46H
```

Since 45 and 46 are valid BCD values, this addition does not require a visible correction to the digits. Nevertheless, the program consistently executes `DAA` afterward.

### Line 3: `DAA`

Corrects the accumulator for packed-BCD arithmetic.

For this example:

```
A = 46H
```

The value remains `46H` after adjustment because the digits are already valid BCD.

Now consider the more interesting case:

```
A = 09H
ADI 01H → A = 0AH
DAA     → A = 10H
```

The same routine works across the decimal digit boundary without requiring a special case for every value ending in 9.

### Line 4: `STA 2070H`

Stores the updated seconds value.

For the first example:

```
[2070H] = 46H
```

The memory now contains the new seconds value.

Notice the order:

1. Read the old value.
2. Increment it.
3. Correct BCD.
4. Store the result.

This sequence is reusable for any packed-BCD counter that needs to be incremented.

### Line 5: `CPI 60H`

Checks whether the updated seconds value equals `60H`.

For the ordinary example:

```
A = 46H
CPI 60H
```

The comparison produces Z = 0 because the values are unequal.

For the rollover example, suppose the old seconds value was `59H`:

```
59H + 01H → 5AH
DAA       → 60H
CPI 60H   → Z = 1
```

The `CPI` instruction does not change A. It only sets the comparison flags.

### Line 6: `RNZ`

This is a conditional return.

For the ordinary increment:

```
A = 46H
Z = 0
RNZ → return to MAIN
```

The routine ends early. It does not execute the reset instructions.

For the rollover case:

```
A = 60H
Z = 1
RNZ → no return
```

Execution continues to the next instruction.

This is how the program avoids unnecessary work on 59 out of every 60 seconds: it returns immediately unless the counter reaches the boundary.

### Line 7: `MVI A, 00H`

This line executes only after the comparison has detected rollover.

It loads zero into A:

```
A = 00H
```

The processor is preparing to reset seconds.

### Line 8: `STA 2070H`

Stores the zero into the seconds location:

```
[2070H] = 00H
```

Seconds have now rolled over from 59 to 00.

### Line 9: `CALL INCMIN`

This calls the minutes routine.

The minutes routine increments minutes by one. If minutes do not reach 60, it returns immediately. If minutes do reach 60, it resets them and calls the hours routine.

This is called cascading rollover.

### Line 10: `RET`

After the minutes routine returns, `RET` returns from `INCSEC` to the instruction after the original `CALL INCSEC` in the main loop.

It is essential to understand that there are two separate returns:

- The `RET` inside `INCMIN` returns to the instruction after `CALL INCMIN`.
- The `RET` at the end of `INCSEC` returns to the main loop.

Each `RET` uses the appropriate address saved by its corresponding `CALL`.

### Complete trace: ordinary increment

Starting from `12:30:45`:

| Step | Instruction   | A       | Seconds memory |
| ---- | ------------- | ------- | -------------- |
| 1    | `LDA 2070H` | `45H` | `45H`        |
| 2    | `ADI 01H`   | `46H` | `45H`        |
| 3    | `DAA`       | `46H` | `45H`        |
| 4    | `STA 2070H` | `46H` | `46H`        |
| 5    | `CPI 60H`   | `46H` | `46H`        |
| 6    | `RNZ`       | `46H` | `46H`        |

Result:

```
12:30:46
```

### Complete trace: seconds rollover

Starting from `12:30:59`:

| Step | Instruction     | A                  | Seconds memory |
| ---- | --------------- | ------------------ | -------------- |
| 1    | `LDA 2070H`   | `59H`            | `59H`        |
| 2    | `ADI 01H`     | `5AH`            | `59H`        |
| 3    | `DAA`         | `60H`            | `59H`        |
| 4    | `STA 2070H`   | `60H`            | `60H`        |
| 5    | `CPI 60H`     | `60H`            | `60H`        |
| 6    | `RNZ`         | `60H`            | `60H`        |
| 7    | `MVI A,00H`   | `00H`            | `60H`        |
| 8    | `STA 2070H`   | `00H`            | `00H`        |
| 9    | `CALL INCMIN` | Depends on routine | `00H`        |
| 10   | `RET`         | —                 | `00H`        |

The final seconds value is `00H`, and minutes have been incremented.

One subtle point: between `MVI A,00H` and `STA 2070H`, A is already zero but seconds memory is still `60H`. The memory becomes zero only when the store executes.

Reusable tip: When tracing assembly code, always distinguish between a value held in a register and a value already written to memory. They are not necessarily the same at every step.

## Part 7 — Incrementing minutes

```
INCMIN: LDA 2071H
        ADI 01H
        DAA
        STA 2071H
        CPI 60H
        RNZ

        MVI A, 00H
        STA 2071H
        CALL INCHOUR
        RET
```

The minutes routine follows the same logic as the seconds routine. Only the memory address and the next subroutine differ.

### Line-by-line explanation

| Instruction      | What it does                              |
| ---------------- | ----------------------------------------- |
| `INCMIN:`      | Defines the minutes subroutine            |
| `LDA 2071H`    | Reads the current minutes into A          |
| `ADI 01H`      | Adds one                                  |
| `DAA`          | Corrects the packed-BCD result            |
| `STA 2071H`    | Stores the updated minutes                |
| `CPI 60H`      | Checks whether minutes have reached 60    |
| `RNZ`          | Returns immediately if minutes are not 60 |
| `MVI A,00H`    | Loads zero for rollover                   |
| `STA 2071H`    | Resets minutes to zero                    |
| `CALL INCHOUR` | Increments hours                          |
| `RET`          | Returns to the caller                     |

### Example: `12:59:00` plus one minute

Suppose the seconds routine has detected a rollover and called `INCMIN`.

The minutes routine starts with:

```
[2071H] = 59H
```

It performs:

```
LDA 2071H → A = 59H
ADI 01H   → A = 5AH
DAA       → A = 60H
STA       → [2071H] = 60H
CPI 60H   → Z = 1
RNZ       → no return
```

It then resets minutes and calls `INCHOUR`.

The time becomes:

```
13:00:00
```

### Why is the minutes routine a separate subroutine?

You could place all the code inside one long block, but separate routines provide three advantages:

1. The same rollover pattern is easy to recognize.
2. Each routine has a single responsibility.
3. You can test each routine independently.

Reusable tip: When two operations follow the same algorithm but use different data locations, compare their code line by line. Often you can create one conceptual template and substitute the relevant addresses and limits.

## Part 8 — Incrementing hours

```
INCHOUR: LDA 2072H
         ADI 01H
         DAA
         STA 2072H
         CPI 24H
         RNZ

         MVI A, 00H
         STA 2072H
         RET
```

The hours routine is similar, but it uses a different rollover limit.

### Why `CPI 24H` instead of `CPI 60H`?

The clock uses a 24-hour format:

```
00, 01, 02, ... 22, 23, then 00
```

After incrementing 23, the result is 24. That is the boundary at which the hours counter must reset.

Therefore, the routine compares A against `24H`.

### Line-by-line explanation

| Instruction   | Purpose                                                       |
| ------------- | ------------------------------------------------------------- |
| `INCHOUR:`  | Marks the beginning of the hours subroutine                   |
| `LDA 2072H` | Loads hours into A                                            |
| `ADI 01H`   | Increments hours                                              |
| `DAA`       | Corrects the BCD result                                       |
| `STA 2072H` | Stores the incremented hours                                  |
| `CPI 24H`   | Tests the 24-hour rollover boundary                           |
| `RNZ`       | Returns if hours are not 24                                   |
| `MVI A,00H` | Prepares zero for rollover                                    |
| `STA 2072H` | Resets hours to zero                                          |
| `RET`       | Returns to the minutes routine or whichever routine called it |

### Midnight rollover

Suppose the clock begins with:

```
23:59:59
```

The following sequence occurs:

1. `INCSEC` increments seconds to `60H`, then resets them to `00H`.
2. `INCSEC` calls `INCMIN`.
3. `INCMIN` increments minutes to `60H`, then resets them to `00H`.
4. `INCMIN` calls `INCHOUR`.
5. `INCHOUR` increments hours to `24H`, then resets them to `00H`.
6. The subroutines return to their callers.

The final memory contents are:

```
[2070H] = 00H
[2071H] = 00H
[2072H] = 00H
```

The time is `00:00:00`.

### The general rollover pattern

This is the central algorithm you can reuse:

```
Increment the current counter.
If the counter is below its limit:
    Return.
Otherwise:
    Reset the current counter.
    Increment the next counter.
    Return.
```

It applies to seconds and minutes, hours, decimal digit counters, and many other counters. The exact representation and rollover boundary depend on the question.

## Part 9 — Understand the delay routine in detail

The delay routine is often the most confusing part for beginners because it uses three counters at the same time.

Here is the code:

```
DELAY:  MVI C, 03H

DL1:    MVI D, 0FFH
DL2:    MVI E, 0FFH

DL3:    DCR E
        JNZ DL3

        DCR D
        JNZ DL2

        DCR C
        JNZ DL1

        RET
```

First, understand its purpose.

The 8085 executes instructions very quickly. If you want to make a clock advance approximately once per second, you need to consume enough processor time between increments.

The delay routine does not measure time directly. Instead, it repeatedly executes instructions.

### 9.1 Line 1: `MVI C, 03H`

This initializes C:

```
C = 03H = 3 decimal
```

C is the outer loop counter.

The outer loop will repeat three times.

### 9.2 Line 2: `DL1: MVI D, 0FFH`

This initializes D to:

```
D = FFH = 255 decimal
```

D is the middle loop counter.

Every time execution reaches `DL1`, D is reinitialized to 255.

This matters because D must count down independently during each outer iteration.

### 9.3 Line 3: `DL2: MVI E, 0FFH`

This initializes E:

```
E = FFH = 255 decimal
```

E is the inner loop counter.

Every time the middle loop starts a new iteration, E is reset to 255.

### 9.4 Line 4: `DL3: DCR E`

Decrements E.

The first few values are:

```
FFH → FEH → FDH → FCH → ...
```

Eventually:

```
02H → 01H → 00H
```

At that point, the Zero flag is set.

### 9.5 Line 5: `JNZ DL3`

This checks the Zero flag set by `DCR E`.

- If E is not zero, the processor jumps back to `DL3`.
- If E is zero, the processor falls through to the next instruction.

Because E starts at `FFH`, the inner loop executes 255 times per middle-loop iteration.

### 9.6 Line 6: `DCR D`

After E reaches zero, execution continues here.

D is decremented once:

```
D = FFH → FEH
```

### 9.7 Line 7: `JNZ DL2`

If D is not zero, execution returns to `DL2`.

At `DL2`, E is initialized to `FFH` again. The inner loop then runs another 255 times.

When D eventually reaches zero, the middle loop ends.

### 9.8 Line 8: `DCR C`

Once D has completed all its iterations, C is decremented.

```
C = 03H → 02H
```

### 9.9 Line 9: `JNZ DL1`

If C is not zero, execution returns to `DL1`.

D is initialized to `FFH` again, and the middle and inner loops repeat.

Once C reaches zero, the complete delay loop ends.

### 9.10 Line 10: `RET`

Returns to the instruction after `CALL DELAY` in the main program.

The clock then proceeds to `CALL INCSEC`.

### 9.11 Visualize the nested loops

### Outer loop: C = 3

Repeats the middle loop three times

### Middle loop: D = 255

Repeats the inner loop 255 times

### Inner loop: E = 255

Executes DCR E and JNZ DL3

### Return from DELAY

The delay is complete

The total number of executions of `DCR E` is:

\\[ 3 \times 255 \times 255 = 195075 \\]

This is a count of inner-loop iterations, not an exact count of processor clock cycles.

### 9.12 How to design nested loops in other questions

Suppose you want an outer loop to execute four times and an inner loop to execute six times per outer iteration.

```
        MVI B, 04H

OUTER:  MVI C, 06H

INNER:  ; Repeated work goes here
        DCR C
        JNZ INNER

        DCR B
        JNZ OUTER
```

The inner loop executes:

\\[ 4 \times 6 = 24 \\]

times in total.

The most important rule is to put each initialization at the correct level:

- Initialize B once, before the outer loop.
- Initialize C inside the outer loop, because C must restart for every outer iteration.

If you move `MVI C,06H` above `OUTER`, the inner counter will not be reinitialized on each outer iteration. That changes the behavior.

### 9.13 A practical warning about zero

Consider:

```
MVI B, 00H
LOOP: DCR B
      JNZ LOOP
```

It does not execute zero iterations.

The first decrement changes B from `00H` to `FFH`. The loop continues until B reaches zero again, after a complete 8-bit wraparound.

For an 8-bit counter decremented and tested this way:

- Starting from `01H` gives one iteration.
- Starting from `05H` gives five iterations.
- Starting from `00H` gives 256 iterations.

Exam tip: Always calculate the loop count from the initialization value and the actual placement of the decrement instruction. Do not assume that zero means no execution.

## Part 10 — BCD arithmetic and flags, explained more deeply

The clock depends on a subtle interaction between arithmetic, BCD correction, comparison, and conditional return.

### 10.1 Ordinary binary addition versus BCD addition

The 8085's `ADI` instruction performs binary addition. It does not automatically know that your data represents decimal digits.

Consider the following:

```
MVI A, 09H
ADI 01H
```

The processor adds the bit patterns:

```
0000 1001
0000 0001
---------
0000 1010
```

The result is `0AH`.

But in packed BCD, each four-bit group must represent a decimal digit from 0 through 9. The lower nibble `AH` is invalid as a decimal digit.

After:

```
DAA
```

the accumulator becomes `10H`, which is valid packed BCD.

### 10.2 Why does `DAA` correct the result?

The instruction uses the result of the preceding addition and relevant auxiliary-carry and carry conditions to determine whether to apply a correction.

In the common case of adding 1 to a valid packed-BCD counter:

- If the low digit remains between 0 and 9, no low-digit correction is needed.
- If the low digit exceeds 9, the instruction applies a correction.
- If the overall result requires a decimal carry into the higher digit, the correction handles that as well.

For example:

| Previous BCD value | After `ADI 01H` | After `DAA` |
| ------------------ | ----------------- | ------------- |
| `07H`            | `08H`           | `08H`       |
| `08H`            | `09H`           | `09H`       |
| `09H`            | `0AH`           | `10H`       |
| `19H`            | `1AH`           | `20H`       |
| `29H`            | `2AH`           | `30H`       |
| `59H`            | `5AH`           | `60H`       |

The routine does not need a separate branch for every digit boundary.

Important distinction: `DAA` is intended to adjust the accumulator after BCD addition. It is not a general instruction for taking any arbitrary binary number and converting it into a decimal display value.

### 10.3 Understand the flags with a real example

Take the clock instruction sequence:

```
LDA 2070H
ADI 01H
DAA
STA 2070H
CPI 60H
RNZ
```

There are several stages:

| Stage   | What happens           | Which flags matter?      |
| ------- | ---------------------- | ------------------------ |
| `LDA` | Loads seconds into A   | No flags changed         |
| `ADI` | Adds one               | Arithmetic flags updated |
| `DAA` | Adjusts the BCD result | Arithmetic flags updated |
| `STA` | Stores A               | No flags changed         |
| `CPI` | Compares A with 60H    | Comparison flags updated |
| `RNZ` | Checks Z               | Reads the Zero flag      |

The key detail is that `CPI 60H` is the last flag-changing instruction before `RNZ`.

Therefore, `RNZ` acts on the result of the comparison, not the earlier addition.

This is a general rule for conditional branches: identify the instruction that most recently produced the relevant flags.

### 10.4 What if another instruction changes the flags?

Consider:

```
CPI 60H
DCR B
JNZ LOOP
```

The `CPI` sets flags based on the comparison. But `DCR B` then changes Z.

As a result, `JNZ LOOP` checks whether B became zero, not whether A equaled `60H`.

This can be intentional, but if you meant to branch on the comparison, the intervening `DCR B` would be a mistake.

### 10.5 `DAA` and carry

The `DAA` instruction can affect the Carry flag as well as the other arithmetic flags. In particular, when the BCD correction produces a carry into a higher decimal position, CY can be set.

For the clock's rollover decision, the program does not rely on CY. It uses:

```
CPI 60H
RNZ
```

This is a useful design choice because the rollover condition is equality with a known BCD boundary.

## Part 11 — All five flags: what they mean and when to use them

You should be able to explain these flags in your practical exam and recognize when a program uses them.

### 11.1 Zero flag (Z)

Z is set when the result of an operation is zero.

Examples:

```
MVI A, 05H
CPI 05H
```

The comparison values are equal, so Z = 1.

```
MVI A, 04H
CPI 05H
```

The comparison values differ, so Z = 0.

Common uses:

- `JZ`: jump if zero.
- `JNZ`: jump if not zero.
- `RZ`: return if zero.
- `RNZ`: return if not zero.

In the clock, Z = 1 after `CPI 60H` indicates a rollover boundary has been reached.

### 11.2 Carry flag (CY)

CY records an arithmetic carry out of the most significant bit during addition, or a borrow during subtraction.

For example:

```
MVI A, 0FFH
ADI 01H
```

The 8-bit result wraps around to `00H`, and CY is set.

The Carry flag can also be useful in multi-byte arithmetic, where a carry from a lower byte must be propagated into a higher byte.

For example, when adding two 16-bit values, you may need to add the low bytes first and then use the carry from that addition while processing the high bytes.

The clock does not use this mechanism to propagate time increments. It explicitly calls the next subroutine when a counter reaches its boundary.

### 11.3 Sign flag (S)

S reflects the most significant bit of the 8-bit result.

- If bit 7 is 0, S = 0.
- If bit 7 is 1, S = 1.

For example, after a suitable arithmetic operation produces `80H`, the result's most significant bit is 1, so S = 1.

The sign flag can help with signed arithmetic, but be careful: a set sign flag alone does not always prove that a mathematical result is negative unless the values are interpreted as signed 8-bit numbers.

### 11.4 Parity flag (P)

P indicates whether the result contains an even number of 1 bits.

For example:

```
03H = 0000 0011
```

There are two set bits, so the parity is even and P = 1.

By contrast:

```
07H = 0000 0111
```

There are three set bits, so the parity is odd and P = 0.

### 11.5 Auxiliary Carry flag (AC)

AC indicates a carry from bit 3 into bit 4 during addition. It is relevant to BCD arithmetic and the correction performed by `DAA`.

It can also be affected by subtraction operations.

### Flag reference table

| Flag | Set when                         | Common use                                     |
| ---- | -------------------------------- | ---------------------------------------------- |
| Z    | Result is zero                   | Equality tests and loop termination            |
| CY   | Carry or borrow condition occurs | Multi-byte arithmetic and unsigned comparisons |
| S    | Result bit 7 is 1                | Signed-result checks                           |
| P    | Result has even parity           | Parity checking                                |
| AC   | Auxiliary carry/borrow condition | BCD adjustment and arithmetic                  |

For this particular clock, the most important flag is Z, because it controls `RNZ` after the rollover comparison and `JNZ` in the delay loops.

## Part 12 — The full program as one logical sequence

Now connect all the routines together.

START

Initialize SP and clock memory

MAIN

Display → Delay → Increment seconds

### INCSEC

Increment seconds, apply DAA, compare with 60H

If not 60H: return to MAIN.

If 60H: reset seconds and call INCMIN.

### INCMIN

Increment minutes, apply DAA, compare with 60H

If not 60H: return to INCSEC.

If 60H: reset minutes and call INCHOUR.

### INCHOUR

Increment hours, apply DAA, compare with 24H

If not 24H: return to INCMIN.

If 24H: reset hours and return.

JMP MAIN

Start the next clock cycle

The program is built around three ideas:

1. A main loop that repeats the clock cycle.
2. Counter subroutines that increment and test individual time units.
3. Conditional rollover that triggers the next counter only when required.

These are general software-design concepts, not just 8085-specific tricks.

## Part 13 — How to use GNUSim8085 to understand the program

Do not immediately run the full program and hope it works. Test it in small stages.

### Stage 1: Check initialization

Assemble the program and run it until the main loop.

Inspect memory:

| Address   | Expected value |
| --------- | -------------- |
| `2070H` | `00H`        |
| `2071H` | `00H`        |
| `2072H` | `00H`        |

If these values are wrong, investigate initialization before debugging the rollover logic.

### Stage 2: Check an ordinary seconds increment

For testing, set the initial seconds value to `45H` and temporarily skip the long delay. Run one increment routine.

Expected result:

```
Before: 2070H = 45H
After:  2070H = 46H
```

Inspect A and the flags after each instruction to understand the execution sequence.

### Stage 3: Check the decimal boundary

Initialize seconds to `09H`.

Expected sequence:

```
LDA 2070H → A = 09H
ADI 01H   → A = 0AH
DAA       → A = 10H
STA       → 2070H = 10H
CPI 60H   → Z = 0
RNZ       → returns
```

This test verifies that BCD adjustment is working as intended.

### Stage 4: Check seconds rollover

Set:

```
2070H = 59H
2071H = 30H
2072H = 12H
```

Execute `INCSEC`.

Expected final memory:

```
2070H = 00H
2071H = 31H
2072H = 12H
```

### Stage 5: Check midnight rollover

Set:

```
2070H = 59H
2071H = 59H
2072H = 23H
```

Execute the rollover logic.

Expected final memory:

```
2070H = 00H
2071H = 00H
2072H = 00H
```

When manually changing the memory values, use the memory editor or initialization code supported by your simulator. Make sure the program reaches the intended subroutine rather than restarting and clearing your test values.

## Part 14 — Apply these ideas to other assembly questions

The strongest way to learn assembly is to identify reusable patterns.

### Example 1: Increment a memory value

Question: Increment the byte at `2050H` by one.

```
LDA 2050H
INR A
STA 2050H
```

This is appropriate for ordinary binary data.

If the byte represents packed BCD, use the BCD-adjustment pattern instead:

```
LDA 2050H
ADI 01H
DAA
STA 2050H
```

The distinction is important: choose the arithmetic operation based on how the data is represented.

### Example 2: Repeat an operation five times

```
MVI B, 05H

LOOP:   ; Perform the required operation
        DCR B
        JNZ LOOP
```

The body executes five times, provided B is not modified elsewhere.

### Example 3: Compare a value with a limit

```
LDA 2050H
CPI 10H
JZ EQUAL
```

This checks equality. It does not check whether the value is greater than or less than the limit.

For ordering comparisons, you must examine the appropriate flags produced by `CPI` and choose a suitable conditional branch.

### Example 4: Reuse a calculation

```
MAIN:   CALL PROCESS
        JMP MAIN

PROCESS:
        ; Calculation goes here
        RET
```

Use a subroutine when a block of work is reusable or when separating the program into logical sections makes it easier to understand.

### Example 5: Process an entire block of data

Suppose a question asks you to copy multiple bytes from one memory area to another.

The general algorithm is:

1. Load the source address into a register pair.
2. Load the destination address into another register pair.
3. Initialize a count.
4. Read one source byte.
5. Write it to the destination.
6. Advance the source and destination pointers.
7. Decrement the count and repeat.

The loop-control structure remains the same. Only the instructions inside the loop change.

This is the key to solving new assembly questions: learn the patterns and then adapt the body of the loop to the required operation.

## Part 15 — Practice questions to test your understanding

Try these before looking up the answers. They are designed to check whether you can reason about instructions rather than merely recognize their names.

## Six-question mini test

6/6 answered

1\. After \`MVI A,09H\` and \`ADI 01H\`, what is in A before DAA?

09H

0AH

10H

0BH

Correct

Ordinary binary addition produces 0AH. DAA subsequently corrects it to 10H.

2\. If A = 46H, what happens after \`CPI 60H\`?

A becomes 14H

A becomes 60H

A stays 46H and Z = 0

A stays 46H and Z = 1

Correct

CPI updates comparison flags but preserves the accumulator.

3\. A loop starts with B = 04H and executes \`DCR B\` followed by \`JNZ LOOP\`. How many times does the loop body execute?

3

4

5

256

Correct answer: 4

The counter changes 04H → 03H → 02H → 01H → 00H. The loop stops when Z becomes 1.

4\. Why must E be initialized inside the middle loop of the delay routine?

To reset the program counter

To reload the inner counter for each middle-loop iteration

To preserve CY

To initialize the stack

Correct

Without reinitialization, the inner loop would not execute the intended number of times on later middle-loop iterations.

5\. What happens when \`RNZ\` executes with Z = 1?

It always jumps to MAIN

It returns from the subroutine

It does not return and execution continues

It clears the accumulator

Correct

RNZ returns only when Z = 0.

6\. Which instruction makes the clock return to its main loop after an ordinary seconds increment?

DAA

CPI

RNZ

DCR

Correct

When seconds are not 60H, CPI sets Z = 0, and RNZ returns to the caller.

Check answers

## Score: 5 / 6

Try again

## Final takeaway

You should now be able to read the Digital Clock program as a sequence of understandable operations rather than a collection of unfamiliar mnemonics.

For every new 8085 question, ask yourself these questions in order:

1. What data am I processing, and where is it stored?
2. Which registers will hold the data temporarily?
3. Which instructions perform the required operation?
4. Which flags will the operation change?
5. What condition determines whether execution continues, branches, or returns?
6. Does the task require a loop or a subroutine?
7. What are the expected results for ordinary and boundary cases?

If you can answer those seven questions before writing the code, you will have a reliable method for approaching the Digital Clock experiment and other introductory 8085 assembly programs.
