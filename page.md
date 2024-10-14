
- [OVERVIEW](#overview)
- [32-bit paging](#32-bit-paging)
- [PAE paging](#pae-paging)
- [IA-32e paging](#ia-32e-paging)
    - [PML4E Page Map Level 4 Entry](#pml4e-page-map-level-4-entry)
    - [PDPTE Page Directory Pointer Table Entry](#pdpte-page-directory-pointer-table-entry)
    - [PDE Page Directory Entry](#pde-page-directory-entry)
    - [PTE Page Table Entry](#pte-page-table-entry)
    - [Translation overview](#translation-overview)
      - [4KB 页大小转换](#4kb-页大小转换)
      - [2MB 页大小转换](#2mb-页大小转换)
      - [1GB 页大小转换](#1gb-页大小转换)
- [CPUID Support](#cpuid-support)
- [Access Rights](#access-rights)
  - [Supervisor-Mode Accesses](#supervisor-mode-accesses)
  - [User-Mode Accesses](#user-mode-accesses)
  - [Protection Keys](#protection-keys)
- [Page-Fault Error Code](#page-fault-error-code)
- [PAT Page Attribute Table](#pat-page-attribute-table)
----

# OVERVIEW

分页开启通过 `CR0.PG` 标志位开启，一旦开启了分页，就会有三种模式会被使用，而 `CR4.PAE` 和 `IA32_EFER.LME` 俩个标志位将决定使用哪种分页模式。

- **32-bit paging**： 
  - CR4.PAE = 0
  - IA32_EFER.LME = any
- **PAE paging**： 
  - CR4.PAE = 1
  - IA32_EFER.LME = 0
- **IA-32e paging** ：
  - CR4.PAE = 1
  - IA32_EFER.LME = 0 

三种模式的不同之处
![](./src/Difference%20of%20page%20modes.png)

# 32-bit paging



# PAE paging

32位扩展，no

# IA-32e paging

**Use of CR3 with IA-32e Paging and CR4.PCIDE = 0**

| 位数  | 描述                         |
| ----- | ---------------------------- |
| 0-2   | 保留 (必须为 0)              |
| 3     | PWT (页级写穿)               |
| 4     | PCD (页级缓存禁止)           |
| 5-11  | 保留 (必须为 0)              |
| 12-63 | 页表基地址 (PML4 的物理地址) |

- 页表基地址: 页表地址的 12-63 位，所以页表必须4kb对齐
> **所有页表有关的结构体都必须4KB对齐！！！**


### PML4E Page Map Level 4 Entry

| 位数    | 描述                                                                                                 |
| ------- | ---------------------------------------------------------------------------------------------------- |
| 0 (P)   | 存在位；必须为1以指向下一级页表                                                                      |
| 1 (R/W) | 读/写；如果为0，可能不允许写入此条目指向的页表(见[Access-accesses](#access-rights))                  |
| 2 (U/S) | 用户/监督；如果为0，不允许用户模式访问此条目指向的页表(见[Access-accesses](#access-rights))          |
| 3 (PWT) | 页级写通；或间接确定访问此条目指向的页表时使用的内存类型(见 [PAT](#pat-page-attribute-table) )       |
| 4 (PCD) | 页级缓存禁用；或间接确定访问此条目指向的页表时使用的内存类型(见 [PAT](#pat-page-attribute-table) )   |
| 5 (A)   | 访问位；指示软件是否已访问此条目指向的页表                                                           |
| 11:6    | 保留（必须为0）                                                                                      |
| 51:12   | 此条目指向的页目录表的物理地址                                                                       |
| 62:52   | 忽略                                                                                                 |
| 63 (XD) | 如果 IA32_EFER.NXE = 1，执行禁用（如果为1，不允许从此条目指向的页表中获取指令）否则，保留（必须为0） |



### PDPTE Page Directory Pointer Table Entry

**1 GB大页模式**
| 位数     | 描述                                                                             |
| -------- | -------------------------------------------------------------------------------- |
| 0 (P)    | 存在位；必须为1以映射1 GB的页                                                    |
| 1 (R/W)  | 读/写；如果为0，可能不允许写入此条目映射的1 GB页面（见第4.6节）                  |
| 2 (U/S)  | 用户/监督；如果为0，不允许用户模式访问此条目映射的1 GB页面（见第4.6节）          |
| 3 (PWT)  | 页级写通；间接确定访问此条目映射的1 GB页面时使用的内存类型（见第4.9.2节）        |
| 4 (PCD)  | 页级缓存禁用；间接确定访问此条目映射的1 GB页面时使用的内存类型（见第4.9.2节）    |
| 5 (A)    | 访问位；指示软件是否已访问此条目映射的1 GB页面（见第4.8节）                      |
| 6 (D)    | 脏位；指示软件是否已写入此条目映射的1 GB页面（见第4.8节）                        |
| 7 (PS)   | 页大小；必须为1，否则此条目指向页目录表                                          |
| 8 (G)    | 全局位；如果 CR4.PGE = 1，确定此条目指向的页是否为全局页（见第4.10节），否则忽略 |
| 11:9     | 忽略                                                                             |
| 12 (PAT) | 间接确定访问此条目映射的1 GB页面时使用的内存类型（见第4.9.2节）                  |
| 29:13    | 保留（必须为0）                                                                  |
| 51:30    | 此条目映射的1 GB页面的物理地址                                                   |
| 58:52    | 忽略                                                                             |
| 62:59    | 保护密钥；如果 CR4.PKE = 1，确定页面的[保护密钥](#protection-keys)，否则忽略     |
| 63 (XD)  | 如果 IA32_EFER.NXE = 1，执行禁用；否则，保留（必须为0）                          |

> PAT 在所有支持IA-32e paging的处理器中都被支持

**指向下一级**
| 位数     | 描述                                                                                                   |
| -------- | ------------------------------------------------------------------------------------------------------ |
| 0 (P)    | 存在位；必须为1以引用页目录                                                                            |
| 1 (R/W)  | 读/写；如果为0，可能不允许写入此条目控制的1 GB区域（见第4.6节）                                        |
| 2 (U/S)  | 用户/监督；如果为0，不允许用户模式访问此条目控制的1 GB区域（见第4.6节）                                |
| 3 (PWT)  | 页级写通；间接确定访问此条目引用的页目录时使用的内存类型（见第4.9.2节）                                |
| 4 (PCD)  | 页级缓存禁用；间接确定访问此条目引用的页目录时使用的内存类型（见第4.9.2节）                            |
| 5 (A)    | 访问位；指示此条目是否已用于线性地址翻译（见第4.8节）                                                  |
| 6        | 忽略                                                                                                   |
| 7 (PS)   | 页大小；必须为0，否则此条目映射一个1 GB页面                                                            |
| 11:8     | 忽略                                                                                                   |
| (M–1):12 | 此条目引用的4 KB对齐页目录的物理地址                                                                   |
| 51:M     | 保留（必须为0）                                                                                        |
| 62:52    | 忽略                                                                                                   |
| 63 (XD)  | 如果 IA32_EFER.NXE = 1，执行禁用；如果为1，不允许从此条目控制的1 GB区域获取指令；否则，保留（必须为0） |

---

### PDE Page Directory Entry

**2 MB大页模式**
| 位数     | 描述                                                                             |
| -------- | -------------------------------------------------------------------------------- |
| 0 (P)    | 存在位；必须为1以映射2 MB的页                                                    |
| 1 (R/W)  | 读/写；如果为0，可能不允许写入此条目映射的2 MB页面（见第4.6节）                  |
| 2 (U/S)  | 用户/监督；如果为0，不允许用户模式访问此条目映射的2 MB页面（见第4.6节）          |
| 3 (PWT)  | 页级写通；间接确定访问此条目映射的2 MB页面时使用的内存类型（见第4.9.2节）        |
| 4 (PCD)  | 页级缓存禁用；间接确定访问此条目映射的2 MB页面时使用的内存类型（见第4.9.2节）    |
| 5 (A)    | 访问位；指示软件是否已访问此条目映射的2 MB页面（见第4.8节）                      |
| 6 (D)    | 脏位；指示软件是否已写入此条目映射的2 MB页面（见第4.8节）                        |
| 7 (PS)   | 页大小；必须为1，否则此条目指向页表                                              |
| 8 (G)    | 全局位；如果 CR4.PGE = 1，确定此条目指向的页是否为全局页（见第4.10节），否则忽略 |
| 11:9     | 忽略                                                                             |
| 12 (PAT) | 间接确定访问此条目映射的2 MB页面时使用的内存类型（见第4.9.2节）                  |
| 29:13    | 保留（必须为0）                                                                  |
| 51:30    | 此条目映射的2 MB页面的物理地址                                                   |
| 58:52    | 忽略                                                                             |
| 62:59    | 保护密钥；如果 CR4.PKE = 1，确定页面的保护密钥（见第4.6.2节），否则忽略          |
| 63 (XD)  | 如果 IA32_EFER.NXE = 1，执行禁用；否则，保留（必须为0）                          |


**指向下一级**
| 位数     | 描述                                                                                                   |
| -------- | ------------------------------------------------------------------------------------------------------ |
| 0 (P)    | 存在位；必须为1以引用页表                                                                              |
| 1 (R/W)  | 读/写；如果为0，可能不允许写入此条目控制的2 MB区域（见第4.6节）                                        |
| 2 (U/S)  | 用户/监督；如果为0，不允许用户模式访问此条目控制的2 MB区域（见第4.6节）                                |
| 3 (PWT)  | 页级写通；间接确定访问此条目引用的页表时使用的内存类型（见第4.9.2节）                                  |
| 4 (PCD)  | 页级缓存禁用；间接确定访问此条目引用的页表时使用的内存类型（见第4.9.2节）                              |
| 5 (A)    | 访问位；指示此条目是否已用于线性地址翻译（见第4.8节）                                                  |
| 6        | 忽略                                                                                                   |
| 7 (PS)   | 页大小；必须为0，否则此条目映射一个2 MB页面（见表4-17）                                                |
| 11:8     | 忽略                                                                                                   |
| (M–1):12 | 此条目引用的4 KB对齐页表的物理地址                                                                     |
| 51:M     | 保留（必须为0）                                                                                        |
| 62:52    | 忽略                                                                                                   |
| 63 (XD)  | 如果 IA32_EFER.NXE = 1，执行禁用；如果为1，不允许从此条目控制的2 MB区域获取指令；否则，保留（必须为0） |

---

### PTE Page Table Entry

| 位数    | 描述                                                                             |
| ------- | -------------------------------------------------------------------------------- |
| 0 (P)   | 存在位；必须为1以映射4 KB的页                                                    |
| 1 (R/W) | 读/写；如果为0，可能不允许写入此条目映射的4 KB页面（见第4.6节）                  |
| 2 (U/S) | 用户/监督；如果为0，不允许用户模式访问此条目映射的4 KB页面（见第4.6节）          |
| 3 (PWT) | 页级写通；间接确定访问此条目映射的4 KB页面时使用的内存类型（见第4.9.2节）        |
| 4 (PCD) | 页级缓存禁用；间接确定访问此条目映射的4 KB页面时使用的内存类型（见第4.9.2节）    |
| 5 (A)   | 访问位；指示软件是否已访问此条目映射的4 KB页面（见第4.8节）                      |
| 6 (D)   | 脏位；指示软件是否已写入此条目映射的4 KB页面（见第4.8节）                        |
| 7 (PAT) | 间接确定访问此条目映射的4 KB页面时使用的内存类型（见第4.9.2节）                  |
| 8 (G)   | 全局位；如果 CR4.PGE = 1，确定此条目指向的页是否为全局页（见第4.10节），否则忽略 |
| 11:9    | 忽略                                                                             |
| 51:12   | 此条目映射的4 KB页面的物理地址                                                   |
| 58:52   | 忽略                                                                             |
| 62:59   | 保护密钥；如果 CR4.PKE = 1，确定页面的保护密钥（见第4.6.2节），否则忽略          |
| 63 (XD) | 如果 IA32_EFER.NXE = 1，执行禁用；否则，保留（必须为0）                          |

### Translation overview

#### 4KB 页大小转换

![](./src/Linear-Address%20Translation%20to%20a%204-KByte%20Page%20using%20IA-32e%20Paging.png)

#### 2MB 页大小转换

![](./src/Linear-Address%20Translation%20to%20a%202-MByte%20Page%20using%20IA-32e%20Paging.png)

#### 1GB 页大小转换

![](./src/Linear-Address%20Translation%20to%20a%201-GByte%20Page%20using%20IA-32e%20Paging.png)


#  CPUID Support

1. **PSE（Page-Size Extensions，页大小扩展）**
   - **CPUID 检查**：`CPUID.01H:EDX.PSE [bit 3] = 1`，表示处理器支持 PSE。
   - **启用方式**：可以通过设置控制寄存器 `CR4` 中的 `PSE` 位（位 4）来启用 PSE。
   - **作用**：启用 PSE 允许使用 4 MB 的大页，而不是默认的 4 KB 小页。在 32 位分页模式下，这对性能有帮助，因为减少了页表的复杂性和访问频率。

2. **PAE（Physical-Address Extension，物理地址扩展）**
   - **CPUID 检查**：`CPUID.01H:EDX.PAE [bit 6] = 1`，表示处理器支持 PAE。
   - **启用方式**：可以通过设置 `CR4.PAE` 位（位 5）来启用 PAE。
   - **作用**：启用 PAE 后，处理器支持 36 位物理地址空间，允许操作系统访问超过 4 GB 的物理内存（通过分页机制）。IA-32e 模式（64 位模式）也需要开启 PAE。

3. **PGE（Global-Page Support，全局页支持）**
   - **CPUID 检查**：`CPUID.01H:EDX.PGE [bit 13] = 1`，表示处理器支持 PGE。
   - **启用方式**：通过设置 `CR4.PGE` 位（位 7）来启用 PGE。
   - **作用**：启用 PGE 后，某些页可以被标记为全局页，不会因上下文切换（进程切换）而被刷新。这有助于减少频繁切换时的 TLB（Translation Lookaside Buffer，转换后援缓冲区）刷新，提高性能。

4. **PAT（Page-Attribute Table，页属性表）**
   - **CPUID 检查**：`CPUID.01H:EDX.PAT [bit 16] = 1`，表示处理器支持 PAT。
   - **作用**：PAT 是一个 8 项表格，控制不同页的缓存行为（例如是否使用写回缓存、写通缓存等）。当 PAT 支持时，分页结构中的三个位可以选择内存类型，使用 PAT 来决定缓存策略。

5. **PSE-36（40-bit Physical Address Extension with 4 MB Pages，40 位物理地址扩展）**
   - **CPUID 检查**：`CPUID.01H:EDX.PSE-36 [bit 17] = 1`，表示处理器支持 PSE-36。
   - **作用**：PSE-36 允许在 32 位分页模式下，使用 4 MB 页支持最多 40 位的物理地址，这意味着可以支持比 PAE 更大的物理内存地址。

6. **PCID（Process-Context Identifiers，进程上下文标识符）**
   - **CPUID 检查**：`CPUID.01H:ECX.PCID [bit 17] = 1`，表示处理器支持 PCID。
   - **启用方式**：可以通过设置 `CR4.PCIDE` 位（位 17）来启用 PCID。
   - **作用**：PCID 是一种优化，它允许在 TLB 中区分不同进程的上下文，减少上下文切换时的 TLB 刷新，从而提高效率。

7. **SMEP（Supervisor-Mode Execution Prevention，管理模式执行防护）**
   - **CPUID 检查**：`CPUID.(EAX=07H,ECX=0H):EBX.SMEP [bit 7] = 1`，表示处理器支持 SMEP。
   - **启用方式**：通过设置 `CR4.SMEP` 位（位 20）来启用 SMEP。
   - **作用**：SMEP 防止在管理模式（例如操作系统内核模式）下执行用户模式地址空间中的代码，增强系统的安全性。

8. **SMAP（Supervisor-Mode Access Prevention，管理模式访问防护）**
   - **CPUID 检查**：`CPUID.(EAX=07H,ECX=0H):EBX.SMAP [bit 20] = 1`，表示处理器支持 SMAP。
   - **启用方式**：通过设置 `CR4.SMAP` 位（位 21）来启用 SMAP。
   - **作用**：SMAP 防止管理模式下访问用户模式内存，从而增强系统的安全性。

9. **PKU（Protection Keys，保护密钥）**
   - **CPUID 检查**：`CPUID.(EAX=07H,ECX=0H):ECX.PKU [bit 3] = 1`，表示处理器支持保护密钥。
   - **启用方式**：通过设置 `CR4.PKE` 位（位 22）来启用 PKU。
   - **作用**：PKU 是内存保护机制，可以为不同的内存区域设置密钥，提供更灵活的访问控制。

10. **NX（No-Execute，禁止执行位）**
    - **CPUID 检查**：`CPUID.80000001H:EDX.NX [bit 20] = 1`，表示处理器支持 NX 位。
    - **启用方式**：可以通过设置 `IA32_EFER.NXE` 位（位 11）来启用 NX。
    - **作用**：NX 位用于分页机制，允许某些页面被标记为不可执行，从而防止执行特定内存区域中的代码（如栈溢出攻击）。

11. **1 GB 页支持**
    - **CPUID 检查**：`CPUID.80000001H:EDX.Page1GB [bit 26] = 1`，表示处理器支持 1 GB 页。
    - **作用**：启用后，系统可以使用 1 GB 大页进行分页，从而减少页表占用，提高性能。

12. **LM（Long Mode，长模式）**
    - **CPUID 检查**：`CPUID.80000001H:EDX.LM [bit 29] = 1`，表示处理器支持 64 位模式（长模式）。
    - **启用方式**：可以通过设置 `IA32_EFER.LME` 位（位 8）来启用长模式。
    - **作用**：长模式支持 64 位地址空间和 IA-32e 分页（即 64 位分页）。

13. **MAXPHYADDR 和线性地址宽度**
    - **CPUID 检查**：`CPUID.80000008H:EAX[7:0]` 报告处理器支持的物理地址宽度，最大可以为 52 位。
    - **线性地址宽度**：`CPUID.80000008H:EAX[15:8]` 报告处理器支持的线性地址宽度。一般情况下，如果处理器支持长模式（`CPUID.80000001H:EDX.LM`），线性地址宽度为 48 位，否则为 32 位。

# Access Rights

一般来说，线性地址的访问权限有俩种，分别是`Supervisor-mode access` 和 `User-mode access`.所有的指令请求和数据权限,都是由当前特权等级 `Current privilege level (CPL)` 决定的。

- **CPL < 3**： Supervisor-mode accesses
- **CPL = 3**： User-mode accesses.


有些操作会隐式地使用线性地址访问系统数据结构；无论当前权限级别CPL如何，对这些数据结构的访问结果会被当做监督模式访问。这类访问的例子包括：
- 访问全局描述符表（GDT）或局部描述符表（LDT）以加载段描述符 
- 在传递中断或异常时访问中断描述符表（IDT）
- 以及在任务切换或 CPL 变更时访问任务状态段（TSS）

所有这些访问都被称为**隐式监督模式(mplicit supervisor-mode accesses)访问**，无论 CPL 为何。
而在 CPL < 3 时进行的其他访问被称为**显式监督模式(explicit supervisor-mode accesses)访问**。




如果至少一个分页结构条目中的 U/S 标志（位 2）为 0，则该地址为 Supervisor-mode access 地址。否则，该地址为 User-mode access 地址。

下面给出分页模式如何确认访问权限的方式:
## Supervisor-Mode Accesses

1. **Data Reads from Supervisor-Mode Addresses**
   - Data may be read (implicitly or explicitly) from any supervisor-mode address.

2. **Data Reads from User-Mode Addresses**
   - Access rights depend on the value of `CR4.SMAP`:
     - If `CR4.SMAP = 0`, data may be read from any user-mode address with a protection key for which read access is permitted.
     - If `CR4.SMAP = 1`, access rights depend on the value of `EFLAGS.AC` and whether the access is implicit or explicit:
       - If `EFLAGS.AC = 1` and the access is explicit, data may be read from any user-mode address with a protection key for which read access is permitted.
       - If `EFLAGS.AC = 0` or the access is implicit, data may not be read from any user-mode address.
     - See Section 4.6.2 for how protection keys are associated with user-mode addresses and permitted accesses for each protection key.

3. **Data Writes to Supervisor-Mode Addresses**
   - Access rights depend on the value of `CR0.WP`:
     - If `CR0.WP = 0`, data may be written to any supervisor-mode address.
     - If `CR0.WP = 1`, data may be written to any supervisor-mode address with a translation where the `R/W` flag (bit 1) is 1 in every paging-structure entry controlling the translation; data may not be written to any supervisor-mode address where the `R/W` flag is 0 in any paging-structure entry controlling the translation.

4. **Data Writes to User-Mode Addresses**
   - Access rights depend on the values of `CR0.WP` and `CR4.SMAP`:
     - If `CR0.WP = 0`:
       - If `CR4.SMAP = 0`, data may be written to any user-mode address with a protection key for which write access is permitted.
       - If `CR4.SMAP = 1`, access rights depend on the value of `EFLAGS.AC` and whether the access is implicit or explicit:
         - If `EFLAGS.AC = 1` and the access is explicit, data may be written to any user-mode address with a protection key for which write access is permitted.
         - If `EFLAGS.AC = 0` or the access is implicit, data may not be written to any user-mode address.
     - If `CR0.WP = 1`:
       - If `CR4.SMAP = 0`, data may be written to any user-mode address with a translation where the `R/W` flag is 1 in every paging-structure entry controlling the translation and with a protection key for which write access is permitted; data may not be written to any user-mode address with a translation where the `R/W` flag is 0 in any paging-structure entry controlling the translation.
       - If `CR4.SMAP = 1`, access rights depend on the value of `EFLAGS.AC` and whether the access is implicit or explicit:
         - If `EFLAGS.AC = 1` and the access is explicit, data may be written to any user-mode address with a translation where the `R/W` flag is 1 in every paging-structure entry controlling the translation and with a protection key for which write access is permitted; data may not be written to any user-mode address where the `R/W` flag is 0 in any paging-structure entry controlling the translation.
         - If `EFLAGS.AC = 0` or the access is implicit, data may not be written to any user-mode address.

5. **Instruction Fetches from Supervisor-Mode Addresses**
   - For 32-bit paging or if `IA32_EFER.NXE = 0`, instructions may be fetched from any supervisor-mode address.
   - For PAE paging or IA-32e paging with `IA32_EFER.NXE = 1`, instructions may be fetched from any supervisor-mode address with a translation where the `XD` flag (bit 63) is 0 in every paging-structure entry controlling the translation; instructions may not be fetched from any supervisor-mode address with a translation where the `XD` flag is 1 in any paging-structure entry controlling the translation.

6. **Instruction Fetches from User-Mode Addresses**
   - Access rights depend on the value of `CR4.SMEP`:
     - If `CR4.SMEP = 0`, access rights depend on the paging mode and the value of `IA32_EFER.NXE`:
       - For 32-bit paging or if `IA32_EFER.NXE = 0`, instructions may be fetched from any user-mode address.
       - For PAE paging or IA-32e paging with `IA32_EFER.NXE = 1`, instructions may be fetched from any user-mode address with a translation where the `XD` flag is 0 in every paging-structure entry controlling the translation.
     - If `CR4.SMEP = 1`, instructions may not be fetched from any user-mode address.

## User-Mode Accesses

1. **Data Reads**
   - Access rights depend on the mode of the linear address:
     - Data may be read from any user-mode address with a protection key for which read access is permitted.
     - Data may not be read from any supervisor-mode address.
     - See Section 4.6.2 for how protection keys are associated with user-mode addresses and permitted accesses for each protection key.

2. **Data Writes**
   - Access rights depend on the mode of the linear address:
     - Data may be written to any user-mode address with a translation where the `R/W` flag is 1 in every paging-structure entry controlling the translation and with a protection key for which write access is permitted.
     - Data may not be written to any supervisor-mode address.

3. **Instruction Fetches**
   - Access rights depend on the mode of the linear address, the paging mode, and the value of `IA32_EFER.NXE`:
     - For 32-bit paging or if `IA32_EFER.NXE = 0`, instructions may be fetched from any user-mode address.
     - For PAE paging or IA-32e paging with `IA32_EFER.NXE = 1`, instructions may be fetched from any user-mode address with a translation where the `XD` flag is 0 in every paging-structure entry controlling the translation.
     - Instructions may not be fetched from any supervisor-mode address.



## Protection Keys

当 **CR4.PKE = 1**，所有的线性地址都与页表结构门中的保护密钥位联系在一起，`PKRU`寄存器决定了每个密钥对应的用户模式地址是否是可读或者可写。

当 **CR4.PKE = 0，或者 IA-32e 分页未激活**，则处理器不会将线性地址与保护密钥关联，也不会使用本节中描述的访问控制机制。在这两种情况下，中对具有保护密钥的用户模式地址的引用应被视为对任何用户模式地址的引用。

PKRU（用户页面保护密钥权限）寄存器是一个32位寄存器，其格式如下：对于每个 i（0 ≤ i ≤ 15），PKRU[2i] 是保护密钥 i 的访问禁用位(ADi, access-disable bit)，PKRU[2i+1] 是保护密钥 i 的写入禁用位(WDi, write-disable bit)。

软件可以使用 `RDPKRU` 和 `WRPKRU` 指令，并将 ECX 设置为 0 来读取和写入 PKRU。此外，PKRU 寄存器是 XSAVE-managed 的状态，因此也可以通过 XSAVE 指令集中的指令进行读取和写入。
> 关于 XSAVE 指令集的更多信息，请参见《Intel® 64 和 IA-32 架构软件开发者手册》第一卷第 13 章“使用 XSAVE 特性集管理状态”。

线性地址的保护密钥如何控制对该地址的访问取决于线性地址的模式：
- 线性地址的保护仅控制对该地址的数据访问，不会影响从该地址获取指令。
- 监督模式(supervisor-mode)下地址的保护密钥会被忽略，不会控制对该地址的数据访问。
- 对于用户模式地址的保护密钥 i，其使用依赖于 PKRU 寄存器的值：
  - 如果 ADi = 1，则不允许任何数据访问。
  - 如果 WDi = 1，某些数据写入访问可能会被拒绝：
    - 不允许用户模式写入访问。
    - 如果 CR0.WP = 1，则不允许监督模式写入访问。如果 CR0.WP = 0，WDi 不会影响监督模式对具有保护密钥 i 的用户模式地址的写入访问。
  


# Page-Fault Error Code

![](./src/Page-Fault%20Error%20Code.png)

- **P 标志 (bit 0)**：  
  如果线性地址没有对应的翻译（即分页结构条目中的 P 标志为 0），则该标志为 0。表示页不存在。
  
- **W/R 标志 (bit 1)**：  
  如果引发页错误的访问是写操作，则该标志为 1；否则为 0。这个标志描述引发页错误的访问类型，而不是分页结构中指定的访问权限。
  
- **U/S 标志 (bit 2)**：  
  如果用户模式访问引发了页错误，该标志为 1；如果是监督模式访问引发的错误，则为 0。这个标志描述引发页错误的访问类型，而不是分页结构中指定的访问权限。用户模式和监督模式的访问定义见第 4.6 节。
  
- **RSVD 标志 (bit 3)**：  
  如果线性地址没有对应的翻译，且这是由于分页结构条目中的保留位被设置为 1，则该标志为 1。（因为在 P 标志为 0 的分页结构条目中不会检查保留位，所以只有当 P 标志为 1 时，RSVD 标志才可能被设置为 1。）
  
- **I/D 标志 (bit 4)**：  
  如果引发页错误的访问是指令获取，并且满足以下条件之一：
  1. CR4.SMEP = 1；
  2. 或者 CR4.PAE = 1 且 IA32_EFER.NXE = 1，则该标志为 1，否则为 0。该标志描述引发页错误的访问类型，而不是分页结构中指定的访问权限。
  
- **PK 标志 (bit 5)**：  
  如果满足以下条件，PK 标志为 1：
  1. IA32_EFER.LMA = CR4.PKE = 1；
  2. 引发页错误的是数据访问；
  3. 线性地址是用户模式地址，并且该地址使用了某个保护密钥 i；
  4. PKRU 寄存器中的 ADi = 1，或者满足以下所有条件：WDi = 1，访问是写操作，并且 CR0.WP = 1 或者该操作是用户模式访问。

- **SGX 标志 (bit 15)**：  
  如果异常与分页无关，并且是由于违反 SGX 特定的访问控制要求引发的，该标志为 1。因为这种违反只能在不存在普通页错误时发生，所以该标志只在 P 标志（bit 0）为 1、RSVD 标志（bit 3）和 PK 标志（bit 5）都为 0 时被设置为 1。

# PAT Page Attribute Table
如果支持 PAT，分页会与 PAT 和内存类型范围寄存器(Memory-type range registers, MTRRs) 共同决定内存类型，具体如 11.5.2.2 节的表 11-7 所述。

PAT 是一个 64 位的 MSR（IA32_PAT，MSR 索引为 277H），包含 8 个 8 位条目。

对于对物理地址的任何访问，表格将 MTRRs 指定的物理地址内存类型与从 PAT 中选择的内存类型结合起来。具体来说，它来自 PAT 的条目 i，其中 i 定义如下：

- 对于 CR3 中地址的分页结构条目的访问（例如，使用 IA-32e 分页的 PML4 表）：
  - 如果 IA-32e 分页中 CR4.PCIDE = 1，则 i = 0。
  - 否则，i = 2\*PCD + PWT，PCD 和 PWT 的值来自 CR3。


- 对于地址位于另一个 `分页结构条目Y` 的 `分页结构条目X` 的访问，i = 2\*PCD + PWT，PCD 和 PWT 的值来自 Y。

- 对于线性地址转换后的物理地址的访问，i = 4\*PAT + 2\*PCD + PWT，PAT、PCD 和 PWT 的值分别来自相关的 PTE（如果转换使用 4-KB 页面）、相关的 PDE（如果转换使用 2-MB 或 4-MB 页面）或相关的 PDPTE（如果转换使用 1-GB 页面）。

