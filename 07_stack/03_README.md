# 03.asm — CALL, RET, and the Return Address

## Purpose

This program shows how `call` and `ret` use the stack to transfer control to a function and then return to the caller. The function also changes a value in `RAX`.

## Execution flow

Let the value of `RSP` before the call be **S**.

1. `_start` executes `mov rax, 10`. `RAX` now contains 10. The stack is unchanged.
2. `call calculate` pushes the address of `after_call` (the instruction immediately following the call) onto the stack. In 64-bit mode this return address occupies 8 bytes, so `RSP` changes from **S** to **S - 8**. Execution jumps to `calculate`.
3. `add rax, 5` changes `RAX` from 10 to 15. This arithmetic instruction does not change `RSP`.
4. `ret` takes the return address from the top of the stack, transfers execution to `after_call`, and increases `RSP` from **S - 8** back to **S**.
5. At `after_call`, `RAX` is still 15. `nop` has no effect on registers or the stack.
6. The program exits with the Linux x86-64 `exit` system call.

## Stack movement

| Point | Stack action | RSP |
|---|---|---:|
| Before `call calculate` | No return address stored yet | S |
| After `call calculate` | Return address pushed | S - 8 |
| During `add rax, 5` | No stack operation | S - 8 |
| After `ret` | Return address popped | S |

This example does not explicitly use `push` or `pop`, but `call` and `ret` perform stack operations internally. `call` saves where execution must resume; `ret` retrieves that address. The stack is balanced when the function returns.

## Runtime observations

The program prints nothing. The expected result after the call is `RAX = 15`, and `RSP` is restored to its pre-call value. The expected process exit status is 0. A debugger is needed to inspect `RAX` at `after_call`; the source has no output instruction. NASM was not available in the environment used to prepare this README, so these are expected observations from the code rather than recorded execution results.

## Assemble and run on 64-bit Linux

```bash
nasm -f elf64 03.asm -o 03.o
ld -o 03 03.o
./03
echo $?
```

Expected exit status: `0`.
