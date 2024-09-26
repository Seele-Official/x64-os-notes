
- [ELF data representation](#elf-data-representation)
- [ELF header](#elf-header)
- [Program Header Table](#program-header-table)
- [Section Header Table](#section-header-table)
  - [Section header entries](#section-header-entries)

----
# ELF data representation

| 名称          | 大小 | 对齐方式 | 目的                     |
| ------------- | ---- | -------- | ------------------------ |
| Elf64_Addr    | 8    | 8        | Unsigned program address |
| Elf64_Off     | 8    | 8        | Unsigned file offset     |
| Elf64_Half    | 2    | 2        | Unsigned medium integer  |
| Elf64_Word    | 4    | 4        | Unsigned integer         |
| Elf64_Sword   | 4    | 4        | Signed integer           |
| Elf64_Xword   | 8    | 8        | Unsigned long integer    |
| Elf64_Sxword  | 8    | 8        | Signed long integer      |
| unsigned char | 1    | 1        | Unsigned small integer   |



# ELF header
```c
struct Elf64_Ehdr
{
    unsigned char e_ident[16]; /* ELF identification */
    Elf64_Half e_type; /* Object file type */
    Elf64_Half e_machine; /* Machine type */
    Elf64_Word e_version; /* Object file version */
    Elf64_Addr e_entry; /* Entry point address */
    Elf64_Off e_phoff; /* Program header offset */
    Elf64_Off e_shoff; /* Section header offset */
    Elf64_Word e_flags; /* Processor-specific flags */
    Elf64_Half e_ehsize; /* ELF header size */
    Elf64_Half e_phentsize; /* Size of program header entry */
    Elf64_Half e_phnum; /* Number of program header entries */
    Elf64_Half e_shentsize; /* Size of section header entry */
    Elf64_Half e_shnum; /* Number of section header entries */
    Elf64_Half e_shstrndx; /* Section name string table index */
};

```
# Program Header Table
```c
struct Elf64_Phdr
{
    Elf64_Word p_type; /* Type of segment */
    Elf64_Word p_flags; /* Segment attributes */
    Elf64_Off p_offset; /* Offset in file */
    Elf64_Addr p_vaddr; /* Virtual address in memory */
    Elf64_Addr p_paddr; /* Reserved */
    Elf64_Xword p_filesz; /* Size of segment in file */
    Elf64_Xword p_memsz; /* Size of segment in memory */
    Elf64_Xword p_align; /* Alignment of segment */
};
```
# Section Header Table

## Section header entries
```c
struct Elf64_Shdr
{
    Elf64_Word sh_name; /* Section name */
    Elf64_Word sh_type; /* Section type */
    Elf64_Xword sh_flags; /* Section attributes */
    Elf64_Addr sh_addr; /* Virtual address in memory */
    Elf64_Off sh_offset; /* Offset in file */
    Elf64_Xword sh_size; /* Size of section */
    Elf64_Word sh_link; /* Link to other section */
    Elf64_Word sh_info; /* Miscellaneous information */
    Elf64_Xword sh_addralign; /* Address alignment boundary */
    Elf64_Xword sh_entsize; /* Size of entries, if section has table */
};
```
