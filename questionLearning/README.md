# 8085 Microprocessor — Assembly Language Programs

A collection of classic 8085 ALP (Assembly Language Programming) exercises covering arithmetic, logical, block data, and search/sort operations.

---

## Table of Contents

1. [Addition of Two 8-bit Numbers](#1-addition-of-two-8-bit-numbers)
2. [Addition of Two 8-bit Numbers with Carry](#2-addition-of-two-8-bit-numbers-with-carry)
3. [Addition of Two 16-bit Numbers](#3-addition-of-two-16-bit-numbers)
4. [1's Complement of a 16-bit Number](#4-1s-complement-of-a-16-bit-number)
5. [2's Complement of a 16-bit Number](#5-2s-complement-of-a-16-bit-number)
6. [Multiplication of Two 8-bit Numbers](#6-multiplication-of-two-8-bit-numbers)
7. [Division of Two 8-bit Numbers](#7-division-of-two-8-bit-numbers)
8. [Block Move](#8-block-move)
9. [Block Compare](#9-block-compare)
10. [Block Exchange](#10-block-exchange)
11. [Block Insert](#11-block-insert)
12. [Block Delete](#12-block-delete)
13. [Moving Display (LED Pattern)](#13-moving-display-led-pattern)
14. [Binary to BCD Conversion](#14-binary-to-bcd-conversion)
15. [Binary to Gray Code Conversion](#15-binary-to-gray-code-conversion)
16. [Hex to ASCII Conversion](#16-hex-to-ascii-conversion)
17. [Square of a Number](#17-square-of-a-number)
18. [Positive or Negative Number Check](#18-positive-or-negative-number-check)
19. [Odd or Even Number Check](#19-odd-or-even-number-check)
20. [Sum of an Array of Numbers](#20-sum-of-an-array-of-numbers)
21. [Search Data in an Array](#21-search-data-in-an-array)
22. [Largest Number in an Array](#22-largest-number-in-an-array)

---

## 1. Addition of Two 8-bit Numbers

**Logic:** Load two 8-bit numbers from memory, add them, and store the result (no carry handling).

```asm
LXI H, 1000H     ; HL -> first number
MOV A, M         ; A = first number
INX H            ; HL -> second number
ADD M            ; A = A + second number
INX H
STA 1002H        ; Store result
HLT
```

---

## 2. Addition of Two 8-bit Numbers with Carry

**Logic:** Same as above, but also stores the carry (0 or 1) generated during addition, separately from the sum.

```asm
LXI H, 1000H
MOV A, M
INX H
MVI B, 00H       ; B = carry flag storage
ADD M
JNC LOOP         ; No carry -> skip
INR B            ; Carry occurred -> B = 1
LOOP:  STA 1004H ; Store sum
MOV A, C
STA 1003H        ; Store carry
HLT
```

---

## 3. Addition of Two 16-bit Numbers

**Logic:** Uses register pair addition (`DAD`) to add two 16-bit numbers, storing the sum and any carry-out.

```asm
LHLD C050H       ; HL = first 16-bit number
XCHG             ; DE = first number
LHLD C052H       ; HL = second 16-bit number
DAD D            ; HL = HL + DE
MVI C, 00H
JNC LOOP
INR C            ; Carry occurred
LOOP:  SHLD C054H ; Store 16-bit sum
MVI A, C
STA C056H         ; Store carry
HLT
```

---

## 4. 1's Complement of a 16-bit Number

**Logic:** Complement each byte of a 16-bit number using `CMA` (complement accumulator).

```asm
LXI H, C050H
MOV A, M
CMA
STA C052H
INX H
MOV A, M
CMA
STA C053H
HLT
```

---

## 5. 2's Complement of a 16-bit Number

**Logic:** 2's complement = 1's complement + 1. The carry from the low byte's +1 must propagate into the high byte.

```asm
LXI H, C050H
MVI B, 00H
MOV A, M
CMA
ADI 01H
STA C052H
JNC LOOP
INR B
LOOP:
INX H
MOV A, M
CMA
ADD B
STA C053H
HLT
```

---

## 6. Multiplication of Two 8-bit Numbers

**Logic:** Repeated addition — add the multiplicand to itself, multiplier number of times, tracking overflow into a 16-bit result.

```asm
LXI H, 0C050H
MOV B, M         ; B = multiplicand
MVI A, 00H
MVI D, 00H       ; D = high byte of result
INX H
MOV C, M         ; C = multiplier (loop counter)
LOOP:  ADD B
JNC LOOP2
INR D            ; carry -> increment high byte
LOOP2: DCR C
JNZ LOOP
STA 0C053H       ; Store low byte
MOV A, D
STA 0C054H       ; Store high byte
HLT
```

---

## 7. Division of Two 8-bit Numbers

**Logic:** Repeated subtraction — subtract the divisor from the dividend, counting how many times it can be subtracted (quotient), with the remainder left over.

```asm
LXI H, 0C050H
MOV A, M         ; A = dividend
MVI B, 00H       ; B = quotient counter
INX H
MOV C, M         ; C = divisor
LOOP:  SUB C
INR B
CMP C
JNC LOOP
; Remaining: store quotient (B) and remainder (A) to memory
STA 0C053H       ; Remainder
MOV A, B
STA 0C054H       ; Quotient
HLT
```

---

## 8. Block Move

**Logic:** Copies a block of data from a source address to a destination address, byte by byte.

```asm
MVI C, 05H
LXI H, 2100H     ; Source
LXI D, 2200H     ; Destination

LOOP: MOV A, M
      STAX D
      INX H
      INX D
      DCR C
      JNZ LOOP
HLT
```

---

## 9. Block Compare

**Logic:** Compares two blocks of data byte-by-byte. Stores `00H` if all bytes match, `FFH` if any mismatch is found.

```asm
LXI H, 2100H
LXI D, 2200H
MVI C, 05H
MVI B, 00H

LOOP: LDAX D
      CMP M
      JNZ NOMATCH
      INX H
      INX D
      DCR C
      JNZ LOOP
      JMP STORE

NOMATCH: MVI B, FFH

STORE: MOV A, B
       STA 2300H
HLT
```

---

## 10. Block Exchange

**Logic:** Swaps the contents of two memory blocks element by element using a temporary register.

```asm
LXI H, 2100H
LXI D, 2200H
MVI C, 05H

LOOP: LDAX D
      MOV B, A
      MOV A, M
      STAX D
      MOV A, B
      MOV M, A
      INX H
      INX D
      DCR C
      JNZ LOOP
HLT
```

---

## 11. Block Insert

**Logic:** Shifts existing elements to make room, then inserts a new value at the desired position.

```asm
LXI H, 2104H
LXI D, 2105H
MVI C, 03H

SHIFT: MOV A, M
       STAX D
       DCX H
       DCX D
       DCR C
       JNZ SHIFT

MVI A, 99H
STAX D
HLT
```

---

## 12. Block Delete

**Logic:** Shifts elements left to overwrite and remove a value from the block.

```asm
LXI D, 2102H
LXI H, 2103H
MVI C, 02H

SHIFT: MOV A, M
       STAX D
       INX H
       INX D
       DCR C
       JNZ SHIFT
HLT
```

---

## 13. Moving Display (LED Pattern)

**Logic:** Uses `RLC` (rotate left) to shift a lit LED bit pattern across an output port, with a software delay loop between shifts.

```asm
MVI A, 80H
OUT 03H          ; Configure I/O port

MVI A, 01H
START: OUT 01H
       CALL DELAY
       RLC
       JMP START

DELAY: MVI B, FFH
LOOP1: MVI C, FFH
LOOP2: DCR C
       JNZ LOOP2
       DCR B
       JNZ LOOP1
       RET
```

---

## 14. Binary to BCD Conversion

**Logic:** Repeatedly subtracts 10 from the binary value to find the tens digit, then packs tens and ones digits into a single BCD byte.

```asm
LXI H, 2000H
MOV A, M

MVI B, 0AH       ; decimal 10
MVI C, 00H       ; tens counter

DIVIDE: CMP B
        JC STORE
        SUB B
        INR C
        JMP DIVIDE

STORE: MOV D, A  ; D = ones digit
       MOV A, C  ; A = tens digit
       RLC
       RLC
       RLC
       RLC
       ORA D     ; Combine tens and ones into packed BCD

       LXI H, 2001H
       MOV M, A
       HLT
```

---

## 15. Binary to Gray Code Conversion

**Logic:** Gray code = binary XORed with itself shifted right by one bit (`RRC` then `XRA`).

```asm
LDA 2000H
MOV B, A
RRC
XRA B
STA 2001H
HLT
```

---

## 16. Hex to ASCII Conversion

**Logic:** Masks the lower nibble, checks if it's a digit (0–9) or a letter (A–F), and adds the appropriate ASCII offset.

```asm
LDA 2000H
ANI 0FH
CPI 0AH
JC SKIP
ADI 07H

SKIP: ADI 30H
      STA 2001H
      HLT
```

---

## 17. Square of a Number

**Logic:** Uses the input number as an offset into a pre-computed lookup table of squares (page C1H) to directly fetch the result.

```asm
LDA C500H
MOV L, A
MVI H, C1H
MOV A, M
STA C501H
HLT
```

---

## 18. Positive or Negative Number Check

**Logic:** Rotates the number left through carry to check the MSB (sign bit). If MSB = 1, the number is negative.

```asm
LXI H, C050H
MOV A, M
RAL              ; MSB -> Carry flag
JC LOOP          ; Carry set -> negative

MVI A, 00H       ; MSB = 0 -> positive
STA C051H
HLT

LOOP: MVI A, 01H ; MSB = 1 -> negative
STA C051H
HLT
```

---

## 19. Odd or Even Number Check

**Logic:** Rotates the number right through carry to check the LSB. If LSB = 1, the number is odd.

```asm
LDA C050H
RAR              ; LSB -> Carry flag
JC LOOP          ; Carry set -> odd

MVI A, 00H       ; LSB = 0 -> even
STA C055H
HLT

LOOP: MVI A, 01H ; LSB = 1 -> odd
STA C055H
HLT
```

---

## 20. Sum of an Array of Numbers

**Logic:** Adds all elements of an array, tracking overflow (carry) separately to form a 16-bit sum.

```asm
LXI H, C050H
MOV C, M         ; C = number of elements

MVI A, 00H       ; A = running sum
MVI B, 00H       ; B = carry count

INX H

LOOP: ADD M
      JNC NEXT
      INR B

NEXT: INX H
      DCR C
      JNZ LOOP

STA C060H        ; Store low byte of sum
MOV A, B
STA C061H        ; Store high byte (carry count)
HLT
```

---

## 21. Search Data in an Array

**Logic:** Linearly scans the array, comparing each element with the target value until a match is found or the array ends.

```asm
LDA C050H        ; Data to search
LXI H, C051H     ; Start of array
MVI C, 05H       ; Number of elements

LOOP: CMP M
      JZ FOUND
      INX H
      DCR C
      JNZ LOOP

MVI A, 00H       ; Not found
STA C060H
HLT

FOUND: MVI A, 01H ; Found
       STA C060H
       HLT
```

---

## 22. Largest Number in an Array

**Logic:** Assumes the first element is the largest, then compares each subsequent element, updating the largest value as needed.

```asm
LXI H, C050H     ; Number of elements
MOV C, M

INX H
MOV A, M         ; Assume first element is largest
DCR C

LOOP: INX H
      CMP M
      JNC NEXT   ; A >= M -> keep A
      MOV A, M   ; A < M -> update largest

NEXT: DCR C
      JNZ LOOP

STA C060H        ; Store largest value
HLT
```

---

## Notes

- All programs are written for the **8085 microprocessor** instruction set.
- Memory addresses (e.g., `1000H`, `C050H`) are examples — adjust based on your simulator/assembler's available memory map.
- `HLT` halts execution; ensure no unintended fall-through into subroutine code in a real ROM/EPROM setup.
- These programs assume a simulator environment (e.g., GNUSim8085) unless otherwise noted.
