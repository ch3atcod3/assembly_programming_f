# 04.asm — A 32-bit Function Stack Frame

## Purpose

This is the 32-bit counterpart of the function stack-frame example. It uses `ESP` as the stack pointer and `EBP` as the frame pointer. In 32-bit mode, stack entries and return addresses in this program occupy 4 bytes.

## Execution flow

Let the initial value of `ESP` in `_start` be **S**.

1. `call work` pushes the address of the instruction after the call (the caller's return address) onto the stack. `ESP` decreases by 4 bytes to **S - 4**, and execution jumps to `work`.
2. `push ebp` saves the caller's old frame pointer. `ESP` decreases to **S - 8**.
3. `mov ebp, esp` establishes the new frame pointer: `EBP = S - 8`.
4. `sub esp, 8` reserves 8 bytes for two 4-byte local variables. `ESP` becomes **S - 16**.
5. `mov dword [ebp - 4], 100` stores 100 in the first local slot. `mov dword [ebp - 8], 200` stores 200 in the second. These stores do not change `ESP` or `EBP`.
6. `mov eax, [ebp - 4]` loads 100 into `EAX`; `mov ebx, [ebp - 8]` loads 200 into `EBX`.
7. `mov esp, ebp` releases the local-variable area and restores `ESP` to **S - 8**.
8. `pop ebp` restores the caller's saved frame pointer and advances `ESP` to **S - 4**.
9. `ret` pops the return address and advances `ESP` to **S**, returning to `_start` after `call work`.
10. `_start` exits using the Linux i386 system call convention: `EAX = 1` selects `exit`, and `EBX = 0` is the status. `int 0x80` invokes the system call.

## Stack layout and movement

At the deepest point in `work`:

| Address relative to S | Contents |
|---|---|
| S - 4 | Return address pushed by `call` |
| S - 8 | Saved caller `EBP` |
| S - 12 (`EBP - 4`) | Local value 100 |
| S - 16 (`EBP - 8`) | Local value 200; current `ESP` |

`ESP` decreases when values are pushed or space is reserved, and increases when saved values are popped or the frame is dismantled. `EBP` remains fixed while the locals are accessed, so the program can refer to them at stable offsets.

## Runtime observations

The program has no print instructions. Inside `work`, the expected register values are `EAX = 100` and `EBX = 200`. The function restores the stack before returning. The caller then exits with status 0. This is a 32-bit ELF program and requires 32-bit execution support on the Linux system. NASM was not available in the environment used to prepare this README, so these are expected observations from the source, not recorded execution results.

## Assemble and run on Linux with 32-bit support

```bash
nasm -f elf32 04.asm -o 04.o
ld -m elf_i386 -o 04 04.o
./04
echo $?
```

Expected exit status: `0`. If the system cannot execute 32-bit ELF programs, install/enable the appropriate 32-bit runtime support or use a suitable 32-bit Linux environment.
