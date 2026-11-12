# 03.asm 

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

