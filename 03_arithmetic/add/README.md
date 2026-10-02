# ADD / ADC
Flags are read **immediately after the ADD/ADC**. The exit sequence rewrites the flags ,reading them after it would show the XOR's result, not the addition's. 

## add1: 8 bit, 120 + 10

```
  0111 1000   (120)
+ 0000 1010   (10)
= 1000 0010   (0x82 = 130 unsigned, -126 signed)
```

| Flag | Value | Why |
|---|---|---|
| CF | 0 | 130 ≤ 255, so there is no carry out of bit 7 |
| OF | 1 | Carry into bit 7 is 1 and carry out of bit 7 is 0 . In signed terms, 120 + 10 = 130 > 127, so two positives produced a negative |
| SF | 1 | Bit 7 of `1000 0010` is 1 |
| ZF | 0 | Result is not 0 |
| PF | 1 | `1000 0010` has two 1-bits (even) |
| AF | 1 | Low nibbles: 1000 + 1010 = 1 0010, so there is a carry out of bit 3 (8 + 10 = 18 > 15) |

## add2: 16 bit, 32000 + 500

```
  0x7D00   (32000) + 0x01F4   (500)
= 0x7EF4   (32500)   0111 1110 1111 0100
```

| Flag | Value | Why |
|---|---|---|
| CF | 0 | 32500 ≤ 65535 |
| OF | 0 | 32500 ≤ 32767; no carry into or out of bit 15 |
| SF | 0 | Bit 15 of 0x7EF4 is 0 |
| ZF | 0 | Result is not 0 |
| PF | 0 | Low byte 0xF4 = `1111 0100` has five 1-bits (odd) |
| AF | 0 | No carry out of bit 3 |

## add3: 16 bit, 0xFFFF + 1, then ADC AX, 0

### Step 1: `add ax, [num2]`

```
  1111 1111 1111 1111   (0xFFFF = 65535,-1 signed)
+ 0000 0000 0000 0001
= 1 0000 0000 0000 0000   -> AX = 0x0000, carry out = 1
```

| Flag | Value | Why |
|---|---|---|
| CF | 1 | 65536 does not fit in 16 bits, so there is a carry out of bit 15 |
| OF | 0 | Carry into bit 15 = 1 and carry out = 1, so no signed overflow.|
| SF | 0 | Result bit 15 is 0 |
| ZF | 1 | AX = 0 |
| PF | 1 | Low byte `0000 0000` has zero 1-bits (even) |
| AF | 1 |  There is a carry out of bit 3 |

### Step 2: adc ax, 0   (AX = 0 + 0 + CF)

ADC adds the carry left by step 1, so AX = 0 + 0 + 1 = 1.

| Flag | Value | Why |
|---|---|---|
| CF | 0 | 0 + 0 + 1 = 1 does not carry out of bit 15  |
| OF | 0 | Small positive result, no signed overflow |
| SF | 0 | Bit 15 is 0 |
| ZF | 0 | AX = 1 |
| PF | 0 | `0000 0001` has one 1-bit (odd) |
| AF | 0 | No carry out of bit 3 |
