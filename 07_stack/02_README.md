# 02.asm 

## Execution flow

Let the caller's initial stack pointer be **S**.

1. `_start` executes `call work`. The `call` instruction pushes the address of the instruction immediately after the call onto the stack (8 bytes in 64 bit mode). `RSP` becomes **S - 8**, and execution jumps to `work`.
2. `push rbp` saves the caller's old frame pointer. `RSP` becomes **S - 16**.
3. `mov rbp, rsp` makes `RBP = S - 16`. This is the base of the new function frame.
4. `sub rsp, 16` reserves 16 bytes for two 8 byte local variables. `RSP` becomes **S - 32**.
5. `mov qword [rbp - 8], 100` stores 100 in the first local-variable slot. `mov qword [rbp - 16], 200` stores 200 in the second slot. These memory writes do not themselves move either stack pointer.
6. `mov rax, [rbp - 8]` loads 100 into `RAX`; `mov rbx, [rbp - 16]` loads 200 into `RBX`.
7. `mov rsp, rbp` releases the 16 bytes reserved for locals by restoring `RSP` to **S - 16**.
8. `pop rbp` restores the caller's saved frame pointer and moves `RSP` to **S - 8**.
9. `ret` pops the saved return address, moves `RSP` back to **S**, and resumes execution in `_start` after `call work`.
10. `_start` exits using the Linux x86-64 `exit` system call.

## Stack layout and movement

At the deepest point in `work`, the layout (from higher to lower addresses) is:

| Address relative to S | Contents |
|---|---|
| S - 8 | Return address pushed by `call` |
| S - 16 | Saved caller `RBP` |
| S - 24 (`RBP - 8`) | Local value 100 |
| S - 32 (`RBP - 16`) | Local value 200; current `RSP` |

The stack grows toward lower addresses. `sub rsp, 16` reserves space; it does not automatically initialise that space. The two `mov` instructions then write the local values into it. `mov rsp, rbp`, `pop rbp`, and `ret` unwind the frame in reverse order.


