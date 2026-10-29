
# 8085 Microprocessor Lab: Elevator and Traffic Light Simulation

Your experiment contains two programs:

1. Elevator simulation: The floor number increases from 0 to 5 and then decreases from 5 to 0 repeatedly.
2. Traffic light control: Different output patterns are sent to a port with delays between them, repeating continuously.

To understand these programs, you need to learn five things:

- What each 8085 instruction does.
- How memory and I/O ports work.
- How flags control decisions and loops.
- How `CALL`, `RET`, and delay loops work.
- How to design an assembly program from a problem statement.

I'll explain the concepts from the beginning, use your exact code, trace how it executes, and then give you a method you can apply to other GNUSim8085 lab questions.

# Part 1 — Understand the basic idea

## 1. What is the 8085 microprocessor?

The 8085 is an 8-bit microprocessor. It executes instructions one after another, using registers, memory, and I/O ports to process data and control external devices.

Think of it like this:

8085 program

Instructions stored in memory

8085 processor

Registers, ALU, flags, program counter

Memory

Stores the current floor or other data

I/O ports

Send output patterns to a device

In your elevator program, memory stores the current floor number. The processor reads it, sends it to an output port, changes it, and repeats.

In your traffic light program, the processor sends output patterns to a port, waits, sends another pattern, waits again, and repeats.

## 2. What are registers?

Registers are small storage locations inside the processor. They are much faster to access than external memory.

The 8085 has several important registers.

| Register              | Purpose in your programs                                         |
| --------------------- | ---------------------------------------------------------------- |
| A — Accumulator      | Holds the floor number, output pattern, and intermediate results |
| B and C               | Together form a 16-bit counter for delays                        |
| D                     | Counts how many times the long delay must repeat                 |
| H and L               | Form a register pair often used for memory addresses             |
| PC — Program Counter | Holds the address of the next instruction                        |
| SP — Stack Pointer   | Tracks the top of the stack                                      |
| Flag register         | Records results such as zero, carry, and sign                    |

### What is an accumulator?

The accumulator is register `A`.

For example:

```
MVI A, 05H
```

This means:

> Put the hexadecimal value 05 into register A.

After this instruction:

```
A = 05H
```

And:

```
INR A
```

means increase A by one.

```
Before: A = 05H
After:  A = 06H
```

The `H` suffix means the number is hexadecimal. `05H` equals decimal 5, and `0AH` equals decimal 10.

### What does an 8-bit register mean?

An 8-bit register can store values from:

\\[ 0 \text{ to } 255 \\]

In hexadecimal, that is:

\\[ 00H \text{ to } FFH \\]

The B and C registers can be combined into a 16-bit register pair, giving values from `0000H` to `FFFFH`.

This is why your delay routines use:

```
LXI B, FFFFH
```

It initializes the BC pair to its maximum 16-bit value.

## 3. What does `H` mean in an instruction?

In 8085 assembly, `H` at the end of a number indicates hexadecimal notation.

| Hexadecimal | Decimal |
| ----------- | ------- |
| `00H`     | 0       |
| `01H`     | 1       |
| `02H`     | 2       |
| `05H`     | 5       |
| `0AH`     | 10      |
| `10H`     | 16      |
| `FFH`     | 255     |
| `FFFFH`   | 65535   |

Be careful: hexadecimal `10H` is decimal 16, not decimal 10.

# Part 2 — Understand every instruction used

Before studying the full programs, learn these instruction groups. They form the foundation of both experiments.

## 4. Data transfer instructions

These move data between registers, memory, and ports.

| Instruction      | Meaning                          |
| ---------------- | -------------------------------- |
| `MVI A, 05H`   | Put 05H into A                   |
| `MOV A, B`     | Copy B into A                    |
| `LDA 8000H`    | Load A from memory address 8000H |
| `STA 8000H`    | Store A at memory address 8000H  |
| `LXI B, FFFFH` | Load BC with FFFFH               |
| `OUT 00H`      | Send A to output port 00H        |
| `IN 00H`       | Read input port 00H into A       |

The distinction between these two instructions is especially important.

### `LDA` versus `MVI`

```
MVI A, 05H
```

Puts the literal value 05H into A.

```
LDA 8000H
```

Reads the value already stored at memory address 8000H into A.

Imagine memory as a row of numbered boxes:

Address

8000H

Stored value

03H

`LDA 8000H` makes A = 03H.

`STA 8000H` stores the current value of A back in that box.

Remember:

- `LDA` = memory to accumulator.
- `STA` = accumulator to memory.
- `MVI` = immediate value to register or memory.
- `MOV` = copy data between registers or between a register and memory through M.

## 5. Arithmetic instructions

| Instruction | Meaning            | Flags affected  |
| ----------- | ------------------ | --------------- |
| `INR A`   | Increase A by 1    | S, Z, P, AC     |
| `DCR A`   | Decrease A by 1    | S, Z, P, AC     |
| `DCX B`   | Decrease BC by 1   | No flags        |
| `INX H`   | Increase HL by 1   | No flags        |
| `CPI 05H` | Compare A with 05H | S, Z, P, AC, CY |

Two instructions that look similar are very different:

- `DCR B` decreases the 8-bit B register.
- `DCX B` decreases the entire 16-bit BC register pair.

Your delay loop needs `DCX B` because the counter starts at `FFFFH`.

For example:

```
BC = 0002H
DCX B
BC = 0001H

DCX B
BC = 0000H
```

The instruction `DCX B` does not change the flags. This matters when designing loops.

## 6. Logic and branching instructions

| Instruction    | Meaning                            |
| -------------- | ---------------------------------- |
| `ORA C`      | Perform bitwise OR between A and C |
| `JNZ LABEL`  | Jump if Zero flag = 0              |
| `JZ LABEL`   | Jump if Zero flag = 1              |
| `JMP LABEL`  | Jump unconditionally               |
| `CALL LABEL` | Call a subroutine                  |
| `RET`        | Return from a subroutine           |
| `HLT`        | Stop execution                     |

You will use `JNZ`, `JZ`, and `JMP` frequently in lab programs.

- `JNZ` asks: Is the Zero flag 0?
- `JZ` asks: Is the Zero flag 1?
- `JMP` does not check a flag at all.

The next section explains how those flags work.

## 7. Flags — the most important concept for loops

The 8085 has five main condition flags.

| Flag | Full name       | Meaning                                                     |
| ---- | --------------- | ----------------------------------------------------------- |
| S    | Sign            | Indicates whether the result's most significant bit is 1    |
| Z    | Zero            | Set to 1 when the result is zero                            |
| AC   | Auxiliary Carry | Used mainly for carry between the lower nibbles             |
| P    | Parity          | Set to 1 if the result has an even number of 1 bits         |
| CY   | Carry           | Indicates carry or borrow in relevant arithmetic operations |

For your experiments, the Zero flag (Z) is the most important.

### How does the Zero flag work?

Suppose:

```
MVI A, 05H
DCR A
```

The result is `04H`. The Zero flag is 0 because the result is not zero.

Now suppose:

```
MVI A, 01H
DCR A
```

The result is `00H`. The Zero flag becomes 1.

That gives you the basic logic behind a loop:

```
DCR B
JNZ LOOP
```

In plain English:

1. Decrease B.
2. Check the Zero flag.
3. If Z = 0, return to `LOOP`.
4. If Z = 1, continue after `JNZ`.

### What is the difference between `JNZ` and `JZ`?

`JNZ LOOP`

Jump if the Zero flag is 0.

Z = 0

Jump to LOOP

`JZ START`

Jump if the Zero flag is 1.

Z = 1

Jump to START

Do not memorize `JNZ` as "jump if the register is nonzero." More precisely, it jumps when the Zero flag from the most recent flag-affecting instruction is 0.

This distinction is essential because some instructions do not change flags.

### Which flags are affected by your instructions?

| Instruction                                 | Flags affected               | Important detail                                    |
| ------------------------------------------- | ---------------------------- | --------------------------------------------------- |
| `MVI`, `MOV`, `LDA`, `STA`, `OUT` | None                         | Data transfer does not set flags                    |
| `INR`, `DCR`                            | S, Z, P, AC                  | CY is preserved                                     |
| `INX`, `DCX`                            | None                         | The 16-bit counter can reach zero without setting Z |
| `CPI`                                     | S, Z, P, AC, CY              | Compares A with an immediate value                  |
| `ORA`                                     | S, Z, P; AC and CY are reset | Used to check whether a value is zero               |
| `JNZ`, `JZ`, `JMP`, `CALL`, `RET` | None                         | Branching itself does not change flags              |

The rule to remember is:

A conditional jump reads flags; it does not calculate them.

# Part 3 — Elevator simulation

Here is the purpose of your first program:

- Start at floor 0.
- Display each floor as the elevator moves upward.
- Stop at floor 5.
- Move downward until floor 0.
- Repeat forever.

## 8. Understand the program's overall structure

Your program has four main sections:

| Section        | Purpose                                    |
| -------------- | ------------------------------------------ |
| `START`      | Initialize the floor to zero               |
| `UP`         | Display the floor and count upward         |
| `DOWN_START` | Set the floor to five                      |
| `DOWN`       | Display the floor and count downward       |
| `DELAY`      | Slow the elevator display so it is visible |

Labels such as `START`, `UP`, and `DOWN` are names for locations in your program. They are not instructions by themselves.

## 9. Initialization: `START`

```
START:  MVI A, 00H
        STA 8000H
```

Line by line:

`MVI A, 00H`

Loads zero into the accumulator.

```
A = 00H
```

`STA 8000H`

Stores the value in A at memory address `8000H`.

```
Memory[8000H] = 00H
```

You are using memory address `8000H` to remember the elevator's current floor.

Why use memory when A already holds the value?

Because the program repeatedly calls a delay subroutine. That routine changes A, so the elevator saves its floor in memory and reloads it whenever needed.

This is a useful assembly programming pattern:

> Store important data in memory when a subroutine might overwrite the register containing it.

## 10. Upward movement: `UP`

Your code:

```
UP:     LDA 8000H
        OUT 00H
        CALL DELAY

        LDA 8000H
        CPI 05H
        JZ DOWN_START

        INR A
        STA 8000H
        JMP UP
```

Let's understand every line.

### Step 1 — Load the current floor

```
LDA 8000H
```

Reads the saved floor from memory into A.

For example:

```
Memory[8000H] = 03H
A = 03H
```

### Step 2 — Display the floor

```
OUT 00H
```

Sends the value in A to output port `00H`.

If A contains `03H`, the port receives `03H`.

The actual display depends on how the elevator hardware or simulator maps port bits to its floor indicators. The instruction itself only sends the value.

### Step 3 — Wait

```
CALL DELAY
```

The processor jumps to the `DELAY` subroutine. When it finishes, `RET` returns execution to the instruction after `CALL`.

This makes the floor display remain visible for some time before changing.

### Step 4 — Check whether the elevator has reached floor 5

```
LDA 8000H
CPI 05H
JZ DOWN_START
```

First, the program reloads the current floor. This is necessary because the delay routine uses A.

Next:

```
CPI 05H
```

means compare A with 05H, without changing A.

Conceptually, the processor calculates:

\\[ A - 05H \\]

to set the flags. The result is not stored in A.

If A equals 05H, the result is zero, so Z = 1.

Then:

```
JZ DOWN_START
```

jumps to `DOWN_START` if Z = 1.

In other words:

- If the current floor is 5, begin the downward phase.
- Otherwise, continue increasing the floor.

### Step 5 — Increase the floor

```
INR A
STA 8000H
```

If the current floor is `03H`, `INR A` changes it to `04H`.

`STA 8000H` saves the new floor.

### Step 6 — Repeat upward movement

```
JMP UP
```

This is an unconditional jump back to `UP`. The program displays the next floor and repeats the same process.

## 11. The upward movement as a trace

Here is how the floor changes. The output column shows the value sent to port `00H`.

| Iteration | Floor displayed | Comparison with 05H | Next action            |
| --------- | --------------- | ------------------- | ---------------------- |
| 1         | 00H             | Not equal           | Increase to 01H        |
| 2         | 01H             | Not equal           | Increase to 02H        |
| 3         | 02H             | Not equal           | Increase to 03H        |
| 4         | 03H             | Not equal           | Increase to 04H        |
| 5         | 04H             | Not equal           | Increase to 05H        |
| 6         | 05H             | Equal               | Jump to `DOWN_START` |

Notice the order: the current floor is output first, and then checked.

That explains why the elevator displays floor 5 before changing direction.

## 12. Changing direction: `DOWN_START`

```
DOWN_START:
        MVI A, 05H
        STA 8000H
```

This initializes the saved floor to 5 again.

You might wonder why this is necessary because the upward phase already reached 5.

It makes the starting value for the downward section explicit. It also means the value saved in memory is definitely 05H when the downward loop begins.

## 13. Downward movement: `DOWN`

```
DOWN:   LDA 8000H
        OUT 00H
        CALL DELAY

        LDA 8000H
        CPI 00H
        JZ START

        DCR A
        STA 8000H
        JMP DOWN
```

This is the downward equivalent of the upward loop.

### Step 1 — Display the current floor

```
LDA 8000H
OUT 00H
CALL DELAY
```

Loads the current floor, outputs it, and waits.

### Step 2 — Check whether the elevator is at floor 0

```
LDA 8000H
CPI 00H
JZ START
```

If A equals 00H, the Zero flag becomes 1 and the program jumps back to `START`.

That begins a new cycle.

### Step 3 — Decrease the floor

```
DCR A
STA 8000H
JMP DOWN
```

If the floor is 03H, `DCR A` changes it to 02H. The new floor is saved, and the program returns to `DOWN`.

The downward sequence is:

```
05 → 04 → 03 → 02 → 01 → 00
```

When it reaches 0, the next iteration jumps back to `START`.

## 14. Important observation about your elevator code

Your program displays floor `05H` twice at the turning point and floor `00H` twice at the cycle boundary.

Why?

- At floor 5, the `UP` loop outputs 05H, detects the maximum, and jumps to `DOWN_START`. The `DOWN` loop then outputs 05H again.
- At floor 0, the `DOWN` loop outputs 00H, detects the minimum, and jumps to `START`. The `UP` loop then outputs 00H again.

This is a consequence of the instruction order, not an error in the comparison logic. If your lab expects a display at each endpoint before reversing direction, your code does that, with a repeated endpoint display.

# Part 4 — The delay subroutine

## 15. Understand your `DELAY` code

```
DELAY:  LXI B, FFFFH

LOOP:   DCX B
        MOV A, B
        ORA C
        JNZ LOOP
        RET
```

This is one of the most useful patterns in your experiment. You can reuse the same idea in many 8085 programs.

### Step 1 — Initialize the counter

```
LXI B, FFFFH
```

Loads the BC register pair with:

```
B = FFH
C = FFH
BC = FFFFH = 65535 decimal
```

B is the high byte and C is the low byte.

Together, they form a 16-bit countdown counter.

### Step 2 — Decrease the counter

```
DCX B
```

Decreases BC by one.

The counter begins at `FFFFH`, then becomes `FFFEH`, `FFFDH`, and so on, until it reaches `0000H`.

### Step 3 — Copy B into A

```
MOV A, B
```

The processor copies the high byte of the counter into A.

For example:

```
B = 12H
C = 00H

After MOV A, B:
A = 12H
```

This instruction does not affect flags.

### Step 4 — Combine A and C using OR

```
ORA C
```

This performs a bitwise OR between A and C, storing the result in A.

Because A contains a copy of B, this effectively checks whether either byte of BC is nonzero.

Remember the OR truth table:

| Input 1 | Input 2 | Result |
| ------- | ------- | ------ |
| 0       | 0       | 0      |
| 0       | 1       | 1      |
| 1       | 0       | 1      |
| 1       | 1       | 1      |

If both B and C are zero, the result of the OR is zero, so Z = 1.

If either byte contains a nonzero bit, Z = 0.

### Step 5 — Repeat if the counter is not zero

```
JNZ LOOP
```

If Z = 0, the program returns to `LOOP` and decrements BC again.

If Z = 1, it continues to `RET`.

### Step 6 — Return to the calling program

```
RET
```

Returns to the instruction after `CALL DELAY`.

## 16. Why not use only `DCX B` and `JNZ`?

You might try this:

```
LOOP:   DCX B
        JNZ LOOP
```

That does not work as intended because `DCX B` does not affect the Zero flag.

The `JNZ` would use whatever Zero flag was left by an earlier flag-affecting instruction. It would not reliably detect whether BC reached zero.

Your original code solves this problem:

```
DCX B
MOV A, B
ORA C
JNZ LOOP
```

The `ORA C` instruction updates the Zero flag based on the two counter bytes. That is what makes the loop work.

This is a general assembly programming lesson:

Whenever you use a conditional jump, identify the instruction that most recently set the flag it checks.

## 17. How many times does the delay loop execute?

BC starts at `FFFFH`, which is 65,535 in decimal.

The loop decrements it until it reaches zero. Therefore, the `DCX B` instruction executes 65,535 times per call to `DELAY`.

The delay is not exactly a fixed number of seconds just because the counter value is fixed. Actual timing depends on instruction execution time and the simulator or hardware clock.

# Part 5 — Traffic light simulation

Your second program uses a similar idea but has a different purpose.

Instead of storing and incrementing a floor number, it sends predefined output patterns to a port.

## 18. Understand the overall sequence

Your code:

```
START:  MVI A, 01H
        OUT 01H
        CALL LONG_DELAY

        MVI A, 02H
        OUT 01H
        CALL SHORT_DELAY

        MVI A, 04H
        OUT 01H
        CALL LONG_DELAY

        MVI A, 02H
        OUT 01H
        CALL SHORT_DELAY

        JMP START
```

The output sequence is:

| Phase  | Value loaded into A | Port output | Delay |
| ------ | ------------------- | ----------- | ----- |
| 1      | `01H`             | `01H`     | Long  |
| 2      | `02H`             | `02H`     | Short |
| 3      | `04H`             | `04H`     | Long  |
| 4      | `02H`             | `02H`     | Short |
| Repeat | `01H`             | `01H`     | Long  |

The program repeats this sequence indefinitely.

The intended red, yellow, and green meanings depend on the hardware wiring. If the three LEDs are connected to bits 0, 1, and 2 respectively, `01H`, `02H`, and `04H` activate one bit at a time.

## 19. Why are the values 01H, 02H, and 04H?

Convert each hexadecimal value into binary.

| Hexadecimal | Binary       | Bit activated |
| ----------- | ------------ | ------------- |
| `01H`     | `00000001` | Bit 0         |
| `02H`     | `00000010` | Bit 1         |
| `04H`     | `00000100` | Bit 2         |

Each value is a power of two:

\\[ 01H=2^0,\qquad 02H=2^1,\qquad 04H=2^2 \\]

That makes these values useful for controlling individual LEDs through output bits.

The instruction:

```
OUT 01H
```

sends the accumulator's current value to port `01H`. Notice that the port address is `01H`; it is not the same thing as the value `01H` stored in A.

## 20. How the main traffic light loop works

Consider the first phase:

```
MVI A, 01H
OUT 01H
CALL LONG_DELAY
```

1. Put `01H` into A.
2. Send `01H` to output port `01H`.
3. Call the long-delay subroutine.
4. When the delay finishes, continue to the next phase.

The next phase follows the same pattern, but sends `02H` and uses `SHORT_DELAY`.

The third phase sends `04H` and uses `LONG_DELAY`.

The fourth phase sends `02H` and uses `SHORT_DELAY`.

Finally:

```
JMP START
```

returns to the beginning, so the traffic light sequence repeats.

Notice the difference from the elevator program:

- Elevator: calculate the next floor using `INR` and `DCR`.
- Traffic light: load predefined patterns using `MVI`.

This is a useful distinction when converting a problem statement into code.

# Part 6 — Long delay and short delay

## 21. Long delay: line-by-line explanation

```
LONG_DELAY:
        MVI D, 05H

L1:     LXI B, FFFFH

L2:     DCX B
        MOV A, B
        ORA C
        JNZ L2

        DCR D
        JNZ L1
        RET
```

This code uses two loops: an inner loop and an outer loop.

Outer loop — `L1`

Repeats the entire delay five times using D.

Inner loop — `L2`

Counts BC down from FFFFH to 0000H.

Check D

Decrease D. If it is not zero, repeat the inner delay.

### Step 1 — Initialize the outer counter

```
MVI D, 05H
```

D is set to 5.

It controls how many times the inner countdown is performed.

### Step 2 — Initialize the inner counter

```
L1: LXI B, FFFFH
```

Every time the program reaches `L1`, BC is reloaded with `FFFFH`.

This is critical: the inner counter is reset on every outer-loop iteration.

### Step 3 — Run the inner loop

```
L2:     DCX B
        MOV A, B
        ORA C
        JNZ L2
```

This is exactly the same countdown pattern as the elevator's `DELAY` routine.

It continues until BC reaches zero.

### Step 4 — Decrease the outer counter

```
DCR D
JNZ L1
```

After the inner loop finishes:

- D changes from 05H to 04H.
- `JNZ L1` jumps back because D is nonzero.
- BC is reinitialized to FFFFH.
- The inner loop runs again.

After five inner-loop runs, D becomes zero, and the program reaches `RET`.

### Trace of the outer loop

| Outer iteration | D at start | D after `DCR D` | Repeat? |
| --------------- | ---------- | ----------------- | ------- |
| 1               | 05H        | 04H               | Yes     |
| 2               | 04H        | 03H               | Yes     |
| 3               | 03H        | 02H               | Yes     |
| 4               | 02H        | 01H               | Yes     |
| 5               | 01H        | 00H               | No      |

Thus, the long delay performs five complete inner countdowns.

## 22. Short delay

```
SHORT_DELAY:
        LXI B, FFFFH

L3:     DCX B
        MOV A, B
        ORA C
        JNZ L3
        RET
```

The short delay has only one inner countdown.

The long delay repeats the same countdown five times, while the short delay repeats it once.

In broad terms:

- Short delay = one countdown.
- Long delay = five countdowns.

The long delay is therefore substantially longer, though the exact ratio depends on instruction overhead and execution timing.

# Part 7 — Understanding loops for any 8085 program

This is the most reusable part of your experiment.

## 23. Every loop has three basic elements

A loop generally needs:

1. Initialization: Set the counter to its starting value.
2. Update: Change the counter during each iteration.
3. Condition: Decide whether to repeat or exit.

For example:

```
MVI B, 05H

LOOP:   ; Loop body
        DCR B
        JNZ LOOP
```

This is an 8-bit countdown loop.

Let's trace it.

| Iteration | B before `DCR B` | B after `DCR B` | Z | `JNZ LOOP` |
| --------- | ------------------ | ----------------- | - | ------------ |
| 1         | 05H                | 04H               | 0 | Jump         |
| 2         | 04H                | 03H               | 0 | Jump         |
| 3         | 03H                | 02H               | 0 | Jump         |
| 4         | 02H                | 01H               | 0 | Jump         |
| 5         | 01H                | 00H               | 1 | Exit         |

The loop body executes five times.

## 24. How to decide between `DCR`, `DCX`, `JNZ`, and `JZ`

Use this decision guide.

Need an 8-bit counter?

Use `MVI B, count`, followed by `DCR B`. Then use `JNZ` if the decrement sets the Zero flag and you want to repeat until B reaches zero.

Need a 16-bit counter?

Use `LXI B, FFFFH`, followed by `DCX B`. Since `DCX` does not affect flags, use an instruction such as `MOV A,B` followed by `ORA C` before `JNZ`.

Need to stop when a value equals a target?

Use `CPI target` followed by `JZ EXIT`. This is the pattern used in your elevator program.

Need to repeat a fixed sequence forever?

Give the sequence a label and end it with `JMP` to that label, as in your traffic light program.

## 25. A common mistake: `DCR B` with the wrong jump

Consider:

```
MVI B, 04H

LOOP:   DCR B
        JZ LOOP
```

This is not a correct countdown loop if the intention is to execute four times and then exit.

When B reaches zero, `JZ LOOP` jumps back to `LOOP`. The next `DCR B` wraps B from `00H` to `FFH`, so the counter continues through the full 8-bit range.

The usual countdown pattern is:

```
MVI B, 04H

LOOP:   DCR B
        JNZ LOOP
```

Here, the program repeats while B is nonzero and exits when B becomes zero.

Memory aid: For a counter that decreases to zero, think "decrease, then jump if not zero."

## 26. How to count loop iterations correctly

Never guess the number of iterations from the initial value alone. Trace the counter and check whether the test happens before or after the update.

For example:

```
MVI B, 04H

LOOP:   DCR B
        JNZ LOOP
```

The loop body after the label includes `DCR B`. It executes four times.

But consider:

```
MVI B, 04H

LOOP:   MOV A, B
        DCR B
        JNZ LOOP
```

Here, `MOV A,B` also executes four times. The same count applies because the first decrement happens before the first conditional jump, and the counter starts at four.

Now consider a loop that checks before decrementing:

```
MVI B, 04H

LOOP:   MOV A, B
        ORA A
        JZ EXIT
        DCR B
        JMP LOOP

EXIT:
```

The body before `JZ EXIT` runs five times if you count the final zero-check, but the nonzero processing happens four times. This distinction between loop-body executions and successful processing iterations is important in viva questions.

# Part 8 — `CALL`, `RET`, and the stack

Both programs use subroutines to avoid writing the same delay code repeatedly.

## 27. What is a subroutine?

A subroutine is a reusable section of code that performs a particular task.

For example:

```
CALL DELAY
```

means:

> Pause the main program's execution and execute the instructions under the `DELAY` label.

When the delay finishes:

```
RET
```

returns execution to the main program.

Consider this simplified sequence:

```
START:  MVI A, 05H
        CALL DELAY
        INR A
        HLT

DELAY:  LXI B, FFFFH
        ; Delay loop
        RET
```

Execution proceeds as follows:

1. Load 05H into A.
2. Reach `CALL DELAY`.
3. Save the return address on the stack and jump to `DELAY`.
4. Execute the delay routine.
5. Execute `RET`.
6. Return to the instruction after `CALL DELAY`.
7. Execute `INR A`.

The accumulator's value becomes 06H, assuming the delay routine has not changed it. In your actual delay routine, A is modified, which is why the elevator reloads its floor from memory after the delay.

## 28. What is the stack?

The stack is a region of memory used to store temporary information, including return addresses for subroutines.

When the processor executes `CALL`, it saves the return address on the stack. When it executes `RET`, it retrieves that address and returns to it.

The Stack Pointer (SP) tracks the top of the stack.

Main program

`CALL DELAY`

Stack

Stores the return address

DELAY subroutine

`...` then `RET`

Return to the instruction after CALL

The 8085 stack normally grows toward lower memory addresses when values are pushed. `CALL` and `RET` handle the return address automatically, provided the stack has been initialized appropriately.

For a standalone GNUSim8085 program, make sure the stack pointer is configured appropriately if your simulator setup does not initialize it for you. A common explicit initialization is:

```
LXI SP, 27FFH
```

The selected address must be valid RAM and must not conflict with your program, data, or other stack usage in the particular memory layout.

## 29. What happens if you forget `RET`?

Suppose you write:

```
DELAY:  LXI B, FFFFH
        ; Countdown loop
```

but forget `RET`.

The processor will not automatically return to the instruction after `CALL`. It will continue executing whatever instructions follow the delay code in memory.

This can make the program behave unpredictably.

Remember:

- `CALL` enters a subroutine.
- `RET` returns from a subroutine.
- `JMP` jumps somewhere without saving a return address.

# Part 9 — GNUSim8085 practical procedure

The exact menus can vary between GNUSim8085 versions, but the essential workflow is the same.

## 30. How to run the elevator program

1. Open GNUSim8085 and create a new program.
2. Enter the complete elevator code, including the `DELAY` routine.
3. Assemble the program and check for syntax errors.
4. Set the program counter to the program's starting address if required.
5. Initialize the stack pointer if needed for `CALL` and `RET`.
6. Run the program, or execute it step by step.
7. Observe register A, memory address `8000H`, and output port `00H`.
8. Verify that the output progresses from 00H to 05H and back to 00H.

For the traffic light program, observe output port `01H` and verify the sequence `01H → 02H → 04H → 02H`.

### What should you inspect while debugging?

| Item          | Elevator                           | Traffic light                       |
| ------------- | ---------------------------------- | ----------------------------------- |
| Accumulator A | Current floor and temporary values | Current output pattern              |
| Memory        | `8000H` stores the current floor | Not required for the main sequence  |
| BC            | Delay counter                      | Delay counter                       |
| D             | Not used by the delay              | Outer long-delay counter            |
| Zero flag     | Check `CPI` and `ORA` results  | Check `ORA` and `DCR D` results |
| Output port   | `00H`                            | `01H`                             |
| Stack         | Required for `CALL` and `RET`  | Required for `CALL` and `RET`   |

## 31. How to use single-step execution

When learning the program, avoid immediately running it at full speed. Step through a few iterations first.

For the elevator, watch this sequence:

```
LDA 8000H
OUT 00H
CALL DELAY
LDA 8000H
CPI 05H
JZ DOWN_START
INR A
STA 8000H
JMP UP
```

Ask yourself at each instruction:

- What is in A?
- What is stored at `8000H`?
- Has the output port changed?
- Which instruction last changed the flags?
- Will `JZ` jump or continue?
- Where will the program counter go next?

For the traffic light, step through the first phase and inspect how `CALL LONG_DELAY` enters the subroutine, how the loops change BC and D, and how `RET` resumes the main sequence.

# Part 10 — How to write the programs in an exam

## 32. A systematic method for solving assembly questions

When your examiner gives you a new problem, do not immediately start memorizing or writing instructions. Translate the problem into a sequence of operations.

## 1

Identify the inputs and outputs

Ask what data is required, where it comes from, and where the result must go. For your elevator, the floor is stored in memory and sent to port 00H.

## 2

Identify the counters and variables

Decide which registers will hold the counter, data, or temporary values. Use memory if a value must survive a subroutine that changes its register.

## 3

Write the logic in plain English

For example: display the floor, wait, check whether it is 5, otherwise increment and repeat.

## 4

Choose the instructions

Use `MVI` or `LXI` for initialization, arithmetic instructions for updates, `CPI` for comparisons, and conditional jumps for decisions.

## 5

Check the flags and loop termination

For every `JZ` or `JNZ`, identify the preceding instruction that sets the relevant flag. Confirm that the loop eventually exits, unless it is intentionally infinite.

## 6

Trace a small example

Simulate the first few iterations on paper. Check the values of the registers, memory, flags, and branch decisions before running the complete program.

## 33. Memory aids for the most common instructions

These are useful for both writing code and answering viva questions.

| Instruction | Memory aid                                          |
| ----------- | --------------------------------------------------- |
| `MVI`     | Move an immediate value into a register or memory   |
| `LXI`     | Load a 16-bit register pair                         |
| `MOV`     | Copy between locations                              |
| `LDA`     | Load A from memory                                  |
| `STA`     | Store A into memory                                 |
| `INR`     | Increase by one                                     |
| `DCR`     | Decrease by one                                     |
| `INX`     | Increase a register pair                            |
| `DCX`     | Decrease a register pair                            |
| `CPI`     | Compare A with an immediate value                   |
| `ORA`     | Bitwise OR; often used to test whether data is zero |
| `JZ`      | Jump when zero                                      |
| `JNZ`     | Jump when not zero                                  |
| `JMP`     | Jump without a condition                            |
| `CALL`    | Call a subroutine                                   |
| `RET`     | Return from a subroutine                            |
| `OUT`     | Send A to an output port                            |
| `HLT`     | Halt execution                                      |

### The most useful loop template

For a small countdown:

```
MVI B, 05H

LOOP:   ; Your instructions
        DCR B
        JNZ LOOP
```

For a larger delay using BC:

```
LXI B, FFFFH

LOOP:   DCX B
        MOV A, B
        ORA C
        JNZ LOOP
```

For checking whether a value equals a target:

```
LDA 8000H
CPI 05H
JZ TARGET
```

For repeating a sequence indefinitely:

```
START:  ; Your sequence
        JMP START
```

These are patterns to adapt, not universal templates. The right choice depends on the data size, where the condition is tested, and whether the loop should terminate.

# Part 11 — Exam checklist and common mistakes

Before submitting either program, check these points.

## Pre-submission checklist

0/10

Every label is spelled consistently, including its references.

Every conditional jump has a valid flag-setting instruction before it.

The counter is initialized before the loop begins.

The loop updates its counter in the correct direction.

The boundary condition is checked at the intended time.

Every CALL has a matching RET in its subroutine.

The correct output port is used.

The code has the required HLT or intentional infinite loop.

The expected register and memory values have been traced.

The program assembles without errors in GNUSim8085.

Reset checklist

## 34. Common mistakes to avoid

Mistake 1: Using `JNZ` immediately after `DCX B`.

`DCX B` does not update the flags. Check BC with `MOV A,B` and `ORA C` first.

Mistake 2: Assuming `MOV A,B` changes flags.

It does not. The following `ORA C` is what updates the relevant flags in your delay loop.

Mistake 3: Forgetting to reset an inner counter.

In `LONG_DELAY`, `LXI B, FFFFH` must be inside the outer loop so BC starts over for every long-delay repetition.

Mistake 4: Confusing an output port with a memory address.

`STA 8000H` writes to memory, whereas `OUT 00H` writes to an I/O port. They perform different operations.

Mistake 5: Expecting `CALL` to preserve all registers.

A subroutine can modify registers, including A. If the main program needs a value afterward, preserve it or reload it, as your elevator program does.

Mistake 6: Using the wrong condition for loop termination.

For a countdown that should stop at zero, `DCR B` followed by `JNZ LOOP` is a standard pattern. `JZ LOOP` reverses the intended decision.

Mistake 7: Forgetting that comparisons happen at a specific point in the program.

In your elevator program, the display instruction comes before the boundary check. That is why the endpoint is output before the direction changes.

# Part 12 — Viva questions with answers

1\. Why is the 8085 called an 8-bit microprocessor?2. What is the purpose of accumulator A?3. What is the difference between INR and INX?4. What is the difference between DCR and DCX?5. Why is CPI used in the elevator program?6. Why does JZ jump to DOWN_START?7. Why are B and C used together in the delay?8. Why is ORA C used after MOV A,B?9. What is the difference between CALL and JMP?10. What does RET do?11. Why does the long delay use register D?12. Why does the traffic light program use 01H, 02H and 04H?13. Why is JMP START used in the traffic light program?14. What happens if the Zero flag is already 1 before JNZ?15. Does OUT affect the flags?

# Final revision: what you should remember

You do not need to memorize every assembly program as a completely separate problem. Learn the underlying patterns and adapt them to the question.

### Elevator

`LDA → OUT → CALL DELAY → CPI → JZ → INR/DCR → STA → JMP`

Read the floor, display it, wait, test the boundary, update the floor, and repeat.

### Traffic light

`MVI → OUT → CALL DELAY → MVI → OUT → ... → JMP START`

Send each output pattern, wait for the required duration, and repeat the sequence.

### 16-bit delay

`LXI B → DCX B → MOV A,B → ORA C → JNZ LOOP`

Initialize BC, decrement it, test whether both bytes are zero, and repeat until the counter reaches zero.

### Long delay

`MVI D → inner countdown → DCR D → JNZ outer loop`

Use an outer counter to repeat the complete inner delay a fixed number of times.

The single most useful habit is to trace the register values and flags after each instruction. When you can explain why a conditional jump is taken, which register changes, and where execution goes next, you are no longer simply memorizing assembly code—you are understanding how to construct and debug it.
