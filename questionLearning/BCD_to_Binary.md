# 8085 ALP to Convert Packed BCD to Binary

---

## Theory

A **packed BCD (Binary Coded Decimal)** number stores two decimal digits in a single byte — the **upper nibble** represents the *tens* digit, and the **lower nibble** represents the *units* digit. For example, the decimal number **47** is stored as packed BCD `47H`, where:

- Upper nibble = `4` (tens digit)
- Lower nibble = `7` (units digit)

However, this is *not* the same as the actual binary value of 47. The binary equivalent of decimal 47 is `2FH`, not `47H`. So to convert packed BCD into true binary, the processor must:

1. Separate the two nibbles (tens digit and units digit).
2. Multiply the tens digit by 10 (since it represents the "tens place").
3. Add the units digit to that result.
4. The final sum is the actual binary value of the original decimal number.

Since the 8085 has no hardware multiplier, **multiplication by 10 is done using repeated addition** — adding 10, tens-digit number of times.

**Formula:**
```
Binary Value = (Tens Digit x 10) + Units Digit
```

**Example:**
If packed BCD = `47H`:
- Tens digit = 4, Units digit = 7
- Binary value = (4 x 10) + 7 = 47 (decimal) = `2FH` (hex)

---

## Logic / Algorithm

1. Load the packed BCD byte from memory.
2. Extract the **lower nibble** (units digit) by masking with `0FH`.
3. Extract the **upper nibble** (tens digit) by masking with `F0H`, then shift it right 4 times (using `RRC` four times) to bring it into the lower nibble position.
4. Multiply the tens digit by 10 using a repeated-addition loop.
5. Add the units digit to the result of the multiplication.
6. Store the final binary value in memory.

---

## Annotated Program

```asm
; ============================================================
; 8085 ALP: Convert Packed BCD Number to Binary
; Input  : Packed BCD number stored at memory location 2000H
; Output : Equivalent binary value stored at memory location 2001H
; ============================================================

LXI H, 2000H     ; HL = 2000H -> point to memory location holding
                  ; the packed BCD input number

MOV A, M          ; A = [2000H] -> load the packed BCD number into
                  ; the accumulator (e.g., A = 47H for decimal 47)

; ------------------------------------------------------------
; Step 1: Extract the UNITS digit (lower nibble)
; ------------------------------------------------------------

MOV B, A          ; B = A -> save a copy of the original BCD number
                  ; (so we can extract the upper nibble later)

ANI 0FH           ; A = A AND 0FH -> mask out upper nibble,
                  ; keeping only the lower nibble (units digit)
                  ; e.g., 47H AND 0FH = 07H

MOV C, A          ; C = units digit -> store units digit safely in C

; ------------------------------------------------------------
; Step 2: Extract the TENS digit (upper nibble)
; ------------------------------------------------------------

MOV A, B          ; A = original BCD number (restore from B)

ANI F0H           ; A = A AND F0H -> mask out lower nibble,
                  ; keeping only the upper nibble (tens digit)
                  ; e.g., 47H AND F0H = 40H

RRC               ; Rotate A right (no carry link) -> shifts all
RRC               ; bits right by 1 each time.
RRC               ; Four RRC instructions together shift the
RRC               ; upper nibble into the lower nibble position.
                  ; e.g., 40H -> 04H (tens digit isolated = 4)

; ------------------------------------------------------------
; Step 3: Multiply tens digit by 10 (repeated addition)
; ------------------------------------------------------------

MVI D, 0AH        ; D = 0AH (decimal 10) -> the value to be added
                  ; repeatedly (since tens digit means "x10")

MOV E, A          ; E = tens digit -> used as the loop counter
                  ; (how many times to add 10)

MVI A, 00H        ; A = 00H -> reset accumulator to 0,
                  ; this will hold the running multiplication result

MULTIPLY: ADD D   ; A = A + D -> add 10 to the accumulator
          DCR E   ; E = E - 1 -> decrement loop counter
          JNZ MULTIPLY
                  ; If E != 0, repeat the loop
                  ; (this effectively computes tens_digit x 10)

; ------------------------------------------------------------
; Step 4: Add the units digit to complete the conversion
; ------------------------------------------------------------

ADD C             ; A = A + C -> add the units digit that was
                  ; saved earlier
                  ; A now holds: (tens digit x 10) + units digit
                  ; i.e., the true binary value of the decimal number

; ------------------------------------------------------------
; Step 5: Store the final binary result
; ------------------------------------------------------------

LXI H, 2001H      ; HL = 2001H -> point to output memory location

MOV M, A          ; [2001H] = A -> store the computed binary value

HLT               ; Halt the program execution
```

---

## Dry Run Example

Suppose memory location `2000H` contains `47H` (packed BCD for decimal 47):

| Step | Operation | A | B | C | D | E | Explanation |
|------|-----------|---|---|---|---|---|-------------|
| 1 | `MOV A,M` | 47H | - | - | - | - | Load packed BCD |
| 2 | `MOV B,A` | 47H | 47H | - | - | - | Save copy |
| 3 | `ANI 0FH` | 07H | 47H | - | - | - | Extract units digit |
| 4 | `MOV C,A` | 07H | 47H | 07H | - | - | Save units digit = 7 |
| 5 | `MOV A,B` | 47H | 47H | 07H | - | - | Restore original |
| 6 | `ANI F0H` | 40H | 47H | 07H | - | - | Extract tens nibble |
| 7 | `RRC x4` | 04H | 47H | 07H | - | - | Tens digit isolated = 4 |
| 8 | `MVI D,0AH` | 04H | 47H | 07H | 0AH | - | D = 10 |
| 9 | `MOV E,A` | 04H | 47H | 07H | 0AH | 04H | Loop count = 4 |
| 10 | `MVI A,00H` | 00H | 47H | 07H | 0AH | 04H | Reset accumulator |
| 11 | Loop x4 (`ADD D`) | 28H | 47H | 07H | 0AH | 00H | 10+10+10+10 = 40 (28H) |
| 12 | `ADD C` | 2FH | 47H | 07H | 0AH | 00H | 40 + 7 = 47 decimal = 2FH |
| 13 | `MOV M,A` | 2FH | - | - | - | - | Store 2FH at 2001H |

**Result:** Memory location `2001H` = `2FH` = decimal **47** ✅

---

## Key Concepts Used

| Instruction | Purpose |
|-------------|---------|
| `ANI 0FH` | Masks upper nibble, isolates lower nibble (units digit) |
| `ANI F0H` | Masks lower nibble, isolates upper nibble (tens digit) |
| `RRC` (x4) | Rotates upper nibble into lower nibble position (nibble swap without using carry) |
| `ADD D` in loop | Performs multiplication by repeated addition (8085 has no `MUL` instruction) |
| `DCR` / `JNZ` | Implements a counted loop |

---

## Notes

- This program assumes the input is a **valid packed BCD** number where both nibbles are in the range 0–9.
- The result is limited to 8 bits, so this works correctly for packed BCD inputs up to `99H` (decimal 99), since the largest possible binary result (99) still fits within a single byte (max 255).
- Written for the **8085 microprocessor** instruction set; assumes a simulator environment (e.g., GNUSim8085) unless otherwise noted.
