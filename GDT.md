GDT（Global Descriptor Table，全局描述符表）中的段描述符用于定义内存段的属性，分为32位和64位模式。每个描述符都是一个 **8字节（64位）** 的结构，描述了一个内存段的基址、长度、访问权限和其他特性。

### 32位模式下的段描述符结构

#### 段描述符的格式：
| 位段  | 名称                       |
| ----- | -------------------------- |
| 0-15  | 段界限 15:0 (Limit 15:0)   |
| 16-31 | 段基址 15:0 (Base 15:0)    |
| 32-39 | 段基址 23:16 (Base 23:16)  |
| 40-47 | 访问字节 (Access Byte)     |
| 48-51 | 段界限 19:16 (Limit 19:16) |
| 52-55 | 标志 (Flags)               |
| 56-63 | 段基址 31:24 (Base 31:24)  |

#### 字段说明：
1. **段基址 (Base Address)**：
   - 分为三部分：`Base[31:24]`, `Base[23:16]`, `Base[15:0]`。它们共同构成一个 32 位的地址，用于指向内存段的起始位置。

2. **段界限 (Segment Limit)**：
   - 分为 `Limit[19:16]` 和 `Limit[15:0]` 两部分，共20位。
   - 它表示段的大小，但实际上这个值要乘以 `G` 位的系数（1KB或4KB）来得到段的真实大小。

3. **Access Byte（访问权限字节）**：
   - **位 7 (Present, P)**：段是否存在，1表示段有效，0表示段无效。如果是0，当程序访问该段时会产生异常。
   - **位 6-5 (Descriptor Privilege Level, DPL)**：特权级，共两位，表示从 0（最高权限）到 3（最低权限）。段的访问权限由此字段决定。
   - **位 4 (Descriptor Type, S)**：描述符类型，0 表示系统段描述符（如 TSS），1 表示代码段或数据段描述符。
   - **位 3 (Executable, E)**：是否为代码段，1 表示代码段，0 表示数据段。
   - **位 2 (Direction/Conforming, DC)**：
     - 如果是数据段：表示段增长方向，0表示向上增长，1表示向下增长。
     - 如果是代码段：表示是否为可一致代码段（Conforming），1表示一致代码段，可以从较低权限级别跳转到该段。
   - **位 1 (Writable/Readable, W/R)**：
     - 如果是数据段：表示是否可写，1表示可写，0表示只读。
     - 如果是代码段：表示是否可读，1表示可读，0表示不可读（代码段总是可执行）。
   - **位 0 (Accessed, A)**：表示段是否已经被访问过。1表示访问过，0表示未被访问。CPU会自动设置这个位。

4. **Flag Byte（标志字节）**：
   - **位 7 (Granularity, G)**：粒度位，决定段界限的单位。0表示字节单位，1表示4KB单位。如果 `G=1`，段界限会乘以 4096。
   - **位 6 (Size, D/B)**：决定操作数大小。0表示16位段，1表示32位段。在代码段中，1表示允许32位操作数和地址，在数据段中，表示32位栈指针。
   - **位 5 (Long Mode, L)**：仅适用于64位模式下的代码段，1表示这是一个64位代码段。
   - **位 4 (Available for System, AVL)**：系统软件可使用的位，通常为0。

### 64位模式下的段描述符结构

在64位模式下，段选择器的作用变小，尤其对于代码段，它们不再进行基址转换和段界限检查，64位模式使用线性地址作为段基址。

#### 64位模式下的代码段描述符结构：
| 位段   | 名称                       |
| ------ | -------------------------- |
| 0-15   | 段界限 15:0 (Limit 15:0)   |
| 16-31  | 段基址 15:0 (Base 15:0)    |
| 32-39  | 段基址 23:16 (Base 23:16)  |
| 40-47  | 访问字节 (Access Byte)     |
| 48-51  | 段界限 19:16 (Limit 19:16) |
| 52-55  | 标志 (Flags)               |
| 56-63  | 段基址 31:24 (Base 31:24)  |
| 64-95  | 段基址 64:32 (Base 64:32)  |
| 95-127 | 保留                       |


#### 字段说明：
1. **段基址 (Base Address)**：在64位模式下的代码段，基址和段界限被忽略。即使有段基址字段，也被CPU忽略。实际使用的是分页机制和线性地址。
   
2. **段界限 (Segment Limit)**：同样被忽略。

3. **Access Byte（访问权限字节）**：
   - 与32位模式类似，依然有效。特别地，代码段的`L`位必须设置为1来表示这是一个64位代码段。

4. **Flag Byte（标志字节）**：
   - **L（Long Mode）**：如果这是一个64位代码段，`L` 位需要设置为1。这只在代码段中有效。
   - **G（Granularity）**：在64位模式下依然有效，用于确定段的粒度是否是4KB（通常无意义，因为段基址和界限被忽略）。
   - **D/B (Size)**：在64位代码段中，此位被忽略。在非64位段中，它依然控制32位或16位操作数。
   - **AVL**：可供软件使用。

### 总结

- **32位模式**下，GDT 的段描述符用于完整的段基址和界限检查，段的基址和界限对段内存访问起关键作用。
- **64位模式**下，段基址和段界限被忽略，只有代码段和数据段的特定标志（如 `L` 位）才会影响执行。分页机制取代了段的地址转换功能。

在64位模式下，代码段的描述符作用减少，重点是启用分页机制，而对于64位代码段，需要将 `L` 位设置为1。




