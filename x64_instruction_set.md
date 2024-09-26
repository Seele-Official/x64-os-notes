# x64 Instruction Set (AT&T Syntax)

## Table of Contents
- [x64 Instruction Set (AT\&T Syntax)](#x64-instruction-set-att-syntax)
  - [Table of Contents](#table-of-contents)
  - [Data Movement Instructions](#data-movement-instructions)
  - [Arithmetic Instructions](#arithmetic-instructions)
  - [Bitwise Instructions](#bitwise-instructions)
  - [Control Transfer Instructions](#control-transfer-instructions)
  - [Stack Operations](#stack-operations)
  - [System Instructions](#system-instructions)
  - [Input/Output Instructions](#inputoutput-instructions)
  - [Control Registers (CR)](#control-registers-cr)
    - [Additional Notes:](#additional-notes)

---

## Data Movement Instructions

| Instruction | Description                              |
|-------------|------------------------------------------|
| `mov`       | Move data from one location to another.  |
| `lea`       | Load effective address.                 |
| `push`      | Push operand onto the stack.            |
| `pop`       | Pop operand from the stack.             |
| `movzx`     | Move with zero extension.               |
| `movsx`     | Move with sign extension.               |
| `xchg`      | Exchange the values of two operands.    |
| `cmove`     | Move if equal (EFLAGS-based).           |
| `cmovz`     | Conditional move if zero (synonym).     |

---

## Arithmetic Instructions

| Instruction | Description                              |
|-------------|------------------------------------------|
| `add`       | Add two operands.                       |
| `sub`       | Subtract second operand from the first. |
| `mul`       | Unsigned multiply.                      |
| `imul`      | Signed multiply.                        |
| `div`       | Unsigned divide.                        |
| `idiv`      | Signed divide.                          |
| `inc`       | Increment operand by one.               |
| `dec`       | Decrement operand by one.               |
| `neg`       | Negate operand.                         |
| `adc`       | Add with carry.                         |
| `sbb`       | Subtract with borrow.                   |

---

## Bitwise Instructions

| Instruction | Description                              |
|-------------|------------------------------------------|
| `and`       | Bitwise AND.                            |
| `or`        | Bitwise OR.                             |
| `xor`       | Bitwise XOR.                            |
| `not`       | Bitwise NOT (ones complement).          |
| `shl`       | Shift bits left (logical).              |
| `shr`       | Shift bits right (logical).             |
| `sar`       | Shift bits right (arithmetic).          |
| `rol`       | Rotate bits left.                       |
| `ror`       | Rotate bits right.                      |

---

## Control Transfer Instructions

| Instruction | Description                              |
|-------------|------------------------------------------|
| `jmp`       | Jump to specified location.             |
| `je`        | Jump if equal (ZF=1).                   |
| `jne`       | Jump if not equal (ZF=0).               |
| `jg`        | Jump if greater (SF=OF, ZF=0).          |
| `jl`        | Jump if less (SF!=OF).                  |
| `ja`        | Jump if above (CF=0, ZF=0).             |
| `call`      | Call procedure.                         |
| `ret`       | Return from procedure.                  |
| `loop`      | Loop with counter in `ecx`.             |
| `iret`      | Return from interrupt.                  |

---

## Stack Operations

| Instruction | Description                              |
|-------------|------------------------------------------|
| `push`      | Push data onto the stack.               |
| `pop`       | Pop data from the stack.                |
| `pusha`     | Push all general-purpose registers.     |
| `popa`      | Pop all general-purpose registers.      |
| `pushf`     | Push flags register.                    |
| `popf`      | Pop flags register.                     |
| `pushal`    | Push all registers (32-bit).            |
| `popal`     | Pop all registers (32-bit).             |

---

## System Instructions

| Instruction | Description                              |
|-------------|------------------------------------------|
| `lgdt`      | Load Global Descriptor Table register.  |
| `lidt`      | Load Interrupt Descriptor Table register.|
| `ltr`       | Load Task Register.                     |
| `lmsw`      | Load machine status word.               |
| `invlpg`    | Invalidate page in TLB.                 |
| `hlt`       | Halt CPU until interrupt occurs.        |
| `cli`       | Clear interrupt flag.                   |
| `sti`       | Set interrupt flag.                     |
| `rdmsr`     | Read from model-specific register.      |
| `wrmsr`     | Write to model-specific register.       |
| `cpuid`     | Get CPU information.                    |

---

## Input/Output Instructions

| Instruction | Description                              |
|-------------|------------------------------------------|
| `in`        | Read from port.                         |
| `out`       | Write to port.                          |
| `insb`      | Input byte from port to memory.         |
| `outsb`     | Output byte from memory to port.        |
| `insw`      | Input word from port to memory.         |
| `outsw`     | Output word from memory to port.        |

---

## Control Registers (CR)

| Instruction | Description                              |
|-------------|------------------------------------------|
| `mov`       | Move between control registers (`CR0`, `CR2`, `CR3`, `CR4`). |
| `lcr0`      | Load Control Register 0.                |
| `scr4`      | Store Control Register 4.               |



---

### Additional Notes:
- **Registers**: Use `%rax`, `%rbx`, etc., for 64-bit registers in AT&T syntax.
- **Operand order**: In AT&T syntax, the source operand comes before the destination.
- **Segments**: Some segment instructions like `mov %ds, %ax` are rarely used in 64-bit mode due to flat memory models.

- AT&T 语法和 Intel 语法的主要区别是源操作数和目标操作数的顺序不同。在 AT&T 语法中，源操作数在前，目标操作数在后。
- 所有寄存器名称以 `%` 开头，如 `%rax`、`%rsp`。
- 常数以 `$` 开头，如 `$1`。
- 内存地址用括号表示，如 `(%rax)` 表示 `%rax` 指向的内存地址。
- 指令后缀用于标识操作数的大小，如 `b`（字节）、`w`（字）、`l`（双字）、`q`（四字）。
  - 例子：`movl` 表示操作双字，`movq` 表示操作四字。