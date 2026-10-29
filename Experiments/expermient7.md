
# 8085 Microprocessor Lab — Experiment 7: Serial Communication Using SIM and RIM

These notes cover the theory, both programs, every instruction, the register and flag changes, the expected output, how to execute the experiment in GNUSim8085, and the questions your professor may ask in a viva.

The aim is not just to memorize the code. It is to understand what each instruction does, why it is needed, and how to explain it when your professor points to any line in the program.

I'll use simple language and the same programs you have been practising. Your version uses `LXI D, 01H` for a short delay and has the data entered manually in GNUSim8085.

# Part 1 — Understand the experiment before learning the code

## 1. What is the aim of this experiment?

The experiment has two parts.

Part A — Transmission: Send an 8-bit data byte, one bit at a time, through the 8085's SOD pin using the `SIM` instruction.

Part B — Reception: Receive eight bits through the SID pin using the `RIM` instruction, reconstruct the byte, and store it in memory.

The sample data is `A5H`. If the receiver receives the correct eight bits, the expected result is `A5H` stored at address `4300H`.&#x20;

Experiment-7-Serial Communication using 8085 Microprocessor.pptx

## 2. What is communication?

Communication means transferring information from one place to another.

For example, imagine sending the number `165` from one computer to another. The sender has the data, and the receiver must obtain the same data.

In this experiment:

- The 8085 microprocessor acts as the sender.
- Its SOD pin carries outgoing serial bits.
- Its SID pin accepts incoming serial bits.
- The receiving program rebuilds the original byte.

Think of SOD as a mouth that sends information and SID as an ear that receives information.

## 3. What is a bit and a byte?

A bit is the smallest unit of digital information. It can have one of two values:

- `0` — LOW
- `1` — HIGH

A byte consists of eight bits.

For example:

\\[ \text{A5H} = 10100101_2 \\]

Here, `A5H` is a hexadecimal representation of the byte, and `10100101` is its binary representation.

The eight bits are numbered from D7 to D0:

The byte A5H

D7

## 1

D6

## 0

D5

## 1

D4

## 0

D3

## 0

D2

## 1

D1

## 0

D0

## 1

D7 is the most significant bit (MSB). D0 is the least significant bit (LSB).

The numerical weights of the bits are:

| Bit | Weight |
| --- | ------ |
| D7  | 128    |
| D6  | 64     |
| D5  | 32     |
| D4  | 16     |
| D3  | 8      |
| D2  | 4      |
| D1  | 2      |
| D0  | 1      |

For `10100101`, the bits with value 1 have weights 128, 32, 4 and 1.

\\[ 128+32+4+1=165 \\]

Therefore, `A5H` = 165 decimal.

## 4. What is hexadecimal?

Hexadecimal is a number system with 16 symbols:

`0 1 2 3 4 5 6 7 8 9 A B C D E F`

The letters represent values from 10 to 15.

| Number system | Example      |
| ------------- | ------------ |
| Binary        | `10100101` |
| Hexadecimal   | `A5H`      |
| Decimal       | `165`      |

The `H` at the end means the number is hexadecimal. Some assemblers also accept a `0x` prefix, but your GNUSim8085 code uses the `H` notation.

Memory aid: One hexadecimal digit represents four binary bits. Therefore, two hexadecimal digits represent one byte.

For example:

- `A` = `1010`
- `5` = `0101`
- `A5H` = `1010 0101`

## 5. What is serial communication?

Serial communication transfers data one bit at a time over a serial data line.

Suppose you want to send `A5H`, which is `10100101`.

The transmitter sends:

`1 → 0 → 1 → 0 → 0 → 1 → 0 → 1`

Each bit is sent during its own time interval.

### Serial versus parallel communication

[Serial Communication. Definisi Serial Communication | by Jatniko Nur Mutaqin | Medium](https://images.openai.com/static-rsc-4/-3Bjppnb5Gj70xYkID-7m9o9kaLs9iPlCHCywYXXVMlJdCUGkze2I72OqAzNL6WgADpaSuKb29TDwbiw1tcUeoFTfRwIP7ppKb1DkX_hK5NnE7FA3t9sbo57veBlnTulpcFrnz2SkjgQ9Jr7l3OqzrWdbWiU19rUjX-Vvy4JIko?purpose=inline)

Parallel communication

Multiple bits travel at the same time over separate data lines. For an 8-bit transfer, eight data lines can carry the eight bits simultaneously.

[Digital Hierarchy | PPT](https://images.openai.com/static-rsc-4/mbPUZJ4eDIIGaNvauHpSuDaq7yaSPnbg2z_eL0AT79wHNCWvKnWwy4LHDaFm24f7i2iGrikKtuIj5jdklZNv79fhLGzUSdwqGH2gFTvt3t4Ncb4eOFwVSC3qVALNKGbmacpcHQuuvpMReWxiZoOAirixsUzVQDAsivq3A5mn00g?purpose=inline)

Serial communication

Bits travel one after another over a serial data line. It uses fewer data wires, but the receiver must sample the bits at the right times.

For this experiment, the 8085 uses serial communication through two dedicated pins.

## 6. What are SID and SOD?

The 8085 has two dedicated serial data pins.&#x20;

Experiment-7-Serial Communication using 8085 Microprocessor.pptx

SOD — Serial Output Data

#### Pin 4

Used to send a serial bit out of the microprocessor. The `SIM` instruction controls it.

SID — Serial Input Data

#### Pin 5

Used to receive a serial bit into the microprocessor. The `RIM` instruction reads it.

Remember this distinction:

- `SIM` controls the output.
- `RIM` reads the input.
- `SOD` is output.
- `SID` is input.

### How are they connected in a hardware demonstration?

A common loopback experiment connects SOD to SID using a jumper wire. The transmitted bit can then be read back by the receiver.

On a physical trainer kit, this requires the appropriate connection. In a simulator, an equivalent connection or input simulation must be supported. Your current GNUSim8085 screenshots have not shown a control for supplying SID bits, so the successful transmission and reception of `A5H` cannot yet be assumed.

# Part 2 — Essential 8085 concepts you need before reading the programs

## 7. What is a register?

A register is a small storage location inside the processor. It holds data that the CPU is currently using.

The 8085 has several registers, and each has a specific purpose in our programs.

| Register         | Size        | Purpose in this experiment                                      |
| ---------------- | ----------- | --------------------------------------------------------------- |
| A — Accumulator | 8 bits      | Holds the byte, prepares the output bit, and reads serial input |
| B                | 8 bits      | Counts the eight bits                                           |
| C                | 8 bits      | Collects the received bits                                      |
| D and E          | 8 bits each | Form the DE register pair for the delay counter                 |
| H and L          | 8 bits each | Form the HL pair to access the data in memory                   |
| PC               | 16 bits     | Holds the address of the next instruction                       |
| SP               | 16 bits     | Points to the top of the stack                                  |
| Flag register    | 8 bits      | Stores condition flags, including CY and Z                      |

### What is a register pair?

Two 8-bit registers can be combined to form a 16-bit register pair.

For example:

- BC = B and C combined.
- DE = D and E combined.
- HL = H and L combined.

If `HL = 4200H`, then the HL pair contains the 16-bit address `4200H`.

## 8. What is the accumulator?

The accumulator is register A. It is the main register used for arithmetic, logic and many data-processing operations in the 8085.

In Part A, A initially holds the data byte `A5H`. The program repeatedly rotates this byte to extract its bits.

In Part B, `RIM` loads the serial input level into bit D7 of A. The program then rotates that bit into the carry flag.

Important: The accumulator does not keep the original byte throughout either program. It is reused for different purposes.

## 9. What is the carry flag (CY)?

The carry flag is one of the condition flags in the 8085's flag register.

It can contain either `0` or `1`.

A carry can be generated by arithmetic operations, but in this experiment the important use is rotation.

Consider:

```
MVI A, 80H
RLC
```

`80H` in binary is `10000000`.

After `RLC`:

- A becomes `01H`.
- CY becomes `1`.

Why? The old D7 bit was 1. `RLC` moves it to CY and also wraps it around into D0.

You tested this successfully in GNUSim8085, which confirms that the rotate instruction and carry flag work as expected.

## 10. What is the stack?

The stack is a region of memory used for temporary storage, especially during subroutine calls and returns.

The stack follows the Last In, First Out (LIFO) principle.

Imagine placing three books on top of one another. The last book placed on the stack is the first one removed.

The stack pointer, SP, identifies the top of the stack.

### Why does Part A use the stack?

Part A executes:

```
PUSH PSW
...
POP PSW
```

The program must temporarily replace A with a control byte for `SIM`. Before doing that, it saves the rotated data and flags so they can be restored for the next bit.

### Why does the program initialize SP?

```
LXI SP, 4FFFH
```

This sets the stack pointer to `4FFFH`.

It provides a known stack location before `PUSH`, `POP`, `CALL` and `RET` are used.

Part B also uses `CALL` and `RET` for its delay subroutine, so it is good practice to initialize SP there as well.

## 11. What is a flag?

Flags record information about the result of certain instructions.

The 8085 has five commonly discussed condition flags:

| Flag                  | Meaning                                                                         |
| --------------------- | ------------------------------------------------------------------------------- |
| S — Sign             | Reflects the most significant bit of certain results                            |
| Z — Zero             | Set when a result is zero                                                       |
| AC — Auxiliary Carry | Used mainly for carry between the lower and upper nibbles                       |
| P — Parity           | Indicates whether the result has even parity                                    |
| CY — Carry           | Indicates a carry or borrow in arithmetic, or a bit shifted out during rotation |

For this experiment, CY is the most important flag because the programs use it to move one bit at a time.

`JNC` and `JNZ` also matter:

- `JNC` means Jump if No Carry. It jumps when CY = 0.
- `JNZ` means Jump if Not Zero. It jumps when Z = 0.

We will examine them in the actual code below.

## 12. Important 8085 instructions used in this experiment

Before going line by line, learn the purpose of each instruction.

| Instruction | Simple meaning                                               |
| ----------- | ------------------------------------------------------------ |
| `LXI`     | Load a 16-bit value into a register pair or SP               |
| `MVI`     | Load an 8-bit immediate value into a register                |
| `MOV`     | Copy data between registers or between a register and memory |
| `RLC`     | Rotate accumulator left; old D7 goes to CY and D0            |
| `RAL`     | Rotate accumulator left through CY                           |
| `PUSH`    | Save a register pair or PSW on the stack                     |
| `POP`     | Restore data from the stack                                  |
| `ORI`     | Bitwise OR the accumulator with an immediate value           |
| `JNC`     | Jump if CY = 0                                               |
| `JNZ`     | Jump if Z = 0                                                |
| `DCR`     | Decrease a register by one                                   |
| `CALL`    | Jump to a subroutine while saving the return address         |
| `RET`     | Return from a subroutine                                     |
| `SIM`     | Set interrupt masks and control serial output                |
| `RIM`     | Read interrupt masks/status and serial input                 |
| `STA`     | Store accumulator contents at a direct memory address        |
| `HLT`     | Stop program execution                                       |

One key distinction: `MOV` copies a value; it does not erase the source. For example, `MOV C,A` copies A into C, but A remains unchanged.

## 13. What is addressing?

Addressing describes how an instruction identifies the data it needs.

Our programs use three important forms.

### A. Immediate addressing

The value is written directly in the instruction.

```
MVI B, 08H
```

This loads the literal value `08H` into B.

### B. Register addressing

The instruction identifies a register.

```
MOV C, A
```

This copies the value in A into C.

### C. Memory addressing

The instruction accesses a memory location.

```
MOV A, M
```

Here, `M` means the memory location addressed by HL. If `HL = 4200H`, this instruction loads A with the byte stored at `4200H`.

Compare these:

- `MOV A, L` copies the value in register L.
- `MOV A, M` copies the value from memory at the address stored in HL.

This distinction is important in a viva.

# Part 3 — Part A: Serial data transmission

## 14. What exactly does the transmitter need to do?

The input byte is stored at `4200H`.

The transmitter must:

1. Read the byte from memory.
2. Count eight bits.
3. Extract the next bit using `RLC`.
4. Prepare the control value for `SIM`.
5. Send the bit through SOD.
6. Wait for a short delay.
7. Repeat until all eight bits have been sent.

The input byte is `A5H`, which is `10100101` in binary. This program transmits the most significant bit first.

## 15. Part A code — the version you have been practising

```
LXI SP, 4FFFH
LXI H, 4200H
MOV A, M
MVI B, 08H

TRANSMIT: RLC
PUSH PSW
MVI A, 00H
JNC SETBIT
MVI A, 80H

SETBIT: ORI 40H
SIM
CALL DELAY
POP PSW
DCR B
JNZ TRANSMIT
HLT

DELAY: LXI D, 01H
BACK: DCX D
MOV A, D
ORA E
JNZ BACK
RET
```

For your GNUSim8085 setup, you have entered `A5H` manually at address `4200H`. That is why this version does not require the `ORG 4200H` and `DB 0A5H` lines from the presentation.

## 16. Line-by-line explanation of Part A

### Line 1 — `LXI SP, 4FFFH`

This initializes the stack pointer.

The program uses `PUSH PSW`, `POP PSW`, `CALL` and `RET`. These operations need the stack.

Expected observation: SP becomes `4FFFH`.

### Line 2 — `LXI H, 4200H`

This loads the address `4200H` into the HL register pair.

The purpose is to point HL at the memory location containing the byte to transmit.

Expected observation:

- H = `42H`
- L = `00H`
- HL = `4200H`

Remember that HL contains the address, not the data byte itself.

### Line 3 — `MOV A, M`

`M` means the memory location addressed by HL.

Since HL = `4200H`, the instruction is effectively:

```
A = memory[4200H]
```

The byte at `4200H` is `A5H`, so A becomes `A5H`.

Expected observation: A = `A5H`.

### Line 4 — `MVI B, 08H`

This initializes the bit counter to eight.

The byte contains eight bits, and the program sends one bit during each iteration.

Expected observation: B = `08H`.

### Line 5 — `TRANSMIT: RLC`

`TRANSMIT` is a label. It gives a name to the address of the instruction that follows it.

`RLC` rotates the accumulator left.

For `A = A5H`:

\\[ A5H = 10100101_2 \\]

The most significant bit is 1. `RLC` moves that bit into CY and wraps it around into D0.

After the first `RLC`:

- CY = 1
- A = `4BH`

Binary:

`10100101` becomes `01001011`.

This is how the program extracts the next bit to send.

### Line 6 — `PUSH PSW`

PSW stands for Program Status Word. In the 8085, it comprises the accumulator and flag register.

`PUSH PSW` saves A and the flags on the stack.

Why is that necessary?

Because the next instructions will replace A with `00H`, `80H` and then `C0H` or `40H` to configure the serial output. Without saving the rotated data, the original byte being rotated would be lost.

The program needs to restore the rotated byte after transmitting the current bit.

### Line 7 — `MVI A, 00H`

This prepares a default control value for sending a zero.

A becomes `00H`.

The carry flag from the previous `RLC` is still available. `MVI` does not modify the condition flags.

### Line 8 — `JNC SETBIT`

This means Jump if No Carry.

- If CY = 0, execution jumps to `SETBIT`.
- If CY = 1, execution continues to the next instruction.

The program uses this instruction to decide whether the current bit is zero or one.

### Line 9 — `MVI A, 80H`

This instruction executes only when CY = 1.

It loads `80H` into A:

`80H = 10000000₂`

This sets D7 to 1, which is the output data bit for SOD.

If CY = 0, the jump skips this instruction and A remains `00H`.

### Line 10 — `SETBIT: ORI 40H`

This is one of the most important lines in the transmitter.

`40H` in binary is:

`01000000`

Bit D6 is the Serial Output Enable (SOE) bit. It must be set to 1 to enable the SOD output.

The OR operation combines the current A value with `40H`.

Case 1: the bit being sent is zero.

```
A before ORI = 00H
Value         = 40H
A after ORI  = 40H
```

Case 2: the bit being sent is one.

```
A before ORI = 80H
Value         = 40H
A after ORI  = C0H
```

So the two control values are:

- `40H` = `01000000` — send 0 with output enabled.
- `C0H` = `11000000` — send 1 with output enabled.

### Line 11 — `SIM`

`SIM` stands for Set Interrupt Mask.

It uses the accumulator's control bits to set interrupt masks and control serial output. For this experiment, the important bits are D7 and D6.

- D7 specifies the output bit.
- D6 enables serial output.

When `SIM` executes, the selected bit is driven onto SOD if the simulator or hardware supports that output.

This is exactly what you observed in Part A: when the blue highlight is on `SIM`, A shows `C0H` or `40H`.

That observation confirms that your program is preparing the correct control value.

### Line 12 — `CALL DELAY`

This calls the delay subroutine.

A receiver needs time to sample each bit. The delay helps keep successive bits separated in time.

`CALL` also saves the return address on the stack so the program can return to the instruction after `CALL` when the subroutine finishes.

### Line 13 — `POP PSW`

This restores the accumulator and flags saved by `PUSH PSW`.

After this instruction, A contains the rotated data byte again, ready for the next `RLC`.

This pairing is essential:

`PUSH PSW` saves the data.

`POP PSW` restores it.

### Line 14 — `DCR B`

This decrements B by one.

The counter starts at `08H`, and one bit has now been sent.

After the first completed iteration, B becomes `07H`.

### Line 15 — `JNZ TRANSMIT`

This repeats the transmission loop while the counter is not zero.

- If B is not zero, jump back to `TRANSMIT`.
- If B is zero, continue to `HLT`.

The program therefore sends eight bits.

### Line 16 — `HLT`

`HLT` stops execution.

Once all eight iterations are complete, the program reaches this instruction.

There is no instruction to print the data on a screen. The transmission is performed through the serial output control.

## 17. The delay subroutine in Part A

Your current version is:

```
DELAY: LXI D, 01H
BACK: DCX D
MOV A, D
ORA E
JNZ BACK
RET
```

Let's understand each instruction.

### `DELAY: LXI D, 01H`

Loads `0001H` into the DE register pair.

- D = `00H`
- E = `01H`
- DE = `0001H`

The label `DELAY` marks the beginning of the subroutine.

### `BACK: DCX D`

`DCX D` decreases the DE register pair by one.

`0001H - 0001H = 0000H`

So DE becomes `0000H`.

Unlike `DCR`, `DCX` does not update the condition flags.

### `MOV A, D`

Copies D into A.

Since D is `00H`, A becomes `00H`.

### `ORA E`

Performs a bitwise OR between A and E.

At this point:

- A = `00H`
- E = `00H`

Therefore, A remains `00H`.

`ORA` updates the condition flags. Since the result is zero, Z becomes 1.

### `JNZ BACK`

JNZ means Jump if Not Zero.

Here, the result was zero, so Z = 1 and the jump is not taken. Execution continues to `RET`.

### `RET`

Returns to the instruction immediately after the `CALL DELAY` that invoked this subroutine.

### Is `LXI D, 01H` a good delay?

It is useful for your short simulator test, but it creates an extremely short delay: the loop executes only once.

For actual hardware serial communication, the receiving end needs a suitable bit period. A delay of one count may be too short for reliable reception.

Your presentation uses `LXI D, 0FFFH` for a longer delay. You have deliberately shortened it to `01H` while practising in GNUSim8085.&#x20;

Experiment-7-Serial Communication using 8085 Microprocessor.pptx

# Part 4 — Understanding the complete transmission of A5H

## 18. How the bits are extracted

The original byte is:

`A5H = 10100101`

The program uses `RLC` to move the next bit into CY.

Here is the first iteration in detail.

Before the first `RLC`:

- A = `A5H` = `10100101`
- B = `08H`

After the first `RLC`:

- A = `4BH` = `01001011`
- CY = 1

The program saves A and the flags using `PUSH PSW`.

It then builds the serial output control byte:

- CY = 1
- `MVI A, 80H`
- `ORI 40H`
- A = `C0H`

When `SIM` executes, the intended output bit is 1.

After the delay, `POP PSW` restores the rotated byte `4BH`. The next `RLC` extracts the next bit, which is 0.

## 19. All eight output bits

The expected control values immediately before `SIM` are:

| Iteration | Bit being sent | A before `SIM` |
| --------- | -------------- | ---------------- |
| 1         | 1              | `C0H`          |
| 2         | 0              | `40H`          |
| 3         | 1              | `C0H`          |
| 4         | 0              | `40H`          |
| 5         | 0              | `40H`          |
| 6         | 1              | `C0H`          |
| 7         | 0              | `40H`          |
| 8         | 1              | `C0H`          |

The sequence is:

`C0H → 40H → C0H → 40H → 40H → C0H → 40H → C0H`

The corresponding serial bits are:

`1 → 0 → 1 → 0 → 0 → 1 → 0 → 1`

This is the visible register-level evidence you can demonstrate in GNUSim8085.

Important distinction: Seeing these values confirms that the program prepares the intended bit sequence. To confirm actual transmission on a pin, you need an observable SOD output or an equivalent simulator/hardware measurement.

## 20. Why do we send MSB first?

MSB means Most Significant Bit.

For `A5H`, D7 is 1, so the first bit sent is 1. The program then sends D6, D5, and so on until D0.

The receiver must use a compatible order. If the transmitter sends MSB first but the receiver assembles the bits in a different order, the resulting byte may be incorrect.

# Part 5 — Part B: Serial data reception

## 21. What is the receiver supposed to do?

The receiver performs the reverse operation.

Instead of sending a bit, it reads the current SID input bit, moves that bit into CY, and adds it to the partial byte stored in C.

After eight bits, C should contain the complete received byte.

The program then stores C at address `4300H`.

## 22. Part B code — the version you have been practising

```
LXI SP, 4FFFH
MVI B, 08H
MVI C, 00H

LOOP1: RIM
RLC
MOV A, C
RAL
MOV C, A
CALL DELAY
DCR B
JNZ LOOP1

MOV A, C
STA 4300H
HLT

DELAY: LXI D, 01H
BACK: DCX D
MOV A, D
ORA E
JNZ BACK
RET
```

This program expects the incoming bits to be supplied through SID. It does not read the transmitter's memory address `4200H` directly.

## 23. Why does the receiver initialize B and C?

### `LXI SP, 4FFFH`

Initializes the stack pointer because the program uses `CALL` and `RET`.

### `MVI B, 08H`

Sets B to eight because the receiver must sample eight bits.

### `MVI C, 00H`

Clears C.

Register C is the result accumulator for the received byte. Starting from zero means there is no old data mixed into the new result.

At this point:

- B = `08H`
- C = `00H`

## 24. `RIM` — the most important instruction in Part B

```
LOOP1: RIM
```

`RIM` stands for Read Interrupt Mask.

It loads the accumulator with the current interrupt/serial status. The current SID input level is placed into bit D7 of A.

For example, if SID is high at the instant `RIM` executes, bit D7 of A becomes 1.

If SID is low, bit D7 becomes 0.

The other bits of A contain interrupt status and mask information. That is why `RIM` is not simply a command that loads either `00H` or `80H`.

The receiver cannot invent missing input bits. `RIM` only samples what the SID input presents at that moment.

This is directly related to the issue you encountered in GNUSim8085: your standalone `RIM` test returned A = `00H`, and CY and C stayed zero during the receiving program. That is consistent with the receiver sampling zero bits, but it does not by itself establish whether the simulator supports SID input control.

## 25. `RLC` — move the sampled bit into CY

```
RLC
```

After `RIM`, the bit of interest is D7.

`RLC` moves the old D7 into the carry flag.

So:

- If the sampled SID bit was 1, CY becomes 1.
- If the sampled SID bit was 0, CY becomes 0.

The remaining accumulator bits are rotated too, but the receiver uses CY to preserve the serial bit of interest.

Why not simply use the accumulator's entire value? Because `RIM` also contains interrupt-status information. The program needs just one serial bit per iteration.

## 26. `MOV A, C` — retrieve the partial byte

```
MOV A, C
```

The receiver copies the partial byte from C into A.

For example, if C contains `02H`, A becomes `02H`.

This instruction does not modify CY, so the serial bit obtained by `RLC` remains available.

## 27. `RAL` — insert the new bit

```
RAL
```

This rotates A left through CY.

It moves the previous contents of A one position left and inserts the carry bit into bit D0.

Conceptually:

\\[ \text{New partial byte} = (\text{Old partial byte shifted left}) + \text{New bit} \\]

For example, suppose:

- C = `02H` = `00000010`
- CY = 1

After `MOV A,C`, A = `02H`.

After `RAL`:

`00000010` becomes `00000101`.

So A = `05H`.

The previously collected bits have shifted left, and the new bit has entered at the right.

## 28. `MOV C, A` — save the updated result

```
MOV C, A
```

Copies the updated partial byte back into C.

In the example above, C becomes `05H`.

The three instructions work together:

```
MOV A, C
RAL
MOV C, A
```

Remember the sequence:

1. Retrieve the partial result.
2. Shift it left and insert the new bit from CY.
3. Save the updated result.

This is the core of the receiver's byte-reconstruction algorithm.

## 29. Worked example: how C builds A5H

Assume SID supplies the bits `1, 0, 1, 0, 0, 1, 0, 1` in that order.

For every bit, the receiver performs:

`RIM → RLC → MOV A,C → RAL → MOV C,A`

Here is how the result grows:

| Iteration | Incoming bit | C before | C after |
| --------- | ------------ | -------- | ------- |
| 1         | 1            | `00H`  | `01H` |
| 2         | 0            | `01H`  | `02H` |
| 3         | 1            | `02H`  | `05H` |
| 4         | 0            | `05H`  | `0AH` |
| 5         | 0            | `0AH`  | `14H` |
| 6         | 1            | `14H`  | `29H` |
| 7         | 0            | `29H`  | `52H` |
| 8         | 1            | `52H`  | `A5H` |

Let's examine the third iteration because it shows the method clearly.

Before the third iteration, C = `02H`.

The incoming bit is 1, so after `RIM` and `RLC`, CY = 1.

Next:

```
MOV A, C
```

A becomes `02H` = `00000010`.

Then:

```
RAL
```

The value shifts left through CY:

`00000010` becomes `00000101`.

Finally:

```
MOV C, A
```

C becomes `05H`.

The receiver has now collected the first three bits, `101`.

After eight correct iterations, C should contain `A5H`.

## 30. Why does the receiver use `DCR B` and `JNZ LOOP1`?

```
DCR B
JNZ LOOP1
```

The counter starts at `08H`.

Each iteration samples one serial bit, so B must decrease once per iteration.

After the first iteration, B = `07H`.

After the eighth iteration, B = `00H`.

`DCR B` updates the Zero flag. When B becomes zero, Z = 1, so `JNZ LOOP1` is not taken and the program proceeds to the next instruction.

This is a standard 8085 loop pattern. You will use it in many other experiments.

Memory aid: Initialize the counter, perform the work, decrement the counter, and jump back while it is not zero.

## 31. How does the receiver store the final result?

After the loop:

```
MOV A, C
STA 4300H
HLT
```

### `MOV A, C`

Copies the reconstructed byte from C into A.

If the byte was received correctly, A becomes `A5H`.

### `STA 4300H`

Stores the accumulator at the specified memory address.

In simple terms:

`memory[4300H] = A`

If A = `A5H`, memory address `4300H` should contain `A5H`.

### `HLT`

Stops the program.

The expected final state, provided the correct bits were received, is:

| Location         | Expected value                         |
| ---------------- | -------------------------------------- |
| Memory `4200H` | `A5H` — original transmitted data   |
| Register C       | `A5H` — reconstructed received byte |
| Memory `4300H` | `A5H` — stored received result      |
| Register B       | `00H` — eight iterations completed  |

In your current simulator setup, the receiver has been sampling zeros. Therefore, do not report `4300H = A5H` as an observed result unless you actually see that value in memory.

# Part 6 — Understand the delay subroutine in Part B

Part B uses the same delay subroutine as Part A:

```
DELAY: LXI D, 01H
BACK: DCX D
MOV A, D
ORA E
JNZ BACK
RET
```

The meaning is identical:

- `LXI D, 01H` initializes DE to one.
- `DCX D` decreases it to zero.
- `MOV A,D` copies D to A.
- `ORA E` checks whether D or E is nonzero.
- `JNZ BACK` repeats if the result is nonzero.
- `RET` returns to the main program.

The delay gives time between successive sampling operations. In a real communication setup, the transmitter's bit timing and receiver's sampling timing must be compatible.

Your shortened delay is suitable for practising the instruction flow, but it should not automatically be considered a reliable real-hardware bit period.

# Part 7 — Understand the full flow of both programs

The two programs have different jobs, but they share several concepts.

### Part A — Transmitter

Read `A5H` from memory `4200H`

`RLC` extracts the next bit into CY

Prepare `C0H` or `40H`

`SIM` controls the SOD output

Delay → restore saved data → decrement B

Repeat eight times.

### Part B — Receiver

`RIM` samples the SID input

`RLC` transfers the sampled bit into CY

`RAL` shifts the partial byte and inserts the bit

`MOV C,A` saves the partial result

Delay → decrement B → repeat

Store final byte at `4300H`

Repeat eight times, then store the reconstructed byte.

The main conceptual difference is that Part A extracts bits from an existing byte, whereas Part B builds a byte from incoming bits.

# Part 8 — What you should see in GNUSim8085

This is especially important because you have already executed both programs.

## 32. Part A: What to inspect

You entered `A5H` at memory address `4200H` and successfully assembled the transmitter.

When stepping through the program, observe these values:

| Program stage              | What to check                          |
| -------------------------- | -------------------------------------- |
| After `LXI H, 4200H`     | HL = `4200H`                         |
| After `MOV A, M`         | A = `A5H`                            |
| After `MVI B, 08H`       | B = `08H`                            |
| After `RLC`              | CY contains the next bit               |
| Immediately before `SIM` | A = `C0H` or `40H`                 |
| After each completed loop  | B decreases by one                     |
| At `HLT`                 | Execution stops after eight iterations |

You already verified the most important part: when the blue highlight is on `SIM`, A shows `C0H` or `40H`.

That is the expected control-byte sequence for `A5H`.

## 33. Part B: What to inspect

In the receiver, look at the following points:

| Instruction             | What to inspect                                  |
| ----------------------- | ------------------------------------------------ |
| `MVI B, 08H`          | B = `08H`                                      |
| `MVI C, 00H`          | C = `00H`                                      |
| `RIM`                 | Inspect A's most significant bit, D7             |
| `RLC`                 | Inspect CY                                       |
| `RAL` and `MOV C,A` | Inspect C                                        |
| `DCR B`               | Check the counter decreases                      |
| `STA 4300H`           | Inspect memory address `4300H` after execution |

Your standalone `RLC` test worked. Your `RIM` test returned `A = 00H`, and C remained zero during reception. That is consistent with SID supplying zero at the sampled times.

The I/O Ports panel you showed does not expose an obvious SID input control. Therefore, your current simulator setup has not yet demonstrated a complete serial loopback.

On hardware, the usual loopback arrangement connects SOD to SID with a jumper wire. In software, the simulator must support an equivalent connection or a way to supply the input bits. Merely executing the two programs consecutively does not guarantee reception.

For your exam: know the intended result and how to verify it, but distinguish the expected result from what you have actually observed.

# Part 9 — Common mistakes and important concepts

## 34. Why is `PUSH PSW` needed in Part A?

Because the transmitter temporarily replaces A with `C0H` or `40H` to configure `SIM`.

Without saving the rotated byte, the program would lose the data it was supposed to continue rotating.

`PUSH PSW` saves the accumulator and flags. `POP PSW` restores them.

## 35. Why is `ORI 40H` needed?

Because D6 must be 1 to enable serial output.

`40H = 01000000₂`, so OR-ing it with either `00H` or `80H` produces the required control value:

- `00H OR 40H = 40H`
- `80H OR 40H = C0H`

Without enabling SOD, setting D7 alone is not sufficient to enable the output.

## 36. Why do we use `RLC` and `RAL`?

They have different roles.

- `RLC` rotates A left, sending old D7 to CY and wrapping it into D0. In Part A, this extracts the next bit from the byte. In Part B, it moves the sampled SID bit into CY.
- `RAL` rotates A left through CY. In Part B, it inserts the new bit into the partial byte.

A useful memory aid:

RLC extracts; RAL inserts.

## 37. Why do we use `B = 08H`?

Because one byte has eight bits.

Each loop iteration handles one bit. Therefore, the counter starts at eight and is decremented once per iteration.

## 38. Why is a delay required?

The receiver needs time to sample the input, and the sender must leave each output bit available long enough for the receiving end to read it.

The delay creates a time interval between bits. The exact delay required depends on the communication timing and hardware or simulator.

## 39. Does `RIM` only read SID?

No. It reads the serial input level into D7 of A and also provides interrupt status and mask information in other accumulator bits.

This is why the program uses `RLC` to transfer D7 into CY.

## 40. Does `SIM` only send serial data?

No. Its name means Set Interrupt Mask. It also controls interrupt masks. The transmitter uses the relevant accumulator bits to control SOD.

## 41. What does `STA 4300H` do?

It stores the accumulator at the specified memory address.

If A = `A5H`, then memory at `4300H` becomes `A5H`.

It does not display the value on the screen by itself.

# Part 10 — Viva questions with answers

Use these to practise answering your professor in simple, precise language.

1\. What is the aim of this experiment?

2\. What is serial communication?

3\. What is the difference between serial and parallel communication?

4\. What are SID and SOD?

5\. What does SIM stand for?

6\. What does RIM stand for?

7\. Why is A5H used?

8\. Why is B initialized to 08H?

9\. Why is C cleared in Part B?

10\. What does RLC do?

11\. What does RAL do?

12\. Why is PUSH PSW used in Part A?

13\. Why is POP PSW used?

14\. What is the purpose of ORI 40H?

15\. What are the two values prepared before SIM?

16\. What is the purpose of CALL and RET?

17\. Why is a delay subroutine used?

18\. What does DCR B do?

19\. What does JNZ do?

20\. What does JNC do?

21\. What does STA 4300H do?

22\. What is the expected final received byte?

23\. Why is SP initialized?

24\. Why does C remain zero in your simulator?

25\. How can the hardware receiver be tested?

# Part 11 — How to memorize both programs

Don't memorize each instruction as an unrelated line. Memorize the purpose of each block.

## Part A — Transmission memory aid

Initialize → Read → Count → Rotate → Save → Prepare → Send → Delay → Restore → Count down → Repeat

| Block              | Instructions                                          | Purpose                |
| ------------------ | ----------------------------------------------------- | ---------------------- |
| Initialize stack   | `LXI SP, 4FFFH`                                     | Prepare stack          |
| Access data        | `LXI H,4200H` / `MOV A,M`                         | Load original byte     |
| Initialize counter | `MVI B,08H`                                         | Eight bits             |
| Extract bit        | `RLC`                                               | Next bit into CY       |
| Save data          | `PUSH PSW`                                          | Protect rotated byte   |
| Prepare output     | `MVI A,00H` / `JNC` / `MVI A,80H` / `ORI 40H` | Construct control byte |
| Transmit           | `SIM`                                               | Set SOD output         |
| Timing             | `CALL DELAY`                                        | Wait between bits      |
| Restore            | `POP PSW`                                           | Recover rotated data   |
| Loop               | `DCR B` / `JNZ TRANSMIT`                          | Repeat eight times     |
| Stop               | `HLT`                                               | Finish                 |

## Part B — Reception memory aid

Initialize → Read → Rotate → Load partial result → Insert bit → Save result → Delay → Count down → Repeat → Store

| Block              | Instructions                | Purpose                   |
| ------------------ | --------------------------- | ------------------------- |
| Initialize stack   | `LXI SP, 4FFFH`           | Prepare stack             |
| Initialize counter | `MVI B,08H`               | Eight bits                |
| Clear result       | `MVI C,00H`               | Start with zero           |
| Read input         | `RIM`                     | Sample SID                |
| Extract bit        | `RLC`                     | SID bit into CY           |
| Load partial byte  | `MOV A,C`                 | Retrieve accumulated data |
| Insert bit         | `RAL`                     | Shift left through CY     |
| Save result        | `MOV C,A`                 | Update partial byte       |
| Timing             | `CALL DELAY`              | Wait before next sample   |
| Loop               | `DCR B` / `JNZ LOOP1`   | Repeat eight times        |
| Store              | `MOV A,C` / `STA 4300H` | Save received byte        |
| Stop               | `HLT`                     | Finish                    |

# Part 12 — Final revision sheet

Before your practical or viva, make sure you can answer these without looking at your notes:

- What is serial communication?
- What are SID and SOD, and which pins are they on?
- What is the difference between SIM and RIM?
- Why is `A5H` equal to `10100101`?
- What does `RLC` do to A and CY?
- What does `RAL` do?
- Why does Part A save and restore PSW?
- Why are `40H` and `C0H` used before `SIM`?
- Why is B initialized to eight?
- How does the loop stop?
- What does the delay routine do?
- Why does Part B require incoming SID bits?
- Why is the expected received byte stored at `4300H`?
- How would you verify the result in GNUSim8085?

The most important conceptual distinction is this:

Part A prepares bits for output. Part B samples bits from input. Correct receiver code alone cannot guarantee that the right byte is reconstructed; the correct serial bits must actually reach SID at the sampling times.

If you understand that distinction, the role of every major instruction in both programs becomes much easier to explain.
