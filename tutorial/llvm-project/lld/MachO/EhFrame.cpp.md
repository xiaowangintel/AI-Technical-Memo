# EhFrame.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lld/MachO/EhFrame.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: Implements Mach-O linker logic for symbol graphs, address assignment, and output-file construction.
- **Purpose (CN) / 用途（中文）**: 实现 Mach-O 链接器的符号图、地址分配以及输出文件构建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```cpp
   1: //===- EhFrame.cpp --------------------------------------------------------===//
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

### Lines 9-15 / 第 9-15 行

```cpp
   9: #include "EhFrame.h"
  10: #include "InputFiles.h"
  11: 
  12: #include "lld/Common/ErrorHandler.h"
  13: #include "llvm/BinaryFormat/Dwarf.h"
  14: #include "llvm/Support/Endian.h"
  15: 
```

- **L9**: Includes \`EhFrame.h\` so this file can use declarations from that header. / 引入 \`EhFrame.h\`，使当前文件能够使用该头文件中的声明。
- **L10**: Includes \`InputFiles.h\` so this file can use declarations from that header. / 引入 \`InputFiles.h\`，使当前文件能够使用该头文件中的声明。
- **L11**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L12**: Includes \`lld/Common/ErrorHandler.h\` so this file can use declarations from that header. / 引入 \`lld/Common/ErrorHandler.h\`，使当前文件能够使用该头文件中的声明。
- **L13**: Includes \`llvm/BinaryFormat/Dwarf.h\` so this file can use declarations from that header. / 引入 \`llvm/BinaryFormat/Dwarf.h\`，使当前文件能够使用该头文件中的声明。
- **L14**: Includes \`llvm/Support/Endian.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/Endian.h\`，使当前文件能够使用该头文件中的声明。
- **L15**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 16-29 / 第 16-29 行

```cpp
  16: using namespace llvm;
  17: using namespace lld;
  18: using namespace lld::macho;
  19: using namespace llvm::support::endian;
  20: 
  21: uint64_t EhReader::readLength(size_t *off) const {
  22:   const size_t errOff = *off;
  23:   if (*off + 4 > data.size())
  24:     failOn(errOff, "CIE/FDE too small");
  25:   uint64_t len = read32le(data.data() + *off);
  26:   *off += 4;
  27:   if (len == dwarf::DW_LENGTH_DWARF64) {
  28:     // FIXME: test this DWARF64 code path
  29:     if (*off + 8 > data.size())
```

- **L16**: Imports namespace \`llvm\` into the current scope for shorter symbol references. / 将命名空间 \`llvm\` 导入当前作用域，以便更简洁地引用符号。
- **L17**: Imports namespace \`lld\` into the current scope for shorter symbol references. / 将命名空间 \`lld\` 导入当前作用域，以便更简洁地引用符号。
- **L18**: Imports namespace \`lld::macho\` into the current scope for shorter symbol references. / 将命名空间 \`lld::macho\` 导入当前作用域，以便更简洁地引用符号。
- **L19**: Imports namespace \`llvm::support::endian\` into the current scope for shorter symbol references. / 将命名空间 \`llvm::support::endian\` 导入当前作用域，以便更简洁地引用符号。
- **L20**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L21**: Defines function or method \`readLength\`. / 定义函数或方法 \`readLength\`。
- **L22**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L23**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L24**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L25**: Declares function or method \`read32le\`. / 声明函数或方法 \`read32le\`。
- **L26**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L27**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L28**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L29**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。

### Lines 30-38 / 第 30-38 行

```cpp
  30:       failOn(errOff, "CIE/FDE too small");
  31:     len = read64le(data.data() + *off);
  32:     *off += 8;
  33:   }
  34:   if (*off + len > data.size())
  35:     failOn(errOff, "CIE/FDE extends past the end of the section");
  36:   return len;
  37: }
  38: 
```

- **L30**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L31**: Declares function or method \`read64le\`. / 声明函数或方法 \`read64le\`。
- **L32**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L33**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L34**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L35**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L36**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L37**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L38**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 39-45 / 第 39-45 行

```cpp
  39: void EhReader::skipValidLength(size_t *off) const {
  40:   uint32_t len = read32le(data.data() + *off);
  41:   *off += 4;
  42:   if (len == dwarf::DW_LENGTH_DWARF64)
  43:     *off += 8;
  44: }
  45: 
```

- **L39**: Defines function or method \`skipValidLength\`. / 定义函数或方法 \`skipValidLength\`。
- **L40**: Declares function or method \`read32le\`. / 声明函数或方法 \`read32le\`。
- **L41**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L42**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L43**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L44**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L45**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 46-52 / 第 46-52 行

```cpp
  46: // Read a byte and advance off by one byte.
  47: uint8_t EhReader::readByte(size_t *off) const {
  48:   if (*off + 1 > data.size())
  49:     failOn(*off, "unexpected end of CIE/FDE");
  50:   return data[(*off)++];
  51: }
  52: 
```

- **L46**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L47**: Defines function or method \`readByte\`. / 定义函数或方法 \`readByte\`。
- **L48**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L49**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L50**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L51**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L52**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 53-60 / 第 53-60 行

```cpp
  53: uint32_t EhReader::readU32(size_t *off) const {
  54:   if (*off + 4 > data.size())
  55:     failOn(*off, "unexpected end of CIE/FDE");
  56:   uint32_t v = read32le(data.data() + *off);
  57:   *off += 4;
  58:   return v;
  59: }
  60: 
```

- **L53**: Defines function or method \`readU32\`. / 定义函数或方法 \`readU32\`。
- **L54**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L55**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L56**: Declares function or method \`read32le\`. / 声明函数或方法 \`read32le\`。
- **L57**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L58**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L59**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L60**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 61-74 / 第 61-74 行

```cpp
  61: uint64_t EhReader::readPointer(size_t *off, uint8_t size) const {
  62:   if (*off + size > data.size())
  63:     failOn(*off, "unexpected end of CIE/FDE");
  64:   uint64_t v;
  65:   if (size == 8)
  66:     v = read64le(data.data() + *off);
  67:   else {
  68:     assert(size == 4);
  69:     v = read32le(data.data() + *off);
  70:   }
  71:   *off += size;
  72:   return v;
  73: }
  74: 
```

- **L61**: Defines function or method \`readPointer\`. / 定义函数或方法 \`readPointer\`。
- **L62**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L63**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L64**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L65**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L66**: Declares function or method \`read64le\`. / 声明函数或方法 \`read64le\`。
- **L67**: Begins the fallback branch for the preceding conditional logic. / 开始前一个条件逻辑的后备分支。
- **L68**: Declares function or method \`assert\`. / 声明函数或方法 \`assert\`。
- **L69**: Declares function or method \`read32le\`. / 声明函数或方法 \`read32le\`。
- **L70**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L71**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L72**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L73**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L74**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 75-87 / 第 75-87 行

```cpp
  75: // Read a null-terminated string.
  76: StringRef EhReader::readString(size_t *off) const {
  77:   if (*off > data.size())
  78:     failOn(*off, "corrupted CIE (failed to read string)");
  79:   const size_t maxlen = data.size() - *off;
  80:   auto *c = reinterpret_cast<const char *>(data.data() + *off);
  81:   size_t len = strnlen(c, maxlen);
  82:   if (len == maxlen) // we failed to find the null terminator
  83:     failOn(*off, "corrupted CIE (failed to read string)");
  84:   *off += len + 1; // skip the null byte too
  85:   return StringRef(c, len);
  86: }
  87: 
```

- **L75**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L76**: Defines function or method \`readString\`. / 定义函数或方法 \`readString\`。
- **L77**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L78**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L79**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L80**: Declares function or method \`data\`. / 声明函数或方法 \`data\`。
- **L81**: Declares function or method \`strnlen\`. / 声明函数或方法 \`strnlen\`。
- **L82**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L83**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L84**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L85**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L86**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L87**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 88-97 / 第 88-97 行

```cpp
  88: void EhReader::skipLeb128(size_t *off) const {
  89:   const size_t errOff = *off;
  90:   while (*off < data.size()) {
  91:     uint8_t val = data[(*off)++];
  92:     if ((val & 0x80) == 0)
  93:       return;
  94:   }
  95:   failOn(errOff, "corrupted CIE (failed to read LEB128)");
  96: }
  97: 
```

- **L88**: Defines function or method \`skipLeb128\`. / 定义函数或方法 \`skipLeb128\`。
- **L89**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L90**: Starts a loop that repeats while the condition remains true. / 开始一个在条件为真时持续执行的循环。
- **L91**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L92**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L93**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L94**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L95**: Declares function or method \`failOn\`. / 声明函数或方法 \`failOn\`。
- **L96**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L97**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 98-108 / 第 98-108 行

```cpp
  98: void EhReader::failOn(size_t errOff, const Twine &msg) const {
  99:   fatal(toString(file) + ":(__eh_frame+0x" +
 100:         Twine::utohexstr(dataOff + errOff) + "): " + msg);
 101: }
 102: 
 103: /*
 104:  * Create a pair of relocs to write the value of:
 105:  *   `b - (offset + a)` if Invert == false
 106:  *   `(a + offset) - b` if Invert == true
 107:  */
 108: template <bool Invert = false>
```

- **L98**: Defines function or method \`failOn\`. / 定义函数或方法 \`failOn\`。
- **L99**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L100**: Declares function or method \`utohexstr\`. / 声明函数或方法 \`utohexstr\`。
- **L101**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L102**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L103**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L104**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L105**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L106**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L107**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L108**: Introduces template parameters for a generic declaration or definition. / 为后续泛型声明或定义引入模板参数。

### Lines 109-122 / 第 109-122 行

```cpp
 109: static void createSubtraction(PointerUnion<Symbol *, InputSection *> a,
 110:                               PointerUnion<Symbol *, InputSection *> b,
 111:                               uint64_t off, uint8_t length,
 112:                               SmallVectorImpl<Relocation> *newRelocs) {
 113:   auto subtrahend = a;
 114:   auto minuend = b;
 115:   if (Invert)
 116:     std::swap(subtrahend, minuend);
 117:   assert(isa<Symbol *>(subtrahend));
 118:   Relocation subtrahendReloc(target->subtractorRelocType, /*pcrel=*/false,
 119:                              length, off, /*addend=*/0, subtrahend);
 120:   Relocation minuendReloc(target->unsignedRelocType, /*pcrel=*/false, length,
 121:                           off, (Invert ? 1 : -1) * off, minuend);
 122:   newRelocs->push_back(subtrahendReloc);
```

- **L109**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L110**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L111**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L112**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L113**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L114**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L115**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L116**: Declares function or method \`swap\`. / 声明函数或方法 \`swap\`。
- **L117**: Declares function or method \`assert\`. / 声明函数或方法 \`assert\`。
- **L118**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L119**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L120**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L121**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L122**: Declares function or method \`push_back\`. / 声明函数或方法 \`push_back\`。

### Lines 123-131 / 第 123-131 行

```cpp
 123:   newRelocs->push_back(minuendReloc);
 124: }
 125: 
 126: void EhRelocator::makePcRel(uint64_t off,
 127:                             PointerUnion<Symbol *, InputSection *> target,
 128:                             uint8_t length) {
 129:   createSubtraction(isec->symbols[0], target, off, length, &newRelocs);
 130: }
 131: 
```

- **L123**: Declares function or method \`push_back\`. / 声明函数或方法 \`push_back\`。
- **L124**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L125**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L126**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L127**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L128**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L129**: Declares function or method \`createSubtraction\`. / 声明函数或方法 \`createSubtraction\`。
- **L130**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L131**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 132-140 / 第 132-140 行

```cpp
 132: void EhRelocator::makeNegativePcRel(
 133:     uint64_t off, PointerUnion<Symbol *, InputSection *> target,
 134:     uint8_t length) {
 135:   createSubtraction</*Invert=*/true>(isec, target, off, length, &newRelocs);
 136: }
 137: 
 138: void EhRelocator::commit() {
 139:   isec->relocs.insert(isec->relocs.end(), newRelocs.begin(), newRelocs.end());
 140: }
```

- **L132**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L133**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L134**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L135**: Declares function or method \`true>\`. / 声明函数或方法 \`true>\`。
- **L136**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L137**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L138**: Defines function or method \`commit\`. / 定义函数或方法 \`commit\`。
- **L139**: Declares function or method \`insert\`. / 声明函数或方法 \`insert\`。
- **L140**: Closes the current scope or body. / 关闭当前作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: Implements Mach-O linker logic for symbol graphs, address assignment, and output-file construction. / 实现 Mach-O 链接器的符号图、地址分配以及输出文件构建逻辑。
- **Scale / 规模**: 140 lines, 5 direct includes, 0 named types, and 21 detected routines. / 共 140 行，含 5 个直接包含、0 个具名类型、21 个检测到的例程。
- **Mach-O linking / Mach-O 链接**: The code manages Mach-O atoms, symbols, segments, or output-file construction. / 该代码管理 Mach-O 原子、符号、段或输出文件构建。
- **DWARF handling / DWARF 处理**: The file reads, writes, links, or interprets DWARF debug information. / 该文件读取、写出、链接或解释 DWARF 调试信息。
- **Relocation processing / 重定位处理**: The implementation reasons about relocation records, fixups, or address adjustments. / 该实现处理重定位记录、修正项或地址调整。
- **Symbol management / 符号管理**: The code tracks declarations, references, visibility, or resolution of symbols. / 该代码跟踪符号的声明、引用、可见性或解析过程。
- **Section/segment layout / 节/段布局**: The file manipulates input or output sections, segments, or related layout state. / 该文件操作输入/输出节、段或相关布局状态。

## Dependencies / 依赖关系

- **LLVM / LLVM**: `llvm/BinaryFormat/Dwarf.h`, `llvm/Support/Endian.h`.
- **lld / lld**: `lld/Common/ErrorHandler.h`.
- **System or local / 系统或本地**: `EhFrame.h`, `InputFiles.h`.
- **Header roles / 头文件角色**: standard-library or local support header / 标准库或本地支持头文件 (2), lld shared linker infrastructure / lld 共享链接基础设施 (1), binary-format constants and helpers / 二进制格式常量与辅助工具 (1), support-library helpers / Support 库辅助功能 (1).
- **Visible routines / 可见例程**: `readLength`, `failOn`, `read32le`, `read64le`, `skipValidLength`, `readByte`, `readU32`, `readPointer`, `assert`, `readString`, `data`, `strnlen`.
