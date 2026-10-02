# SUB / SBB: flags and why

Programs run: `sub1.asm` (8 bit), `sub2.asm` (16 bit), `sub3.asm` (16 bit `sub` then `sbb`).




## sub1: 8 bit, 50 - 80

```
  0011 0010   (50) - 0101 0000   (80)
= 1110 0010   (0xE2 = 226 unsigned, 0XE2 = -30 signed as 256 - 30)
```

| Flag | Value | Why |
|---|---|---|
| CF | 1 | 50 < 80 unsigned, so a borrow is needed (the result wraps to 256 - 30 = 226) |
| OF | 0 | Both operands are positive, so no signed overflow is possible; -30 is within -128 - 127 |
| SF | 1 | Bit 7 of `1110 0010` is 1 (negative as signed) |
| ZF | 0 | Result is not 0 |
| PF | 1 | `1110 0010` has four 1-bits (even) |
| AF | 0 | Low nibbles needs no borrow |

## sub2: 16 bit, 1000 - 2000

```
  0x03E8   (1000) - 0x07D0   (2000)
= 0xFC18   (64536 unsigned , -1000 signed as 65536 - 1000)
```

| Flag | Value | Why |
|---|---|---|
| CF | 1 | 1000 < 2000 unsigned, so a borrow is needed |
| OF | 0 | Both operands are positive, and -1000 is within the signed 16-bit range |
| SF | 1 | Bit 15 of 0xFC18 is 1 |
| ZF | 0 | Result is not 0 |
| PF | 1 | Low byte 0x18 = `0001 1000` has two 1-bits, only the low byte counts |
| AF | 0 | Low nibbles needs no borrow |

## sub3: 16 bit, 0 - 1, then SBB AX, 0

### Step 1: `sub ax, [num2]`

```
  0x0000 - 0x0001
= 0xFFFF   (65535 unsigned, -1 signed)
```

| Flag | Value | Why |
|---|---|---|
| CF | 1 | 0 < 1 unsigned, so a borrow is needed |
| OF | 0 | 0 - 1 = -1 is within the signed 16-bit range, so the signed result is correct |
| SF | 1 | Bit 15 of 0xFFFF is 1 |
| ZF | 0 | Result is not 0 |
| PF | 1 | Low byte 0xFF has eight 1-bits (even) |
| AF | 1 | Low nibbles needs a borrow from bit 4 |

### Step 2: `sbb ax, 0`  (AX = AX - 0 - CF)

SBB subtracts the borrow left by step 1: 0xFFFF - 0 - 1 = 0xFFFE (-2).

| Flag | Value | Why |
|---|---|---|
| CF | 0 | 0xFFFF ≥ 0 + 1, so no borrow this time (the old CF was consumed) |
| OF | 0 | -1 - 1 = -2 is within the signed range |
| SF | 1 | Bit 15 of 0xFFFE is 1 |
| ZF | 0 | Result is not 0 |
| PF | 0 | Low byte 0xFE = `1111 1110` has seven 1-bits |
| AF | 0 | Low nibbles: 0xF - 0x0 - 1 = 0xE, no borrow |
