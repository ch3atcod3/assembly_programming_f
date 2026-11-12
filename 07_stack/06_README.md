# 06.asm

## Execution flow

Let the initial value of `ESP` be **S**.

1. `mov eax, 0x11223344` loads the first 32-bit value into `EAX`. `mov ebx, 0x1234` loads the second value into `EBX`. The stack is unchanged.
2. `push eax` decreases `ESP` by 4 bytes to **S - 4** and stores `0x11223344`.
3. `push ebx` decreases `ESP` by another 4 bytes to **S - 8** and stores `0x1234` at the top of the stack.
4. `nop` does nothing to the stack.
5. `pop ebx` retrieves `0x1234` into `EBX` and increases `ESP` to **S - 4**.
6. `pop eax` retrieves `0x11223344` into `EAX` and increases `ESP` to **S**.
7. The program sets `EAX = 1` to select the Linux i386 `exit` system call and clears `EBX` to set exit status 0. `int 0x80` invokes the system call.

## Stack movement 
| Instruction | Value retrieved/stored | ESP after instruction |
|---|---|---:|
| Start | — | S |
| `push eax` | Stores `0x11223344` | S - 4 |
| `push ebx` | Stores `0x1234` | S - 8 |
| `pop ebx` | Retrieves `0x1234` | S - 4 |
| `pop eax` | Retrieves `0x11223344` | S |

The program follows last-in, first-out order: the second value pushed (`0x1234`) is the first one popped. After both pops, `ESP` is restored to its original value. The program then overwrites `EAX` and `EBX` with the exit syscall number and status, so the popped values are not preserved until termination.

