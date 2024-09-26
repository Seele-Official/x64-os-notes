- [通用寄存器 (General Purpose Registers)](#通用寄存器-general-purpose-registers)
- [指针寄存器 (Pointer Registers)](#指针寄存器-pointer-registers)
- [段寄存器 (Segment Registers)](#段寄存器-segment-registers)
- [标志寄存器 (RFLAGS Register)](#标志寄存器-rflags-register)
- [控制寄存器 (Control Registers)](#控制寄存器-control-registers)
  - [CR0](#cr0)
  - [CR2](#cr2)
  - [CR3](#cr3)
  - [CR4](#cr4)
  - [CR8](#cr8)
  - [CR1, CR5-7, CR9-15](#cr1-cr5-7-cr9-15)
- [特殊模块寄存器(MSR-Model Specific Registers)](#特殊模块寄存器msr-model-specific-registers)
  - [IA32\_EFER](#ia32_efer)
  - [FS.base, GS.base](#fsbase-gsbase)
  - [KernelGSBase](#kernelgsbase)
- [调试寄存器 (Debug Registers)](#调试寄存器-debug-registers)
  - [DR0 - DR3](#dr0---dr3)
  - [DR6](#dr6)
  - [DR7](#dr7)
- [测试寄存器 (Test Registers)](#测试寄存器-test-registers)
- [保护模式寄存器 (Protected Mode Registers)](#保护模式寄存器-protected-mode-registers)
  - [GDTR](#gdtr)
  - [LDTR](#ldtr)
  - [TR](#tr)
  - [IDTR](#idtr)
----
## 通用寄存器 (General Purpose Registers)

| 64-bit | 32-bit | 16-bit | 8-bit 高位 | 8-bit | 描述 |
|--------|--------|--------|------------|-------|------|
| RAX    | EAX    | AX     | AH         | AL    | 累加寄存器 (主要用于算术运算) |
| RBX    | EBX    | BX     | BH         | BL    | 基址寄存器 (通常用于存储基址) |
| RCX    | ECX    | CX     | CH         | CL    | 计数器寄存器 (常用于循环和字符串操作) |
| RDX    | EDX    | DX     | DH         | DL    | 数据寄存器 (常用于 I/O 操作) |
| RSI    | ESI    | SI     | N/A        | SIL   | 源索引寄存器 (在字符串操作中用作源地址) |
| RDI    | EDI    | DI     | N/A        | DIL   | 目的索引寄存器 (在字符串操作中用作目的地址) |
| RSP    | ESP    | SP     | N/A        | SPL   | 栈指针寄存器 (指向当前栈顶) |
| RBP    | EBP    | BP     | N/A        | BPL   | 基址指针寄存器 (常用于栈帧指针) |
| R8     | R8D    | R8W    | N/A        | R8B   | 通用寄存器 |
| R9     | R9D    | R9W    | N/A        | R9B   | 通用寄存器 |
| R10    | R10D   | R10W   | N/A        | R10B  | 通用寄存器 |
| R11    | R11D   | R11W   | N/A        | R11B  | 通用寄存器 |
| R12    | R12D   | R12W   | N/A        | R12B  | 通用寄存器 |
| R13    | R13D   | R13W   | N/A        | R13B  | 通用寄存器 |
| R14    | R14D   | R14W   | N/A        | R14B  | 通用寄存器 |
| R15    | R15D   | R15W   | N/A        | R15B  | 通用寄存器 |


*注意：在使用 REX.W 指令前缀时，不能访问 `AH`、`BH`、`CH` 和 `DH`。当操作数包含 64 位寄存器时，汇编器会自动添加此前缀。*

## 指针寄存器 (Pointer Registers)

| 64-bit | 32-bit | 16-bit | 描述 |
|--------|--------|--------|------|
| RIP    | EIP    | IP     | 指令指针 (用于存储当前执行的指令地址) |

## 段寄存器 (Segment Registers)

| 寄存器 | 描述 |
|--------|------|
| CS     | 代码段 (存储当前代码的段选择子) |
| DS     | 数据段 (存储当前数据的段选择子) |
| SS     | 堆栈段 (存储当前栈的段选择子) |
| ES     | 额外段 (常用于字符串操作) |
| FS     | 通用段 |
| GS     | 通用段 |

`CS`、`DS`、`ES` 和 `SS` 的段被当作其基址为 0，无论 GDT 中的段描述符是什么。`FS` 和 `GS` 是例外，它们有 MSR 来改变其基址。

所有段的限制检查都被禁用。

## 标志寄存器 (RFLAGS Register)

| Bit(s) | Label | 描述 (Description) |
|--------|-------|--------------------|
| 0      | CF    | Carry Flag         |
| 1      | 1     | Reserved           |
| 2      | PF    | Parity Flag        |
| 3      | 0     | Reserved           |
| 4      | AF    | Auxiliary Carry Flag |
| 5      | 0     | Reserved           |
| 6      | ZF    | Zero Flag          |
| 7      | SF    | Sign Flag          |
| 8      | TF    | Trap Flag          |
| 9      | IF    | Interrupt Enable Flag |
| 10     | DF    | Direction Flag     |
| 11     | OF    | Overflow Flag      |
| 12-13  | IOPL  | I/O Privilege Level |
| 14     | NT    | Nested Task        |
| 15     | 0     | Reserved           |
| 16     | RF    | Resume Flag        |
| 17     | VM    | Virtual-8086 Mode  |
| 18     | AC    | Alignment Check / Access Control |
| 19     | VIF   | Virtual Interrupt Flag |
| 20     | VIP   | Virtual Interrupt Pending |
| 21     | ID    | ID Flag            |
| 22-63  | 0     | Reserved           |


## 控制寄存器 (Control Registers)

### CR0

| Bit(s) | Label | 描述 (Description) |
|--------|-------|--------------------|
| 0      | PE    | **Protected Mode Enable (保护模式启用)**: 当设置时，CPU 进入保护模式，使用分页和分段等保护机制。清除时，CPU 处于实模式。|
| 1      | MP    | **Monitor Co-Processor (协处理器监控)**: 控制WAIT/FWAIT指令是否监视协处理器的状态。如果设置，只有当 `TS` 位设置时，WAIT/FWAIT 指令才会生成协处理器不可用异常。|
| 2      | EM    | **Emulation (仿真)**: 当设置时，禁用浮点运算，并且每个浮点指令都会生成设备不可用异常。这通常用于软件仿真浮点运算。|
| 3      | TS    | **Task Switched (任务切换)**: 由操作系统用于指示任务切换已经发生。该位设置后，处理器在执行浮点指令时会生成设备不可用异常，从而通知操作系统是否需要保存或恢复浮点状态。|
| 4      | ET    | **Extension Type (扩展类型)**: 标识浮点处理器的类型。如果设置，处理器为 80387 及以后版本。若清除，处理器为 80287。|
| 5      | NE    | **Numeric Error (数字错误)**: 当设置时，启用基于 I/O 的浮点错误处理机制 (与 `FNINIT` 和 `FNSTSW` 指令相关联)。当清除时，处理器生成非掩码浮点错误异常。|
| 6-15   | 0     | **Reserved (保留)**: 未使用，必须设置为 0。|
| 16     | WP    | **Write Protect (写保护)**: 当设置时，即使在特权级为 0 的情况下，分页机制仍会保护只读页面不被写入。|
| 17     | 0     | **Reserved (保留)**: 未使用，必须设置为 0。|
| 18     | AM    | **Alignment Mask (对齐掩码)**: 用于控制 4 字节对齐的检查。当设置时，在用户模式下不正确的内存对齐将生成对齐检查异常。|
| 19-28  | 0     | **Reserved (保留)**: 未使用，必须设置为 0。|
| 29     | NW    | **Not-Write Through (非写直通)**: 控制写缓存策略。当设置时，内存写操作不会使用 "write-through" 缓存策略，而是使用 "write-back" 缓存策略。|
| 30     | CD    | **Cache Disable (缓存禁用)**: 当设置时，禁用所有缓存功能。用于调试或内存映射 I/O。|
| 31     | PG    | **Paging (分页)**: 当设置时，启用分页机制。通过设置此位，CPU 从逻辑地址转换为线性地址，并且通过页表将线性地址转换为物理地址。|
| 32-63  | 0     | **Reserved (保留)**: 未使用，必须设置为 0。|

*注意：CR0 是唯一可以通过两种方式读写的控制寄存器，而其他控制寄存器只能通过 MOV 指令访问。*



### CR2

此控制寄存器包含触发页面错误的线性（虚拟）地址，可在页面错误的中断处理程序中使用。


### CR3
CR3 控制着页表的物理基地址，它在不同情况下有不同的结构：
- 当 `CR4.PCIDE = 0` 时，低 12 位用于其他控制标志，高 52 位存储页表的物理基地址。
- 当 `CR4.PCIDE = 1` 时，低 12 位中的 0-11 位存储 PCID（进程上下文标识符），高 52 位依旧存储页表的物理基地址。


| 位数    | 描述                                       |
|---------|--------------------------------------------|
| 0-2     | 保留 (必须为 0)                            |
| 3       | PWT (页级写穿)                             |
| 4       | PCD (页级缓存禁止)                         |
| 5-11    | 保留 (必须为 0)                            |
| 12-63   | 页表基地址 (PML4 的物理地址)               |

*注：CR3 必须 4KB 对齐，PCID 是一个重要的性能优化，用于避免上下文切换时刷空 TLB（翻译后备缓冲区）。*


### CR4

| Bit(s) | Label | 描述 |
|--------|-------|------|
| 0      | VME   | **虚拟 8086 模式扩展**：启用对虚拟 8086 模式的支持，用于处理虚拟 8086 环境中的中断和异常。 |
| 1      | PVI   | **保护模式虚拟中断**：在保护模式下启用虚拟中断标志，允许更细粒度的中断控制。 |
| 2      | TSD   | **时间戳仅在 ring 0 启用**：限制 `RDTSC` 指令，仅允许在 ring 0（内核模式）下执行时间戳读取，提升安全性。 |
| 3      | DE    | **调试扩展**：启用扩展调试寄存器的支持，以便于在调试过程中获取更多信息。 |
| 4      | PSE   | **页面大小扩展**：启用 4MB 大小的分页，而不仅限于 4KB 的标准页。 |
| 5      | PAE   | **物理地址扩展**：启用 36 位的物理地址扩展（最多支持 64GB 的物理内存），这对于 32 位处理器特别有用。 |
| 6      | MCE   | **机器检查异常**：启用机器检查异常，当发生硬件故障时，可以捕获异常并进行处理。 |
| 7      | PGE   | **页面全局启用**：启用全局页面功能，允许某些页表项在所有进程间共享，减少切换进程时的 TLB 刷新。 |
| 8      | PCE   | **性能监视计数器启用**：允许用户模式程序使用 `RDPMC` 指令读取性能监视计数器。 |
| 9      | OSFXSR| **操作系统支持 FXSAVE/FXRSTOR**：启用操作系统对 `FXSAVE` 和 `FXRSTOR` 指令的支持，处理浮点上下文切换。 |
| 10     | OSXMMEXCPT | **操作系统支持未屏蔽的 SIMD 浮点异常**：启用对 SIMD 浮点异常的支持，这对于一些多媒体和科学计算指令非常重要。 |
| 11     | UMIP  | **用户模式指令预防**：禁止用户模式下执行某些特权指令（如 SGDT、SIDT），提高系统安全性。 |
| 12     | 0     | 保留。 |
| 13     | VMXE  | **虚拟机扩展启用**：启用虚拟机扩展（Intel VT-x），用于虚拟化技术。 |
| 14     | SMXE  | **安全模式扩展启用**：启用更安全的执行模式，用于保护某些敏感操作。 |
| 15     | 0     | 保留。 |
| 16     | FSGSBASE | **启用 FSGSBASE 指令**：允许 `RDFSBASE`、`RDGSBASE`、`WRFSBASE` 和 `WRGSBASE` 指令访问 FS 和 GS 段寄存器的基址。 |
| 17     | PCIDE | **PCID 启用**：启用进程上下文标识符（PCID），用于优化页表切换时的 TLB 缓存，减少性能开销。 |
| 18     | OSXSAVE | **操作系统支持 XSAVE**：启用操作系统对 `XSAVE` 和扩展处理器状态保存的支持，优化状态切换。 |
| 19     | 0     | 保留。 |
| 20     | SMEP  | **超级用户模式执行保护**：防止内核模式代码执行来自用户模式的代码，提升系统安全性。 |
| 21     | SMAP  | **超级用户模式访问保护**：防止内核模式代码访问来自用户模式的内存，进一步提升安全性。 |
| 22     | PKE   | **用户模式页面的保护键启用**：启用保护键功能，用于增强用户模式页面的访问控制。 |
| 23     | CET   | **控制流强制技术启用**：启用 CET 技术，用于防御控制流劫持攻击。 |
| 24     | PKS   | **超级用户模式页面的保护键启用**：启用保护键功能，用于增强超级用户模式页面的访问控制。 |
| 25-63  | 0     | 保留。 |

`CR4` 寄存器控制着处理器的许多高级功能，特别是安全性、虚拟化和分页管理。VME 和 PVI 允许处理器更灵活地管理中断和异常，TSD 则限制了时间戳指令的使用，防止非特权代码滥用。此外，PAE 和 PSE 等位控制着分页的不同模式，而 SMEP 和 SMAP 则为内核与用户空间的隔离提供了额外的安全保障。
### CR8

CR8 是在 64 位模式下通过 REX 前缀访问的新寄存器。CR8 用于优先级外部中断，并被称为任务优先级寄存器（TPR）。

AMD64 架构允许软件定义最多 15 个外部中断优先级类。优先级类从 1 到 15 编号，其中优先级类 1 为最低，优先级类 15 为最高。CR8 使用低 4 位来指定任务优先级，其余 60 位保留，必须写入零。

系统软件可以使用 TPR 寄存器暂时阻止低优先级中断打断高优先级任务。这通过将 TPR 加载为对应要阻止的最高优先级中断的值来实现。例如，将 TPR 加载为 9（1001b）将阻止所有优先级为 9 或更低的中断，同时允许优先级为 10 或更高的中断被识别。将 TPR 加载为 0 启用所有外部中断。将 TPR 加载为 15（1111b）禁用所有外部中断。

TPR 在复位时被清零。

| Bit(s) | 目的 (Purpose) |
|--------|----------------|
| 0-3    | Priority       |
| 4-63   | Reserved       |

### CR1, CR5-7, CR9-15

保留，访问这些寄存器时 CPU 将抛出 #ud 异常。


## 特殊模块寄存器(MSR-Model Specific Registers)

### IA32_EFER

扩展功能使能寄存器（EFER）是一个在 AMD K6 处理器中引入的模型特定寄存器，用于启用 SYSCALL/SYSRET 指令，并在 AMD64 架构中成为体系结构的一部分，被 Intel 采纳。其 MSR 编号是 0xC0000080。
`IA32_EFER` 是一个重要的寄存器，用于控制处理器的高级功能。常见的用途是启用 64 位长模式，以及提高系统安全性和虚拟化支持。典型情况下，SCE 用于启用系统调用指令，而 LME 和 LMA 则控制着是否允许 64 位的执行模式。NXE 位则提供了一层额外的防御机制，防止不可执行的内存页运行代码，有助于防范某些类型的攻击。


| Bit(s) | Label | 描述 |
|--------|-------|------|
| 0      | SCE   | **系统调用扩展**：启用 `SYSCALL` 和 `SYSRET` 指令，这些指令用于从用户模式切换到内核模式并返回。 |
| 1-7    | 0     | 保留：未使用，必须写入0。 |
| 8      | LME   | **长模式启用**：启用 64 位长模式，此位需要结合 CR0 和 CR4 的设置才能完全启用 64 位模式。 |
| 10     | LMA   | **长模式活动**：此位指示当前处理器是否运行在长模式下。当 LME 被设置且分页启用时，LMA 会自动被处理器设置为1。 |
| 11     | NXE   | **不可执行启用**：启用不可执行页面特性，该功能使内存页可以标记为不可执行，从而防止代码执行，以提高安全性。 |
| 12     | SVME  | **安全虚拟机启用**：启用 AMD 的安全虚拟机扩展 (SVM)，用于增强虚拟化中的安全性。 |
| 13     | LMSLE | **长模式段限制启用**：启用长模式下的段限制检查。通常在长模式下段限制无效，但启用该位后会强制进行检查。 |
| 14     | FFXSR | **快速 FXSAVE/FXRSTOR**：启用对 `FXSAVE` 和 `FXRSTOR` 指令的优化处理，以提高处理浮点状态保存和恢复的效率。 |
| 15     | TCE   | **转换缓存扩展**：与虚拟化技术相关的缓存优化，用于提高性能。 |
| 16-63  | 0     | 保留：未使用，必须写入0。 |


### FS.base, GS.base

MSR 地址为 0xC0000100（FS）和 0xC0000101（GS）的寄存器包含 FS 和 GS 段寄存器的基地址。这些地址通常用于用户代码中的线程指针和内核代码中的 CPU 本地指针。可以安全地包含任何内容，因为使用段并不会赋予用户代码额外的特权。

在较新的 CPU 中，这些寄存器也可以使用 WRFSBASE 和 WRGSBASE 指令在任何特权级别下进行写入。

### KernelGSBase

MSR 地址为 0xC0000102。基本上是一个缓冲区，在执行 swapgs 指令后与 GS.base 进行交换。通常用于分隔内核和用户对 GS 寄存器的使用。

## 调试寄存器 (Debug Registers)

### DR0 - DR3

包含最多 4 个断点的线性地址。如果启用了分页，它们会被转换为物理地址。

### DR6

允许调试器确定发生了哪些调试条件。当启用的调试异常被触发时，低位 0-3 会在进入调试异常处理程序之前被设置。

### DR7

| Bit   | 描述 (Description) |
|-------|--------------------|
| 0     | Local DR0 Breakpoint |
| 1     | Global DR0 Breakpoint |
| 2     | Local DR1 Breakpoint |
| 3     | Global DR1 Breakpoint |
| 4     | Local DR2 Breakpoint |
| 5     | Global DR2 Breakpoint |
| 6     | Local DR3 Breakpoint |
| 7     | Global DR3 Breakpoint |
| 16-17 | Conditions for DR0 |
| 18-19 | Size of DR0 Breakpoint |
| 20-21 | Conditions for DR1 |
| 22-23 | Size of DR1 Breakpoint |
| 24-25 | Conditions for DR2 |
| 26-27 | Size of DR2 Breakpoint |
| 28-29 | Conditions for DR3 |
| 30-31 | Size of DR3 Breakpoint |

本地断点位在硬件任务切换时会失效，而全局断点则不会。
00b 表示执行中断，01b 表示写入监视点，11b 表示读/写监视点。10b 保留用于 I/O 读/写（不支持）。

## 测试寄存器 (Test Registers)

| 名称 (Name) | 描述 (Description) |
|-------------|--------------------|
| TR3 - TR5   | Undocumented       |
| TR6         | Test Command Register |
| TR7         | Test Data Register |

## 保护模式寄存器 (Protected Mode Registers)

### GDTR

GDTR 保存全局描述符表的基地址和大小。用于内存段选择。

| 位数      | 描述                                 |
|-----------|--------------------------------------|
| 0-15      | GDT 限长 (表示全局描述符表的长度)   |
| 16-79     | GDT 基址 (保存全局描述符表的 64 位地址) | |

### LDTR

存储 LDT 的段选择子。

### TR

存储 TSS 的段选择子。

### IDTR
IDTR 保存中断描述符表的基地址和大小。用于处理中断和异常。

| 位数      | 描述                                 |
|-----------|--------------------------------------|
| 0-15      | IDT 限长 (表示中断描述符表的长度)   |
| 16-79     | IDT 基址 (保存中断描述符表的 64 位地址) |
