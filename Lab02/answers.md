# Lab 02 Answers
**Student Name:** Savannah Copeland

---

## How to decode an instruction encoding

For each question below, find the hex encoding in your disassembly log, convert it to 32-bit binary, then fill in each field using the bit layout diagram.

**Worked Example: `MOVZ X1, #1` (encoding: `d2800021`)**

Step 1 — Convert hex to binary (4 bits per digit):
```
d    2    8    0    0    0    2    1
1101 0010 1000 0000 0000 0000 0010 0001
```

Step 2 — Map bits to the MOVZ field layout:

| 31 | 30-29 | 28-23  | 22-21 | 20-5             | 4-0   |
|----|-------|--------|-------|------------------|-------|
| sf | 10    | 100101 | hw    | imm16            | Rd    |
| 1  | 10    | 100101 | 00    | 0000000000000001 | 00001 |

Step 3 — Identify each field:
- `sf` = 1 → 64-bit register
- `hw` = 00 → no shift (LSL #0)
- `imm16` = 0000000000000001 = 1
- `Rd` = 00001 = X1

---

## Section 3.2 — Interpreting Instruction Encodings

---

**1. `MOVZ X0, #5`**

Hex encoding (from disassembly log): `0x`

Binary (32-bit):
```
_d       _2        _8      _0       _0       _0       _a       _0
_1101___ _0010___ __1000__ _0000___ _0000___ _0000___ _1010___ _0000___
```

| 31 | 30-29 | 28-23  | 22-21 | 20-5  |              4-0 |
|----|-------|--------|-------|-------|             -----|
| sf | 10    | 100101 | hw    | imm16          |  Rd     |
| 1  | 10    | 100101 | 00    |0000000000000101| 00000   |

- `sf` = 1
- `hw` = 00
- `imm16` = 0000000000000101
- `Rd` = 00000

---

**2. `ADD X4, X4, X0`**

Hex encoding (from disassembly log): `0x`

Binary (32-bit):
```
_8       _b       _0       _0       _0       _0       _8       _4
_1000___ _1011___ _0000___ _0000___ _0000___ _0000___ _1000___ _0100___
```

| 31 | 30 | 29 | 28-24 | 23-22 | 20-16 | 15-10 | 9-5 | 4-0 |
|----|----|----|-------|-------|-------|-------|-----|-----|
| sf | op | S  | 01011 | shift | Rm    | imm6  | Rn  | Rd  |
| 1  | 0  | 0  | 01011 | 00    |00000  | 000000|00100|00100|
- `Rm` (binary) = 00000
- `Rn` (binary) = 00100
- `Rd` (binary) = 00100

---

**3. `SUBS X0, X0, X1`**

Hex encoding (from disassembly log): `0x`

Binary (32-bit):
```
eb010000
_e       _b       _0       _1       _0       _0       _0       _0
_1110___ _1011___ _0000___ _0001___ _0000___ _0000___ _0000___ _0000___
```

| 31 | 30 | 29 | 28-24 | 23-22 | 20-16 | 15-10 | 9-5 | 4-0 |
|----|----|----|-------|-------|-------|-------|-----|-----|
| sf | op | S  | 01011 | shift | Rm    | imm6  | Rn  | Rd  |
| 1  | 1  | 1  | 01011 |  00   | 00001 |000000 |00000|00000|

Compare the `op` and `S` bits to `ADD` above:
- How does the encoding differ to signal that condition flags should be updated?
in SUBS S is a 1, which signals to update flags, the op only changes the arithmetic used.

---

**4. `B.NE sum_loop`**

Hex encoding (from disassembly log): `0x`

Binary (32-bit):
```
54ffffa1
_5       _4       _f       _f       _f       _f       _a       _1
_0101___ _0100___ _1111___ _1111___ _1111___ _1111___ __1001__ __0001__
```

| 31-24    | 23-5  | 4 | 3-0  |
|----------|-------|---|------|
| 01010100 | imm19 | 0 | cond |
| 01010100 |1111111111111111100|  1| 0001 |

- `imm19` (binary) = 1111111111111111100
- `imm19` as a two's complement integer = -4
- Byte offset (imm19 × 4) = -16
- `B.NE` address (from disassembly) = 0X1c
- `sum_loop` address (from disassembly) =0X10
- Do they match? no.

---

## Section 4.1 — Logical Immediate Values

`MOVZ` and `MOVK` each write a 16-bit immediate into one of four slots in a 64-bit register. The `LSL` shift selects which slot:

| bits 63-48 | bits 47-32 | bits 31-16 | bits 15-0 |
|------------|------------|------------|-----------|
| LSL #48    | LSL #32    | LSL #16    | LSL #0    |

`MOVZ` writes the selected slot and **zeros** all others.
`MOVK` writes the selected slot and **keeps** all other bits unchanged.

Use this layout to trace the value of X5 step by step before answering.

---
MOVZ
| bits 63-48 | bits 47-32 | bits 31-16 | bits 15-0 |
|------------|------------|------------|-----------|
| 0x0000   | 0x0000    | 0x0000   | 0x5678   |

MOVK
| bits 63-48 | bits 47-32 | bits 31-16 | bits 15-0 |
|------------|------------|------------|-----------|
| 0x0000   | 0x0000    | 0x1234    | 0x5678    |

**X5** (after `MOVZ` + `MOVK`):
`X5 = 0x0000000012345678`


| bits 63-48 | bits 47-32 | bits 31-16 | bits 15-0 |
|------------|------------|------------|-----------|
| 0x0000   | 0x3ffc    | 0x0000    | 0x3ffc    |
| 0x0000   | 0x0000   | 0x1234    | 0x5678   |
| 0x0000   | 0x0000   | 0x0000    | 0x1678   |

**X6** (after `AND X6, X5, #0x00003ffc00003ffc`):
`X6 = 0x0x0000000000001678`

| bits 63-48 | bits 47-32 | bits 31-16 | bits 15-0 |
|------------|------------|------------|-----------|
| 0x0000   | 0x3ffc    | 0x0000    | 0x3ffc    |
| 0x0000   | 0x0000   | 0x1234    | 0x5678   |
| 0x0000   | 0x3ffc   | 0x1234    | 0x5678   |


**X7** (after `ORR X7, X5, #0x00003ffc00003ffc`):
`X7 = 0x00003ffc12345678`

---

## Section 5 — Instruction Aliases

- What is the base instruction that `CMP X0, X1` translates to?
SUBS
- What is the full expanded form (including all operands)?
SUBS destination register, first operand, second operand
