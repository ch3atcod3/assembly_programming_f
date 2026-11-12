# 05.asm — Pushing and Popping 64-bit Values

## Purpose

This program demonstrates that a 64-bit `push`/`pop` uses 8-byte stack slots in 64-bit mode. It pushes a large 64-bit value and a smaller value, then pops them in reverse order.

## Execution flow

Let the initial `RSP` be **S**.

1. `mov rax, 0x1122334455667788` loads the 64-bit pattern into `RAX`. `mov rbx, 0x1234` loads the second value into `RBX`. Neither instruction changes the stack.
2. `push rax` decreases `RSP` by 8 bytes to **S - 8** and stores `0x1122334455667788` on the stack.
3. `push rbx` decreases `RSP` by another 8 bytes to **S - 16** and stores `0x1234` at the new top.
4. `nop` has no effect.
5. `pop rbx` retrieves the most recently pushed value, `0x1234`, into `RBX`; `RSP` increases to **S - 8**.
6. `pop rax` retrieves `0x1122334455667788` into `RAX`; `RSP` increases to **S**.
7. The program then sets `RAX = 60` and `RDI = 0`, and invokes the Linux x86-64 exit system call.

## Stack movement and LIFO order

| Instruction | Value retrieved/stored | RSP after instruction |
|---|---|---:|
| Start | — | S |
| `push rax` | Stores `0x1122334455667788` | S - 8 |
| `push rbx` | Stores `0x1234` | S - 16 |
| `pop rbx` | Retrieves `0x1234` | S - 8 |
| `pop rax` | Retrieves `0x1122334455667788` | S |

The smaller value is popped first because it was pushed last. After both pops, the stack pointer returns to its starting value. Notice that the program subsequently overwrites `RAX` with 60 to select the exit system call, so the popped 64-bit value is no longer in `RAX` at program termination.

## Runtime observations

The program is silent because it does not print the register contents. Immediately after the two pops, the expected values are `RBX = 0x1234` and `RAX = 0x1122334455667788`; immediately before the exit system call, `RAX` is changed to 60. The expected exit status is 0. NASM was not available in the environment used to prepare this README, so these are source-derived expected observations, not recorded execution results.

## Assemble and run on 64-bit Linux

```bash
nasm -f elf64 05.asm -o 05.o
ld -o 05 05.o
./05
echo $?
```

Expected exit status: `0`.
