# AArch64.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/BinaryFormat/ELFRelocs/AArch64.def`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This definition file lists reusable Based on released ABI: https://github.com/ARM-software/abi-aa, aaelf64. ELF64 Null relocation: also 0x100 for ELF64 entries for X-macro style expansion in LLVM's object-file and debug binary format descriptions layer. / 该定义文件在 LLVM 的目标文件与调试二进制格式描述层中列出可复用条目，供 X-macro 风格展开生成 AArch64 相关逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

```cpp
#ifndef ELF_RELOC
#error "ELF_RELOC must be defined"
#endif

// Based on released ABI: https://github.com/ARM-software/abi-aa, aaelf64.
// ELF64
// Null relocation: also 0x100 for ELF64
ELF_RELOC(R_AARCH64_NONE,                                0)
// Data relocations
ELF_RELOC(R_AARCH64_ABS64,                           0x101)
ELF_RELOC(R_AARCH64_ABS32,                           0x102)
ELF_RELOC(R_AARCH64_ABS16,                           0x103)
ELF_RELOC(R_AARCH64_PREL64,                          0x104)
ELF_RELOC(R_AARCH64_PREL32,                          0x105)
ELF_RELOC(R_AARCH64_PREL16,                          0x106)
// Static AArch64 relocations
ELF_RELOC(R_AARCH64_MOVW_UABS_G0,                    0x107)
ELF_RELOC(R_AARCH64_MOVW_UABS_G0_NC,                 0x108)
ELF_RELOC(R_AARCH64_MOVW_UABS_G1,                    0x109)
ELF_RELOC(R_AARCH64_MOVW_UABS_G1_NC,                 0x10a)
ELF_RELOC(R_AARCH64_MOVW_UABS_G2,                    0x10b)
ELF_RELOC(R_AARCH64_MOVW_UABS_G2_NC,                 0x10c)
ELF_RELOC(R_AARCH64_MOVW_UABS_G3,                    0x10d)
ELF_RELOC(R_AARCH64_MOVW_SABS_G0,                    0x10e)
```

- **L1**: Starts a preprocessor guard or conditional branch keyed by `ELF_RELOC`. / 开始一个由 `ELF_RELOC` 控制的预处理保护或条件分支。
- **L2**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L3**: Closes the current preprocessor conditional or header guard. / 结束当前的预处理条件块或头文件保护。
- **L4**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L5**: Comment documents the nearby API, invariant, or algorithmic intent: `Based on released ABI: https://github.com/ARM-software/abi-aa, aaelf64.`. / 这行注释说明了附近 API、不变量或算法意图：`Based on released ABI: https://github.com/ARM-software/abi-aa, aaelf64.`。
- **L6**: Comment documents the nearby API, invariant, or algorithmic intent: `ELF64`. / 这行注释说明了附近 API、不变量或算法意图：`ELF64`。
- **L7**: Comment documents the nearby API, invariant, or algorithmic intent: `Null relocation: also 0x100 for ELF64`. / 这行注释说明了附近 API、不变量或算法意图：`Null relocation: also 0x100 for ELF64`。
- **L8**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L9**: Comment documents the nearby API, invariant, or algorithmic intent: `Data relocations`. / 这行注释说明了附近 API、不变量或算法意图：`Data relocations`。
- **L10**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L11**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L12**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L13**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L14**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L15**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L16**: Comment documents the nearby API, invariant, or algorithmic intent: `Static AArch64 relocations`. / 这行注释说明了附近 API、不变量或算法意图：`Static AArch64 relocations`。
- **L17**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L18**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L19**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L20**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L21**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L22**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L23**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L24**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 25-48

```cpp
ELF_RELOC(R_AARCH64_MOVW_SABS_G1,                    0x10f)
ELF_RELOC(R_AARCH64_MOVW_SABS_G2,                    0x110)
ELF_RELOC(R_AARCH64_LD_PREL_LO19,                    0x111)
ELF_RELOC(R_AARCH64_ADR_PREL_LO21,                   0x112)
ELF_RELOC(R_AARCH64_ADR_PREL_PG_HI21,                0x113)
ELF_RELOC(R_AARCH64_ADR_PREL_PG_HI21_NC,             0x114)
ELF_RELOC(R_AARCH64_ADD_ABS_LO12_NC,                 0x115)
ELF_RELOC(R_AARCH64_LDST8_ABS_LO12_NC,               0x116)
ELF_RELOC(R_AARCH64_TSTBR14,                         0x117)
ELF_RELOC(R_AARCH64_CONDBR19,                        0x118)
ELF_RELOC(R_AARCH64_JUMP26,                          0x11a)
ELF_RELOC(R_AARCH64_CALL26,                          0x11b)
ELF_RELOC(R_AARCH64_LDST16_ABS_LO12_NC,              0x11c)
ELF_RELOC(R_AARCH64_LDST32_ABS_LO12_NC,              0x11d)
ELF_RELOC(R_AARCH64_LDST64_ABS_LO12_NC,              0x11e)
ELF_RELOC(R_AARCH64_MOVW_PREL_G0,                    0x11f)
ELF_RELOC(R_AARCH64_MOVW_PREL_G0_NC,                 0x120)
ELF_RELOC(R_AARCH64_MOVW_PREL_G1,                    0x121)
ELF_RELOC(R_AARCH64_MOVW_PREL_G1_NC,                 0x122)
ELF_RELOC(R_AARCH64_MOVW_PREL_G2,                    0x123)
ELF_RELOC(R_AARCH64_MOVW_PREL_G2_NC,                 0x124)
ELF_RELOC(R_AARCH64_MOVW_PREL_G3,                    0x125)
ELF_RELOC(R_AARCH64_LDST128_ABS_LO12_NC,             0x12b)
ELF_RELOC(R_AARCH64_MOVW_GOTOFF_G0,                  0x12c)
```

- **L25**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L26**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L27**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L28**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L29**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L30**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L31**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L32**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L33**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L34**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L35**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L36**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L37**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L38**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L39**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L40**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L41**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L42**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L43**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L44**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L45**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L46**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L47**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L48**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 49-72

```cpp
ELF_RELOC(R_AARCH64_MOVW_GOTOFF_G0_NC,               0x12d)
ELF_RELOC(R_AARCH64_MOVW_GOTOFF_G1,                  0x12e)
ELF_RELOC(R_AARCH64_MOVW_GOTOFF_G1_NC,               0x12f)
ELF_RELOC(R_AARCH64_MOVW_GOTOFF_G2,                  0x130)
ELF_RELOC(R_AARCH64_MOVW_GOTOFF_G2_NC,               0x131)
ELF_RELOC(R_AARCH64_MOVW_GOTOFF_G3,                  0x132)
ELF_RELOC(R_AARCH64_GOTREL64,                        0x133)
ELF_RELOC(R_AARCH64_GOTREL32,                        0x134)
ELF_RELOC(R_AARCH64_GOT_LD_PREL19,                   0x135)
ELF_RELOC(R_AARCH64_LD64_GOTOFF_LO15,                0x136)
ELF_RELOC(R_AARCH64_ADR_GOT_PAGE,                    0x137)
ELF_RELOC(R_AARCH64_LD64_GOT_LO12_NC,                0x138)
ELF_RELOC(R_AARCH64_LD64_GOTPAGE_LO15,               0x139)
ELF_RELOC(R_AARCH64_PLT32,                           0x13a)
ELF_RELOC(R_AARCH64_GOTPCREL32,                      0x13b)
ELF_RELOC(R_AARCH64_PATCHINST,                       0x13c)
ELF_RELOC(R_AARCH64_FUNCINIT64,                      0x13d)
// General dynamic TLS relocations
ELF_RELOC(R_AARCH64_TLSGD_ADR_PREL21,                0x200)
ELF_RELOC(R_AARCH64_TLSGD_ADR_PAGE21,                0x201)
ELF_RELOC(R_AARCH64_TLSGD_ADD_LO12_NC,               0x202)
ELF_RELOC(R_AARCH64_TLSGD_MOVW_G1,                   0x203)
ELF_RELOC(R_AARCH64_TLSGD_MOVW_G0_NC,                0x204)
// Local dynamic TLS relocations
```

- **L49**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L50**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L51**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L52**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L53**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L54**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L55**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L56**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L57**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L58**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L59**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L60**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L61**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L62**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L63**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L64**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L65**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L66**: Comment documents the nearby API, invariant, or algorithmic intent: `General dynamic TLS relocations`. / 这行注释说明了附近 API、不变量或算法意图：`General dynamic TLS relocations`。
- **L67**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L68**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L69**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L70**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L71**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L72**: Comment documents the nearby API, invariant, or algorithmic intent: `Local dynamic TLS relocations`. / 这行注释说明了附近 API、不变量或算法意图：`Local dynamic TLS relocations`。

### Lines 73-96

```cpp
ELF_RELOC(R_AARCH64_TLSLD_ADR_PREL21,                0x205)
ELF_RELOC(R_AARCH64_TLSLD_ADR_PAGE21,                0x206)
ELF_RELOC(R_AARCH64_TLSLD_ADD_LO12_NC,               0x207)
ELF_RELOC(R_AARCH64_TLSLD_MOVW_G1,                   0x208)
ELF_RELOC(R_AARCH64_TLSLD_MOVW_G0_NC,                0x209)
ELF_RELOC(R_AARCH64_TLSLD_LD_PREL19,                 0x20a)
ELF_RELOC(R_AARCH64_TLSLD_MOVW_DTPREL_G2,            0x20b)
ELF_RELOC(R_AARCH64_TLSLD_MOVW_DTPREL_G1,            0x20c)
ELF_RELOC(R_AARCH64_TLSLD_MOVW_DTPREL_G1_NC,         0x20d)
ELF_RELOC(R_AARCH64_TLSLD_MOVW_DTPREL_G0,            0x20e)
ELF_RELOC(R_AARCH64_TLSLD_MOVW_DTPREL_G0_NC,         0x20f)
ELF_RELOC(R_AARCH64_TLSLD_ADD_DTPREL_HI12,           0x210)
ELF_RELOC(R_AARCH64_TLSLD_ADD_DTPREL_LO12,           0x211)
ELF_RELOC(R_AARCH64_TLSLD_ADD_DTPREL_LO12_NC,        0x212)
ELF_RELOC(R_AARCH64_TLSLD_LDST8_DTPREL_LO12,         0x213)
ELF_RELOC(R_AARCH64_TLSLD_LDST8_DTPREL_LO12_NC,      0x214)
ELF_RELOC(R_AARCH64_TLSLD_LDST16_DTPREL_LO12,        0x215)
ELF_RELOC(R_AARCH64_TLSLD_LDST16_DTPREL_LO12_NC,     0x216)
ELF_RELOC(R_AARCH64_TLSLD_LDST32_DTPREL_LO12,        0x217)
ELF_RELOC(R_AARCH64_TLSLD_LDST32_DTPREL_LO12_NC,     0x218)
ELF_RELOC(R_AARCH64_TLSLD_LDST64_DTPREL_LO12,        0x219)
ELF_RELOC(R_AARCH64_TLSLD_LDST64_DTPREL_LO12_NC,     0x21a)
ELF_RELOC(R_AARCH64_TLSIE_MOVW_GOTTPREL_G1,          0x21b)
ELF_RELOC(R_AARCH64_TLSIE_MOVW_GOTTPREL_G0_NC,       0x21c)
```

- **L73**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L74**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L75**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L76**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L77**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L78**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L79**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L80**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L81**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L82**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L83**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L84**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L85**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L86**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L87**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L88**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L89**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L90**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L91**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L92**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L93**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L94**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L95**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L96**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 97-120

```cpp
ELF_RELOC(R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21,       0x21d)
ELF_RELOC(R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC,     0x21e)
ELF_RELOC(R_AARCH64_TLSIE_LD_GOTTPREL_PREL19,        0x21f)
// Local exec TLS relocations
ELF_RELOC(R_AARCH64_TLSLE_MOVW_TPREL_G2,             0x220)
ELF_RELOC(R_AARCH64_TLSLE_MOVW_TPREL_G1,             0x221)
ELF_RELOC(R_AARCH64_TLSLE_MOVW_TPREL_G1_NC,          0x222)
ELF_RELOC(R_AARCH64_TLSLE_MOVW_TPREL_G0,             0x223)
ELF_RELOC(R_AARCH64_TLSLE_MOVW_TPREL_G0_NC,          0x224)
ELF_RELOC(R_AARCH64_TLSLE_ADD_TPREL_HI12,            0x225)
ELF_RELOC(R_AARCH64_TLSLE_ADD_TPREL_LO12,            0x226)
ELF_RELOC(R_AARCH64_TLSLE_ADD_TPREL_LO12_NC,         0x227)
ELF_RELOC(R_AARCH64_TLSLE_LDST8_TPREL_LO12,          0x228)
ELF_RELOC(R_AARCH64_TLSLE_LDST8_TPREL_LO12_NC,       0x229)
ELF_RELOC(R_AARCH64_TLSLE_LDST16_TPREL_LO12,         0x22a)
ELF_RELOC(R_AARCH64_TLSLE_LDST16_TPREL_LO12_NC,      0x22b)
ELF_RELOC(R_AARCH64_TLSLE_LDST32_TPREL_LO12,         0x22c)
ELF_RELOC(R_AARCH64_TLSLE_LDST32_TPREL_LO12_NC,      0x22d)
ELF_RELOC(R_AARCH64_TLSLE_LDST64_TPREL_LO12,         0x22e)
ELF_RELOC(R_AARCH64_TLSLE_LDST64_TPREL_LO12_NC,      0x22f)
// TLS descriptor relocations
ELF_RELOC(R_AARCH64_TLSDESC_LD_PREL19,               0x230)
ELF_RELOC(R_AARCH64_TLSDESC_ADR_PREL21,              0x231)
ELF_RELOC(R_AARCH64_TLSDESC_ADR_PAGE21,              0x232)
```

- **L97**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L98**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L99**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L100**: Comment documents the nearby API, invariant, or algorithmic intent: `Local exec TLS relocations`. / 这行注释说明了附近 API、不变量或算法意图：`Local exec TLS relocations`。
- **L101**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L102**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L103**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L104**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L105**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L106**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L107**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L108**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L109**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L110**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L111**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L112**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L113**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L114**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L115**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L116**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L117**: Comment documents the nearby API, invariant, or algorithmic intent: `TLS descriptor relocations`. / 这行注释说明了附近 API、不变量或算法意图：`TLS descriptor relocations`。
- **L118**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L119**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L120**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 121-144

```cpp
ELF_RELOC(R_AARCH64_TLSDESC_LD64_LO12,               0x233)
ELF_RELOC(R_AARCH64_TLSDESC_ADD_LO12,                0x234)
ELF_RELOC(R_AARCH64_TLSDESC_OFF_G1,                  0x235)
ELF_RELOC(R_AARCH64_TLSDESC_OFF_G0_NC,               0x236)
ELF_RELOC(R_AARCH64_TLSDESC_LDR,                     0x237)
ELF_RELOC(R_AARCH64_TLSDESC_ADD,                     0x238)
ELF_RELOC(R_AARCH64_TLSDESC_CALL,                    0x239)
ELF_RELOC(R_AARCH64_TLSLE_LDST128_TPREL_LO12,        0x23a)
ELF_RELOC(R_AARCH64_TLSLE_LDST128_TPREL_LO12_NC,     0x23b)
ELF_RELOC(R_AARCH64_TLSLD_LDST128_DTPREL_LO12,       0x23c)
ELF_RELOC(R_AARCH64_TLSLD_LDST128_DTPREL_LO12_NC,    0x23d)
// Dynamic relocations
ELF_RELOC(R_AARCH64_COPY,                            0x400)
ELF_RELOC(R_AARCH64_GLOB_DAT,                        0x401)
ELF_RELOC(R_AARCH64_JUMP_SLOT,                       0x402)
ELF_RELOC(R_AARCH64_RELATIVE,                        0x403)
// 0x404 and 0x405 are now R_AARCH64_TLS_IMPDEF1 and R_AARCH64_TLS_IMPDEF2
// We follow GNU and define TLS_IMPDEF1 as TLS_DTPMOD64 and TLS_IMPDEF2 as
// TLS_DTPREL64
ELF_RELOC(R_AARCH64_TLS_DTPMOD64,                    0x404)
ELF_RELOC(R_AARCH64_TLS_DTPREL64,                    0x405)
ELF_RELOC(R_AARCH64_TLS_TPREL64,                     0x406)
ELF_RELOC(R_AARCH64_TLSDESC,                         0x407)
ELF_RELOC(R_AARCH64_IRELATIVE,                       0x408)
```

- **L121**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L122**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L123**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L124**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L125**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L126**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L127**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L128**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L129**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L130**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L131**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L132**: Comment documents the nearby API, invariant, or algorithmic intent: `Dynamic relocations`. / 这行注释说明了附近 API、不变量或算法意图：`Dynamic relocations`。
- **L133**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L134**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L135**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L136**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L137**: Comment documents the nearby API, invariant, or algorithmic intent: `0x404 and 0x405 are now R_AARCH64_TLS_IMPDEF1 and R_AARCH64_TLS_IMPDEF2`. / 这行注释说明了附近 API、不变量或算法意图：`0x404 and 0x405 are now R_AARCH64_TLS_IMPDEF1 and R_AARCH64_TLS_IMPDEF2`。
- **L138**: Comment documents the nearby API, invariant, or algorithmic intent: `We follow GNU and define TLS_IMPDEF1 as TLS_DTPMOD64 and TLS_IMPDEF2 as`. / 这行注释说明了附近 API、不变量或算法意图：`We follow GNU and define TLS_IMPDEF1 as TLS_DTPMOD64 and TLS_IMPDEF2 as`。
- **L139**: Comment documents the nearby API, invariant, or algorithmic intent: `TLS_DTPREL64`. / 这行注释说明了附近 API、不变量或算法意图：`TLS_DTPREL64`。
- **L140**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L141**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L142**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L143**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L144**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 145-168

```cpp
// PAuthABI static and dynamic relocations: defined in pauthabielf64,
// https://github.com/ARM-software/abi-aa
ELF_RELOC(R_AARCH64_AUTH_ABS64,                      0x244)
ELF_RELOC(R_AARCH64_AUTH_MOVW_GOTOFF_G0,             0x245)
ELF_RELOC(R_AARCH64_AUTH_MOVW_GOTOFF_G0_NC,          0x246)
ELF_RELOC(R_AARCH64_AUTH_MOVW_GOTOFF_G1,             0x247)
ELF_RELOC(R_AARCH64_AUTH_MOVW_GOTOFF_G1_NC,          0x248)
ELF_RELOC(R_AARCH64_AUTH_MOVW_GOTOFF_G2,             0x249)
ELF_RELOC(R_AARCH64_AUTH_MOVW_GOTOFF_G2_NC,          0x24a)
ELF_RELOC(R_AARCH64_AUTH_MOVW_GOTOFF_G3,             0x24b)
ELF_RELOC(R_AARCH64_AUTH_GOT_LD_PREL19,              0x24c)
ELF_RELOC(R_AARCH64_AUTH_LD64_GOTOFF_LO15,           0x24d)
ELF_RELOC(R_AARCH64_AUTH_ADR_GOT_PAGE,               0x24e)
ELF_RELOC(R_AARCH64_AUTH_LD64_GOT_LO12_NC,           0x24f)
ELF_RELOC(R_AARCH64_AUTH_LD64_GOTPAGE_LO15,          0x250)
ELF_RELOC(R_AARCH64_AUTH_GOT_ADD_LO12_NC,            0x251)
ELF_RELOC(R_AARCH64_AUTH_GOT_ADR_PREL_LO21,          0x252)
ELF_RELOC(R_AARCH64_AUTH_TLSDESC_ADR_PAGE21,         0x253)
ELF_RELOC(R_AARCH64_AUTH_TLSDESC_LD64_LO12,          0x254)
ELF_RELOC(R_AARCH64_AUTH_TLSDESC_ADD_LO12,           0x255)
ELF_RELOC(R_AARCH64_AUTH_RELATIVE,                   0x411)
ELF_RELOC(R_AARCH64_AUTH_GLOB_DAT,                   0x412)
ELF_RELOC(R_AARCH64_AUTH_TLSDESC,                    0x413)
ELF_RELOC(R_AARCH64_AUTH_IRELATIVE,                  0x414)
```

- **L145**: Comment documents the nearby API, invariant, or algorithmic intent: `PAuthABI static and dynamic relocations: defined in pauthabielf64,`. / 这行注释说明了附近 API、不变量或算法意图：`PAuthABI static and dynamic relocations: defined in pauthabielf64,`。
- **L146**: Comment documents the nearby API, invariant, or algorithmic intent: `https://github.com/ARM-software/abi-aa`. / 这行注释说明了附近 API、不变量或算法意图：`https://github.com/ARM-software/abi-aa`。
- **L147**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L148**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L149**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L150**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L151**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L152**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L153**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L154**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L155**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L156**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L157**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L158**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L159**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L160**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L161**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L162**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L163**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L164**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L165**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L166**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L167**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L168**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 169-192

```cpp

// ELF32
// ELF_RELOC(R_AARCH64_P32_NONE,                         0)
ELF_RELOC(R_AARCH64_P32_ABS32,                       0x001)
ELF_RELOC(R_AARCH64_P32_ABS16,                       0x002)
ELF_RELOC(R_AARCH64_P32_PREL32,                      0x003)
ELF_RELOC(R_AARCH64_P32_PREL16,                      0x004)
ELF_RELOC(R_AARCH64_P32_MOVW_UABS_G0,                0x005)
ELF_RELOC(R_AARCH64_P32_MOVW_UABS_G0_NC,             0x006)
ELF_RELOC(R_AARCH64_P32_MOVW_UABS_G1,                0x007)
ELF_RELOC(R_AARCH64_P32_MOVW_SABS_G0,                0x008)
ELF_RELOC(R_AARCH64_P32_LD_PREL_LO19,                0x009)
ELF_RELOC(R_AARCH64_P32_ADR_PREL_LO21,               0x00a)
ELF_RELOC(R_AARCH64_P32_ADR_PREL_PG_HI21,            0x00b)
ELF_RELOC(R_AARCH64_P32_ADD_ABS_LO12_NC,             0x00c)
ELF_RELOC(R_AARCH64_P32_LDST8_ABS_LO12_NC,           0x00d)
ELF_RELOC(R_AARCH64_P32_LDST16_ABS_LO12_NC,          0x00e)
ELF_RELOC(R_AARCH64_P32_LDST32_ABS_LO12_NC,          0x00f)
ELF_RELOC(R_AARCH64_P32_LDST64_ABS_LO12_NC,          0x010)
ELF_RELOC(R_AARCH64_P32_LDST128_ABS_LO12_NC,         0x011)
ELF_RELOC(R_AARCH64_P32_TSTBR14,                     0x012)
ELF_RELOC(R_AARCH64_P32_CONDBR19,                    0x013)
ELF_RELOC(R_AARCH64_P32_JUMP26,                      0x014)
ELF_RELOC(R_AARCH64_P32_CALL26,                      0x015)
```

- **L169**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L170**: Comment documents the nearby API, invariant, or algorithmic intent: `ELF32`. / 这行注释说明了附近 API、不变量或算法意图：`ELF32`。
- **L171**: Comment documents the nearby API, invariant, or algorithmic intent: `ELF_RELOC(R_AARCH64_P32_NONE, 0)`. / 这行注释说明了附近 API、不变量或算法意图：`ELF_RELOC(R_AARCH64_P32_NONE, 0)`。
- **L172**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L173**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L174**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L175**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L176**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L177**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L178**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L179**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L180**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L181**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L182**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L183**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L184**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L185**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L186**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L187**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L188**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L189**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L190**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L191**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L192**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 193-216

```cpp
ELF_RELOC(R_AARCH64_P32_MOVW_PREL_G0,                0x016)
ELF_RELOC(R_AARCH64_P32_MOVW_PREL_G0_NC,             0x017)
ELF_RELOC(R_AARCH64_P32_MOVW_PREL_G1,                0x018)
ELF_RELOC(R_AARCH64_P32_GOT_LD_PREL19,               0x019)
ELF_RELOC(R_AARCH64_P32_ADR_GOT_PAGE,                0x01a)
ELF_RELOC(R_AARCH64_P32_LD32_GOT_LO12_NC,            0x01b)
ELF_RELOC(R_AARCH64_P32_LD32_GOTPAGE_LO14,           0x01c)
ELF_RELOC(R_AARCH64_P32_PLT32,                       0x01d)
ELF_RELOC(R_AARCH64_P32_TLSGD_ADR_PREL21,            0x050)
ELF_RELOC(R_AARCH64_P32_TLSGD_ADR_PAGE21,            0x051)
ELF_RELOC(R_AARCH64_P32_TLSGD_ADD_LO12_NC,           0x052)
ELF_RELOC(R_AARCH64_P32_TLSLD_ADR_PREL21,            0x053)
ELF_RELOC(R_AARCH64_P32_TLSLD_ADR_PAGE21,            0x054)
ELF_RELOC(R_AARCH64_P32_TLSLD_ADD_LO12_NC,           0x055)
ELF_RELOC(R_AARCH64_P32_TLSLD_LD_PREL19,             0x056)
ELF_RELOC(R_AARCH64_P32_TLSLD_MOVW_DTPREL_G1,        0x057)
ELF_RELOC(R_AARCH64_P32_TLSLD_MOVW_DTPREL_G0,        0x058)
ELF_RELOC(R_AARCH64_P32_TLSLD_MOVW_DTPREL_G0_NC,     0x059)
ELF_RELOC(R_AARCH64_P32_TLSLD_ADD_DTPREL_HI12,       0x05a)
ELF_RELOC(R_AARCH64_P32_TLSLD_ADD_DTPREL_LO12,       0x05b)
ELF_RELOC(R_AARCH64_P32_TLSLD_ADD_DTPREL_LO12_NC,    0x05c)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST8_DTPREL_LO12,     0x05d)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST8_DTPREL_LO12_NC,  0x05e)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST16_DTPREL_LO12,    0x05f)
```

- **L193**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L194**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L195**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L196**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L197**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L198**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L199**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L200**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L201**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L202**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L203**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L204**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L205**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L206**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L207**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L208**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L209**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L210**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L211**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L212**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L213**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L214**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L215**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L216**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 217-240

```cpp
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST16_DTPREL_LO12_NC, 0x060)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST32_DTPREL_LO12,    0x061)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST32_DTPREL_LO12_NC, 0x062)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST64_DTPREL_LO12,    0x063)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST64_DTPREL_LO12_NC, 0x064)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST128_DTPREL_LO12,   0x065)
ELF_RELOC(R_AARCH64_P32_TLSLD_LDST128_DTPREL_LO12_NC,0x066)
ELF_RELOC(R_AARCH64_P32_TLSIE_ADR_GOTTPREL_PAGE21,   0x067)
ELF_RELOC(R_AARCH64_P32_TLSIE_LD32_GOTTPREL_LO12_NC, 0x068)
ELF_RELOC(R_AARCH64_P32_TLSIE_LD_GOTTPREL_PREL19,    0x069)
ELF_RELOC(R_AARCH64_P32_TLSLE_MOVW_TPREL_G1,         0x06a)
ELF_RELOC(R_AARCH64_P32_TLSLE_MOVW_TPREL_G0,         0x06b)
ELF_RELOC(R_AARCH64_P32_TLSLE_MOVW_TPREL_G0_NC,      0x06c)
ELF_RELOC(R_AARCH64_P32_TLSLE_ADD_TPREL_HI12,        0x06d)
ELF_RELOC(R_AARCH64_P32_TLSLE_ADD_TPREL_LO12,        0x06e)
ELF_RELOC(R_AARCH64_P32_TLSLE_ADD_TPREL_LO12_NC,     0x06f)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST8_TPREL_LO12,      0x070)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST8_TPREL_LO12_NC,   0x071)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST16_TPREL_LO12,     0x072)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST16_TPREL_LO12_NC,  0x073)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST32_TPREL_LO12,     0x074)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST32_TPREL_LO12_NC,  0x075)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST64_TPREL_LO12,     0x076)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST64_TPREL_LO12_NC,  0x077)
```

- **L217**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L218**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L219**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L220**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L221**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L222**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L223**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L224**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L225**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L226**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L227**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L228**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L229**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L230**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L231**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L232**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L233**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L234**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L235**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L236**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L237**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L238**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L239**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L240**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

### Lines 241-258

```cpp
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST128_TPREL_LO12,    0x078)
ELF_RELOC(R_AARCH64_P32_TLSLE_LDST128_TPREL_LO12_NC, 0x079)
ELF_RELOC(R_AARCH64_P32_TLSDESC_LD_PREL19,           0x07a)
ELF_RELOC(R_AARCH64_P32_TLSDESC_ADR_PREL21,          0x07b)
ELF_RELOC(R_AARCH64_P32_TLSDESC_ADR_PAGE21,          0x07c)
ELF_RELOC(R_AARCH64_P32_TLSDESC_LD32_LO12,           0x07d)
ELF_RELOC(R_AARCH64_P32_TLSDESC_ADD_LO12,            0x07e)
ELF_RELOC(R_AARCH64_P32_TLSDESC_CALL,                0x07f)
// Dynamic relocations
ELF_RELOC(R_AARCH64_P32_COPY,                        0x0b4)
ELF_RELOC(R_AARCH64_P32_GLOB_DAT,                    0x0b5)
ELF_RELOC(R_AARCH64_P32_JUMP_SLOT,                   0x0b6)
ELF_RELOC(R_AARCH64_P32_RELATIVE,                    0x0b7)
ELF_RELOC(R_AARCH64_P32_TLS_DTPREL,                  0x0b8)
ELF_RELOC(R_AARCH64_P32_TLS_DTPMOD,                  0x0b9)
ELF_RELOC(R_AARCH64_P32_TLS_TPREL,                   0x0ba)
ELF_RELOC(R_AARCH64_P32_TLSDESC,                     0x0bb)
ELF_RELOC(R_AARCH64_P32_IRELATIVE,                   0x0bc)
```

- **L241**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L242**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L243**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L244**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L245**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L246**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L247**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L248**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L249**: Comment documents the nearby API, invariant, or algorithmic intent: `Dynamic relocations`. / 这行注释说明了附近 API、不变量或算法意图：`Dynamic relocations`。
- **L250**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L251**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L252**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L253**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L254**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L255**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L256**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L257**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。
- **L258**: Invokes macro `ELF_RELOC` to emit generated declarations, attributes, or table entries. / 调用宏 `ELF_RELOC` 来生成声明、属性或表项。

## Key Concepts / 关键概念

- EN: Layer: `BinaryFormat` belongs to LLVM's object-file and debug binary format descriptions subsystem.
  - CN: 层次：`BinaryFormat` 属于 LLVM 的目标文件与调试二进制格式描述子系统。
- EN: X-macro pattern: the file is intended to be included multiple times under different macro definitions to generate tables or boilerplate.
  - CN: X-macro 模式：该文件预期在不同宏定义下被多次包含，以生成表格或样板代码。

## Dependencies / 依赖关系

- EN: Macro consumers: entries here are activated by macros such as `ELF_RELOC`, `ABI`, `ARM`, `ELF64`, `R_AARCH64_NONE`, `R_AARCH64_ABS64`, `R_AARCH64_ABS32`, `R_AARCH64_ABS16`, which are expected to be defined by including files.
  - CN: 宏消费者：这里的条目通过 `ELF_RELOC`, `ABI`, `ARM`, `ELF64`, `R_AARCH64_NONE`, `R_AARCH64_ABS64`, `R_AARCH64_ABS32`, `R_AARCH64_ABS16` 等宏被激活，而这些宏通常由包含它的文件预先定义。
