# 01.asm 
## Execution flow

1. `_start` begins execution. Let the initial value of `RSP` be **S**.
2. `mov rax, 100` places decimal 100 in `RAX`; `mov rbx, 200` places decimal 200 in `RBX`. These instructions do not change the stack.
3. `push rax` decreases `RSP` by 8 bytes, to **S - 8**, and stores 100 at the new top of the stack.
4. `push rbx` decreases `RSP` by another 8 bytes, to **S - 16**, and stores 200 above the earlier value. The most recently pushed value is now at the top.
5. `nop` does nothing and does not change `RSP`.
6. `pop rcx` reads the top value (200) into `RCX`, then increases `RSP` by 8 bytes to **S - 8**.
7. `pop rdx` reads the next value (100) into `RDX`, then increases `RSP` by 8 bytes to **S**.
8. The program uses the Linux x86-64 `exit` system call (`RAX = 60`, `RDI = 0`) to terminate.

## Stack movement

| Instruction | Value at the top | RSP after instruction |
|---|---:|---:|
| Start | — | S |
| `push rax` | 100 | S - 8 |
| `push rbx` | 200 | S - 16 |
| `pop rcx` | 100 is next | S - 8 |
| `pop rdx` | stack restored | S |

The stack is LIFO: 200 is popped before 100, even though 100 was pushed first. After both pops, `RSP` returns to its initial value. The resulting register values are `RCX = 200` and `RDX = 100`.

#