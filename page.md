
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


### PML4E Page Map Level 4 Entry

| 位数    | 描述                                                                                                 |
| ------- | ---------------------------------------------------------------------------------------------------- |
| 0 (P)   | 存在位；必须为1以指向下一级页表                                                                      |
| 1 (R/W) | 读/写；如果为0，可能不允许写入此条目指向的页表（见第4.6节）                                          |
| 2 (U/S) | 用户/监督；如果为0，不允许用户模式访问此条目指向的页表（见第4.6节）                                  |
| 3 (PWT) | 页级写通；间接确定访问此条目指向的页表时使用的内存类型（见第4.9.2节）                                |
| 4 (PCD) | 页级缓存禁用；间接确定访问此条目指向的页表时使用的内存类型（见第4.9.2节）                            |
| 5 (A)   | 访问位；指示软件是否已访问此条目指向的页表（见第4.8节）                                              |
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
| 7 (PS)   | 页大小；必须为1，否则此条目指向页目录表（见表4-16）                              |
| 8 (G)    | 全局位；如果 CR4.PGE = 1，确定此条目指向的页是否为全局页（见第4.10节），否则忽略 |
| 11:9     | 忽略                                                                             |
| 12 (PAT) | 间接确定访问此条目映射的1 GB页面时使用的内存类型（见第4.9.2节）                  |
| 29:13    | 保留（必须为0）                                                                  |
| 51:30    | 此条目映射的1 GB页面的物理地址                                                   |
| 58:52    | 忽略                                                                             |
| 62:59    | 保护密钥；如果 CR4.PKE = 1，确定页面的[保护密钥](#protection-keys)，否则忽略          |
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
| 7 (PS)   | 页大小；必须为0，否则此条目映射一个1 GB页面（见表4-15）                                                |
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

一般来说，线性地址的访问权限有俩种，分别是`supervisor-mode access` 和 `user-mode access`.所有的指令请求和数据权限,都是由当前特权等级 `current privilege level (CPL)` 决定的。

- **CPL < 3**： supervisor-mode accesses
- **CPL = 3**： user-mode accesses.

mplicit supervisor-mode accesses

explicit supervisor-mode accesses

如果至少一个分页结构条目中的 U/S 标志（位 2）为 0，则该地址为管理员模式地址。否则，该地址为用户模式地址。

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

---

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