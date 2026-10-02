# DIV

Programs run: `div1.asm` (8 bit divisor), `div2.asm` (16 bit divisor), `div3.asm` (32 bit divisor).



## What DIV does to the flags

All six arithmetic flags (CF, OF, SF, ZF, AF, PF) are undefined after DIV. DIV does not report anything through the flags. The flags GDB shows are not a property of the quotient or remainder, and they may differ between CPUs. Since `mov` does not modify flags and these programs run no flag changing instruction before the DIV, GDB will most likely show the same flags before and after, typically only the `IF` and `AF` flag, but even if some change, they carry no meaning.

What DIV signals instead is an **exception**: a divide error is raised if

1. the divisor is 0, or
2. the quotient does not fit in the destination register.

All three programs avoid both, shown below.

| Form | Dividend | Quotient | Remainder | Fails if quotient exceeds |
|---|---|---|---|---|
| `div r/m8` | AX | AL | AH | 255 |
| `div r/m16` | DX:AX | AX | DX | 65535 |
| `div r/m32` | EDX:EAX | EAX | EDX | 4,294,967,295 |

---

## div1: 100 / 7 (8-bit divisor)

```
AX = 100 = 0x0064, BL = 7
100 = 14 × 7 + 2   (14 × 7 = 98)
-> AL = 14 = 0x0E, AH = 2 = 0x02, AX = 0x020E
```

- Divisor is non-zero and quotient 14 ≤ 255, so no #DE.
- Flags CF, OF, SF, ZF, AF, PF all are undefined.

## div2: 50000 / 300 (16-bit divisor)

```
DX:AX = 0x0000:0xC350 = 50000, BX = 300 = 0x012C
50000 = 166 × 300 + 200   (166 × 300 = 49800)
-> AX = 166 = 0x00A6, DX = 200 = 0x00C8
```

- Divisor is non-zero and quotient 166 ≤ 65535, so no #DE. (Equivalently DX = 0 < 300, the condition for the quotient to fit.)
- All six flas are undefined.

## div3: 300000000 / 1000 (32-bit divisor)

```
EDX:EAX = 0x00000000:0x11E1A300 = 300,000,000, EBX = 1000 = 0x3E8
300,000,000 = 300,000 × 1000 + 0
EAX = 300,000 = 0x000493E0, EDX = 0
```

- Divisor is non-zero and quotient 300,000 ≤ 2^32 - 1, so no #DE. The remainder is 0 because 1000 divides 300,000,000 exactly.
-All six flas are undefined.
