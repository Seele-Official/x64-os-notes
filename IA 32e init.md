

操作系统应遵循以下顺序来初始化 IA-32e 模式：
1. 从保护模式开始，通过设置 CR0.PG = 0 来禁用分页。使用 MOV CR0 指令禁用分页。
   > *MOV CR0 必须在 Identity-mapped（同一映射）中使用。就是指逻辑地址（也就是虚拟地址）和物理地址是相同的。*
2. 通过设置 CR4.PAE = 1 来启用物理地址扩展 (PAE)。如果无法启用PAE，则在尝试初始化 IA-32e 模式时将导致 #GP 错误。
3. 使用 4 级页表 (PML4) 的物理基址加载 CR3。
4. 通过设置 IA32_EFER.LME = 1 来启用 IA-32e 模式。
5. 通过设置 CR0.PG = 1 来启用分页。这会导致处理器将 IA32_EFER.LMA 位设置为 1。
   > *启用分页的 MOV CR0 指令和以下指令必须位于同一映射页面中，直到跳转到非同一映射页面的分支。*

```GAS
longmode_init:
    movl    $0b1101101000, %eax  ; Set PAE, MCE, PGE; OSFXSR, OSXMMEXCPT (enable SSE)
    movl    %eax, %cr4
    movl    $0xA000, %eax
    movl    %eax, %cr3
    movl    $0xC0000080, %ecx    ; EFER MSR
    rdmsr
    orl     $0x100, %eax         ; enable long mode
    wrmsr

    movl    $0xC0000011, %eax    ; clear EM, MP (enable SSE) and WP
    movl    %eax, %cr0           ; enable paging with cache disabled
    lgdt    GDT64_value(%rip)    ; read 80-bit address
    ljmp    $8, $.bootboot_startcore

    .code64
.bootboot_startcore:
    xorq    %rax, %rax           ; load long mode segments
    movw    $0x10, %ax
    movw    %ax, %ds
    movw    %ax, %es
    movw    %ax, %ss
    movw    %ax, %fs
    movw    %ax, %gs

```