# 8085 Microprocessor — Assembly Language Programs (Part 2)

Additional 8085 ALP exercises: square root, factorial, and block data manipulation.

---

## Table of Contents

1. [Square Root of a Number](#1-square-root-of-a-number)
2. [Factorial of a Number](#2-factorial-of-a-number)
3. [Block Insert](#3-block-insert)
4. [Block Delete](#4-block-delete)

---

## 1. Square Root of a Number

**Logic:** Uses the property that the sum of the first *n* odd numbers equals *n²*. Repeatedly subtracts successive odd numbers (1, 3, 5, 7, ...) from the input until it can no longer subtract; the count of successful subtractions is the integer square root.

```asm
LDA C050H        ; Load input number into A

MVI B, 01H       ; B = 01H (first odd number)
MVI C, 00H       ; C = count of subtractions (result)

LOOP: CMP B      ; Compare A with current odd number
      JC DONE    ; If A < B, stop — C holds the square root

      SUB B      ; A = A - B
      INR C      ; Increment result count

      INR B      ; B = B + 2 (next odd number)
      INR B      ; Sequence: 1, 3, 5, 7, ...

      JMP LOOP   ; Repeat

DONE: MOV A, C   ; Move result to A
      STA C051H  ; Store square root at C051H
      HLT        ; Stop
```

---

## 2. Factorial of a Number

**Logic:** Multiplies the running factorial by successive values from N down to 1, using a repeated-addition multiplication subroutine (since 8085 has no native `MUL` instruction).

```asm
LDA C050H        ; Load the number into A

MOV B, A         ; B = number
MVI A, 01H       ; A = 1 (initial factorial value)

LOOP: CALL MULT  ; Multiply factorial (A) by B

      DCR B      ; B = B - 1
      JNZ LOOP   ; Repeat until B = 0

STA C051H        ; Store factorial result
HLT              ; Stop

; --------------------------------
; Multiplication subroutine
; A x B using repeated addition
; --------------------------------

MULT: MOV D, A   ; D = multiplicand (current factorial)
      MVI A, 00H ; A = 0 (result accumulator)
      MOV C, B   ; C = multiplier

MLOOP: ADD D     ; A = A + D
       DCR C     ; Decrease multiplier
       JNZ MLOOP ; Repeat

       RET       ; Return from subroutine
```

> **Note:** This works correctly for small values of N (results fitting in 8 bits, i.e., N ≤ 5, since 6! = 720 overflows a byte). For larger factorials, a 16-bit result storage and multiplication routine would be needed.

---

## 3. Block Insert

**Logic:** Shifts existing array elements one position to the right (starting from the end) to create space, then inserts a new value at the freed position.

```asm
LXI H, 2104H     ; HL -> last element to shift
LXI D, 2105H     ; DE -> destination (one ahead)

MVI C, 03H       ; Number of elements to shift

SHIFT: MOV A, M
       STAX D
       DCX H
       DCX D
       DCR C
       JNZ SHIFT

MVI A, 99H       ; New value to insert
STAX D
HLT
```

---

## 4. Block Delete

**Logic:** Shifts array elements one position to the left, overwriting the element to be deleted.

```asm
LXI D, 2102H     ; DE -> destination (deletion point)
LXI H, 2103H     ; HL -> source (element after deletion point)

MVI C, 02H       ; Number of elements to shift

SHIFT: MOV A, M
       STAX D
       INX H
       INX D
       DCR C
       JNZ SHIFT
HLT
```

---

## Notes

- All programs are written for the **8085 microprocessor** instruction set.
- Memory addresses (e.g., `C050H`, `2104H`) are examples — adjust based on your simulator/assembler's available memory map.
- The factorial routine's multiplication subroutine (`MULT`) uses repeated addition since the 8085 lacks a hardware multiply instruction.
- These programs assume a simulator environment (e.g., GNUSim8085) unless otherwise noted.
