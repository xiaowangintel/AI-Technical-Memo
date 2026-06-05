# Relocations.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lld/MachO/Relocations.h`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: Declares Mach-O linker logic for symbol graphs, address assignment, and output-file construction.
- **Purpose (CN) / 用途（中文）**: 声明 Mach-O 链接器的符号图、地址分配以及输出文件构建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```cpp
   1: //===- Relocations.h --------------------------------------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
```

- **L1**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L2**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L3**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L4**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L5**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L6**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L7**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L8**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 9-16 / 第 9-16 行

```cpp
   9: #ifndef LLD_MACHO_RELOCATIONS_H
  10: #define LLD_MACHO_RELOCATIONS_H
  11: 
  12: #include "llvm/ADT/BitmaskEnum.h"
  13: #include "llvm/ADT/PointerUnion.h"
  14: #include "llvm/BinaryFormat/MachO.h"
  15: #include "llvm/Support/Endian.h"
  16: 
```

- **L9**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L10**: Defines macro \`LLD_MACHO_RELOCATIONS_H\` for conditional compilation or textual reuse. / 定义宏 \`LLD_MACHO_RELOCATIONS_H\`，供条件编译或文本复用使用。
- **L11**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L12**: Includes \`llvm/ADT/BitmaskEnum.h\` so this file can use declarations from that header. / 引入 \`llvm/ADT/BitmaskEnum.h\`，使当前文件能够使用该头文件中的声明。
- **L13**: Includes \`llvm/ADT/PointerUnion.h\` so this file can use declarations from that header. / 引入 \`llvm/ADT/PointerUnion.h\`，使当前文件能够使用该头文件中的声明。
- **L14**: Includes \`llvm/BinaryFormat/MachO.h\` so this file can use declarations from that header. / 引入 \`llvm/BinaryFormat/MachO.h\`，使当前文件能够使用该头文件中的声明。
- **L15**: Includes \`llvm/Support/Endian.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/Endian.h\`，使当前文件能够使用该头文件中的声明。
- **L16**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 17-23 / 第 17-23 行

```cpp
  17: #include <cstddef>
  18: #include <cstdint>
  19: 
  20: namespace lld::macho {
  21: LLVM_ENABLE_BITMASK_ENUMS_IN_NAMESPACE();
  22: 
  23: class Symbol;
```

- **L17**: Includes \`cstddef\` so this file can use declarations from that header. / 引入 \`cstddef\`，使当前文件能够使用该头文件中的声明。
- **L18**: Includes \`cstdint\` so this file can use declarations from that header. / 引入 \`cstdint\`，使当前文件能够使用该头文件中的声明。
- **L19**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L20**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L21**: Invokes a macro that expands subsystem-specific boilerplate or registration logic. / 调用一个宏，以展开子系统特定的样板代码或注册逻辑。
- **L22**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L23**: Begins the declaration of class \`Symbol\`. / 开始声明 class \`Symbol\`。

### Lines 24-37 / 第 24-37 行

```cpp
  24: class InputSection;
  25: 
  26: enum class RelocAttrBits {
  27:   _0 = 0,              // invalid
  28:   BYTE1 = 1 << 0,      // 1 byte datum
  29:   BYTE2 = 1 << 1,      // 2 byte datum
  30:   BYTE4 = 1 << 2,      // 4 byte datum
  31:   BYTE8 = 1 << 3,      // 8 byte datum
  32:   PCREL = 1 << 4,      // Value is PC-relative offset
  33:   ABSOLUTE = 1 << 5,   // Value is an absolute address or fixed offset
  34:   EXTERN = 1 << 6,     // Can have an external symbol
  35:   LOCAL = 1 << 7,      // Can have a local symbol
  36:   ADDEND = 1 << 8,     // *_ADDEND paired prefix reloc
  37:   SUBTRAHEND = 1 << 9, // *_SUBTRACTOR paired prefix reloc
```

- **L24**: Begins the declaration of class \`InputSection\`. / 开始声明 class \`InputSection\`。
- **L25**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L26**: Begins the declaration of enum \`RelocAttrBits\`. / 开始声明枚举 \`RelocAttrBits\`。
- **L27**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L28**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L29**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L30**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L31**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L32**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L33**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L34**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L35**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L36**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L37**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 38-47 / 第 38-47 行

```cpp
  38:   BRANCH = 1 << 10,    // Value is branch target
  39:   GOT = 1 << 11,       // References a symbol in the Global Offset Table
  40:   TLV = 1 << 12,       // References a thread-local symbol
  41:   LOAD = 1 << 13,      // Relaxable indirect load
  42:   POINTER = 1 << 14,   // Non-relaxable indirect load (pointer is taken)
  43:   UNSIGNED = 1 << 15,  // *_UNSIGNED relocs
  44:   LLVM_MARK_AS_BITMASK_ENUM(/*LargestValue*/ (1 << 16) - 1),
  45: };
  46: // Note: SUBTRACTOR always pairs with UNSIGNED (a delta between two symbols).
  47: 
```

- **L38**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L39**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L40**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L41**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L42**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L43**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L44**: Invokes a macro that expands subsystem-specific boilerplate or registration logic. / 调用一个宏，以展开子系统特定的样板代码或注册逻辑。
- **L45**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L46**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L47**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 48-54 / 第 48-54 行

```cpp
  48: struct RelocAttrs {
  49:   llvm::StringRef name;
  50:   RelocAttrBits bits;
  51:   bool hasAttr(RelocAttrBits b) const { return (bits & b) == b; }
  52: };
  53: 
  54: struct Relocation {
```

- **L48**: Begins the declaration of struct \`RelocAttrs\`. / 开始声明 struct \`RelocAttrs\`。
- **L49**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L50**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L51**: Defines function or method \`hasAttr\`. / 定义函数或方法 \`hasAttr\`。
- **L52**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L53**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L54**: Begins the declaration of struct \`Relocation\`. / 开始声明 struct \`Relocation\`。

### Lines 55-65 / 第 55-65 行

```cpp
  55:   uint8_t type = llvm::MachO::GENERIC_RELOC_INVALID;
  56:   bool pcrel = false;
  57:   uint8_t length = 0;
  58:   // The offset from the start of the subsection that this relocation belongs
  59:   // to.
  60:   uint32_t offset = 0;
  61:   // Adding this offset to the address of the referent symbol or subsection
  62:   // gives the destination that this relocation refers to.
  63:   int64_t addend = 0;
  64:   llvm::PointerUnion<Symbol *, InputSection *> referent = nullptr;
  65: 
```

- **L55**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L56**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L57**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L58**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L59**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L60**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L61**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L62**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L63**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L64**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L65**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 66-73 / 第 66-73 行

```cpp
  66:   Relocation() = default;
  67: 
  68:   Relocation(uint8_t type, bool pcrel, uint8_t length, uint32_t offset,
  69:              int64_t addend,
  70:              llvm::PointerUnion<Symbol *, InputSection *> referent)
  71:       : type(type), pcrel(pcrel), length(length), offset(offset),
  72:         addend(addend), referent(referent) {}
  73: 
```

- **L66**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L67**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L68**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L69**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L70**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L71**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L72**: Defines function or method \`addend\`. / 定义函数或方法 \`addend\`。
- **L73**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 74-80 / 第 74-80 行

```cpp
  74:   InputSection *getReferentInputSection() const;
  75: 
  76:   // Must point to an offset within a CStringInputSection or a
  77:   // ConcatInputSection.
  78:   llvm::StringRef getReferentString() const;
  79: };
  80: 
```

- **L74**: Declares function or method \`getReferentInputSection\`. / 声明函数或方法 \`getReferentInputSection\`。
- **L75**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L76**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L77**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L78**: Declares function or method \`getReferentString\`. / 声明函数或方法 \`getReferentString\`。
- **L79**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L80**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 81-90 / 第 81-90 行

```cpp
  81: bool validateSymbolRelocation(const Symbol *, const InputSection *,
  82:                               const Relocation &);
  83: 
  84: /*
  85:  * v: The value the relocation is attempting to encode
  86:  * bits: The number of bits actually available to encode this relocation
  87:  */
  88: void reportRangeError(void *loc, const Relocation &, const llvm::Twine &v,
  89:                       uint8_t bits, int64_t min, uint64_t max);
  90: 
```

- **L81**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L82**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L83**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L84**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L85**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L86**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L87**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L88**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L89**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L90**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 91-98 / 第 91-98 行

```cpp
  91: struct SymbolDiagnostic {
  92:   const Symbol *symbol;
  93:   llvm::StringRef reason;
  94: };
  95: 
  96: void reportRangeError(void *loc, SymbolDiagnostic, const llvm::Twine &v,
  97:                       uint8_t bits, int64_t min, uint64_t max);
  98: 
```

- **L91**: Begins the declaration of struct \`SymbolDiagnostic\`. / 开始声明 struct \`SymbolDiagnostic\`。
- **L92**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L93**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L94**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L95**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L96**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L97**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L98**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 99-105 / 第 99-105 行

```cpp
  99: template <typename Diagnostic>
 100: inline void checkInt(void *loc, Diagnostic d, int64_t v, int bits) {
 101:   if (v != llvm::SignExtend64(v, bits))
 102:     reportRangeError(loc, d, llvm::Twine(v), bits, llvm::minIntN(bits),
 103:                      llvm::maxIntN(bits));
 104: }
 105: 
```

- **L99**: Introduces template parameters for a generic declaration or definition. / 为后续泛型声明或定义引入模板参数。
- **L100**: Defines function or method \`checkInt\`. / 定义函数或方法 \`checkInt\`。
- **L101**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L102**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L103**: Declares function or method \`maxIntN\`. / 声明函数或方法 \`maxIntN\`。
- **L104**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L105**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 106-119 / 第 106-119 行

```cpp
 106: template <typename Diagnostic>
 107: inline void checkUInt(void *loc, Diagnostic d, uint64_t v, int bits) {
 108:   if ((v >> bits) != 0)
 109:     reportRangeError(loc, d, llvm::Twine(v), bits, 0, llvm::maxUIntN(bits));
 110: }
 111: 
 112: inline void writeAddress(uint8_t *loc, uint64_t addr, uint8_t length) {
 113:   switch (length) {
 114:   case 2:
 115:     llvm::support::endian::write32le(loc, addr);
 116:     break;
 117:   case 3:
 118:     llvm::support::endian::write64le(loc, addr);
 119:     break;
```

- **L106**: Introduces template parameters for a generic declaration or definition. / 为后续泛型声明或定义引入模板参数。
- **L107**: Defines function or method \`checkUInt\`. / 定义函数或方法 \`checkUInt\`。
- **L108**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L109**: Declares function or method \`reportRangeError\`. / 声明函数或方法 \`reportRangeError\`。
- **L110**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L111**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L112**: Defines function or method \`writeAddress\`. / 定义函数或方法 \`writeAddress\`。
- **L113**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L114**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L115**: Declares function or method \`write32le\`. / 声明函数或方法 \`write32le\`。
- **L116**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。
- **L117**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L118**: Declares function or method \`write64le\`. / 声明函数或方法 \`write64le\`。
- **L119**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。

### Lines 120-126 / 第 120-126 行

```cpp
 120:   default:
 121:     llvm_unreachable("invalid r_length");
 122:   }
 123: }
 124: 
 125: InputSection *offsetToInputSection(uint64_t *);
 126: 
```

- **L120**: Marks the default \`switch\` alternative. / 标记 \`switch\` 语句中的默认分支。
- **L121**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。
- **L122**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L123**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L124**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L125**: Declares function or method \`offsetToInputSection\`. / 声明函数或方法 \`offsetToInputSection\`。
- **L126**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 127-131 / 第 127-131 行

```cpp
 127: extern const RelocAttrs invalidRelocAttrs;
 128: 
 129: } // namespace lld::Macho
 130: 
 131: #endif
```

- **L127**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L128**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L129**: Closes a named namespace scope. / 关闭一个具名命名空间作用域。
- **L130**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L131**: Ends the active conditional-compilation region. / 结束当前条件编译区域。

## Key Concepts / 关键概念

- **Role / 角色**: Declares Mach-O linker logic for symbol graphs, address assignment, and output-file construction. / 声明 Mach-O 链接器的符号图、地址分配以及输出文件构建逻辑。
- **Scale / 规模**: 131 lines, 6 direct includes, 6 named types, and 14 detected routines. / 共 131 行，含 6 个直接包含、6 个具名类型、14 个检测到的例程。
- **Mach-O linking / Mach-O 链接**: The code manages Mach-O atoms, symbols, segments, or output-file construction. / 该代码管理 Mach-O 原子、符号、段或输出文件构建。
- **Relocation processing / 重定位处理**: The implementation reasons about relocation records, fixups, or address adjustments. / 该实现处理重定位记录、修正项或地址调整。
- **Symbol management / 符号管理**: The code tracks declarations, references, visibility, or resolution of symbols. / 该代码跟踪符号的声明、引用、可见性或解析过程。
- **Section/segment layout / 节/段布局**: The file manipulates input or output sections, segments, or related layout state. / 该文件操作输入/输出节、段或相关布局状态。
- **Concurrency control / 并发控制**: The implementation manages synchronization, thread state, or atomic updates. / 该实现管理同步、线程状态或原子更新。
- **Rich type modeling / 丰富的类型建模**: The file introduces or manipulates named types that capture subsystem state. / 该文件引入或操作用于表达子系统状态的具名类型。

## Dependencies / 依赖关系

- **LLVM / LLVM**: `llvm/ADT/BitmaskEnum.h`, `llvm/ADT/PointerUnion.h`, `llvm/BinaryFormat/MachO.h`, `llvm/Support/Endian.h`.
- **System or local / 系统或本地**: `cstddef`, `cstdint`.
- **Header roles / 头文件角色**: LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2), standard-library or local support header / 标准库或本地支持头文件 (2), binary-format constants and helpers / 二进制格式常量与辅助工具 (1), support-library helpers / Support 库辅助功能 (1).
- **Core types / 核心类型**: `Symbol`, `InputSection`, `RelocAttrBits`, `RelocAttrs`, `Relocation`, `SymbolDiagnostic`.
- **Visible routines / 可见例程**: `LLVM_ENABLE_BITMASK_ENUMS_IN_NAMESPACE`, `hasAttr`, `addend`, `getReferentInputSection`, `getReferentString`, `checkInt`, `maxIntN`, `checkUInt`, `reportRangeError`, `writeAddress`, `write32le`, `write64le`.
