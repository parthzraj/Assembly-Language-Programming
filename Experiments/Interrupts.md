
# 8085 Microprocessor Lab: Interrupt Programming — Complete Beginner's Guide

Your uploaded presentation is about Interrupt Programming using the 8085 Microprocessor. It covers two experiments:

1. Hardware interrupt using RST 7.5 — the processor responds to an external interrupt request and executes an Interrupt Service Routine (ISR).
2. Software interrupt using RST 5 — the program deliberately executes an instruction that transfers control to a fixed memory address.

I'll use your PPT as the primary source and explain the theory, every important instruction, the program logic, affected flags, loops, GNUSim8085 execution, exam strategy, and memory aids in beginner-friendly language.&#x20;

Interrupt Programming using 8085 Microprocessor.pptx

The goal is not just to memorize two programs. It is to understand the rules well enough to solve a similar interrupt question in your practical exam.

# Part 1 — Understand the experiment before writing code

## 1. What is a microprocessor?

A microprocessor is a chip that executes instructions stored in memory.

The 8085 performs operations such as:

- Moving data between registers and memory.
- Adding and subtracting numbers.
- Checking conditions using flags.
- Repeating instructions using loops.
- Responding to external devices through interrupts.

Imagine the processor is a student solving a question paper. Normally, it continues working through the questions in order. If an urgent announcement arrives, it pauses its current work, handles the announcement, and then returns to the question where it stopped.

That is the basic idea of an interrupt.

## 2. What is an interrupt?

An interrupt is a request that makes the processor temporarily stop its current program and execute another routine.

For example, imagine the processor is continuously updating a counter. A push button is connected to the RST 7.5 interrupt pin.

Main program

Counter is running normally

Button press → RST 7.5 request

CPU recognizes the request

ISR — Interrupt Service Routine

Perform the interrupt-specific task

RET → Main program resumes

Continues from the interrupted point

The processor does not normally restart the main program from the beginning. It returns to the saved continuation address.

### What is an ISR?

ISR stands for Interrupt Service Routine.

It is a small program that runs when a particular interrupt is accepted.

For example:

```
ISR75:  PUSH PSW
        PUSH B
        PUSH H

        LXI H, 2300H
        INR M

        POP H
        POP B
        POP PSW

        EI
        RET
```

This routine preserves registers, increments a memory counter, restores the registers, enables future maskable interrupts, and returns to the main program.

We'll break down every instruction later.

## 3. Polling versus interrupt

These are two ways for a processor to respond to an event.

| Polling                                                   | Interrupt                                                     |
| --------------------------------------------------------- | ------------------------------------------------------------- |
| CPU repeatedly checks a device's status.                  | Device requests the CPU's attention.                          |
| CPU may spend time repeatedly checking.                   | CPU can continue its normal work until a request is accepted. |
| Example: continuously checking whether a key was pressed. | Example: responding to a button press through RST 7.5.        |

Exam definition: An interrupt allows a processor to respond to an event without continuously polling the device.

# Part 2 — The concepts you must know first

## 4. Registers in the 8085

A register is a small storage location inside the processor.

| Register              | Simple purpose                                          |
| --------------------- | ------------------------------------------------------- |
| A — Accumulator      | Commonly used for arithmetic and logical operations.    |
| B, C, D, E, H, L      | General-purpose 8-bit registers.                        |
| BC, DE, HL            | Register pairs that can hold 16-bit values.             |
| PC — Program Counter | Holds the address of the next instruction to execute.   |
| SP — Stack Pointer   | Holds the address associated with the top of the stack. |
| Flag register         | Records conditions resulting from certain operations.   |

### What does 8-bit or 16-bit mean?

- An 8-bit register can represent 256 different bit patterns, from `00H` to `FFH`.
- A 16-bit register pair can represent 65,536 different bit patterns, from `0000H` to `FFFFH`.

Here, `H` means the number is written in hexadecimal.

Examples:

- `10H` = decimal 16.
- `22H` = decimal 34.
- `FFH` = decimal 255.
- `003CH` = hexadecimal address 003C.

## 5. What is the stack?

The stack is an area of memory used for temporarily saving information, including return addresses and register values.

Think of it as a pile of books: the last book placed on top is the first one removed.

This principle is called LIFO — Last In, First Out.

The instructions in your experiment use the stack in two ways:

- `CALL` and `RET` save and recover a return address.
- `PUSH` and `POP` save and restore register contents.

### What does the Stack Pointer do?

The Stack Pointer (`SP`) keeps track of the top of the stack. In the 8085, the stack grows toward lower memory addresses.

Your program begins with:

```
LXI SP, 27FFH
```

This initializes the stack pointer to `27FFH`.

Why do this first? Because `CALL`, `RET`, `PUSH`, and `POP` rely on the stack. An incorrectly initialized stack can cause return addresses or saved register values to be stored in the wrong memory locations.

Exam rule: Initialize `SP` before enabling interrupts or using stack-dependent instructions.

## 6. What is a flag?

A flag is a single bit that records a condition resulting from an operation.

The 8085 has five principal condition flags.

| Flag                  | Meaning                       | When it becomes 1                                              |
| --------------------- | ----------------------------- | -------------------------------------------------------------- |
| S — Sign             | Sign of result                | Result's most significant bit is 1.                            |
| Z — Zero             | Zero result                   | Result is `00H`.                                             |
| AC — Auxiliary Carry | Carry between bit 3 and bit 4 | Relevant lower-nibble carry occurs.                            |
| P — Parity           | Even parity                   | Result contains an even number of 1 bits.                      |
| CY — Carry           | Carry or borrow indicator     | An applicable arithmetic operation produces a carry or borrow. |

Not every instruction affects these flags.

For example:

```
INR B
```

Increments B by one and affects S, Z, AC, and P, but does not affect CY.

By contrast:

```
DCX D
```

Decrements the DE register pair and does not affect the condition flags.

This distinction matters in interrupt programming because an ISR can accidentally change flags that the interrupted main program depends on. `PUSH PSW` and `POP PSW` help preserve the accumulator and flag register.

## 7. The most important instructions for this experiment

These are the instructions you should recognize immediately in the practical exam.

| Instruction       | Easy meaning                                                    |
| ----------------- | --------------------------------------------------------------- |
| `MVI A, 0BH`    | Put the 8-bit value `0BH` into A.                             |
| `LXI SP, 27FFH` | Load a 16-bit address into SP.                                  |
| `MOV A, B`      | Copy B into A.                                                  |
| `INR B`         | Increase B by 1.                                                |
| `INR M`         | Increase the memory byte addressed by HL by 1.                  |
| `DCR B`         | Decrease B by 1.                                                |
| `JMP LABEL`     | Jump unconditionally to a label.                                |
| `JNZ LABEL`     | Jump if the Zero flag is 0.                                     |
| `CALL LABEL`    | Save a return address and jump to a subroutine.                 |
| `RET`           | Return using the address saved on the stack.                    |
| `PUSH B`        | Save the BC register pair on the stack.                         |
| `POP B`         | Restore the BC register pair from the stack.                    |
| `EI`            | Enable maskable interrupts globally.                            |
| `DI`            | Disable maskable interrupts globally.                           |
| `SIM`           | Apply interrupt-mask controls and other serial-output controls. |
| `RIM`           | Read interrupt status and mask information.                     |
| `RST 5`         | Transfer control to software RST 5's vector at `0028H`.       |
| `HLT`           | Halt normal instruction execution.                              |

### What does `M` mean?

In 8085 assembly, `M` means the memory location whose address is held in the HL register pair.

Suppose:

```
LXI H, 2300H
MVI M, 05H
INR M
```

Step by step:

1. `LXI H, 2300H` makes HL point to memory address `2300H`.
2. `MVI M, 05H` stores `05H` at address `2300H`.
3. `INR M` increments that stored byte to `06H`.

So the final result is:

\\[ [2300H]=06H \\]

The square brackets mean “the contents of the memory location at this address.”

Remember: `INR H` changes the H register; `INR M` changes the memory byte addressed by HL.

# Part 3 — All the 8085 interrupts

## 8. Hardware interrupts

A hardware interrupt is generated by a signal from outside the CPU.

The 8085 has five hardware interrupt inputs.

## TRAP

Vector: `0024H`

Highest-priority hardware interrupt. It is non-maskable, so `DI` cannot disable it.

## RST 7.5

Vector: `003CH`

Second-highest priority. Rising-edge triggered and internally latched.

## RST 6.5

Vector: `0034H`

Level-triggered and maskable.

## RST 5.5

Vector: `002CH`

Level-triggered and maskable.

## INTR

No fixed vector address

Lowest-priority hardware interrupt. The external device supplies an instruction during interrupt acknowledgement.

Memorize the priority order:

TRAP → RST 7.5 → RST 6.5 → RST 5.5 → INTR

A common viva question asks which interrupt has the highest priority. The answer is TRAP.

### Edge-triggered versus level-triggered

- Edge-triggered: A change in the signal triggers the request. RST 7.5 detects a rising edge, from low to high.
- Level-triggered: The request is recognized while the input remains at its active logic level. RST 6.5 and RST 5.5 are high-level triggered.

RST 7.5 has an internal latch, so its rising-edge request can remain pending until serviced or cleared.

## 9. What is a vector address?

A vector address is the fixed memory address at which the processor starts handling an interrupt.

For example, RST 7.5 has vector address `003CH`.

But there is a problem: consecutive RST vector addresses are only eight bytes apart. A complete interrupt routine may need more than eight bytes.

The solution is to place a short jump instruction at the vector address.

```
ORG 003CH
JMP ISR75

ORG 3000H
ISR75:  ; full interrupt routine starts here
```

This means:

1. The interrupt occurs.
2. The CPU starts executing at `003CH`.
3. `JMP ISR75` redirects execution to the routine at `3000H`.
4. The ISR executes.
5. `RET` returns to the interrupted program.

The same pattern works for the software interrupt in your PPT:

```
ORG 0028H
JMP ISR5

ORG 3100H
ISR5:   ; software interrupt routine
```

### Software interrupt addresses

For the software instructions `RST 0` through `RST 7`, use:

\\[ \text{Vector address}=8\times n \\]

Here \\(n\\) is the RST number, and the address is expressed in hexadecimal.

| Instruction | Vector address |
| ----------- | -------------- |
| `RST 0`   | `0000H`      |
| `RST 1`   | `0008H`      |
| `RST 2`   | `0010H`      |
| `RST 3`   | `0018H`      |
| `RST 4`   | `0020H`      |
| `RST 5`   | `0028H`      |
| `RST 6`   | `0030H`      |
| `RST 7`   | `0038H`      |

Memory aid: RST number × 8 = vector address.

For example, `RST 5` → \\(5\times8=40\\) decimal → `28H`.

# Part 4 — Interrupt enable and masking

## 10. What do EI and DI do?

The 8085 uses a global interrupt-enable flip-flop for maskable hardware interrupts.

- `EI` sets the global enable state.
- `DI` clears it.
- TRAP is not affected by this global control.

After reset, maskable interrupts are disabled.

Even if RST 7.5 is individually unmasked, the CPU also needs `EI` before it can accept that maskable interrupt.

Think of the two controls as two doors:

- The individual mask decides whether a particular interrupt is allowed.
- The global enable decides whether maskable interrupts can be accepted generally.

Both must allow the request.

## 11. What is SIM?

`SIM` stands for Set Interrupt Mask.

It uses bits in the accumulator to control the individual masks for RST 7.5, RST 6.5, and RST 5.5.

Your program contains:

```
MVI A, 0BH
SIM
EI
```

Let's decode it.

`0BH` in binary is:

D7

D6

D5

D4

D3

D2

D1

D0

## 0

## 0

## 0

## 0

## 1

## 0

## 1

## 1

Binary: 0000 1011 = 0BH

The relevant bits are:

| Bit        | Value | Meaning                  |
| ---------- | ----- | ------------------------ |
| D3 — MSE  | 1     | Apply the mask settings. |
| D2 — M7.5 | 0     | Unmask RST 7.5.          |
| D1 — M6.5 | 1     | Mask RST 6.5.            |
| D0 — M5.5 | 1     | Mask RST 5.5.            |

`MSE` means Mask Set Enable. It must be 1 for the individual mask bits to take effect.

So the memory aid is:

`0BH` + `SIM` + `EI` = unmask RST 7.5 and globally enable maskable interrupts.

The RST 6.5 and RST 5.5 lines remain masked.

## 12. What is RIM?

`RIM` stands for Read Interrupt Mask.

It reads the interrupt status and mask information into the accumulator. You can use it to inspect pending requests and the current mask/global-enable states.

Example from your presentation:

```
RIM
ANI 40H
JNZ PENDING
```

Here's the logic:

1. `RIM` places interrupt status information in A.
2. `ANI 40H` performs a bitwise AND with `40H`.
3. This isolates bit D6, which represents the RST 7.5 pending status in the RIM result.
4. If the resulting value is nonzero, `JNZ PENDING` jumps to `PENDING`.

A caution: `ANI` changes the flags, including Z and CY. It is not just a read-only test.

# Part 5 — Program A: Hardware interrupt using RST 7.5

This is the main experiment in your PPT. The following code follows its memory layout and logic.&#x20;

Interrupt Programming using 8085 Microprocessor.pptx

## 13. Complete GNUSim8085 program

The example uses one memory location for the main counter (`2200H`) and another for the interrupt-hit counter (`2300H`).

```
        ORG 2000H

        LXI SP, 27FFH
        MVI A, 0BH
        SIM
        EI

        MVI B, 00H

MAIN:   MOV A, B
        STA 2200H
        CALL DELAY
        INR B
        JMP MAIN

DELAY:  LXI D, 0FFFH

BACK:   DCX D
        MOV A, D
        ORA E
        JNZ BACK
        RET

        ORG 003CH
        JMP ISR75

        ORG 3000H

ISR75:  PUSH PSW
        PUSH B
        PUSH H

        LXI H, 2300H
        INR M

        POP H
        POP B
        POP PSW

        EI
        RET
```

Important: the ISR increments the byte at `2300H`. For a clean demonstration, initialize that location to `00H` before testing. Otherwise, it may contain an old or undefined value.

If your simulator cannot trigger external interrupts, the program can still be assembled, but you need an interrupt control or supported interrupt simulation to test the RST 7.5 path. Exact controls vary between simulator versions.

## 14. Understand the main program, line by line

### A. Initialize the stack

```
LXI SP, 27FFH
```

`LXI` means Load Register Pair Immediate. It loads a 16-bit value into a register pair or SP.

Here:

\\[ SP=27FFH \\]

The processor is now ready to use the stack.

### B. Configure the interrupt

```
MVI A, 0BH
SIM
EI
```

- `MVI A, 0BH`: load the interrupt mask pattern.
- `SIM`: apply the mask configuration.
- `EI`: enable maskable interrupts globally.

Order matters. The mask configuration should be applied, and the stack initialized, before enabling the interrupt.

### C. Initialize the main counter

```
MVI B, 00H
```

This sets B to zero.

B will hold the main counter value.

### D. Store the current count

```
MAIN: MOV A, B
      STA 2200H
```

`MOV A, B` copies B to A.

`STA 2200H` stores A directly at memory address `2200H`.

For example, if `B = 05H`, the result is:

- A = `05H`
- Memory `[2200H] = 05H`

Notice that `MOV A, B` does not change the condition flags.

### E. Call the delay routine

```
CALL DELAY
```

`CALL` saves the return address on the stack and jumps to the subroutine labelled `DELAY`.

When the delay routine executes `RET`, execution resumes at the instruction after `CALL`.

### F. Increment the counter

```
INR B
```

B increases by one.

For example:

| Before  | After   |
| ------- | ------- |
| `00H` | `01H` |
| `01H` | `02H` |
| `09H` | `0AH` |
| `FFH` | `00H` |

The last example illustrates 8-bit overflow. `INR B` wraps from `FFH` to `00H`, but it does not set the Carry flag.

### G. Repeat forever

```
JMP MAIN
```

This jumps back to `MAIN`.

The processor continuously stores the count, delays, increments B, and repeats. This is an infinite loop, which is intentional for the main program.

## 15. Understand the delay routine

```
DELAY:  LXI D, 0FFFH

BACK:   DCX D
        MOV A, D
        ORA E
        JNZ BACK
        RET
```

The routine uses DE as a 16-bit countdown value.

### Instruction 1 — `LXI D, 0FFFH`

Load:

\\[ DE=0FFFH \\]

This means D contains `0FH` and E contains `FFH`.

### Instruction 2 — `DCX D`

Decrease the DE register pair by one.

Examples:

| Before    | After     |
| --------- | --------- |
| `0FFFH` | `0FFEH` |
| `0002H` | `0001H` |
| `0001H` | `0000H` |

`DCX D` does not update the condition flags. Therefore, the processor cannot use `JNZ BACK` to test the countdown result directly.

### Instruction 3 — `MOV A, D`

Copy D into A.

This does not affect the flags.

### Instruction 4 — `ORA E`

Perform a bitwise OR between A and E, storing the result in A.

If either D or E contains a nonzero bit, the OR result is nonzero. If both are zero, the result is zero.

The Zero flag is updated by `ORA`.

### Instruction 5 — `JNZ BACK`

Jump back if Z = 0.

- If DE is nonzero, `ORA E` produces a nonzero result, so the loop repeats.
- If DE is zero, `ORA E` produces `00H`, so Z = 1 and the jump is not taken.

This is the core of the delay loop.

### Instruction 6 — `RET`

Once the countdown finishes, `RET` returns to the main program.

General rule: If an instruction does not update the flag you need, use an appropriate instruction afterward to set that flag before the conditional jump.

For example, `DCX D` does not set Z, but `MOV A,D` followed by `ORA E` creates the required zero/nonzero test.

## 16. Understand the ISR line by line

The ISR is the most important part of the experiment because it demonstrates how to handle an interrupt without corrupting the main program.

```
ISR75:  PUSH PSW
        PUSH B
        PUSH H

        LXI H, 2300H
        INR M

        POP H
        POP B
        POP PSW

        EI
        RET
```

### A. `PUSH PSW`

PSW means Program Status Word. In the 8085, it refers to the accumulator and flag register together.

`PUSH PSW` saves the accumulator and flags on the stack.

Why? The main program may be using A and its flags when the interrupt occurs. The ISR should preserve their original values if it intends to resume the main program without disturbing them.

### B. `PUSH B`

Saves the BC register pair.

The main program uses B as its running counter. Without saving B, an ISR that modified B could change the main counter unexpectedly.

### C. `PUSH H`

Saves the HL register pair before using it as a memory pointer.

### D. Increment the interrupt-hit counter

```
LXI H, 2300H
INR M
```

First, HL is made to point to address `2300H`.

Then `INR M` increases the byte stored at that address by one.

If memory initially contains `00H`, one interrupt produces `01H`, two produce `02H`, and so on, with normal 8-bit wraparound.

This counter is separate from the main counter at `2200H`.

### E. Restore the saved registers

```
POP H
POP B
POP PSW
```

Notice the order:

| Save order   | Restore order       |
| ------------ | ------------------- |
| `PUSH PSW` | `POP PSW` — last |
| `PUSH B`   | `POP B` — second |
| `PUSH H`   | `POP H` — first  |

The actual restoration sequence is the reverse of the save sequence.

This is LIFO: Last In, First Out.

Memory aid: PUSH forward, POP backward.

If you save A, B, and H in that order, restore H, B, and A in reverse order.

### F. Why `EI` before `RET`?

```
EI
RET
```

When a hardware interrupt is accepted, the 8085 automatically clears its global interrupt-enable state. Therefore, the ISR executes `EI` if further maskable interrupts should be accepted.

Then `RET` returns to the saved continuation address.

The 8085's `EI` takes effect after the following instruction. Here, that following instruction is `RET`, which is why this arrangement is commonly used.

## 17. What exactly happens when RST 7.5 occurs?

Memorize this sequence for your viva.

1. The CPU is executing `MAIN` or the delay subroutine.
2. An external rising edge generates an RST 7.5 request.
3. The CPU accepts the request when the necessary enable and mask conditions are satisfied, after completing the current instruction.
4. The CPU saves the continuation address and transfers control to vector address `003CH`.
5. `JMP ISR75` transfers execution to the ISR at `3000H`.
6. The ISR saves registers, increments the byte at `2300H`, and restores the saved registers.
7. `EI` re-enables maskable interrupts, and `RET` resumes the interrupted program.

A useful distinction: the CPU automatically preserves the return address for the interrupt. The `PUSH` instructions in your ISR are additional register-preservation operations.

# Part 6 — Program B: Software interrupt using RST 5

## 18. Complete program from your PPT

This experiment demonstrates that a software RST instruction transfers execution to a predefined vector without an external button press.&#x20;

Interrupt Programming using 8085 Microprocessor.pptx

```
        ORG 2000H

        LXI SP, 27FFH
        MVI A, 22H
        MVI B, 44H

        RST 5
        HLT

        ORG 0028H
        JMP ISR5

        ORG 3100H

ISR5:   PUSH PSW
        ADD B
        STA 2400H
        POP PSW
        RET
```

Expected result:

\\[ 22H+44H=66H \\]

Therefore:

\\[ [2400H]=66H \\]

## 19. Trace the program

### Step 1 — Initialize the stack

```
LXI SP, 27FFH
```

This gives `CALL`-like return operations a valid stack.

### Step 2 — Load two values

```
MVI A, 22H
MVI B, 44H
```

The register contents become:

- A = `22H`
- B = `44H`

### Step 3 — Execute the software interrupt

```
RST 5
```

The instruction transfers control to vector address `0028H`.

At `0028H`, the program contains:

```
JMP ISR5
```

The processor jumps to the full routine at `3100H`.

Unlike a hardware interrupt, a software `RST n` instruction executed by the program does not automatically clear the global interrupt-enable state.

### Step 4 — Preserve A and the flags

```
PUSH PSW
```

The original accumulator value `22H` and flags are saved.

### Step 5 — Add B to A

```
ADD B
```

The calculation is:

\\[ A=22H+44H=66H \\]

Here, `ADD B` means add the contents of B to A and keep the result in A.

Since `66H` is within the 8-bit range, there is no carry out.

For this operation:

- A becomes `66H`.
- CY = 0.
- Z = 0.
- S = 0.
- P = 1, because `66H` contains four set bits.
- AC = 0, because the lower nibbles add without a carry from bit 3 to bit 4.

### Step 6 — Store the result

```
STA 2400H
```

This stores the current accumulator value at memory address `2400H`.

So:

\\[ [2400H]=66H \\]

### Step 7 — Restore the original accumulator and flags

```
POP PSW
```

A becomes `22H` again, and the saved flags are restored.

This means the addition result remains in memory even though A is restored.

### Step 8 — Return and halt

```
RET
```

Control returns to the instruction after `RST 5`.

That instruction is:

```
HLT
```

The program halts.

Important distinction: The ISR's calculated result is at `2400H`, not necessarily in A after the return. The program deliberately restores the original accumulator value.

# Part 7 — Which flags are affected?

This is a common practical and viva topic.

## 20. Flag effects in your two programs

| Instruction                                                    | Flags affected                                                                  |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `MVI`, `MOV`, `LXI`, `STA`, `JMP`, `CALL`, `RET` | None                                                                            |
| `INR B`, `INR M`                                           | S, Z, AC, P; CY unchanged                                                       |
| `DCX D`                                                      | None                                                                            |
| `ORA E`                                                      | S, Z, P; CY = 0 and AC = 0                                                      |
| `JNZ`                                                        | None                                                                            |
| `ADD B`                                                      | S, Z, AC, P, CY                                                                 |
| `ANI 40H`                                                    | S, Z, P; CY = 0 and AC = 0                                                      |
| `PUSH`, `POP`                                              | Do not modify condition flags as an arithmetic result                           |
| `EI`, `DI`, `SIM`, `RIM`                               | No ordinary arithmetic flag result; they control or read interrupt/serial state |

A few distinctions are particularly important:

- `INR` changes flags but does not change CY.
- `DCX` changes a register pair without changing the flags.
- `ORA` sets Z based on the OR result, which makes it useful before `JNZ`.
- `ADD` updates all five condition flags.

### How to answer a flag question in the exam

Use this three-step method:

1. Identify the instruction.
2. Recall whether that instruction modifies flags.
3. If it does, calculate the result and determine each relevant flag.

For example, if asked what happens after `INR B` when B contains `FFH`:

- B becomes `00H`.
- Z = 1.
- S = 0.
- P = 1 because zero has even parity.
- CY remains unchanged.
- AC is set because the lower nibble overflows from `FH` to `0H`.

Do not assume every instruction changes flags just because the register contents change.

# Part 8 — Loops: a general method for any 8085 question

You specifically asked how to handle loops confidently in questions beyond this experiment. This is one of the most reusable assembly-language skills.

## 21. The three components of a loop

A typical counter-controlled loop contains:

1. Initialization: Set the starting value.
2. Body: Perform the required operation.
3. Update and condition: Change the counter and decide whether to repeat.

Example:

```
        MVI B, 04H

LOOP:   ; operation goes here
        DCR B
        JNZ LOOP

        HLT
```

This loop repeats four times if B starts at `04H` and is decremented once per iteration.

### Trace it manually

| Iteration | B before `DCR B` | B after `DCR B` | Z | `JNZ LOOP` |
| --------- | ------------------ | ----------------- | - | ------------ |
| 1         | `04H`            | `03H`           | 0 | Jump         |
| 2         | `03H`            | `02H`           | 0 | Jump         |
| 3         | `02H`            | `01H`           | 0 | Jump         |
| 4         | `01H`            | `00H`           | 1 | Do not jump  |

The loop body executes four times.

## 22. What do JZ and JNZ actually mean?

- `JZ LABEL`: jump if Z = 1.
- `JNZ LABEL`: jump if Z = 0.

They do not directly mean “stop” or “continue.” Their behavior depends on the flag at the moment the jump executes.

For example:

```
DCR B
JNZ LOOP
```

This repeats while B is nonzero.

By contrast:

```
DCR B
JZ DONE
JMP LOOP
```

This jumps to `DONE` when B becomes zero and otherwise repeats.

Both can implement a finite loop.

### The critical rule

A conditional jump uses the current Zero flag, not the number you intended to count.

If another instruction between the decrement and the jump overwrites the flags, the loop may behave differently.

For example:

```
DCR B
MOV A, C
JNZ LOOP
```

`MOV A, C` does not affect flags, so the Zero flag from `DCR B` is still available.

But:

```
DCR B
ORA C
JNZ LOOP
```

`ORA C` updates Z, so the jump now tests the OR result, not the result of decrementing B.

That is a common source of loop errors.

## 23. What if the counter starts at zero?

Consider:

```
MVI B, 00H

LOOP:  DCR B
       JNZ LOOP
```

`DCR B` changes `00H` to `FFH`. Since the result is nonzero, `JNZ` jumps back.

The counter then continues decreasing through `FEH`, `FDH`, and so on until it reaches `00H` again.

The loop executes 256 times, not zero times.

This happens because an 8-bit register wraps around modulo 256.

Memory aid: Zero followed by decrement means 256 iterations to return to zero, if the loop checks only after decrementing.

## 24. The three most useful loop templates

### Pattern A — Repeat N times

```
MVI B, 05H

LOOP:   ; body
        DCR B
        JNZ LOOP
```

Starts at 5 and runs the body five times, assuming no other control-flow changes.

### Pattern B — Count up to a limit

```
MVI B, 00H

LOOP:   INR B
        MOV A, B
        CPI 05H
        JNZ LOOP
```

Counts from 1 through 5. CPI compares A with an immediate value and sets flags without changing A.

### Pattern C — Countdown using a register pair

```
LXI D, 0005H

LOOP:   DCX D
        MOV A, D
        ORA E
        JNZ LOOP
```

Uses a 16-bit counter. ORA tests whether either half is nonzero because DCX does not update flags.

For any new loop question, ask yourself:

- Which register stores the counter?
- What is its initial value?
- Which instruction changes the counter?
- Which instruction establishes the condition flags?
- Which jump uses those flags?
- What value causes the loop to terminate?

If you can answer these six questions, you can usually trace the loop correctly.

# Part 9 — How to execute the programs in GNUSim8085

## 25. Hardware interrupt program

Your PPT describes entering the main program at `2000H`, the RST 7.5 vector jump at `003CH`, and the ISR at `3000H`.&#x20;

Interrupt Programming using 8085 Microprocessor.pptx

Follow this general procedure. Menu names and shortcuts may vary by version.

1. Open GNUSim8085 and enter the assembly program in the editor.
2. Ensure the `ORG` directives, labels, and vector jump are included. Assemble the program and correct any syntax errors.
3. Initialize the interrupt counter at `2300H` to `00H` before testing, if necessary.
4. Run the main program. It repeatedly updates the main counter at `2200H`.
5. Locate the interrupt controls, if your simulator version provides them, and trigger RST 7.5.
6. Use single-step execution to observe the jump to `003CH`, the jump to `3000H`, and the ISR instructions.
7. Verify that the memory value at `2300H` increases and that execution returns to the main program.

### Which debugging controls should you know?

| Control             | Meaning                                                                                            |
| ------------------- | -------------------------------------------------------------------------------------------------- |
| Step In / Step Into | Execute one instruction, entering a called routine when applicable.                                |
| Step Over           | Execute a call as a unit instead of stepping through every instruction inside it, where supported. |
| Step Out            | Continue until the current subroutine returns, where supported.                                    |
| Breakpoint          | Pause execution when it reaches a selected instruction.                                            |
| Run / Continue      | Execute until a breakpoint, halt, or other stopping condition.                                     |
| Register view       | Inspect A, B, C, D, E, H, L, PC, SP, and flags.                                                    |
| Memory view         | Inspect bytes stored at addresses such as `2200H`, `2300H`, and `2400H`.                     |

If a hardware interrupt does not trigger, check whether the simulator supports external interrupt injection. An assembled program does not automatically simulate a button press.

## 26. Software interrupt program

This program is simpler to test because `RST 5` is an instruction in the program itself.

1. Assemble the code.
2. Set the starting execution address to `2000H`, if required.
3. Run or single-step through the program.
4. Watch the transfer to `0028H`, then `3100H`.
5. Inspect memory address `2400H`.
6. Confirm the stored result is `66H`.
7. Observe `HLT` after the ISR returns.

If the program stops before producing the expected result, inspect the program counter and stack pointer. Also verify that the vector at `0028H` contains the intended jump instruction.

# Part 10 — General problem-solving strategy for 8085 assembly

## 27. A five-step method for solving unfamiliar questions

Use this approach for arithmetic, memory operations, loops, interrupts, and code conversions.

## 1

Identify the inputs and outputs

Write down where the input values come from and where the final result must be stored. For example, two numbers in A and B, with the result at `2400H`.

## 2

Choose registers

Decide which register stores the counter, which holds the data, and which register pair points to memory. Avoid reusing a register without considering its existing value.

## 3

Write the instruction sequence

Translate each operation into assembly. For a memory array, initialize HL and decide whether to use `INX H`, `DCX H`, or another pointer update.

## 4

Trace registers, memory, and flags

Manually calculate the first few iterations. Identify exactly which instruction updates the flag used by the conditional jump.

## 5

Verify the final result

Check the ending address, loop count, memory contents, register values, and any carry or overflow behavior. Then test the code in the simulator.

## 28. Common mistakes to avoid

| Mistake                                                              | Why it is wrong                                                          |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Forgetting to initialize SP                                          | Stack operations may overwrite unintended memory or return incorrectly.  |
| Writing `EI` but not configuring the individual interrupt mask     | An individually masked RST line still cannot be accepted.                |
| Using `SIM` without setting MSE appropriately                      | The individual mask settings may not be applied.                         |
| Writing a full ISR directly into a vector slot                       | The next vector starts only eight bytes later.                           |
| Forgetting to save a register modified by the ISR                    | The interrupted program may resume with corrupted data.                  |
| Restoring registers in the wrong order                               | Stack contents can be assigned to the wrong registers.                   |
| Assuming `DCX` updates Z                                           | It does not update the condition flags.                                  |
| Assuming `INR` updates CY                                          | It does not modify CY.                                                   |
| Forgetting that `RST 5` returns after the instruction              | It resumes at the instruction following `RST 5`, not at the beginning. |
| Assuming `POP PSW` preserves the calculation result in A           | It restores the old accumulator value.                                   |
| Using `JNZ` without checking the latest flag-producing instruction | The jump may test an unrelated result.                                   |

# Part 11 — Memory aids for the practical exam

## 29. Quick revision sheet

### Interrupt sequence

Mask → Enable → Request → Vector → ISR → Restore → EI → RET

Initialize SP before enabling interrupts. This sequence is a conceptual guide; the request arrives asynchronously.

### Vector addresses

Software RST n = n × 8

`RST 5` → `0028H`; hardware RST 7.5 → `003CH`.

### Register preservation

PUSH in order; POP in reverse order.

Save the registers your ISR modifies. Use `PUSH PSW` when you need to preserve A and the flags.

### Loop logic

Initialize → Work → Update → Test → Repeat or exit.

For `DCR B` followed by `JNZ`, the body executes four times when B starts at `04H`, assuming the loop begins with B already initialized.

### Flag logic

Check the instruction, then check its flags.

`INR` preserves CY. `DCX` preserves all condition flags. `ORA` sets Z according to the OR result.

## 30. Likely viva questions with answers

### Test yourself

0 of 17 answers revealed

1\. What is an interrupt?

2\. What is the aim of this experiment?

3\. What is an ISR?

4\. Which 8085 interrupt has the highest priority?

5\. Which hardware interrupt is edge-triggered and latched?

6\. What is the vector address of RST 7.5?

7\. What is the vector address of RST 5?

8\. Why do we use JMP at an interrupt vector?

9\. What is the purpose of SIM?

10\. What is the purpose of RIM?

11\. Why do we use EI inside a hardware ISR?

12\. Why initialize the stack pointer?

13\. Why do we save registers in the ISR?

14\. What is the result of the software RST 5 example?

15\. Does INR affect the Carry flag?

16\. Does DCX update the Zero flag?

17\. What is the difference between software RST and a hardware interrupt?

Reveal all answers

# Part 12 — What to do if the examiner gives you a variation

Do not panic if the examiner changes the register, vector, counter, or result address. Identify what has changed and retain the underlying logic.

| If the question changes…         | What you should do                                                                                   |
| --------------------------------- | ---------------------------------------------------------------------------------------------------- |
| RST 7.5 to RST 6.5                | Use vector `0034H` and adjust the SIM mask pattern.                                                |
| RST 5 to RST 3                    | Use vector `0018H`, since \\(3\times8=24\\) decimal = `18H`.                                     |
| Interrupt counter address         | Change the address loaded into HL in the ISR.                                                        |
| Number of loop iterations         | Change the counter initialization and verify the loop's termination condition.                       |
| Register used as the counter      | Replace the corresponding increment/decrement and jump logic, checking which flags are affected.     |
| Result address                    | Change the address in `STA` or the pointer initialization, depending on the addressing mode.       |
| ISR modifies a different register | Save and restore that register if its original value must be preserved.                              |
| Examiner asks for flags           | Trace the arithmetic or logical operation and identify the exact instruction that updates the flags. |

One important caution about changing RST 7.5 to RST 6.5: do not simply change the vector address. You must also ensure the interrupt is individually unmasked, globally enabled, and triggered in a way supported by that input.

## Final preparation checklist

Before your practical exam, make sure you can do these without looking at the notes:

- Define interrupt, ISR, vector address, stack, and polling.
- Recall the five hardware interrupts, their priority order, and vector addresses.
- Explain `EI`, `DI`, `SIM`, and `RIM`.
- Write both programs with correct `ORG` addresses.
- Explain every instruction in the main program and ISR.
- Trace the delay loop and determine why it terminates.
- State which flags are affected by `INR`, `DCX`, `ORA`, and `ADD`.
- Explain why the ISR saves and restores registers.
- Verify `2300H` for the hardware interrupt count and `2400H = 66H` for the software interrupt result.
- Explain why the software RST returns after the instruction and why the hardware ISR uses `EI` when future maskable interrupts must be accepted.

The most useful approach is to learn the reason behind each instruction, then practice tracing the program with a register-and-memory table. Once you understand the vector, stack, flag, and loop rules, variations of these programs become much easier to solve.
