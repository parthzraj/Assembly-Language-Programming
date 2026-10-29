## `CALL` and `RET` in 8085 Microprocessor

A subroutine is a section of code that performs a particular task and can be called whenever that task is needed. It helps us avoid writing the same instructions repeatedly.

For example, suppose a program needs to add the contents of the accumulator to itself five times in different parts of the program. Instead of writing `ADD A` repeatedly, we can write the operation once in a subroutine and call it whenever required.

### 1. What does `CALL` do?

The `CALL` instruction transfers program execution to a subroutine.

Syntax:

```
CALL address
```

Example:

```
CALL 2050H
```

This tells the 8085 to execute the subroutine beginning at memory address `2050H`.

However, before jumping there, the processor must remember where it should return after the subroutine finishes. It does this by saving the return address on the stack.

The return address is the address of the instruction immediately following `CALL`.

### 2. What does `RET` do?

The `RET` (Return) instruction brings execution back to the instruction following the original `CALL`.

```
RET
```

It retrieves the saved return address from the stack and loads it into the Program Counter (PC).

The PC then points to the next instruction in the main program.

### 3. Understand the complete process

Consider this example:

```
2000H: MVI A, 05H
2002H: CALL 2050H
2005H: HLT

2050H: ADD A
2051H: RET
```

Assume the Stack Pointer is initialized to a valid RAM address.

![1793289035711](image/31Call-Ret/1793289035711.png)

Notice that the processor does not return to `2002H`, where `CALL` is located. It returns to `2005H`, the instruction immediately after the three-byte `CALL` instruction.

### 4. What happens to the stack?

The stack is a reserved area of memory used to store temporary information, including return addresses.

The Stack Pointer (SP) holds the address associated with the top of the stack. In the 8085, the stack normally grows toward lower memory addresses.

When executing `CALL`:

1. The processor calculates the return address.
2. It saves that 16-bit address on the stack.
3. The SP decreases by two.
4. The PC is loaded with the subroutine's starting address.

When executing `RET`:

1. The processor retrieves the saved 16-bit return address from the stack.
2. The SP increases by two.
3. The restored address is loaded into the PC.

This is why `CALL` and `RET` work together: `CALL` saves the return location, and `RET` retrieves it.

### 5. Do `CALL` and `RET` affect flags?

Neither `CALL` nor `RET` directly modifies the 8085's condition flags.

| Instruction      | Purpose                                           | Flags affected  |
| ---------------- | ------------------------------------------------- | --------------- |
| `CALL address` | Saves the return address and transfers control    | None            |
| `RET`          | Restores the return address and resumes execution | None            |
| `ADD A`        | Adds A to itself                                  | S, Z, AC, P, CY |

In the example, the flags may change because of `ADD A`, but not because of `CALL` or `RET`.

### 6. `CALL` vs `JMP`

| Feature                                       | `CALL`                      | `JMP`                              |
| --------------------------------------------- | ----------------------------- | ------------------------------------ |
| Transfers execution                           | Yes                           | Yes                                  |
| Saves return address on the stack             | Yes                           | No                                   |
| Automatically returns to the next instruction | With `RET`                  | No                                   |
| Typical purpose                               | Execute a reusable subroutine | Transfer control to another location |

For example, if you use `JMP 2050H`, the processor jumps to `2050H` without saving where it came from. A `RET` at the destination would only work correctly if a valid return address had already been placed on the stack.

### 7. Important exam points

- `CALL` is a 3-byte instruction in the 8085.
- `RET` is a 1-byte instruction.
- The Program Counter (PC) tracks the next instruction to execute.
- The Stack Pointer (SP) tracks the top of the stack.
- The return address is the address immediately following `CALL`.
- The 8085 saves a 16-bit return address because memory addresses are 16 bits wide.
- A subroutine can use registers, but it must preserve any values that the calling program still needs. `CALL` does not automatically save all registers.
- If a subroutine calls another subroutine, the stack keeps the return addresses in last-in, first-out (LIFO) order.

Memory aid: `CALL` = save the way back and go there. `RET` = retrieve the way back and return.


## 3.11 `RNZ` — Return if Not Zero (8085)

`RNZ` is a conditional return instruction. It checks the Zero flag (`Z`) and decides whether the processor should return from the current subroutine.

### 1. Syntax

```
RNZ
```

There is no address or label after `RNZ`. It uses the return address already stored on the stack by a previous `CALL`.

### 2. How does `RNZ` work?

The decision depends on the Zero flag:

| Zero flag | Meaning                                                        | What `RNZ` does                 |
| --------- | -------------------------------------------------------------- | --------------------------------- |
| `Z = 0` | The previous flag-setting operation produced a non-zero result | Returns to the calling program    |
| `Z = 1` | The previous flag-setting operation produced a zero result     | Continues to the next instruction |

Memory trick: `RNZ` means Return if Not Zero.

### 3. Example: Understand it line by line

```
CALL CHECK

; Main program continues here
HLT

CHECK:
    CPI 3CH
    RNZ
    MVI A, 00H
    RET
```

Assume the accumulator initially contains `25H`.

Step 1 — `CALL CHECK`

The processor saves the address of the instruction following `CALL` on the stack and jumps to `CHECK`.

Step 2 — `CPI 3CH`

`CPI` compares the accumulator with `3CH` by internally calculating:

\\[ A-3CH \\]

It does not store the subtraction result in A. It updates the flags.

Here:

\\[ 25H-3CH \neq 0 \\]

Therefore, `Z = 0`.

Step 3 — `RNZ`

Because `Z = 0`, the processor returns immediately to the instruction after `CALL`. The instructions `MVI A, 00H` and `RET` are skipped.

If A had contained `3CH`, the comparison would produce `Z = 1`. In that case, `RNZ` would not return, and the processor would execute `MVI A, 00H`, followed by `RET`.

### 4. Why is `RNZ` useful in a digital clock?

Suppose a seconds counter runs from `00` to `59`, and you need to reset it when it reaches `59H`.

```
CPI 3BH       ; Compare A with 3BH (59 decimal)
RNZ           ; If not equal, return immediately
MVI A, 00H    ; If equal, reset seconds
RET           ; Return to the calling program
```

Here, the clock uses packed BCD, so `3BH` is not valid BCD. For a proper BCD clock, compare against `59H` instead:

```
CPI 59H
RNZ
MVI A, 00H
RET
```

The logic is:

- A is not `59H`: `Z = 0`, so `RNZ` returns without resetting the seconds.
- A is `59H`: `Z = 1`, so execution continues and resets the seconds to `00H`.

The calling program can then increment the minutes counter.

### 5. `RNZ` versus `JNZ`

| Feature                              | `RNZ`                         | `JNZ address`                       |
| ------------------------------------ | ------------------------------- | ------------------------------------- |
| Full meaning                         | Return if Not Zero              | Jump if Not Zero                      |
| Checks Zero flag                     | Yes                             | Yes                                   |
| Uses a specified destination address | No                              | Yes                                   |
| Uses the return address on the stack | Yes, when taken                 | No                                    |
| Typical use                          | Exit a subroutine conditionally | Repeat a loop or branch conditionally |

Important: `RNZ` only makes sense when a valid return address is available on the stack. If the Zero flag is `0` and no valid return address exists, the processor may jump to an unintended address.

Also, `RNZ` does not itself change the flags. The flags are usually set by the preceding instruction, such as `CMP`, `CPI`, `ADD`, or `DCR`.
