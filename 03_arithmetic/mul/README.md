# MUL: flags and why

Programs run: `mul1.asm` (8 bit), `mul2.asm` (16 bit), `mul3.asm` (32 bit).



## What MUL does to the flags

Unsigned MUL produces a result twice as wide as the operands:

| Form | Computes | Upper half |
|---|---|---|
| `mul r/m8` | AX = AL × r/m8 | AH |
| `mul r/m16` | DX:AX = AX × r/m16 | DX |
| `mul r/m32` | EDX:EAX = EAX × r/m32 | EDX |

- **CF and OF are defined:** both are set to 1 if the upper half is non zero and cleared to 0 if the upper half is zero. They are always equal after MUL.
- **SF, ZF, AF, PF are undefined** . Whatever GDB shows for them is not a meaningful result of the multiplication and should not be explained as one.

OF here does not mean signed overflow, it only means the upper half is non zero.

---

## mul1: 8 bit, 25 × 10

```
AL = 25 = 0x19
25 × 10 = 250 = 0x00FA - AH = 0x00, AL = 0xFA
```

| Flag | Value | Why |
|---|---|---|
| CF | 0 | Upper half AH = 0, so the product fits in AL |
| OF | 0 | Same condition as CF |
| SF, ZF, AF, PF | undefined | Not defined by MUL |

## mul2: 16 bit, 3000 × 200

```
3000 = 0x0BB8, 200 = 0x00C8
3000 × 200 = 600000 = 0x000927C0 
DX = 0x0009, AX = 0x27C0
```

| Flag | Value | Why |
|---|---|---|
| CF | 1 | DX = 9 ≠ 0 and 600000 > 65535, so the product needs the upper register |
| OF | 1 | Same condition as CF |
| SF, ZF, AF, PF | undefined | Not defined by MUL |

## mul3: 32 bit, 100000 × 300000

```
100000 = 0x000186A0, 300000 = 0x000493E0
100000 × 300000 = 30,000,000,000 = 0x0000_0006_FC23_AC00
EDX = 0x00000006, EAX = 0xFC23AC00
```

| Flag | Value | Why |
|---|---|---|
| CF | 1 | EDX = 6 ≠ 0 and 3×10^10 > 2^32 - 1 (4,294,967,295), so it does not fit in EAX |
| OF | 1 | Same condition as CF |
| SF, ZF, AF, PF | undefined | Not defined by MUL |
