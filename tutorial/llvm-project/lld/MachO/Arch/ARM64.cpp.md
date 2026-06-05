# ARM64.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lld/MachO/Arch/ARM64.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: Implements Mach-O linker logic for symbol graphs, address assignment, and output-file construction.
- **Purpose (CN) / 用途（中文）**: 实现 Mach-O 链接器的符号图、地址分配以及输出文件构建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```cpp
   1: //===- ARM64.cpp ----------------------------------------------------------===//
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

### Lines 9-18 / 第 9-18 行

```cpp
   9: #include "Arch/ARM64Common.h"
  10: #include "InputFiles.h"
  11: #include "Symbols.h"
  12: #include "SyntheticSections.h"
  13: #include "Target.h"
  14: 
  15: #include "lld/Common/ErrorHandler.h"
  16: #include "mach-o/compact_unwind_encoding.h"
  17: #include "llvm/BinaryFormat/MachO.h"
  18: 
```

- **L9**: Includes \`Arch/ARM64Common.h\` so this file can use declarations from that header. / 引入 \`Arch/ARM64Common.h\`，使当前文件能够使用该头文件中的声明。
- **L10**: Includes \`InputFiles.h\` so this file can use declarations from that header. / 引入 \`InputFiles.h\`，使当前文件能够使用该头文件中的声明。
- **L11**: Includes \`Symbols.h\` so this file can use declarations from that header. / 引入 \`Symbols.h\`，使当前文件能够使用该头文件中的声明。
- **L12**: Includes \`SyntheticSections.h\` so this file can use declarations from that header. / 引入 \`SyntheticSections.h\`，使当前文件能够使用该头文件中的声明。
- **L13**: Includes \`Target.h\` so this file can use declarations from that header. / 引入 \`Target.h\`，使当前文件能够使用该头文件中的声明。
- **L14**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L15**: Includes \`lld/Common/ErrorHandler.h\` so this file can use declarations from that header. / 引入 \`lld/Common/ErrorHandler.h\`，使当前文件能够使用该头文件中的声明。
- **L16**: Includes \`mach-o/compact_unwind_encoding.h\` so this file can use declarations from that header. / 引入 \`mach-o/compact_unwind_encoding.h\`，使当前文件能够使用该头文件中的声明。
- **L17**: Includes \`llvm/BinaryFormat/MachO.h\` so this file can use declarations from that header. / 引入 \`llvm/BinaryFormat/MachO.h\`，使当前文件能够使用该头文件中的声明。
- **L18**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 19-25 / 第 19-25 行

```cpp
  19: using namespace llvm;
  20: using namespace llvm::MachO;
  21: using namespace lld;
  22: using namespace lld::macho;
  23: 
  24: namespace {
  25: 
```

- **L19**: Imports namespace \`llvm\` into the current scope for shorter symbol references. / 将命名空间 \`llvm\` 导入当前作用域，以便更简洁地引用符号。
- **L20**: Imports namespace \`llvm::MachO\` into the current scope for shorter symbol references. / 将命名空间 \`llvm::MachO\` 导入当前作用域，以便更简洁地引用符号。
- **L21**: Imports namespace \`lld\` into the current scope for shorter symbol references. / 将命名空间 \`lld\` 导入当前作用域，以便更简洁地引用符号。
- **L22**: Imports namespace \`lld::macho\` into the current scope for shorter symbol references. / 将命名空间 \`lld::macho\` 导入当前作用域，以便更简洁地引用符号。
- **L23**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L24**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L25**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 26-32 / 第 26-32 行

```cpp
  26: struct ARM64 : ARM64Common {
  27:   ARM64();
  28:   void writeStub(uint8_t *buf, const Symbol &, uint64_t) const override;
  29:   void writeStubHelperHeader(uint8_t *buf) const override;
  30:   void writeStubHelperEntry(uint8_t *buf, const Symbol &,
  31:                             uint64_t entryAddr) const override;
  32: 
```

- **L26**: Begins the declaration of struct \`ARM64\`. / 开始声明 struct \`ARM64\`。
- **L27**: Invokes a macro that expands subsystem-specific boilerplate or registration logic. / 调用一个宏，以展开子系统特定的样板代码或注册逻辑。
- **L28**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L29**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L30**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L31**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L32**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 33-44 / 第 33-44 行

```cpp
  33:   void writeObjCMsgSendStub(uint8_t *buf, Symbol *sym, uint64_t stubsAddr,
  34:                             uint64_t &stubOffset, uint64_t selrefVA,
  35:                             Symbol *objcMsgSend) const override;
  36:   void populateThunk(InputSection *thunk, Symbol *funcSym,
  37:                      int64_t addend) override;
  38: 
  39:   void initICFSafeThunkBody(InputSection *thunk,
  40:                             Symbol *targetSym) const override;
  41:   Symbol *getThunkBranchTarget(InputSection *thunk) const override;
  42:   uint32_t getICFSafeThunkSize() const override;
  43: };
  44: 
```

- **L33**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L34**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L35**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L36**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L37**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L38**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L39**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L40**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L41**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L42**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L43**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L44**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 45-58 / 第 45-58 行

```cpp
  45: } // namespace
  46: 
  47: // Random notes on reloc types:
  48: // ADDEND always pairs with BRANCH26, PAGE21, or PAGEOFF12
  49: // POINTER_TO_GOT: ld64 supports a 4-byte pc-relative form as well as an 8-byte
  50: // absolute version of this relocation. The semantics of the absolute relocation
  51: // are weird -- it results in the value of the GOT slot being written, instead
  52: // of the address. Let's not support it unless we find a real-world use case.
  53: static constexpr std::array<RelocAttrs, 11> relocAttrsArray{{
  54: #define B(x) RelocAttrBits::x
  55:     {"UNSIGNED",
  56:      B(UNSIGNED) | B(ABSOLUTE) | B(EXTERN) | B(LOCAL) | B(BYTE4) | B(BYTE8)},
  57:     {"SUBTRACTOR", B(SUBTRAHEND) | B(EXTERN) | B(BYTE4) | B(BYTE8)},
  58:     {"BRANCH26", B(PCREL) | B(EXTERN) | B(BRANCH) | B(BYTE4)},
```

- **L45**: Closes a named namespace scope. / 关闭一个具名命名空间作用域。
- **L46**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L47**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L48**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L49**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L50**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L51**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L52**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L53**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L54**: Defines macro \`B(x)\` for conditional compilation or textual reuse. / 定义宏 \`B(x)\`，供条件编译或文本复用使用。
- **L55**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L56**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L57**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L58**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。

### Lines 59-71 / 第 59-71 行

```cpp
  59:     {"PAGE21", B(PCREL) | B(EXTERN) | B(BYTE4)},
  60:     {"PAGEOFF12", B(ABSOLUTE) | B(EXTERN) | B(BYTE4)},
  61:     {"GOT_LOAD_PAGE21", B(PCREL) | B(EXTERN) | B(GOT) | B(BYTE4)},
  62:     {"GOT_LOAD_PAGEOFF12",
  63:      B(ABSOLUTE) | B(EXTERN) | B(GOT) | B(LOAD) | B(BYTE4)},
  64:     {"POINTER_TO_GOT", B(PCREL) | B(EXTERN) | B(GOT) | B(POINTER) | B(BYTE4)},
  65:     {"TLVP_LOAD_PAGE21", B(PCREL) | B(EXTERN) | B(TLV) | B(BYTE4)},
  66:     {"TLVP_LOAD_PAGEOFF12",
  67:      B(ABSOLUTE) | B(EXTERN) | B(TLV) | B(LOAD) | B(BYTE4)},
  68:     {"ADDEND", B(ADDEND)},
  69: #undef B
  70: }};
  71: 
```

- **L59**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L60**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L61**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L62**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L63**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L64**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L65**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L66**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L67**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L68**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L69**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L70**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L71**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 72-82 / 第 72-82 行

```cpp
  72: static constexpr uint32_t stubCode[] = {
  73:     0x90000010, // 00: adrp  x16, __la_symbol_ptr@page
  74:     0xf9400210, // 04: ldr   x16, [x16, __la_symbol_ptr@pageoff]
  75:     0xd61f0200, // 08: br    x16
  76: };
  77: 
  78: void ARM64::writeStub(uint8_t *buf8, const Symbol &sym,
  79:                       uint64_t pointerVA) const {
  80:   ::writeStub(buf8, stubCode, sym, pointerVA);
  81: }
  82: 
```

- **L72**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L73**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L74**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L75**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L76**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L77**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L78**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L79**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L80**: Declares function or method \`writeStub\`. / 声明函数或方法 \`writeStub\`。
- **L81**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L82**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 83-91 / 第 83-91 行

```cpp
  83: static constexpr uint32_t stubHelperHeaderCode[] = {
  84:     0x90000011, // 00: adrp  x17, _dyld_private@page
  85:     0x91000231, // 04: add   x17, x17, _dyld_private@pageoff
  86:     0xa9bf47f0, // 08: stp   x16/x17, [sp, #-16]!
  87:     0x90000010, // 0c: adrp  x16, dyld_stub_binder@page
  88:     0xf9400210, // 10: ldr   x16, [x16, dyld_stub_binder@pageoff]
  89:     0xd61f0200, // 14: br    x16
  90: };
  91: 
```

- **L83**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L84**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L85**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L86**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L87**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L88**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L89**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L90**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L91**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 92-101 / 第 92-101 行

```cpp
  92: void ARM64::writeStubHelperHeader(uint8_t *buf8) const {
  93:   ::writeStubHelperHeader<LP64>(buf8, stubHelperHeaderCode);
  94: }
  95: 
  96: static constexpr uint32_t stubHelperEntryCode[] = {
  97:     0x18000050, // 00: ldr  w16, l0
  98:     0x14000000, // 04: b    stubHelperHeader
  99:     0x00000000, // 08: l0: .long 0
 100: };
 101: 
```

- **L92**: Defines function or method \`writeStubHelperHeader\`. / 定义函数或方法 \`writeStubHelperHeader\`。
- **L93**: Declares function or method \`writeStubHelperHeader\`. / 声明函数或方法 \`writeStubHelperHeader\`。
- **L94**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L95**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L96**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L97**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L98**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L99**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L100**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L101**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 102-115 / 第 102-115 行

```cpp
 102: void ARM64::writeStubHelperEntry(uint8_t *buf8, const Symbol &sym,
 103:                                  uint64_t entryVA) const {
 104:   ::writeStubHelperEntry(buf8, stubHelperEntryCode, sym, entryVA);
 105: }
 106: 
 107: static constexpr uint32_t objcStubsFastCode[] = {
 108:     0x90000001, // adrp  x1, __objc_selrefs@page
 109:     0xf9400021, // ldr   x1, [x1, @selector("foo")@pageoff]
 110:     0x90000010, // adrp  x16, _got@page
 111:     0xf9400210, // ldr   x16, [x16, _objc_msgSend@pageoff]
 112:     0xd61f0200, // br    x16
 113:     0xd4200020, // brk   #0x1
 114:     0xd4200020, // brk   #0x1
 115:     0xd4200020, // brk   #0x1
```

- **L102**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L103**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L104**: Declares function or method \`writeStubHelperEntry\`. / 声明函数或方法 \`writeStubHelperEntry\`。
- **L105**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L106**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L107**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L108**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L109**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L110**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L111**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L112**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L113**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L114**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L115**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 116-123 / 第 116-123 行

```cpp
 116: };
 117: 
 118: static constexpr uint32_t objcStubsSmallCode[] = {
 119:     0x90000001, // adrp  x1, __objc_selrefs@page
 120:     0xf9400021, // ldr   x1, [x1, @selector("foo")@pageoff]
 121:     0x14000000, // b     _objc_msgSend
 122: };
 123: 
```

- **L116**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L117**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L118**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L119**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L120**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L121**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L122**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L123**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 124-130 / 第 124-130 行

```cpp
 124: void ARM64::writeObjCMsgSendStub(uint8_t *buf, Symbol *sym, uint64_t stubsAddr,
 125:                                  uint64_t &stubOffset, uint64_t selrefVA,
 126:                                  Symbol *objcMsgSend) const {
 127:   uint64_t objcMsgSendAddr;
 128:   uint64_t objcStubSize;
 129:   uint64_t objcMsgSendIndex;
 130: 
```

- **L124**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L125**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L126**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L127**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L128**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L129**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L130**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 131-144 / 第 131-144 行

```cpp
 131:   if (config->objcStubsMode == ObjCStubsMode::fast) {
 132:     objcStubSize = target->objcStubsFastSize;
 133:     objcMsgSendAddr = in.got->addr;
 134:     objcMsgSendIndex = objcMsgSend->gotIndex;
 135:     ::writeObjCMsgSendFastStub<LP64>(buf, objcStubsFastCode, sym, stubsAddr,
 136:                                      stubOffset, selrefVA, objcMsgSendAddr,
 137:                                      objcMsgSendIndex);
 138:   } else {
 139:     assert(config->objcStubsMode == ObjCStubsMode::small);
 140:     objcStubSize = target->objcStubsSmallSize;
 141:     if (auto *d = dyn_cast<Defined>(objcMsgSend)) {
 142:       objcMsgSendAddr = d->getVA();
 143:       objcMsgSendIndex = 0;
 144:     } else {
```

- **L131**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L132**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L133**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L134**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L135**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L136**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L137**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L138**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L139**: Declares function or method \`assert\`. / 声明函数或方法 \`assert\`。
- **L140**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L141**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L142**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L143**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L144**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 145-154 / 第 145-154 行

```cpp
 145:       objcMsgSendAddr = in.stubs->addr;
 146:       objcMsgSendIndex = objcMsgSend->stubsIndex;
 147:     }
 148:     ::writeObjCMsgSendSmallStub<LP64>(buf, objcStubsSmallCode, sym, stubsAddr,
 149:                                       stubOffset, selrefVA, objcMsgSendAddr,
 150:                                       objcMsgSendIndex);
 151:   }
 152:   stubOffset += objcStubSize;
 153: }
 154: 
```

- **L145**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L146**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L147**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L148**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L149**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L150**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L151**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L152**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L153**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L154**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 155-163 / 第 155-163 行

```cpp
 155: // A thunk is the relaxed variation of stubCode. We don't need the
 156: // extra indirection through a lazy pointer because the target address
 157: // is known at link time.
 158: static constexpr uint32_t thunkCode[] = {
 159:     0x90000010, // 00: adrp  x16, <thunk.ptr>@page
 160:     0x91000210, // 04: add   x16, [x16,<thunk.ptr>@pageoff]
 161:     0xd61f0200, // 08: br    x16
 162: };
 163: 
```

- **L155**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L156**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L157**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L158**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L159**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L160**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L161**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L162**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L163**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 164-177 / 第 164-177 行

```cpp
 164: void ARM64::populateThunk(InputSection *thunk, Symbol *funcSym,
 165:                           int64_t addend) {
 166:   thunk->align = 4;
 167:   thunk->data = {reinterpret_cast<const uint8_t *>(thunkCode),
 168:                  sizeof(thunkCode)};
 169:   thunk->relocs.emplace_back(/*type=*/ARM64_RELOC_PAGEOFF12,
 170:                              /*pcrel=*/false, /*length=*/2,
 171:                              /*offset=*/4, /*addend=*/addend,
 172:                              /*referent=*/funcSym);
 173:   thunk->relocs.emplace_back(/*type=*/ARM64_RELOC_PAGE21,
 174:                              /*pcrel=*/true, /*length=*/2,
 175:                              /*offset=*/0, /*addend=*/addend,
 176:                              /*referent=*/funcSym);
 177: }
```

- **L164**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L165**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L166**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L167**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L168**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L169**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L170**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L171**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L172**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L173**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L174**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L175**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L176**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L177**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 178-188 / 第 178-188 行

```cpp
 178: // Just a single direct branch to the target function.
 179: static constexpr uint32_t icfSafeThunkCode[] = {
 180:     0x14000000, // 08: b    target
 181: };
 182: 
 183: void ARM64::initICFSafeThunkBody(InputSection *thunk, Symbol *targetSym) const {
 184:   // The base data here will not be itself modified, we'll just be adding a
 185:   // reloc below. So we can directly use the constexpr above as the data.
 186:   thunk->data = {reinterpret_cast<const uint8_t *>(icfSafeThunkCode),
 187:                  sizeof(icfSafeThunkCode)};
 188: 
```

- **L178**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L179**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L180**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L181**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L182**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L183**: Defines function or method \`initICFSafeThunkBody\`. / 定义函数或方法 \`initICFSafeThunkBody\`。
- **L184**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L185**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L186**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L187**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L188**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 189-201 / 第 189-201 行

```cpp
 189:   thunk->relocs.emplace_back(/*type=*/ARM64_RELOC_BRANCH26,
 190:                              /*pcrel=*/true, /*length=*/2,
 191:                              /*offset=*/0, /*addend=*/0,
 192:                              /*referent=*/targetSym);
 193: }
 194: 
 195: Symbol *ARM64::getThunkBranchTarget(InputSection *thunk) const {
 196:   assert(thunk->relocs.size() == 1 &&
 197:          "expected a single reloc on ARM64 ICF thunk");
 198:   auto &reloc = thunk->relocs[0];
 199:   assert(isa<Symbol *>(reloc.referent) &&
 200:          "ARM64 thunk reloc is expected to point to a Symbol");
 201: 
```

- **L189**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L190**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L191**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L192**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L193**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L194**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L195**: Defines function or method \`getThunkBranchTarget\`. / 定义函数或方法 \`getThunkBranchTarget\`。
- **L196**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L197**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L198**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L199**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L200**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L201**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 202-210 / 第 202-210 行

```cpp
 202:   return cast<Symbol *>(reloc.referent);
 203: }
 204: 
 205: uint32_t ARM64::getICFSafeThunkSize() const { return sizeof(icfSafeThunkCode); }
 206: 
 207: ARM64::ARM64() : ARM64Common(LP64()) {
 208:   cpuType = CPU_TYPE_ARM64;
 209:   cpuSubtype = CPU_SUBTYPE_ARM64_ALL;
 210: 
```

- **L202**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L203**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L204**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L205**: Defines function or method \`getICFSafeThunkSize\`. / 定义函数或方法 \`getICFSafeThunkSize\`。
- **L206**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L207**: Defines function or method \`ARM64\`. / 定义函数或方法 \`ARM64\`。
- **L208**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L209**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L210**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 211-218 / 第 211-218 行

```cpp
 211:   stubSize = sizeof(stubCode);
 212:   thunkSize = sizeof(thunkCode);
 213: 
 214:   objcStubsFastSize = sizeof(objcStubsFastCode);
 215:   objcStubsFastAlignment = 32;
 216:   objcStubsSmallSize = sizeof(objcStubsSmallCode);
 217:   objcStubsSmallAlignment = 4;
 218: 
```

- **L211**: Declares function or method \`sizeof\`. / 声明函数或方法 \`sizeof\`。
- **L212**: Declares function or method \`sizeof\`. / 声明函数或方法 \`sizeof\`。
- **L213**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L214**: Declares function or method \`sizeof\`. / 声明函数或方法 \`sizeof\`。
- **L215**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L216**: Declares function or method \`sizeof\`. / 声明函数或方法 \`sizeof\`。
- **L217**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L218**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 219-228 / 第 219-228 行

```cpp
 219:   // Branch immediate is two's complement 26 bits, which is implicitly
 220:   // multiplied by 4 (since all functions are 4-aligned: The branch range
 221:   // is -4*(2**(26-1))..4*(2**(26-1) - 1).
 222:   backwardBranchRange = 128 * 1024 * 1024;
 223:   forwardBranchRange = backwardBranchRange - 4;
 224: 
 225:   modeDwarfEncoding = UNWIND_ARM64_MODE_DWARF;
 226:   subtractorRelocType = ARM64_RELOC_SUBTRACTOR;
 227:   unsignedRelocType = ARM64_RELOC_UNSIGNED;
 228: 
```

- **L219**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L220**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L221**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L222**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L223**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L224**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L225**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L226**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L227**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L228**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 229-238 / 第 229-238 行

```cpp
 229:   stubHelperHeaderSize = sizeof(stubHelperHeaderCode);
 230:   stubHelperEntrySize = sizeof(stubHelperEntryCode);
 231: 
 232:   relocAttrs = {relocAttrsArray.data(), relocAttrsArray.size()};
 233: }
 234: 
 235: TargetInfo *macho::createARM64TargetInfo() {
 236:   static ARM64 t;
 237:   return &t;
 238: }
```

- **L229**: Declares function or method \`sizeof\`. / 声明函数或方法 \`sizeof\`。
- **L230**: Declares function or method \`sizeof\`. / 声明函数或方法 \`sizeof\`。
- **L231**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L232**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L233**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L234**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L235**: Defines function or method \`createARM64TargetInfo\`. / 定义函数或方法 \`createARM64TargetInfo\`。
- **L236**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L237**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L238**: Closes the current scope or body. / 关闭当前作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: Implements Mach-O linker logic for symbol graphs, address assignment, and output-file construction. / 实现 Mach-O 链接器的符号图、地址分配以及输出文件构建逻辑。
- **Scale / 规模**: 238 lines, 8 direct includes, 1 named types, and 10 detected routines. / 共 238 行，含 8 个直接包含、1 个具名类型、10 个检测到的例程。
- **Mach-O linking / Mach-O 链接**: The code manages Mach-O atoms, symbols, segments, or output-file construction. / 该代码管理 Mach-O 原子、符号、段或输出文件构建。
- **DWARF handling / DWARF 处理**: The file reads, writes, links, or interprets DWARF debug information. / 该文件读取、写出、链接或解释 DWARF 调试信息。
- **Relocation processing / 重定位处理**: The implementation reasons about relocation records, fixups, or address adjustments. / 该实现处理重定位记录、修正项或地址调整。
- **Symbol management / 符号管理**: The code tracks declarations, references, visibility, or resolution of symbols. / 该代码跟踪符号的声明、引用、可见性或解析过程。
- **Section/segment layout / 节/段布局**: The file manipulates input or output sections, segments, or related layout state. / 该文件操作输入/输出节、段或相关布局状态。
- **Rich type modeling / 丰富的类型建模**: The file introduces or manipulates named types that capture subsystem state. / 该文件引入或操作用于表达子系统状态的具名类型。

## Dependencies / 依赖关系

- **LLVM / LLVM**: `llvm/BinaryFormat/MachO.h`.
- **lld / lld**: `lld/Common/ErrorHandler.h`.
- **System or local / 系统或本地**: `Arch/ARM64Common.h`, `InputFiles.h`, `Symbols.h`, `SyntheticSections.h`, `Target.h`, `mach-o/compact_unwind_encoding.h`.
- **Header roles / 头文件角色**: standard-library or local support header / 标准库或本地支持头文件 (4), supporting declarations / 辅助声明 (2), lld shared linker infrastructure / lld 共享链接基础设施 (1), binary-format constants and helpers / 二进制格式常量与辅助工具 (1).
- **Core types / 核心类型**: `ARM64`.
- **Visible routines / 可见例程**: `ARM64`, `writeStub`, `writeStubHelperHeader`, `writeStubHelperEntry`, `assert`, `getVA`, `initICFSafeThunkBody`, `getThunkBranchTarget`, `getICFSafeThunkSize`, `createARM64TargetInfo`.
