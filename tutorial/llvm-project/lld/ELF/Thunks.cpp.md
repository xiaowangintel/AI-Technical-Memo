# Thunks.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lld/ELF/Thunks.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: This file contains Thunk subclasses.
- **Purpose (CN) / 用途（中文）**: 实现 ELF 链接器逻辑，例如符号解析、节布局、重定位以及驱动行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22 / 第 1-22 行

```cpp
   1: //===- Thunks.cpp --------------------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===---------------------------------------------------------------------===//
   8: //
   9: // This file contains Thunk subclasses.
  10: //
  11: // A thunk is a small piece of code written after an input section
  12: // which is used to jump between "incompatible" functions
  13: // such as MIPS PIC and non-PIC or ARM non-Thumb and Thumb functions.
  14: //
  15: // If a jump target is too far and its address doesn't fit to a
  16: // short jump instruction, we need to create a thunk too, but we
  17: // haven't supported it yet.
  18: //
  19: // i386 and x86-64 don't need thunks.
  20: //
  21: //===---------------------------------------------------------------------===//
  22: 
```

- **L1**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L2**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L3**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L4**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L5**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L6**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L7**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L8**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L9**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L10**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L11**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L12**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L13**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L14**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L15**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L16**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L17**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L18**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L19**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L20**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L21**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L22**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 23-38 / 第 23-38 行

```cpp
  23: #include "Thunks.h"
  24: #include "Config.h"
  25: #include "InputFiles.h"
  26: #include "InputSection.h"
  27: #include "OutputSections.h"
  28: #include "Symbols.h"
  29: #include "SyntheticSections.h"
  30: #include "Target.h"
  31: #include "lld/Common/CommonLinkerContext.h"
  32: #include "llvm/BinaryFormat/ELF.h"
  33: #include "llvm/Support/Casting.h"
  34: #include "llvm/Support/ErrorHandling.h"
  35: #include "llvm/Support/MathExtras.h"
  36: #include <cstdint>
  37: #include <cstring>
  38: 
```

- **L23**: Includes \`Thunks.h\` so this file can use declarations from that header. / 引入 \`Thunks.h\`，使当前文件能够使用该头文件中的声明。
- **L24**: Includes \`Config.h\` so this file can use declarations from that header. / 引入 \`Config.h\`，使当前文件能够使用该头文件中的声明。
- **L25**: Includes \`InputFiles.h\` so this file can use declarations from that header. / 引入 \`InputFiles.h\`，使当前文件能够使用该头文件中的声明。
- **L26**: Includes \`InputSection.h\` so this file can use declarations from that header. / 引入 \`InputSection.h\`，使当前文件能够使用该头文件中的声明。
- **L27**: Includes \`OutputSections.h\` so this file can use declarations from that header. / 引入 \`OutputSections.h\`，使当前文件能够使用该头文件中的声明。
- **L28**: Includes \`Symbols.h\` so this file can use declarations from that header. / 引入 \`Symbols.h\`，使当前文件能够使用该头文件中的声明。
- **L29**: Includes \`SyntheticSections.h\` so this file can use declarations from that header. / 引入 \`SyntheticSections.h\`，使当前文件能够使用该头文件中的声明。
- **L30**: Includes \`Target.h\` so this file can use declarations from that header. / 引入 \`Target.h\`，使当前文件能够使用该头文件中的声明。
- **L31**: Includes \`lld/Common/CommonLinkerContext.h\` so this file can use declarations from that header. / 引入 \`lld/Common/CommonLinkerContext.h\`，使当前文件能够使用该头文件中的声明。
- **L32**: Includes \`llvm/BinaryFormat/ELF.h\` so this file can use declarations from that header. / 引入 \`llvm/BinaryFormat/ELF.h\`，使当前文件能够使用该头文件中的声明。
- **L33**: Includes \`llvm/Support/Casting.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/Casting.h\`，使当前文件能够使用该头文件中的声明。
- **L34**: Includes \`llvm/Support/ErrorHandling.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/ErrorHandling.h\`，使当前文件能够使用该头文件中的声明。
- **L35**: Includes \`llvm/Support/MathExtras.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/MathExtras.h\`，使当前文件能够使用该头文件中的声明。
- **L36**: Includes \`cstdint\` so this file can use declarations from that header. / 引入 \`cstdint\`，使当前文件能够使用该头文件中的声明。
- **L37**: Includes \`cstring\` so this file can use declarations from that header. / 引入 \`cstring\`，使当前文件能够使用该头文件中的声明。
- **L38**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 39-58 / 第 39-58 行

```cpp
  39: using namespace llvm;
  40: using namespace llvm::object;
  41: using namespace llvm::ELF;
  42: using namespace lld;
  43: using namespace lld::elf;
  44: 
  45: namespace {
  46: 
  47: // Base class for AArch64 thunks.
  48: //
  49: // An AArch64 thunk may be either short or long. A short thunk is simply a
  50: // branch (B) instruction, and it may be used to call AArch64 functions when the
  51: // distance from the thunk to the target is less than 128MB. Long thunks can
  52: // branch to any virtual address and they are implemented in the derived
  53: // classes. This class tries to create a short thunk if the target is in range,
  54: // otherwise it creates a long thunk. When BTI is enabled indirect branches
  55: // must land on a BTI instruction. If the destination does not have a BTI
  56: // instruction mayNeedLandingPad is set to true and Thunk::landingPad points
  57: // to an alternative entry point with a BTI.
  58: class AArch64Thunk : public Thunk {
```

- **L39**: Imports namespace \`llvm\` into the current scope for shorter symbol references. / 将命名空间 \`llvm\` 导入当前作用域，以便更简洁地引用符号。
- **L40**: Imports namespace \`llvm::object\` into the current scope for shorter symbol references. / 将命名空间 \`llvm::object\` 导入当前作用域，以便更简洁地引用符号。
- **L41**: Imports namespace \`llvm::ELF\` into the current scope for shorter symbol references. / 将命名空间 \`llvm::ELF\` 导入当前作用域，以便更简洁地引用符号。
- **L42**: Imports namespace \`lld\` into the current scope for shorter symbol references. / 将命名空间 \`lld\` 导入当前作用域，以便更简洁地引用符号。
- **L43**: Imports namespace \`lld::elf\` into the current scope for shorter symbol references. / 将命名空间 \`lld::elf\` 导入当前作用域，以便更简洁地引用符号。
- **L44**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L45**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L46**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L47**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L48**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L49**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L50**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L51**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L52**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L53**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L54**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L55**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L56**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L57**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L58**: Begins the declaration of class \`AArch64Thunk\`. / 开始声明 class \`AArch64Thunk\`。

### Lines 59-78 / 第 59-78 行

```cpp
  59: public:
  60:   AArch64Thunk(Ctx &ctx, Symbol &dest, int64_t addend, bool mayNeedLandingPad)
  61:       : Thunk(ctx, dest, addend), mayNeedLandingPad(mayNeedLandingPad) {}
  62:   bool getMayUseShortThunk();
  63:   void writeTo(uint8_t *buf) override;
  64:   bool needsSyntheticLandingPad() override;
  65: 
  66: protected:
  67:   bool mayNeedLandingPad;
  68: 
  69: private:
  70:   bool mayUseShortThunk = true;
  71:   virtual void writeLong(uint8_t *buf) = 0;
  72:   // A thunk may be written out as a short or long, and we may not know which
  73:   // type at thunk creation time. In some thunk implementations the long thunk
  74:   // has additional mapping symbols. Thus function can be overridden to add
  75:   // these additional mapping symbols.
  76:   virtual void addLongMapSyms() {}
  77: };
  78: 
```

- **L59**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L60**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L61**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L62**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L63**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L64**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L65**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L66**: Switches following class members to \`protected\` access control. / 将后续类成员的访问控制切换为 \`protected\`。
- **L67**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L68**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L69**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L70**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L71**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L72**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L73**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L74**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L75**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L76**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L77**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L78**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 79-93 / 第 79-93 行

```cpp
  79: // AArch64 long range Thunks.
  80: class AArch64ABSLongThunk final : public AArch64Thunk {
  81: public:
  82:   AArch64ABSLongThunk(Ctx &ctx, Symbol &dest, int64_t addend,
  83:                       bool mayNeedLandingPad)
  84:       : AArch64Thunk(ctx, dest, addend, mayNeedLandingPad) {}
  85:   uint32_t size() override { return getMayUseShortThunk() ? 4 : 16; }
  86:   void addSymbols(ThunkSection &isec) override;
  87: 
  88: private:
  89:   void writeLong(uint8_t *buf) override;
  90:   void addLongMapSyms() override;
  91:   ThunkSection *tsec = nullptr;
  92: };
  93: 
```

- **L79**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L80**: Begins the declaration of class \`AArch64ABSLongThunk\`. / 开始声明 class \`AArch64ABSLongThunk\`。
- **L81**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L82**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L83**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L84**: Defines function or method \`AArch64Thunk\`. / 定义函数或方法 \`AArch64Thunk\`。
- **L85**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L86**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L87**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L88**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L89**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L90**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L91**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L92**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L93**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 94-108 / 第 94-108 行

```cpp
  94: // AArch64 long range Thunks compatible with execute-only code.
  95: class AArch64ABSXOLongThunk final : public AArch64Thunk {
  96: public:
  97:   AArch64ABSXOLongThunk(Ctx &ctx, Symbol &dest, int64_t addend,
  98:                         bool mayNeedLandingPad)
  99:       : AArch64Thunk(ctx, dest, addend, mayNeedLandingPad) {}
 100:   uint32_t size() override { return getMayUseShortThunk() ? 4 : 20; }
 101:   void addSymbols(ThunkSection &sec) override;
 102: 
 103: private:
 104:   void writeLong(uint8_t *buf) override;
 105: };
 106: 
 107: class AArch64ADRPThunk final : public AArch64Thunk {
 108: public:
```

- **L94**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L95**: Begins the declaration of class \`AArch64ABSXOLongThunk\`. / 开始声明 class \`AArch64ABSXOLongThunk\`。
- **L96**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L97**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L98**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L99**: Defines function or method \`AArch64Thunk\`. / 定义函数或方法 \`AArch64Thunk\`。
- **L100**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L101**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L102**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L103**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L104**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L105**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L106**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L107**: Begins the declaration of class \`AArch64ADRPThunk\`. / 开始声明 class \`AArch64ADRPThunk\`。
- **L108**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。

### Lines 109-124 / 第 109-124 行

```cpp
 109:   AArch64ADRPThunk(Ctx &ctx, Symbol &dest, int64_t addend,
 110:                    bool mayNeedLandingPad)
 111:       : AArch64Thunk(ctx, dest, addend, mayNeedLandingPad) {}
 112:   uint32_t size() override { return getMayUseShortThunk() ? 4 : 12; }
 113:   void addSymbols(ThunkSection &isec) override;
 114: 
 115: private:
 116:   void writeLong(uint8_t *buf) override;
 117: };
 118: 
 119: // AArch64 BTI Landing Pad
 120: // When BTI is enabled indirect branches must land on a BTI
 121: // compatible instruction. When the destination does not have a
 122: // BTI compatible instruction a Thunk doing an indirect branch
 123: // targets a Landing Pad Thunk that direct branches to the target.
 124: class AArch64BTILandingPadThunk final : public Thunk {
```

- **L109**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L110**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L111**: Defines function or method \`AArch64Thunk\`. / 定义函数或方法 \`AArch64Thunk\`。
- **L112**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L113**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L114**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L115**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L116**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L117**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L118**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L119**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L120**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L121**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L122**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L123**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L124**: Begins the declaration of class \`AArch64BTILandingPadThunk\`. / 开始声明 class \`AArch64BTILandingPadThunk\`。

### Lines 125-147 / 第 125-147 行

```cpp
 125: public:
 126:   AArch64BTILandingPadThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 127:       : Thunk(ctx, dest, addend) {}
 128: 
 129:   uint32_t size() override { return getMayUseShortThunk() ? 4 : 8; }
 130:   void addSymbols(ThunkSection &isec) override;
 131:   void writeTo(uint8_t *buf) override;
 132: 
 133: private:
 134:   bool getMayUseShortThunk();
 135:   void writeLong(uint8_t *buf);
 136:   bool mayUseShortThunk = true;
 137: };
 138: 
 139: // Base class for ARM thunks.
 140: //
 141: // An ARM thunk may be either short or long. A short thunk is simply a branch
 142: // (B) instruction, and it may be used to call ARM functions when the distance
 143: // from the thunk to the target is less than 32MB. Long thunks can branch to any
 144: // virtual address and can switch between ARM and Thumb, and they are
 145: // implemented in the derived classes. This class tries to create a short thunk
 146: // if the target is in range, otherwise it creates a long thunk.
 147: class ARMThunk : public Thunk {
```

- **L125**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L126**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L127**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L128**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L129**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L130**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L131**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L132**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L133**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L134**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L135**: Declares function or method \`writeLong\`. / 声明函数或方法 \`writeLong\`。
- **L136**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L137**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L138**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L139**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L140**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L141**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L142**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L143**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L144**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L145**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L146**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L147**: Begins the declaration of class \`ARMThunk\`. / 开始声明 class \`ARMThunk\`。

### Lines 148-162 / 第 148-162 行

```cpp
 148: public:
 149:   ARMThunk(Ctx &ctx, Symbol &dest, int64_t addend) : Thunk(ctx, dest, addend) {}
 150: 
 151:   bool getMayUseShortThunk();
 152:   uint32_t size() override { return getMayUseShortThunk() ? 4 : sizeLong(); }
 153:   void writeTo(uint8_t *buf) override;
 154:   bool isCompatibleWith(const InputSection &isec,
 155:                         const Relocation &rel) const override;
 156: 
 157:   // Returns the size of a long thunk.
 158:   virtual uint32_t sizeLong() = 0;
 159: 
 160:   // Writes a long thunk to Buf.
 161:   virtual void writeLong(uint8_t *buf) = 0;
 162: 
```

- **L148**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L149**: Defines function or method \`ARMThunk\`. / 定义函数或方法 \`ARMThunk\`。
- **L150**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L151**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L152**: Defines function or method \`getMayUseShortThunk\`. / 定义函数或方法 \`getMayUseShortThunk\`。
- **L153**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L154**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L155**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L156**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L157**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L158**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L159**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L160**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L161**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L162**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 163-179 / 第 163-179 行

```cpp
 163: private:
 164:   // This field tracks whether all previously considered layouts would allow
 165:   // this thunk to be short. If we have ever needed a long thunk, we always
 166:   // create a long thunk, even if the thunk may be short given the current
 167:   // distance to the target. We do this because transitioning from long to short
 168:   // can create layout oscillations in certain corner cases which would prevent
 169:   // the layout from converging.
 170:   bool mayUseShortThunk = true;
 171:   // See comment in AArch64Thunk.
 172:   virtual void addLongMapSyms() {}
 173: };
 174: 
 175: // Base class for Thumb-2 thunks.
 176: //
 177: // This class is similar to ARMThunk, but it uses the Thumb-2 B.W instruction
 178: // which has a range of 16MB.
 179: class ThumbThunk : public Thunk {
```

- **L163**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L164**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L165**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L166**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L167**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L168**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L169**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L170**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L171**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L172**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L173**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L174**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L175**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L176**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L177**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L178**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L179**: Begins the declaration of class \`ThumbThunk\`. / 开始声明 class \`ThumbThunk\`。

### Lines 180-194 / 第 180-194 行

```cpp
 180: public:
 181:   ThumbThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 182:       : Thunk(ctx, dest, addend) {
 183:     alignment = 2;
 184:   }
 185: 
 186:   bool getMayUseShortThunk();
 187:   uint32_t size() override { return getMayUseShortThunk() ? 4 : sizeLong(); }
 188:   void writeTo(uint8_t *buf) override;
 189:   bool isCompatibleWith(const InputSection &isec,
 190:                         const Relocation &rel) const override;
 191: 
 192:   // Returns the size of a long thunk.
 193:   virtual uint32_t sizeLong() = 0;
 194: 
```

- **L180**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L181**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L182**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L183**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L184**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L185**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L186**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L187**: Defines function or method \`getMayUseShortThunk\`. / 定义函数或方法 \`getMayUseShortThunk\`。
- **L188**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L189**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L190**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L191**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L192**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L193**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L194**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 195-211 / 第 195-211 行

```cpp
 195:   // Writes a long thunk to Buf.
 196:   virtual void writeLong(uint8_t *buf) = 0;
 197: 
 198: private:
 199:   // See comment in ARMThunk above.
 200:   bool mayUseShortThunk = true;
 201:   // See comment in AArch64Thunk.
 202:   virtual void addLongMapSyms() {}
 203: };
 204: 
 205: // Specific ARM Thunk implementations. The naming convention is:
 206: // Source State, TargetState, Target Requirement, ABS or PI, Range
 207: class ARMV7ABSLongThunk final : public ARMThunk {
 208: public:
 209:   ARMV7ABSLongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 210:       : ARMThunk(ctx, dest, addend) {}
 211: 
```

- **L195**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L196**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L197**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L198**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L199**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L200**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L201**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L202**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L203**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L204**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L205**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L206**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L207**: Begins the declaration of class \`ARMV7ABSLongThunk\`. / 开始声明 class \`ARMV7ABSLongThunk\`。
- **L208**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L209**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L210**: Defines function or method \`ARMThunk\`. / 定义函数或方法 \`ARMThunk\`。
- **L211**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 212-226 / 第 212-226 行

```cpp
 212:   uint32_t sizeLong() override { return 12; }
 213:   void writeLong(uint8_t *buf) override;
 214:   void addSymbols(ThunkSection &isec) override;
 215: };
 216: 
 217: class ARMV7PILongThunk final : public ARMThunk {
 218: public:
 219:   ARMV7PILongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 220:       : ARMThunk(ctx, dest, addend) {}
 221: 
 222:   uint32_t sizeLong() override { return 16; }
 223:   void writeLong(uint8_t *buf) override;
 224:   void addSymbols(ThunkSection &isec) override;
 225: };
 226: 
```

- **L212**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L213**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L214**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L215**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L216**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L217**: Begins the declaration of class \`ARMV7PILongThunk\`. / 开始声明 class \`ARMV7PILongThunk\`。
- **L218**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L219**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L220**: Defines function or method \`ARMThunk\`. / 定义函数或方法 \`ARMThunk\`。
- **L221**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L222**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L223**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L224**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L225**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L226**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 227-241 / 第 227-241 行

```cpp
 227: class ThumbV7ABSLongThunk final : public ThumbThunk {
 228: public:
 229:   ThumbV7ABSLongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 230:       : ThumbThunk(ctx, dest, addend) {}
 231: 
 232:   uint32_t sizeLong() override { return 10; }
 233:   void writeLong(uint8_t *buf) override;
 234:   void addSymbols(ThunkSection &isec) override;
 235: };
 236: 
 237: class ThumbV7PILongThunk final : public ThumbThunk {
 238: public:
 239:   ThumbV7PILongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 240:       : ThumbThunk(ctx, dest, addend) {}
 241: 
```

- **L227**: Begins the declaration of class \`ThumbV7ABSLongThunk\`. / 开始声明 class \`ThumbV7ABSLongThunk\`。
- **L228**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L229**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L230**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L231**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L232**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L233**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L234**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L235**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L236**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L237**: Begins the declaration of class \`ThumbV7PILongThunk\`. / 开始声明 class \`ThumbV7PILongThunk\`。
- **L238**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L239**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L240**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L241**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 242-256 / 第 242-256 行

```cpp
 242:   uint32_t sizeLong() override { return 12; }
 243:   void writeLong(uint8_t *buf) override;
 244:   void addSymbols(ThunkSection &isec) override;
 245: };
 246: 
 247: // Implementations of Thunks for Arm v6-M. Only Thumb instructions are permitted
 248: class ThumbV6MABSLongThunk final : public ThumbThunk {
 249: public:
 250:   ThumbV6MABSLongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 251:       : ThumbThunk(ctx, dest, addend) {}
 252: 
 253:   uint32_t sizeLong() override { return 12; }
 254:   void writeLong(uint8_t *buf) override;
 255:   void addSymbols(ThunkSection &isec) override;
 256: 
```

- **L242**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L243**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L244**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L245**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L246**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L247**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L248**: Begins the declaration of class \`ThumbV6MABSLongThunk\`. / 开始声明 class \`ThumbV6MABSLongThunk\`。
- **L249**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L250**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L251**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L252**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L253**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L254**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L255**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L256**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 257-271 / 第 257-271 行

```cpp
 257: private:
 258:   void addLongMapSyms() override;
 259:   ThunkSection *tsec = nullptr;
 260: };
 261: 
 262: class ThumbV6MABSXOLongThunk final : public ThumbThunk {
 263: public:
 264:   ThumbV6MABSXOLongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 265:       : ThumbThunk(ctx, dest, addend) {}
 266: 
 267:   uint32_t sizeLong() override { return 20; }
 268:   void writeLong(uint8_t *buf) override;
 269:   void addSymbols(ThunkSection &isec) override;
 270: };
 271: 
```

- **L257**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L258**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L259**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L260**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L261**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L262**: Begins the declaration of class \`ThumbV6MABSXOLongThunk\`. / 开始声明 class \`ThumbV6MABSXOLongThunk\`。
- **L263**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L264**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L265**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L266**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L267**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L268**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L269**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L270**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L271**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 272-290 / 第 272-290 行

```cpp
 272: class ThumbV6MPILongThunk final : public ThumbThunk {
 273: public:
 274:   ThumbV6MPILongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 275:       : ThumbThunk(ctx, dest, addend) {}
 276: 
 277:   uint32_t sizeLong() override { return 16; }
 278:   void writeLong(uint8_t *buf) override;
 279:   void addSymbols(ThunkSection &isec) override;
 280: 
 281: private:
 282:   void addLongMapSyms() override;
 283:   ThunkSection *tsec = nullptr;
 284: };
 285: 
 286: // Architectures v4, v5 and v6 do not support the movt/movw instructions. v5 and
 287: // v6 support BLX to which BL instructions can be rewritten inline. There are no
 288: // Thumb entrypoints for v5 and v6 as there is no Thumb branch instruction on
 289: // these architecture that can result in a thunk.
 290: 
```

- **L272**: Begins the declaration of class \`ThumbV6MPILongThunk\`. / 开始声明 class \`ThumbV6MPILongThunk\`。
- **L273**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L274**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L275**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L276**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L277**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L278**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L279**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L280**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L281**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L282**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L283**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L284**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L285**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L286**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L287**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L288**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L289**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L290**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 291-307 / 第 291-307 行

```cpp
 291: // LDR on v5 and v6 can switch processor state, so for v5 and v6,
 292: // ARMV5LongLdrPcThunk can be used for both Arm->Arm and Arm->Thumb calls. v4
 293: // can also use this thunk, but only for Arm->Arm calls.
 294: class ARMV5LongLdrPcThunk final : public ARMThunk {
 295: public:
 296:   ARMV5LongLdrPcThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 297:       : ARMThunk(ctx, dest, addend) {}
 298: 
 299:   uint32_t sizeLong() override { return 8; }
 300:   void writeLong(uint8_t *buf) override;
 301:   void addSymbols(ThunkSection &isec) override;
 302: 
 303: private:
 304:   void addLongMapSyms() override;
 305:   ThunkSection *tsec = nullptr;
 306: };
 307: 
```

- **L291**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L292**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L293**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L294**: Begins the declaration of class \`ARMV5LongLdrPcThunk\`. / 开始声明 class \`ARMV5LongLdrPcThunk\`。
- **L295**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L296**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L297**: Defines function or method \`ARMThunk\`. / 定义函数或方法 \`ARMThunk\`。
- **L298**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L299**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L300**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L301**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L302**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L303**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L304**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L305**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L306**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L307**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 308-323 / 第 308-323 行

```cpp
 308: // Implementations of Thunks for v4. BLX is not supported, and loads
 309: // will not invoke Arm/Thumb state changes.
 310: class ARMV4PILongBXThunk final : public ARMThunk {
 311: public:
 312:   ARMV4PILongBXThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 313:       : ARMThunk(ctx, dest, addend) {}
 314: 
 315:   uint32_t sizeLong() override { return 16; }
 316:   void writeLong(uint8_t *buf) override;
 317:   void addSymbols(ThunkSection &isec) override;
 318: 
 319: private:
 320:   void addLongMapSyms() override;
 321:   ThunkSection *tsec = nullptr;
 322: };
 323: 
```

- **L308**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L309**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L310**: Begins the declaration of class \`ARMV4PILongBXThunk\`. / 开始声明 class \`ARMV4PILongBXThunk\`。
- **L311**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L312**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L313**: Defines function or method \`ARMThunk\`. / 定义函数或方法 \`ARMThunk\`。
- **L314**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L315**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L316**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L317**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L318**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L319**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L320**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L321**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L322**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L323**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 324-338 / 第 324-338 行

```cpp
 324: class ARMV4PILongThunk final : public ARMThunk {
 325: public:
 326:   ARMV4PILongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 327:       : ARMThunk(ctx, dest, addend) {}
 328: 
 329:   uint32_t sizeLong() override { return 12; }
 330:   void writeLong(uint8_t *buf) override;
 331:   void addSymbols(ThunkSection &isec) override;
 332: 
 333: private:
 334:   void addLongMapSyms() override;
 335:   ThunkSection *tsec = nullptr;
 336: };
 337: 
 338: class ThumbV4PILongBXThunk final : public ThumbThunk {
```

- **L324**: Begins the declaration of class \`ARMV4PILongThunk\`. / 开始声明 class \`ARMV4PILongThunk\`。
- **L325**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L326**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L327**: Defines function or method \`ARMThunk\`. / 定义函数或方法 \`ARMThunk\`。
- **L328**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L329**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L330**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L331**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L332**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L333**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L334**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L335**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L336**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L337**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L338**: Begins the declaration of class \`ThumbV4PILongBXThunk\`. / 开始声明 class \`ThumbV4PILongBXThunk\`。

### Lines 339-353 / 第 339-353 行

```cpp
 339: public:
 340:   ThumbV4PILongBXThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 341:       : ThumbThunk(ctx, dest, addend) {}
 342: 
 343:   uint32_t sizeLong() override { return 16; }
 344:   void writeLong(uint8_t *buf) override;
 345:   void addSymbols(ThunkSection &isec) override;
 346: 
 347: private:
 348:   void addLongMapSyms() override;
 349:   ThunkSection *tsec = nullptr;
 350: };
 351: 
 352: class ThumbV4PILongThunk final : public ThumbThunk {
 353: public:
```

- **L339**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L340**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L341**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L342**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L343**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L344**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L345**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L346**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L347**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L348**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L349**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L350**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L351**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L352**: Begins the declaration of class \`ThumbV4PILongThunk\`. / 开始声明 class \`ThumbV4PILongThunk\`。
- **L353**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。

### Lines 354-370 / 第 354-370 行

```cpp
 354:   ThumbV4PILongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 355:       : ThumbThunk(ctx, dest, addend) {}
 356: 
 357:   uint32_t sizeLong() override { return 20; }
 358:   void writeLong(uint8_t *buf) override;
 359:   void addSymbols(ThunkSection &isec) override;
 360: 
 361: private:
 362:   void addLongMapSyms() override;
 363:   ThunkSection *tsec = nullptr;
 364: };
 365: 
 366: class ARMV4ABSLongBXThunk final : public ARMThunk {
 367: public:
 368:   ARMV4ABSLongBXThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 369:       : ARMThunk(ctx, dest, addend) {}
 370: 
```

- **L354**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L355**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L356**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L357**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L358**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L359**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L360**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L361**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L362**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L363**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L364**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L365**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L366**: Begins the declaration of class \`ARMV4ABSLongBXThunk\`. / 开始声明 class \`ARMV4ABSLongBXThunk\`。
- **L367**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L368**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L369**: Defines function or method \`ARMThunk\`. / 定义函数或方法 \`ARMThunk\`。
- **L370**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 371-388 / 第 371-388 行

```cpp
 371:   uint32_t sizeLong() override { return 12; }
 372:   void writeLong(uint8_t *buf) override;
 373:   void addSymbols(ThunkSection &isec) override;
 374: 
 375: private:
 376:   void addLongMapSyms() override;
 377:   ThunkSection *tsec = nullptr;
 378: };
 379: 
 380: class ThumbV4ABSLongBXThunk final : public ThumbThunk {
 381: public:
 382:   ThumbV4ABSLongBXThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 383:       : ThumbThunk(ctx, dest, addend) {}
 384: 
 385:   uint32_t sizeLong() override { return 12; }
 386:   void writeLong(uint8_t *buf) override;
 387:   void addSymbols(ThunkSection &isec) override;
 388: 
```

- **L371**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L372**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L373**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L374**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L375**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L376**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L377**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L378**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L379**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L380**: Begins the declaration of class \`ThumbV4ABSLongBXThunk\`. / 开始声明 class \`ThumbV4ABSLongBXThunk\`。
- **L381**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L382**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L383**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L384**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L385**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L386**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L387**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L388**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 389-403 / 第 389-403 行

```cpp
 389: private:
 390:   void addLongMapSyms() override;
 391:   ThunkSection *tsec = nullptr;
 392: };
 393: 
 394: class ThumbV4ABSLongThunk final : public ThumbThunk {
 395: public:
 396:   ThumbV4ABSLongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 397:       : ThumbThunk(ctx, dest, addend) {}
 398: 
 399:   uint32_t sizeLong() override { return 16; }
 400:   void writeLong(uint8_t *buf) override;
 401:   void addSymbols(ThunkSection &isec) override;
 402: 
 403: private:
```

- **L389**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L390**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L391**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L392**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L393**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L394**: Begins the declaration of class \`ThumbV4ABSLongThunk\`. / 开始声明 class \`ThumbV4ABSLongThunk\`。
- **L395**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L396**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L397**: Defines function or method \`ThumbThunk\`. / 定义函数或方法 \`ThumbThunk\`。
- **L398**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L399**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L400**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L401**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L402**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L403**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。

### Lines 404-421 / 第 404-421 行

```cpp
 404:   void addLongMapSyms() override;
 405:   ThunkSection *tsec = nullptr;
 406: };
 407: 
 408: // The AVR devices need thunks for R_AVR_LO8_LDI_GS/R_AVR_HI8_LDI_GS
 409: // when their destination is out of range [0, 0x1ffff].
 410: class AVRThunk : public Thunk {
 411: public:
 412:   AVRThunk(Ctx &ctx, Symbol &dest, int64_t addend) : Thunk(ctx, dest, addend) {}
 413:   uint32_t size() override { return 4; }
 414:   void writeTo(uint8_t *buf) override;
 415:   void addSymbols(ThunkSection &isec) override;
 416: };
 417: 
 418: // Hexagon CPUs need thunks for R_HEX_B{9,1{3,5},22}_PCREL,
 419: // R_HEX_{,GD_}PLT_B22_PCREL when their destination is out of
 420: // range.
 421: class HexagonThunk : public Thunk {
```

- **L404**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L405**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L406**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L407**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L408**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L409**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L410**: Begins the declaration of class \`AVRThunk\`. / 开始声明 class \`AVRThunk\`。
- **L411**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L412**: Defines function or method \`AVRThunk\`. / 定义函数或方法 \`AVRThunk\`。
- **L413**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L414**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L415**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L416**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L417**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L418**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L419**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L420**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L421**: Begins the declaration of class \`HexagonThunk\`. / 开始声明 class \`HexagonThunk\`。

### Lines 422-436 / 第 422-436 行

```cpp
 422: public:
 423:   HexagonThunk(Ctx &ctx, const InputSection &isec, Relocation &rel,
 424:                Symbol &dest)
 425:       : Thunk(ctx, dest, 0), relOffset(rel.offset) {
 426:     alignment = 4;
 427:   }
 428:   uint32_t relOffset;
 429:   uint32_t size() override { return ctx.arg.isPic ? 12 : 8; }
 430:   void writeTo(uint8_t *buf) override;
 431:   void addSymbols(ThunkSection &isec) override;
 432: };
 433: 
 434: // MIPS LA25 thunk
 435: class MipsThunk final : public Thunk {
 436: public:
```

- **L422**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L423**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L424**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L425**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L426**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L427**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L428**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L429**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L430**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L431**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L432**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L433**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L434**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L435**: Begins the declaration of class \`MipsThunk\`. / 开始声明 class \`MipsThunk\`。
- **L436**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。

### Lines 437-455 / 第 437-455 行

```cpp
 437:   MipsThunk(Ctx &ctx, Symbol &dest) : Thunk(ctx, dest, 0) {}
 438: 
 439:   uint32_t size() override { return 16; }
 440:   void writeTo(uint8_t *buf) override;
 441:   void addSymbols(ThunkSection &isec) override;
 442:   InputSection *getTargetInputSection() const override;
 443: };
 444: 
 445: // microMIPS R2-R5 LA25 thunk
 446: class MicroMipsThunk final : public Thunk {
 447: public:
 448:   MicroMipsThunk(Ctx &ctx, Symbol &dest) : Thunk(ctx, dest, 0) {}
 449: 
 450:   uint32_t size() override { return 14; }
 451:   void writeTo(uint8_t *buf) override;
 452:   void addSymbols(ThunkSection &isec) override;
 453:   InputSection *getTargetInputSection() const override;
 454: };
 455: 
```

- **L437**: Defines function or method \`MipsThunk\`. / 定义函数或方法 \`MipsThunk\`。
- **L438**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L439**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L440**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L441**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L442**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L443**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L444**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L445**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L446**: Begins the declaration of class \`MicroMipsThunk\`. / 开始声明 class \`MicroMipsThunk\`。
- **L447**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L448**: Defines function or method \`MicroMipsThunk\`. / 定义函数或方法 \`MicroMipsThunk\`。
- **L449**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L450**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L451**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L452**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L453**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L454**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L455**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 456-478 / 第 456-478 行

```cpp
 456: // microMIPS R6 LA25 thunk
 457: class MicroMipsR6Thunk final : public Thunk {
 458: public:
 459:   MicroMipsR6Thunk(Ctx &ctx, Symbol &dest) : Thunk(ctx, dest, 0) {}
 460: 
 461:   uint32_t size() override { return 12; }
 462:   void writeTo(uint8_t *buf) override;
 463:   void addSymbols(ThunkSection &isec) override;
 464:   InputSection *getTargetInputSection() const override;
 465: };
 466: 
 467: class PPC32PltCallStub final : public Thunk {
 468: public:
 469:   // For R_PPC_PLTREL24, Thunk::addend records the addend which will be used to
 470:   // decide the offsets in the call stub.
 471:   PPC32PltCallStub(Ctx &ctx, const InputSection &isec, const Relocation &rel,
 472:                    Symbol &dest)
 473:       : Thunk(ctx, dest, rel.addend), file(isec.file) {}
 474:   uint32_t size() override { return 16; }
 475:   void writeTo(uint8_t *buf) override;
 476:   void addSymbols(ThunkSection &isec) override;
 477:   bool isCompatibleWith(const InputSection &isec, const Relocation &rel) const override;
 478: 
```

- **L456**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L457**: Begins the declaration of class \`MicroMipsR6Thunk\`. / 开始声明 class \`MicroMipsR6Thunk\`。
- **L458**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L459**: Defines function or method \`MicroMipsR6Thunk\`. / 定义函数或方法 \`MicroMipsR6Thunk\`。
- **L460**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L461**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L462**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L463**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L464**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L465**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L466**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L467**: Begins the declaration of class \`PPC32PltCallStub\`. / 开始声明 class \`PPC32PltCallStub\`。
- **L468**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L469**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L470**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L471**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L472**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L473**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L474**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L475**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L476**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L477**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L478**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 479-501 / 第 479-501 行

```cpp
 479: private:
 480:   // Records the call site of the call stub.
 481:   const InputFile *file;
 482: };
 483: 
 484: class PPC32LongThunk final : public Thunk {
 485: public:
 486:   PPC32LongThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 487:       : Thunk(ctx, dest, addend) {}
 488:   uint32_t size() override { return ctx.arg.isPic ? 32 : 16; }
 489:   void writeTo(uint8_t *buf) override;
 490:   void addSymbols(ThunkSection &isec) override;
 491: };
 492: 
 493: // PPC64 Plt call stubs.
 494: // Any call site that needs to call through a plt entry needs a call stub in
 495: // the .text section. The call stub is responsible for:
 496: // 1) Saving the toc-pointer to the stack.
 497: // 2) Loading the target functions address from the procedure linkage table into
 498: //    r12 for use by the target functions global entry point, and into the count
 499: //    register.
 500: // 3) Transferring control to the target function through an indirect branch.
 501: class PPC64PltCallStub final : public Thunk {
```

- **L479**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L480**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L481**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L482**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L483**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L484**: Begins the declaration of class \`PPC32LongThunk\`. / 开始声明 class \`PPC32LongThunk\`。
- **L485**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L486**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L487**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L488**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L489**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L490**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L491**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L492**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L493**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L494**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L495**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L496**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L497**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L498**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L499**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L500**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L501**: Begins the declaration of class \`PPC64PltCallStub\`. / 开始声明 class \`PPC64PltCallStub\`。

### Lines 502-517 / 第 502-517 行

```cpp
 502: public:
 503:   PPC64PltCallStub(Ctx &ctx, Symbol &dest) : Thunk(ctx, dest, 0) {}
 504:   uint32_t size() override { return 20; }
 505:   void writeTo(uint8_t *buf) override;
 506:   void addSymbols(ThunkSection &isec) override;
 507:   bool isCompatibleWith(const InputSection &isec,
 508:                         const Relocation &rel) const override;
 509: };
 510: 
 511: // PPC64 R2 Save Stub
 512: // When the caller requires a valid R2 TOC pointer but the callee does not
 513: // require a TOC pointer and the callee cannot guarantee that it doesn't
 514: // clobber R2 then we need to save R2. This stub:
 515: // 1) Saves the TOC pointer to the stack.
 516: // 2) Tail calls the callee.
 517: class PPC64R2SaveStub final : public Thunk {
```

- **L502**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L503**: Defines function or method \`PPC64PltCallStub\`. / 定义函数或方法 \`PPC64PltCallStub\`。
- **L504**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L505**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L506**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L507**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L508**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L509**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L510**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L511**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L512**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L513**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L514**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L515**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L516**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L517**: Begins the declaration of class \`PPC64R2SaveStub\`. / 开始声明 class \`PPC64R2SaveStub\`。

### Lines 518-540 / 第 518-540 行

```cpp
 518: public:
 519:   PPC64R2SaveStub(Ctx &ctx, Symbol &dest, int64_t addend)
 520:       : Thunk(ctx, dest, addend) {
 521:     alignment = 16;
 522:   }
 523: 
 524:   // To prevent oscillations in layout when moving from short to long thunks
 525:   // we make sure that once a thunk has been set to long it cannot go back.
 526:   bool getMayUseShortThunk() {
 527:     if (!mayUseShortThunk)
 528:       return false;
 529:     if (!isInt<26>(computeOffset())) {
 530:       mayUseShortThunk = false;
 531:       return false;
 532:     }
 533:     return true;
 534:   }
 535:   uint32_t size() override { return getMayUseShortThunk() ? 8 : 32; }
 536:   void writeTo(uint8_t *buf) override;
 537:   void addSymbols(ThunkSection &isec) override;
 538:   bool isCompatibleWith(const InputSection &isec,
 539:                         const Relocation &rel) const override;
 540: 
```

- **L518**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L519**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L520**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L521**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L522**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L523**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L524**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L525**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L526**: Defines function or method \`getMayUseShortThunk\`. / 定义函数或方法 \`getMayUseShortThunk\`。
- **L527**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L528**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L529**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L530**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L531**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L532**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L533**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L534**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L535**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L536**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L537**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L538**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L539**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L540**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 541-555 / 第 541-555 行

```cpp
 541: private:
 542:   // Transitioning from long to short can create layout oscillations in
 543:   // certain corner cases which would prevent the layout from converging.
 544:   // This is similar to the handling for ARMThunk.
 545:   bool mayUseShortThunk = true;
 546:   int64_t computeOffset() const {
 547:     return destination.getVA(ctx) - (getThunkTargetSym()->getVA(ctx) + 4);
 548:   }
 549: };
 550: 
 551: // PPC64 R12 Setup Stub
 552: // When a caller that does not maintain TOC calls a target which may possibly
 553: // use TOC (either non-preemptible with localentry>1 or preemptible), we need to
 554: // set r12 to satisfy the requirement of the global entry point.
 555: class PPC64R12SetupStub final : public Thunk {
```

- **L541**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L542**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L543**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L544**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L545**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L546**: Defines function or method \`computeOffset\`. / 定义函数或方法 \`computeOffset\`。
- **L547**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L548**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L549**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L550**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L551**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L552**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L553**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L554**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L555**: Begins the declaration of class \`PPC64R12SetupStub\`. / 开始声明 class \`PPC64R12SetupStub\`。

### Lines 556-570 / 第 556-570 行

```cpp
 556: public:
 557:   PPC64R12SetupStub(Ctx &ctx, Symbol &dest, bool gotPlt)
 558:       : Thunk(ctx, dest, 0), gotPlt(gotPlt) {
 559:     alignment = 16;
 560:   }
 561:   uint32_t size() override { return 32; }
 562:   void writeTo(uint8_t *buf) override;
 563:   void addSymbols(ThunkSection &isec) override;
 564:   bool isCompatibleWith(const InputSection &isec,
 565:                         const Relocation &rel) const override;
 566: 
 567: private:
 568:   bool gotPlt;
 569: };
 570: 
```

- **L556**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L557**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L558**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L559**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L560**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L561**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L562**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L563**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L564**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L565**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L566**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L567**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L568**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L569**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L570**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 571-586 / 第 571-586 行

```cpp
 571: // A bl instruction uses a signed 24 bit offset, with an implicit 4 byte
 572: // alignment. This gives a possible 26 bits of 'reach'. If the call offset is
 573: // larger than that we need to emit a long-branch thunk. The target address
 574: // of the callee is stored in a table to be accessed TOC-relative. Since the
 575: // call must be local (a non-local call will have a PltCallStub instead) the
 576: // table stores the address of the callee's local entry point. For
 577: // position-independent code a corresponding relative dynamic relocation is
 578: // used.
 579: class PPC64LongBranchThunk : public Thunk {
 580: public:
 581:   uint32_t size() override { return 32; }
 582:   void writeTo(uint8_t *buf) override;
 583:   void addSymbols(ThunkSection &isec) override;
 584:   bool isCompatibleWith(const InputSection &isec,
 585:                         const Relocation &rel) const override;
 586: 
```

- **L571**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L572**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L573**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L574**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L575**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L576**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L577**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L578**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L579**: Begins the declaration of class \`PPC64LongBranchThunk\`. / 开始声明 class \`PPC64LongBranchThunk\`。
- **L580**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L581**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L582**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L583**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L584**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L585**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L586**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 587-607 / 第 587-607 行

```cpp
 587: protected:
 588:   PPC64LongBranchThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 589:       : Thunk(ctx, dest, addend) {}
 590: };
 591: 
 592: class PPC64PILongBranchThunk final : public PPC64LongBranchThunk {
 593: public:
 594:   PPC64PILongBranchThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 595:       : PPC64LongBranchThunk(ctx, dest, addend) {
 596:     assert(!dest.isPreemptible);
 597:     if (std::optional<uint32_t> index =
 598:             ctx.in.ppc64LongBranchTarget->addEntry(&dest, addend)) {
 599:       ctx.mainPart->relaDyn->addRelativeReloc(
 600:           ctx.target->relativeRel, *ctx.in.ppc64LongBranchTarget,
 601:           *index * UINT64_C(8), dest,
 602:           addend + getPPC64GlobalEntryToLocalEntryOffset(ctx, dest.stOther),
 603:           ctx.target->symbolicRel, R_ABS);
 604:     }
 605:   }
 606: };
 607: 
```

- **L587**: Switches following class members to \`protected\` access control. / 将后续类成员的访问控制切换为 \`protected\`。
- **L588**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L589**: Defines function or method \`Thunk\`. / 定义函数或方法 \`Thunk\`。
- **L590**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L591**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L592**: Begins the declaration of class \`PPC64PILongBranchThunk\`. / 开始声明 class \`PPC64PILongBranchThunk\`。
- **L593**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L594**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L595**: Defines function or method \`PPC64LongBranchThunk\`. / 定义函数或方法 \`PPC64LongBranchThunk\`。
- **L596**: Declares function or method \`assert\`. / 声明函数或方法 \`assert\`。
- **L597**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L598**: Defines function or method \`addEntry\`. / 定义函数或方法 \`addEntry\`。
- **L599**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L600**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L601**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L602**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L603**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L604**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L605**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L606**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L607**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 608-625 / 第 608-625 行

```cpp
 608: class PPC64PDLongBranchThunk final : public PPC64LongBranchThunk {
 609: public:
 610:   PPC64PDLongBranchThunk(Ctx &ctx, Symbol &dest, int64_t addend)
 611:       : PPC64LongBranchThunk(ctx, dest, addend) {
 612:     ctx.in.ppc64LongBranchTarget->addEntry(&dest, addend);
 613:   }
 614: };
 615: 
 616: } // end anonymous namespace
 617: 
 618: Defined *Thunk::addSymbol(StringRef name, uint8_t type, uint64_t value,
 619:                           InputSectionBase &section) {
 620:   Defined *d =
 621:       addSyntheticLocal(ctx, name, type, value + offset, /*size=*/0, section);
 622:   syms.push_back(d);
 623:   return d;
 624: }
 625: 
```

- **L608**: Begins the declaration of class \`PPC64PDLongBranchThunk\`. / 开始声明 class \`PPC64PDLongBranchThunk\`。
- **L609**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L610**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L611**: Defines function or method \`PPC64LongBranchThunk\`. / 定义函数或方法 \`PPC64LongBranchThunk\`。
- **L612**: Declares function or method \`addEntry\`. / 声明函数或方法 \`addEntry\`。
- **L613**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L614**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L615**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L616**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L617**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L618**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L619**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L620**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L621**: Declares function or method \`addSyntheticLocal\`. / 声明函数或方法 \`addSyntheticLocal\`。
- **L622**: Declares function or method \`push_back\`. / 声明函数或方法 \`push_back\`。
- **L623**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L624**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L625**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 626-648 / 第 626-648 行

```cpp
 626: void Thunk::setOffset(uint64_t newOffset) {
 627:   for (Defined *d : syms)
 628:     d->value = d->value - offset + newOffset;
 629:   offset = newOffset;
 630: }
 631: 
 632: // AArch64 Thunk base class.
 633: static uint64_t getAArch64ThunkDestVA(Ctx &ctx, const Symbol &s, int64_t a) {
 634:   uint64_t v = s.isInPlt(ctx) ? s.getPltVA(ctx) : s.getVA(ctx, a);
 635:   return v;
 636: }
 637: 
 638: bool AArch64Thunk::getMayUseShortThunk() {
 639:   if (!mayUseShortThunk)
 640:     return false;
 641:   uint64_t s = getAArch64ThunkDestVA(ctx, destination, addend);
 642:   uint64_t p = getThunkTargetSym()->getVA(ctx);
 643:   mayUseShortThunk = llvm::isInt<28>(s - p);
 644:   if (!mayUseShortThunk)
 645:     addLongMapSyms();
 646:   return mayUseShortThunk;
 647: }
 648: 
```

- **L626**: Defines function or method \`setOffset\`. / 定义函数或方法 \`setOffset\`。
- **L627**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L628**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L629**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L630**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L631**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L632**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L633**: Defines function or method \`getAArch64ThunkDestVA\`. / 定义函数或方法 \`getAArch64ThunkDestVA\`。
- **L634**: Declares function or method \`isInPlt\`. / 声明函数或方法 \`isInPlt\`。
- **L635**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L636**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L637**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L638**: Defines function or method \`getMayUseShortThunk\`. / 定义函数或方法 \`getMayUseShortThunk\`。
- **L639**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L640**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L641**: Declares function or method \`getAArch64ThunkDestVA\`. / 声明函数或方法 \`getAArch64ThunkDestVA\`。
- **L642**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L643**: Declares function or method \`isInt\`. / 声明函数或方法 \`isInt\`。
- **L644**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L645**: Declares function or method \`addLongMapSyms\`. / 声明函数或方法 \`addLongMapSyms\`。
- **L646**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L647**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L648**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 649-665 / 第 649-665 行

```cpp
 649: void AArch64Thunk::writeTo(uint8_t *buf) {
 650:   if (!getMayUseShortThunk()) {
 651:     writeLong(buf);
 652:     return;
 653:   }
 654:   uint64_t s = getAArch64ThunkDestVA(ctx, destination, addend);
 655:   uint64_t p = getThunkTargetSym()->getVA(ctx);
 656:   write32(ctx, buf, 0x14000000); // b S
 657:   ctx.target->relocateNoSym(buf, R_AARCH64_CALL26, s - p);
 658: }
 659: 
 660: bool AArch64Thunk::needsSyntheticLandingPad() {
 661:   // Short Thunks use a direct branch, no synthetic landing pad
 662:   // required.
 663:   return mayNeedLandingPad && !getMayUseShortThunk();
 664: }
 665: 
```

- **L649**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L650**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L651**: Declares function or method \`writeLong\`. / 声明函数或方法 \`writeLong\`。
- **L652**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L653**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L654**: Declares function or method \`getAArch64ThunkDestVA\`. / 声明函数或方法 \`getAArch64ThunkDestVA\`。
- **L655**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L656**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L657**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L658**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L659**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L660**: Defines function or method \`needsSyntheticLandingPad\`. / 定义函数或方法 \`needsSyntheticLandingPad\`。
- **L661**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L662**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L663**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L664**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L665**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 666-683 / 第 666-683 行

```cpp
 666: // AArch64 long range Thunks.
 667: void AArch64ABSLongThunk::writeLong(uint8_t *buf) {
 668:   const uint8_t data[] = {
 669:     0x50, 0x00, 0x00, 0x58, //     ldr x16, L0
 670:     0x00, 0x02, 0x1f, 0xd6, //     br  x16
 671:     0x00, 0x00, 0x00, 0x00, // L0: .xword S
 672:     0x00, 0x00, 0x00, 0x00,
 673:   };
 674:   // If mayNeedLandingPad is true then destination is an
 675:   // AArch64BTILandingPadThunk that defines landingPad.
 676:   assert(!mayNeedLandingPad || landingPad != nullptr);
 677:   uint64_t s = mayNeedLandingPad
 678:                    ? landingPad->getVA(ctx, 0)
 679:                    : getAArch64ThunkDestVA(ctx, destination, addend);
 680:   memcpy(buf, data, sizeof(data));
 681:   ctx.target->relocateNoSym(buf + 8, R_AARCH64_ABS64, s);
 682: }
 683: 
```

- **L666**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L667**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L668**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L669**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L670**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L671**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L672**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L673**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L674**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L675**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L676**: Declares function or method \`assert\`. / 声明函数或方法 \`assert\`。
- **L677**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L678**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L679**: Declares function or method \`getAArch64ThunkDestVA\`. / 声明函数或方法 \`getAArch64ThunkDestVA\`。
- **L680**: Declares function or method \`memcpy\`. / 声明函数或方法 \`memcpy\`。
- **L681**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L682**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L683**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 684-698 / 第 684-698 行

```cpp
 684: void AArch64ABSLongThunk::addSymbols(ThunkSection &isec) {
 685:   addSymbol(ctx.saver.save("__AArch64AbsLongThunk_" + destination.getName()),
 686:             STT_FUNC, 0, isec);
 687:   addSymbol("$x", STT_NOTYPE, 0, isec);
 688:   tsec = &isec;
 689:   (void)getMayUseShortThunk();
 690: }
 691: 
 692: void AArch64ABSLongThunk::addLongMapSyms() {
 693:   addSymbol("$d", STT_NOTYPE, 8, *tsec);
 694:   // The ldr in the long Thunk requires 8-byte alignment when
 695:   // unaligned accesses are disabled.
 696:   alignment = 8;
 697: }
 698: 
```

- **L684**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L685**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L686**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L687**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L688**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L689**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L690**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L691**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L692**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L693**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L694**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L695**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L696**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L697**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L698**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 699-719 / 第 699-719 行

```cpp
 699: void AArch64ABSXOLongThunk::writeLong(uint8_t *buf) {
 700:   const uint8_t data[] = {
 701:       0x10, 0x00, 0x80, 0xd2, // movz x16, :abs_g0_nc:S, lsl #0
 702:       0x10, 0x00, 0xa0, 0xf2, // movk x16, :abs_g1_nc:S, lsl #16
 703:       0x10, 0x00, 0xc0, 0xf2, // movk x16, :abs_g2_nc:S, lsl #32
 704:       0x10, 0x00, 0xe0, 0xf2, // movk x16, :abs_g3:S,    lsl #48
 705:       0x00, 0x02, 0x1f, 0xd6, // br   x16
 706:   };
 707:   // If mayNeedLandingPad is true then destination is an
 708:   // AArch64BTILandingPadThunk that defines landingPad.
 709:   assert(!mayNeedLandingPad || landingPad != nullptr);
 710:   uint64_t s = mayNeedLandingPad
 711:                    ? landingPad->getVA(ctx, 0)
 712:                    : getAArch64ThunkDestVA(ctx, destination, addend);
 713:   memcpy(buf, data, sizeof(data));
 714:   ctx.target->relocateNoSym(buf + 0, R_AARCH64_MOVW_UABS_G0_NC, s);
 715:   ctx.target->relocateNoSym(buf + 4, R_AARCH64_MOVW_UABS_G1_NC, s);
 716:   ctx.target->relocateNoSym(buf + 8, R_AARCH64_MOVW_UABS_G2_NC, s);
 717:   ctx.target->relocateNoSym(buf + 12, R_AARCH64_MOVW_UABS_G3, s);
 718: }
 719: 
```

- **L699**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L700**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L701**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L702**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L703**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L704**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L705**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L706**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L707**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L708**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L709**: Declares function or method \`assert\`. / 声明函数或方法 \`assert\`。
- **L710**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L711**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L712**: Declares function or method \`getAArch64ThunkDestVA\`. / 声明函数或方法 \`getAArch64ThunkDestVA\`。
- **L713**: Declares function or method \`memcpy\`. / 声明函数或方法 \`memcpy\`。
- **L714**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L715**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L716**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L717**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L718**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L719**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 720-749 / 第 720-749 行

```cpp
 720: void AArch64ABSXOLongThunk::addSymbols(ThunkSection &sec) {
 721:   addSymbol(ctx.saver.save("__AArch64AbsXOLongThunk_" + destination.getName()),
 722:             STT_FUNC, 0, sec);
 723:   addSymbol("$x", STT_NOTYPE, 0, sec);
 724: }
 725: 
 726: // This Thunk has a maximum range of 4Gb, this is sufficient for all programs
 727: // using the small code model, including pc-relative ones. At time of writing
 728: // clang and gcc do not support the large code model for position independent
 729: // code so it is safe to use this for position independent thunks without
 730: // worrying about the destination being more than 4Gb away.
 731: void AArch64ADRPThunk::writeLong(uint8_t *buf) {
 732:   const uint8_t data[] = {
 733:       0x10, 0x00, 0x00, 0x90, // adrp x16, Dest R_AARCH64_ADR_PREL_PG_HI21(Dest)
 734:       0x10, 0x02, 0x00, 0x91, // add  x16, x16, R_AARCH64_ADD_ABS_LO12_NC(Dest)
 735:       0x00, 0x02, 0x1f, 0xd6, // br   x16
 736:   };
 737:   // if mayNeedLandingPad is true then destination is an
 738:   // AArch64BTILandingPadThunk that defines landingPad.
 739:   assert(!mayNeedLandingPad || landingPad != nullptr);
 740:   uint64_t s = mayNeedLandingPad
 741:                    ? landingPad->getVA(ctx, 0)
 742:                    : getAArch64ThunkDestVA(ctx, destination, addend);
 743:   uint64_t p = getThunkTargetSym()->getVA(ctx);
 744:   memcpy(buf, data, sizeof(data));
 745:   ctx.target->relocateNoSym(buf, R_AARCH64_ADR_PREL_PG_HI21,
 746:                             getAArch64Page(s) - getAArch64Page(p));
 747:   ctx.target->relocateNoSym(buf + 4, R_AARCH64_ADD_ABS_LO12_NC, s);
 748: }
 749: 
```

- **L720**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L721**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L722**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L723**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L724**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L725**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L726**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L727**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L728**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L729**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L730**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L731**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L732**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L733**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L734**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L735**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L736**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L737**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L738**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L739**: Declares function or method \`assert\`. / 声明函数或方法 \`assert\`。
- **L740**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L741**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L742**: Declares function or method \`getAArch64ThunkDestVA\`. / 声明函数或方法 \`getAArch64ThunkDestVA\`。
- **L743**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L744**: Declares function or method \`memcpy\`. / 声明函数或方法 \`memcpy\`。
- **L745**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L746**: Declares function or method \`getAArch64Page\`. / 声明函数或方法 \`getAArch64Page\`。
- **L747**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L748**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L749**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 750-770 / 第 750-770 行

```cpp
 750: void AArch64ADRPThunk::addSymbols(ThunkSection &isec) {
 751:   addSymbol(ctx.saver.save("__AArch64ADRPThunk_" + destination.getName()),
 752:             STT_FUNC, 0, isec);
 753:   addSymbol("$x", STT_NOTYPE, 0, isec);
 754: }
 755: 
 756: void AArch64BTILandingPadThunk::addSymbols(ThunkSection &isec) {
 757:   addSymbol(ctx.saver.save("__AArch64BTIThunk_" + destination.getName()),
 758:             STT_FUNC, 0, isec);
 759:   addSymbol("$x", STT_NOTYPE, 0, isec);
 760: }
 761: 
 762: void AArch64BTILandingPadThunk::writeTo(uint8_t *buf) {
 763:   if (!getMayUseShortThunk()) {
 764:     writeLong(buf);
 765:     return;
 766:   }
 767:   write32(ctx, buf, 0xd503245f); // BTI c
 768:   // Control falls through to target in following section.
 769: }
 770: 
```

- **L750**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L751**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L752**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L753**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L754**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L755**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L756**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L757**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L758**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L759**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L760**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L761**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L762**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L763**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L764**: Declares function or method \`writeLong\`. / 声明函数或方法 \`writeLong\`。
- **L765**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L766**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L767**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L768**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L769**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L770**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 771-788 / 第 771-788 行

```cpp
 771: bool AArch64BTILandingPadThunk::getMayUseShortThunk() {
 772:   if (!mayUseShortThunk)
 773:     return false;
 774:   // If the target is the following instruction then we can fall
 775:   // through without the indirect branch.
 776:   uint64_t s = destination.getVA(ctx, addend);
 777:   uint64_t p = getThunkTargetSym()->getVA(ctx);
 778:   // This function is called before addresses are stable.  We need to
 779:   // work out the range from the thunk to the next section but the
 780:   // address of the start of the next section depends on the size of
 781:   // the thunks in the previous pass.  s - p + offset == 0 represents
 782:   // the first pass where the Thunk and following section are assigned
 783:   // the same offset.  s - p <= 4 is the last Thunk in the Thunk
 784:   // Section.
 785:   mayUseShortThunk = (s - p + offset == 0 || s - p <= 4);
 786:   return mayUseShortThunk;
 787: }
 788: 
```

- **L771**: Defines function or method \`getMayUseShortThunk\`. / 定义函数或方法 \`getMayUseShortThunk\`。
- **L772**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L773**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L774**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L775**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L776**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L777**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L778**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L779**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L780**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L781**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L782**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L783**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L784**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L785**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L786**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L787**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L788**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 789-818 / 第 789-818 行

```cpp
 789: void AArch64BTILandingPadThunk::writeLong(uint8_t *buf) {
 790:   uint64_t s = destination.getVA(ctx, addend);
 791:   uint64_t p = getThunkTargetSym()->getVA(ctx) + 4;
 792:   write32(ctx, buf, 0xd503245f);     // BTI c
 793:   write32(ctx, buf + 4, 0x14000000); // B S
 794:   ctx.target->relocateNoSym(buf + 4, R_AARCH64_CALL26, s - p);
 795: }
 796: 
 797: // ARM Target Thunks
 798: static uint64_t getARMThunkDestVA(Ctx &ctx, const Symbol &s) {
 799:   uint64_t v = s.isInPlt(ctx) ? s.getPltVA(ctx) : s.getVA(ctx);
 800:   return SignExtend64<32>(v);
 801: }
 802: 
 803: // This function returns true if the target is not Thumb and is within 2^26, and
 804: // it has not previously returned false (see comment for mayUseShortThunk).
 805: bool ARMThunk::getMayUseShortThunk() {
 806:   if (!mayUseShortThunk)
 807:     return false;
 808:   uint64_t s = getARMThunkDestVA(ctx, destination);
 809:   if (s & 1) {
 810:     mayUseShortThunk = false;
 811:     addLongMapSyms();
 812:     return false;
 813:   }
 814:   uint64_t p = getThunkTargetSym()->getVA(ctx);
 815:   int64_t offset = s - p - 8;
 816:   mayUseShortThunk = llvm::isInt<26>(offset);
 817:   if (!mayUseShortThunk)
 818:     addLongMapSyms();
```

- **L789**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L790**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L791**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L792**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L793**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L794**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L795**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L796**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L797**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L798**: Defines function or method \`getARMThunkDestVA\`. / 定义函数或方法 \`getARMThunkDestVA\`。
- **L799**: Declares function or method \`isInPlt\`. / 声明函数或方法 \`isInPlt\`。
- **L800**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L801**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L802**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L803**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L804**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L805**: Defines function or method \`getMayUseShortThunk\`. / 定义函数或方法 \`getMayUseShortThunk\`。
- **L806**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L807**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L808**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L809**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L810**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L811**: Declares function or method \`addLongMapSyms\`. / 声明函数或方法 \`addLongMapSyms\`。
- **L812**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L813**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L814**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L815**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L816**: Declares function or method \`isInt\`. / 声明函数或方法 \`isInt\`。
- **L817**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L818**: Declares function or method \`addLongMapSyms\`. / 声明函数或方法 \`addLongMapSyms\`。

### Lines 819-834 / 第 819-834 行

```cpp
 819:   return mayUseShortThunk;
 820: }
 821: 
 822: void ARMThunk::writeTo(uint8_t *buf) {
 823:   if (!getMayUseShortThunk()) {
 824:     writeLong(buf);
 825:     return;
 826:   }
 827: 
 828:   uint64_t s = getARMThunkDestVA(ctx, destination);
 829:   uint64_t p = getThunkTargetSym()->getVA(ctx);
 830:   int64_t offset = s - p - 8;
 831:   write32(ctx, buf, 0xea000000); // b S
 832:   ctx.target->relocateNoSym(buf, R_ARM_JUMP24, offset);
 833: }
 834: 
```

- **L819**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L820**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L821**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L822**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L823**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L824**: Declares function or method \`writeLong\`. / 声明函数或方法 \`writeLong\`。
- **L825**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L826**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L827**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L828**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L829**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L830**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L831**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L832**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L833**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L834**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 835-864 / 第 835-864 行

```cpp
 835: bool ARMThunk::isCompatibleWith(const InputSection &isec,
 836:                                 const Relocation &rel) const {
 837:   // v4T does not have BLX, so also deny R_ARM_THM_CALL
 838:   if (!ctx.arg.armHasBlx && rel.type == R_ARM_THM_CALL)
 839:     return false;
 840: 
 841:   // Thumb branch relocations can't use BLX
 842:   return rel.type != R_ARM_THM_JUMP19 && rel.type != R_ARM_THM_JUMP24;
 843: }
 844: 
 845: // This function returns true if:
 846: // the target is Thumb
 847: // && is within branch range
 848: // && this function has not previously returned false
 849: //    (see comment for mayUseShortThunk)
 850: // && the arch supports Thumb branch range extension.
 851: bool ThumbThunk::getMayUseShortThunk() {
 852:   if (!mayUseShortThunk)
 853:     return false;
 854:   uint64_t s = getARMThunkDestVA(ctx, destination);
 855:   // To use a short thunk the destination must be Thumb and the target must
 856:   // have the wide branch instruction B.w. This instruction is included when
 857:   // Thumb 2 is present, or in v8-M (and above) baseline architectures.
 858:   // armJ1J2BranchEncoding is available in all architectures with a profile and
 859:   // the one v6 CPU that implements Thumb 2 (Arm1156t2-s).
 860:   // Movt and Movw instructions require Thumb 2 or v8-M baseline.
 861:   if ((s & 1) == 0 || !ctx.arg.armJ1J2BranchEncoding ||
 862:       !ctx.arg.armHasMovtMovw) {
 863:     mayUseShortThunk = false;
 864:     addLongMapSyms();
```

- **L835**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L836**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L837**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L838**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L839**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L840**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L841**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L842**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L843**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L844**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L845**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L846**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L847**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L848**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L849**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L850**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L851**: Defines function or method \`getMayUseShortThunk\`. / 定义函数或方法 \`getMayUseShortThunk\`。
- **L852**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L853**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L854**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L855**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L856**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L857**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L858**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L859**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L860**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L861**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L862**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L863**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L864**: Declares function or method \`addLongMapSyms\`. / 声明函数或方法 \`addLongMapSyms\`。

### Lines 865-880 / 第 865-880 行

```cpp
 865:     return false;
 866:   }
 867:   uint64_t p = getThunkTargetSym()->getVA(ctx) & ~1;
 868:   int64_t offset = s - p - 4;
 869:   mayUseShortThunk = llvm::isInt<25>(offset);
 870:   if (!mayUseShortThunk)
 871:     addLongMapSyms();
 872:   return mayUseShortThunk;
 873: }
 874: 
 875: void ThumbThunk::writeTo(uint8_t *buf) {
 876:   if (!getMayUseShortThunk()) {
 877:     writeLong(buf);
 878:     return;
 879:   }
 880: 
```

- **L865**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L866**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L867**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L868**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L869**: Declares function or method \`isInt\`. / 声明函数或方法 \`isInt\`。
- **L870**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L871**: Declares function or method \`addLongMapSyms\`. / 声明函数或方法 \`addLongMapSyms\`。
- **L872**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L873**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L874**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L875**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L876**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L877**: Declares function or method \`writeLong\`. / 声明函数或方法 \`writeLong\`。
- **L878**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L879**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L880**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 881-898 / 第 881-898 行

```cpp
 881:   uint64_t s = getARMThunkDestVA(ctx, destination);
 882:   uint64_t p = getThunkTargetSym()->getVA(ctx);
 883:   int64_t offset = s - p - 4;
 884:   write16(ctx, buf + 0, 0xf000); // b.w S
 885:   write16(ctx, buf + 2, 0xb000);
 886:   ctx.target->relocateNoSym(buf, R_ARM_THM_JUMP24, offset);
 887: }
 888: 
 889: bool ThumbThunk::isCompatibleWith(const InputSection &isec,
 890:                                   const Relocation &rel) const {
 891:   // v4T does not have BLX, so also deny R_ARM_CALL
 892:   if (!ctx.arg.armHasBlx && rel.type == R_ARM_CALL)
 893:     return false;
 894: 
 895:   // ARM branch relocations can't use BLX
 896:   return rel.type != R_ARM_JUMP24 && rel.type != R_ARM_PC24 && rel.type != R_ARM_PLT32;
 897: }
 898: 
```

- **L881**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L882**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L883**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L884**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L885**: Declares function or method \`write16\`. / 声明函数或方法 \`write16\`。
- **L886**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L887**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L888**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L889**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L890**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L891**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L892**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L893**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L894**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L895**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L896**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L897**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L898**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 899-913 / 第 899-913 行

```cpp
 899: void ARMV7ABSLongThunk::writeLong(uint8_t *buf) {
 900:   write32(ctx, buf + 0, 0xe300c000); // movw ip,:lower16:S
 901:   write32(ctx, buf + 4, 0xe340c000); // movt ip,:upper16:S
 902:   write32(ctx, buf + 8, 0xe12fff1c); // bx   ip
 903:   uint64_t s = getARMThunkDestVA(ctx, destination);
 904:   ctx.target->relocateNoSym(buf, R_ARM_MOVW_ABS_NC, s);
 905:   ctx.target->relocateNoSym(buf + 4, R_ARM_MOVT_ABS, s);
 906: }
 907: 
 908: void ARMV7ABSLongThunk::addSymbols(ThunkSection &isec) {
 909:   addSymbol(ctx.saver.save("__ARMv7ABSLongThunk_" + destination.getName()),
 910:             STT_FUNC, 0, isec);
 911:   addSymbol("$a", STT_NOTYPE, 0, isec);
 912: }
 913: 
```

- **L899**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L900**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L901**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L902**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L903**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L904**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L905**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L906**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L907**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L908**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L909**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L910**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L911**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L912**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L913**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 914-930 / 第 914-930 行

```cpp
 914: void ThumbV7ABSLongThunk::writeLong(uint8_t *buf) {
 915:   write16(ctx, buf + 0, 0xf240); // movw ip, :lower16:S
 916:   write16(ctx, buf + 2, 0x0c00);
 917:   write16(ctx, buf + 4, 0xf2c0); // movt ip, :upper16:S
 918:   write16(ctx, buf + 6, 0x0c00);
 919:   write16(ctx, buf + 8, 0x4760); // bx   ip
 920:   uint64_t s = getARMThunkDestVA(ctx, destination);
 921:   ctx.target->relocateNoSym(buf, R_ARM_THM_MOVW_ABS_NC, s);
 922:   ctx.target->relocateNoSym(buf + 4, R_ARM_THM_MOVT_ABS, s);
 923: }
 924: 
 925: void ThumbV7ABSLongThunk::addSymbols(ThunkSection &isec) {
 926:   addSymbol(ctx.saver.save("__Thumbv7ABSLongThunk_" + destination.getName()),
 927:             STT_FUNC, 1, isec);
 928:   addSymbol("$t", STT_NOTYPE, 0, isec);
 929: }
 930: 
```

- **L914**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L915**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L916**: Declares function or method \`write16\`. / 声明函数或方法 \`write16\`。
- **L917**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L918**: Declares function or method \`write16\`. / 声明函数或方法 \`write16\`。
- **L919**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L920**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L921**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L922**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L923**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L924**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L925**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L926**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L927**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L928**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L929**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L930**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 931-950 / 第 931-950 行

```cpp
 931: void ARMV7PILongThunk::writeLong(uint8_t *buf) {
 932:   write32(ctx, buf + 0,
 933:           0xe30fcff0); // P:  movw ip,:lower16:S - (P + (L1-P) + 8)
 934:   write32(ctx, buf + 4,
 935:           0xe340c000); //     movt ip,:upper16:S - (P + (L1-P) + 8)
 936:   write32(ctx, buf + 8, 0xe08cc00f);  // L1: add  ip, ip, pc
 937:   write32(ctx, buf + 12, 0xe12fff1c); //     bx   ip
 938:   uint64_t s = getARMThunkDestVA(ctx, destination);
 939:   uint64_t p = getThunkTargetSym()->getVA(ctx);
 940:   int64_t offset = s - p - 16;
 941:   ctx.target->relocateNoSym(buf, R_ARM_MOVW_PREL_NC, offset);
 942:   ctx.target->relocateNoSym(buf + 4, R_ARM_MOVT_PREL, offset);
 943: }
 944: 
 945: void ARMV7PILongThunk::addSymbols(ThunkSection &isec) {
 946:   addSymbol(ctx.saver.save("__ARMV7PILongThunk_" + destination.getName()),
 947:             STT_FUNC, 0, isec);
 948:   addSymbol("$a", STT_NOTYPE, 0, isec);
 949: }
 950: 
```

- **L931**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L932**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L933**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L934**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L935**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L936**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L937**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L938**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L939**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L940**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L941**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L942**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L943**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L944**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L945**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L946**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L947**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L948**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L949**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L950**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 951-970 / 第 951-970 行

```cpp
 951: void ThumbV7PILongThunk::writeLong(uint8_t *buf) {
 952:   write16(ctx, buf + 0, 0xf64f); // P:  movw ip,:lower16:S - (P + (L1-P) + 4)
 953:   write16(ctx, buf + 2, 0x7cf4);
 954:   write16(ctx, buf + 4, 0xf2c0); //     movt ip,:upper16:S - (P + (L1-P) + 4)
 955:   write16(ctx, buf + 6, 0x0c00);
 956:   write16(ctx, buf + 8, 0x44fc);  // L1: add  ip, pc
 957:   write16(ctx, buf + 10, 0x4760); //     bx   ip
 958:   uint64_t s = getARMThunkDestVA(ctx, destination);
 959:   uint64_t p = getThunkTargetSym()->getVA(ctx) & ~0x1;
 960:   int64_t offset = s - p - 12;
 961:   ctx.target->relocateNoSym(buf, R_ARM_THM_MOVW_PREL_NC, offset);
 962:   ctx.target->relocateNoSym(buf + 4, R_ARM_THM_MOVT_PREL, offset);
 963: }
 964: 
 965: void ThumbV7PILongThunk::addSymbols(ThunkSection &isec) {
 966:   addSymbol(ctx.saver.save("__ThumbV7PILongThunk_" + destination.getName()),
 967:             STT_FUNC, 1, isec);
 968:   addSymbol("$t", STT_NOTYPE, 0, isec);
 969: }
 970: 
```

- **L951**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L952**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L953**: Declares function or method \`write16\`. / 声明函数或方法 \`write16\`。
- **L954**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L955**: Declares function or method \`write16\`. / 声明函数或方法 \`write16\`。
- **L956**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L957**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L958**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L959**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L960**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L961**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L962**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L963**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L964**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L965**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L966**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L967**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L968**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L969**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L970**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 971-992 / 第 971-992 行

```cpp
 971: void ThumbV6MABSLongThunk::writeLong(uint8_t *buf) {
 972:   // Most Thumb instructions cannot access the high registers r8 - r15. As the
 973:   // only register we can corrupt is r12 we must instead spill a low register
 974:   // to the stack to use as a scratch register. We push r1 even though we
 975:   // don't need to get some space to use for the return address.
 976:   write16(ctx, buf + 0, 0xb403); // push {r0, r1} ; Obtain scratch registers
 977:   write16(ctx, buf + 2, 0x4801); // ldr r0, [pc, #4] ; L1
 978:   write16(ctx, buf + 4, 0x9001); // str r0, [sp, #4] ; SP + 4 = S
 979:   write16(ctx, buf + 6, 0xbd01); // pop {r0, pc} ; restore r0 and branch to dest
 980:   write32(ctx, buf + 8, 0x00000000); // L1: .word S
 981:   uint64_t s = getARMThunkDestVA(ctx, destination);
 982:   ctx.target->relocateNoSym(buf + 8, R_ARM_ABS32, s);
 983: }
 984: 
 985: void ThumbV6MABSLongThunk::addSymbols(ThunkSection &isec) {
 986:   addSymbol(ctx.saver.save("__Thumbv6MABSLongThunk_" + destination.getName()),
 987:             STT_FUNC, 1, isec);
 988:   addSymbol("$t", STT_NOTYPE, 0, isec);
 989:   tsec = &isec;
 990:   (void)getMayUseShortThunk();
 991: }
 992: 
```

- **L971**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L972**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L973**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L974**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L975**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L976**: Defines function or method \`write16\`. / 定义函数或方法 \`write16\`。
- **L977**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L978**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L979**: Defines function or method \`write16\`. / 定义函数或方法 \`write16\`。
- **L980**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L981**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L982**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L983**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L984**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L985**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L986**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L987**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L988**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L989**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L990**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L991**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L992**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 993-1019 / 第 993-1019 行

```cpp
 993: void ThumbV6MABSLongThunk::addLongMapSyms() {
 994:   addSymbol("$d", STT_NOTYPE, 8, *tsec);
 995: }
 996: 
 997: void ThumbV6MABSXOLongThunk::writeLong(uint8_t *buf) {
 998:   // Most Thumb instructions cannot access the high registers r8 - r15. As the
 999:   // only register we can corrupt is r12 we must instead spill a low register
1000:   // to the stack to use as a scratch register. We push r1 even though we
1001:   // don't need to get some space to use for the return address.
1002:   write16(ctx, buf + 0, 0xb403);  // push {r0, r1} ; Obtain scratch registers
1003:   write16(ctx, buf + 2, 0x2000);  // movs r0, :upper8_15:S
1004:   write16(ctx, buf + 4, 0x0200);  // lsls r0, r0, #8
1005:   write16(ctx, buf + 6, 0x3000);  // adds r0, :upper0_7:S
1006:   write16(ctx, buf + 8, 0x0200);  // lsls r0, r0, #8
1007:   write16(ctx, buf + 10, 0x3000); // adds r0, :lower8_15:S
1008:   write16(ctx, buf + 12, 0x0200); // lsls r0, r0, #8
1009:   write16(ctx, buf + 14, 0x3000); // adds r0, :lower0_7:S
1010:   write16(ctx, buf + 16, 0x9001); // str r0, [sp, #4] ; SP + 4 = S
1011:   write16(ctx, buf + 18,
1012:           0xbd01); // pop {r0, pc} ; restore r0 and branch to dest
1013:   uint64_t s = getARMThunkDestVA(ctx, destination);
1014:   ctx.target->relocateNoSym(buf + 2, R_ARM_THM_ALU_ABS_G3, s);
1015:   ctx.target->relocateNoSym(buf + 6, R_ARM_THM_ALU_ABS_G2_NC, s);
1016:   ctx.target->relocateNoSym(buf + 10, R_ARM_THM_ALU_ABS_G1_NC, s);
1017:   ctx.target->relocateNoSym(buf + 14, R_ARM_THM_ALU_ABS_G0_NC, s);
1018: }
1019: 
```

- **L993**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L994**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L995**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L996**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L997**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L998**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L999**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1000**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1001**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1002**: Defines function or method \`write16\`. / 定义函数或方法 \`write16\`。
- **L1003**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1004**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1005**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1006**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1007**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1008**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1009**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1010**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1011**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1012**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1013**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1014**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1015**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1016**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1017**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1018**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1019**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1020-1044 / 第 1020-1044 行

```cpp
1020: void ThumbV6MABSXOLongThunk::addSymbols(ThunkSection &isec) {
1021:   addSymbol(ctx.saver.save("__Thumbv6MABSXOLongThunk_" + destination.getName()),
1022:             STT_FUNC, 1, isec);
1023:   addSymbol("$t", STT_NOTYPE, 0, isec);
1024: }
1025: 
1026: void ThumbV6MPILongThunk::writeLong(uint8_t *buf) {
1027:   // Most Thumb instructions cannot access the high registers r8 - r15. As the
1028:   // only register we can corrupt is ip (r12) we must instead spill a low
1029:   // register to the stack to use as a scratch register.
1030:   write16(ctx, buf + 0,
1031:           0xb401); // P:  push {r0}        ; Obtain scratch register
1032:   write16(ctx, buf + 2, 0x4802); //     ldr r0, [pc, #8] ; L2
1033:   write16(ctx, buf + 4, 0x4684); //     mov ip, r0       ; high to low register
1034:   write16(ctx, buf + 6,
1035:           0xbc01); //     pop {r0}         ; restore scratch register
1036:   write16(ctx, buf + 8, 0x44e7); // L1: add pc, ip       ; transfer control
1037:   write16(ctx, buf + 10,
1038:           0x46c0); //     nop              ; pad to 4-byte boundary
1039:   write32(ctx, buf + 12, 0x00000000); // L2: .word S - (P + (L1 - P) + 4)
1040:   uint64_t s = getARMThunkDestVA(ctx, destination);
1041:   uint64_t p = getThunkTargetSym()->getVA(ctx) & ~0x1;
1042:   ctx.target->relocateNoSym(buf + 12, R_ARM_REL32, s - p - 12);
1043: }
1044: 
```

- **L1020**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1021**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1022**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1023**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1024**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1025**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1026**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1027**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1028**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1029**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1030**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1031**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1032**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1033**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1034**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1035**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1036**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1037**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1038**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1039**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1040**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1041**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1042**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1043**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1044**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1045-1063 / 第 1045-1063 行

```cpp
1045: void ThumbV6MPILongThunk::addSymbols(ThunkSection &isec) {
1046:   addSymbol(ctx.saver.save("__Thumbv6MPILongThunk_" + destination.getName()),
1047:             STT_FUNC, 1, isec);
1048:   addSymbol("$t", STT_NOTYPE, 0, isec);
1049:   tsec = &isec;
1050:   (void)getMayUseShortThunk();
1051: }
1052: 
1053: void ThumbV6MPILongThunk::addLongMapSyms() {
1054:   addSymbol("$d", STT_NOTYPE, 12, *tsec);
1055: }
1056: 
1057: void ARMV5LongLdrPcThunk::writeLong(uint8_t *buf) {
1058:   write32(ctx, buf + 0, 0xe51ff004); // ldr pc, [pc,#-4] ; L1
1059:   write32(ctx, buf + 4, 0x00000000); // L1: .word S
1060:   ctx.target->relocateNoSym(buf + 4, R_ARM_ABS32,
1061:                             getARMThunkDestVA(ctx, destination));
1062: }
1063: 
```

- **L1045**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1046**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1047**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1048**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1049**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1050**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1051**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1052**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1053**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1054**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1055**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1056**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1057**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1058**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1059**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1060**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1061**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1062**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1063**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1064-1083 / 第 1064-1083 行

```cpp
1064: void ARMV5LongLdrPcThunk::addSymbols(ThunkSection &isec) {
1065:   addSymbol(ctx.saver.save("__ARMv5LongLdrPcThunk_" + destination.getName()),
1066:             STT_FUNC, 0, isec);
1067:   addSymbol("$a", STT_NOTYPE, 0, isec);
1068:   tsec = &isec;
1069:   (void)getMayUseShortThunk();
1070: }
1071: 
1072: void ARMV5LongLdrPcThunk::addLongMapSyms() {
1073:   addSymbol("$d", STT_NOTYPE, 4, *tsec);
1074: }
1075: 
1076: void ARMV4ABSLongBXThunk::writeLong(uint8_t *buf) {
1077:   write32(ctx, buf + 0, 0xe59fc000); // ldr r12, [pc] ; L1
1078:   write32(ctx, buf + 4, 0xe12fff1c); // bx r12
1079:   write32(ctx, buf + 8, 0x00000000); // L1: .word S
1080:   ctx.target->relocateNoSym(buf + 8, R_ARM_ABS32,
1081:                             getARMThunkDestVA(ctx, destination));
1082: }
1083: 
```

- **L1064**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1065**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1066**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1067**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1068**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1069**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1070**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1071**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1072**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1073**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1074**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1075**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1076**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1077**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1078**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1079**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1080**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1081**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1082**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1083**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1084-1105 / 第 1084-1105 行

```cpp
1084: void ARMV4ABSLongBXThunk::addSymbols(ThunkSection &isec) {
1085:   addSymbol(ctx.saver.save("__ARMv4ABSLongBXThunk_" + destination.getName()),
1086:             STT_FUNC, 0, isec);
1087:   addSymbol("$a", STT_NOTYPE, 0, isec);
1088:   tsec = &isec;
1089:   (void)getMayUseShortThunk();
1090: }
1091: 
1092: void ARMV4ABSLongBXThunk::addLongMapSyms() {
1093:   addSymbol("$d", STT_NOTYPE, 8, *tsec);
1094: }
1095: 
1096: void ThumbV4ABSLongBXThunk::writeLong(uint8_t *buf) {
1097:   write16(ctx, buf + 0, 0x4778); // bx pc
1098:   write16(ctx, buf + 2,
1099:           0xe7fd); // b #-6 ; Arm recommended sequence to follow bx pc
1100:   write32(ctx, buf + 4, 0xe51ff004); // ldr pc, [pc, #-4] ; L1
1101:   write32(ctx, buf + 8, 0x00000000); // L1: .word S
1102:   ctx.target->relocateNoSym(buf + 8, R_ARM_ABS32,
1103:                             getARMThunkDestVA(ctx, destination));
1104: }
1105: 
```

- **L1084**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1085**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1086**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1087**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1088**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1089**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1090**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1091**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1092**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1093**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1094**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1095**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1096**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1097**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1098**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1099**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1100**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1101**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1102**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1103**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1104**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1105**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1106-1129 / 第 1106-1129 行

```cpp
1106: void ThumbV4ABSLongBXThunk::addSymbols(ThunkSection &isec) {
1107:   addSymbol(ctx.saver.save("__Thumbv4ABSLongBXThunk_" + destination.getName()),
1108:             STT_FUNC, 1, isec);
1109:   addSymbol("$t", STT_NOTYPE, 0, isec);
1110:   tsec = &isec;
1111:   (void)getMayUseShortThunk();
1112: }
1113: 
1114: void ThumbV4ABSLongBXThunk::addLongMapSyms() {
1115:   addSymbol("$a", STT_NOTYPE, 4, *tsec);
1116:   addSymbol("$d", STT_NOTYPE, 8, *tsec);
1117: }
1118: 
1119: void ThumbV4ABSLongThunk::writeLong(uint8_t *buf) {
1120:   write16(ctx, buf + 0, 0x4778); // bx pc
1121:   write16(ctx, buf + 2,
1122:           0xe7fd); // b #-6 ; Arm recommended sequence to follow bx pc
1123:   write32(ctx, buf + 4, 0xe59fc000);  // ldr r12, [pc] ; L1
1124:   write32(ctx, buf + 8, 0xe12fff1c);  // bx r12
1125:   write32(ctx, buf + 12, 0x00000000); // L1: .word S
1126:   ctx.target->relocateNoSym(buf + 12, R_ARM_ABS32,
1127:                             getARMThunkDestVA(ctx, destination));
1128: }
1129: 
```

- **L1106**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1107**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1108**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1109**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1110**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1111**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1112**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1113**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1114**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1115**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1116**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1117**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1118**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1119**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1120**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1121**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1122**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1123**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1124**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1125**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1126**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1127**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1128**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1129**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1130-1152 / 第 1130-1152 行

```cpp
1130: void ThumbV4ABSLongThunk::addSymbols(ThunkSection &isec) {
1131:   addSymbol(ctx.saver.save("__Thumbv4ABSLongThunk_" + destination.getName()),
1132:             STT_FUNC, 1, isec);
1133:   addSymbol("$t", STT_NOTYPE, 0, isec);
1134:   tsec = &isec;
1135:   (void)getMayUseShortThunk();
1136: }
1137: 
1138: void ThumbV4ABSLongThunk::addLongMapSyms() {
1139:   addSymbol("$a", STT_NOTYPE, 4, *tsec);
1140:   addSymbol("$d", STT_NOTYPE, 12, *tsec);
1141: }
1142: 
1143: void ARMV4PILongBXThunk::writeLong(uint8_t *buf) {
1144:   write32(ctx, buf + 0, 0xe59fc004);  // P:  ldr ip, [pc,#4] ; L2
1145:   write32(ctx, buf + 4, 0xe08fc00c);  // L1: add ip, pc, ip
1146:   write32(ctx, buf + 8, 0xe12fff1c);  //     bx ip
1147:   write32(ctx, buf + 12, 0x00000000); // L2: .word S - (P + (L1 - P) + 8)
1148:   uint64_t s = getARMThunkDestVA(ctx, destination);
1149:   uint64_t p = getThunkTargetSym()->getVA(ctx) & ~0x1;
1150:   ctx.target->relocateNoSym(buf + 12, R_ARM_REL32, s - p - 12);
1151: }
1152: 
```

- **L1130**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1131**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1132**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1133**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1134**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1135**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1136**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1137**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1138**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1139**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1140**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1141**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1142**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1143**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1144**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1145**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1146**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1147**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1148**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1149**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1150**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1151**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1152**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1153-1173 / 第 1153-1173 行

```cpp
1153: void ARMV4PILongBXThunk::addSymbols(ThunkSection &isec) {
1154:   addSymbol(ctx.saver.save("__ARMv4PILongBXThunk_" + destination.getName()),
1155:             STT_FUNC, 0, isec);
1156:   addSymbol("$a", STT_NOTYPE, 0, isec);
1157:   tsec = &isec;
1158:   (void)getMayUseShortThunk();
1159: }
1160: 
1161: void ARMV4PILongBXThunk::addLongMapSyms() {
1162:   addSymbol("$d", STT_NOTYPE, 12, *tsec);
1163: }
1164: 
1165: void ARMV4PILongThunk::writeLong(uint8_t *buf) {
1166:   write32(ctx, buf + 0, 0xe59fc000); // P:  ldr ip, [pc] ; L2
1167:   write32(ctx, buf + 4, 0xe08ff00c); // L1: add pc, pc, r12
1168:   write32(ctx, buf + 8, 0x00000000); // L2: .word S - (P + (L1 - P) + 8)
1169:   uint64_t s = getARMThunkDestVA(ctx, destination);
1170:   uint64_t p = getThunkTargetSym()->getVA(ctx) & ~0x1;
1171:   ctx.target->relocateNoSym(buf + 8, R_ARM_REL32, s - p - 12);
1172: }
1173: 
```

- **L1153**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1154**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1155**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1156**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1157**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1158**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1159**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1160**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1161**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1162**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1163**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1164**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1165**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1166**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1167**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1168**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1169**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1170**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1171**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1172**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1173**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1174-1197 / 第 1174-1197 行

```cpp
1174: void ARMV4PILongThunk::addSymbols(ThunkSection &isec) {
1175:   addSymbol(ctx.saver.save("__ARMv4PILongThunk_" + destination.getName()),
1176:             STT_FUNC, 0, isec);
1177:   addSymbol("$a", STT_NOTYPE, 0, isec);
1178:   tsec = &isec;
1179:   (void)getMayUseShortThunk();
1180: }
1181: 
1182: void ARMV4PILongThunk::addLongMapSyms() {
1183:   addSymbol("$d", STT_NOTYPE, 8, *tsec);
1184: }
1185: 
1186: void ThumbV4PILongBXThunk::writeLong(uint8_t *buf) {
1187:   write16(ctx, buf + 0, 0x4778); // P:  bx pc
1188:   write16(ctx, buf + 2,
1189:           0xe7fd); //     b #-6 ; Arm recommended sequence to follow bx pc
1190:   write32(ctx, buf + 4, 0xe59fc000);  //     ldr r12, [pc] ; L2
1191:   write32(ctx, buf + 8, 0xe08cf00f);  // L1: add pc, r12, pc
1192:   write32(ctx, buf + 12, 0x00000000); // L2: .word S - (P + (L1 - P) + 8)
1193:   uint64_t s = getARMThunkDestVA(ctx, destination);
1194:   uint64_t p = getThunkTargetSym()->getVA(ctx) & ~0x1;
1195:   ctx.target->relocateNoSym(buf + 12, R_ARM_REL32, s - p - 16);
1196: }
1197: 
```

- **L1174**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1175**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1176**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1177**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1178**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1179**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1180**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1181**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1182**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1183**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1184**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1185**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1186**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1187**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1188**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1189**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1190**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1191**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1192**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1193**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1194**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1195**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1196**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1197**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1198-1223 / 第 1198-1223 行

```cpp
1198: void ThumbV4PILongBXThunk::addSymbols(ThunkSection &isec) {
1199:   addSymbol(ctx.saver.save("__Thumbv4PILongBXThunk_" + destination.getName()),
1200:             STT_FUNC, 1, isec);
1201:   addSymbol("$t", STT_NOTYPE, 0, isec);
1202:   tsec = &isec;
1203:   (void)getMayUseShortThunk();
1204: }
1205: 
1206: void ThumbV4PILongBXThunk::addLongMapSyms() {
1207:   addSymbol("$a", STT_NOTYPE, 4, *tsec);
1208:   addSymbol("$d", STT_NOTYPE, 12, *tsec);
1209: }
1210: 
1211: void ThumbV4PILongThunk::writeLong(uint8_t *buf) {
1212:   write16(ctx, buf + 0, 0x4778); // P:  bx pc
1213:   write16(ctx, buf + 2,
1214:           0xe7fd); //     b #-6 ; Arm recommended sequence to follow bx pc
1215:   write32(ctx, buf + 4, 0xe59fc004);  //     ldr ip, [pc,#4] ; L2
1216:   write32(ctx, buf + 8, 0xe08fc00c);  // L1: add ip, pc, ip
1217:   write32(ctx, buf + 12, 0xe12fff1c); //     bx ip
1218:   write32(ctx, buf + 16, 0x00000000); // L2: .word S - (P + (L1 - P) + 8)
1219:   uint64_t s = getARMThunkDestVA(ctx, destination);
1220:   uint64_t p = getThunkTargetSym()->getVA(ctx) & ~0x1;
1221:   ctx.target->relocateNoSym(buf + 16, R_ARM_REL32, s - p - 16);
1222: }
1223: 
```

- **L1198**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1199**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1200**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1201**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1202**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1203**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1204**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1205**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1206**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1207**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1208**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1209**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1210**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1211**: Defines function or method \`writeLong\`. / 定义函数或方法 \`writeLong\`。
- **L1212**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1213**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1214**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1215**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1216**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1217**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1218**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1219**: Declares function or method \`getARMThunkDestVA\`. / 声明函数或方法 \`getARMThunkDestVA\`。
- **L1220**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1221**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1222**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1223**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1224-1242 / 第 1224-1242 行

```cpp
1224: void ThumbV4PILongThunk::addSymbols(ThunkSection &isec) {
1225:   addSymbol(ctx.saver.save("__Thumbv4PILongThunk_" + destination.getName()),
1226:             STT_FUNC, 1, isec);
1227:   addSymbol("$t", STT_NOTYPE, 0, isec);
1228:   tsec = &isec;
1229:   (void)getMayUseShortThunk();
1230: }
1231: 
1232: void ThumbV4PILongThunk::addLongMapSyms() {
1233:   addSymbol("$a", STT_NOTYPE, 4, *tsec);
1234:   addSymbol("$d", STT_NOTYPE, 16, *tsec);
1235: }
1236: 
1237: // Use the long jump which covers a range up to 8MiB.
1238: void AVRThunk::writeTo(uint8_t *buf) {
1239:   write32(ctx, buf, 0x940c); // jmp func
1240:   ctx.target->relocateNoSym(buf, R_AVR_CALL, destination.getVA(ctx));
1241: }
1242: 
```

- **L1224**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1225**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1226**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1227**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1228**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1229**: Declares function or method \`getMayUseShortThunk\`. / 声明函数或方法 \`getMayUseShortThunk\`。
- **L1230**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1231**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1232**: Defines function or method \`addLongMapSyms\`. / 定义函数或方法 \`addLongMapSyms\`。
- **L1233**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1234**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1235**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1236**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1237**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1238**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1239**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1240**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1241**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1242**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1243-1258 / 第 1243-1258 行

```cpp
1243: void AVRThunk::addSymbols(ThunkSection &isec) {
1244:   addSymbol(ctx.saver.save("__AVRThunk_" + destination.getName()), STT_FUNC, 0,
1245:             isec);
1246: }
1247: 
1248: // Write MIPS LA25 thunk code to call PIC function from the non-PIC one.
1249: void MipsThunk::writeTo(uint8_t *buf) {
1250:   uint64_t s = destination.getVA(ctx);
1251:   write32(ctx, buf, 0x3c190000);                // lui   $25, %hi(func)
1252:   write32(ctx, buf + 4, 0x08000000 | (s >> 2)); // j     func
1253:   write32(ctx, buf + 8, 0x27390000);            // addiu $25, $25, %lo(func)
1254:   write32(ctx, buf + 12, 0x00000000);           // nop
1255:   ctx.target->relocateNoSym(buf, R_MIPS_HI16, s);
1256:   ctx.target->relocateNoSym(buf + 8, R_MIPS_LO16, s);
1257: }
1258: 
```

- **L1243**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1244**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1245**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1246**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1247**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1248**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1249**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1250**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L1251**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1252**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1253**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1254**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1255**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1256**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1257**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1258**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1259-1281 / 第 1259-1281 行

```cpp
1259: void MipsThunk::addSymbols(ThunkSection &isec) {
1260:   addSymbol(ctx.saver.save("__LA25Thunk_" + destination.getName()), STT_FUNC, 0,
1261:             isec);
1262: }
1263: 
1264: InputSection *MipsThunk::getTargetInputSection() const {
1265:   auto &dr = cast<Defined>(destination);
1266:   return dyn_cast<InputSection>(dr.section);
1267: }
1268: 
1269: // Write microMIPS R2-R5 LA25 thunk code
1270: // to call PIC function from the non-PIC one.
1271: void MicroMipsThunk::writeTo(uint8_t *buf) {
1272:   uint64_t s = destination.getVA(ctx);
1273:   write16(ctx, buf, 0x41b9);      // lui   $25, %hi(func)
1274:   write16(ctx, buf + 4, 0xd400);  // j     func
1275:   write16(ctx, buf + 8, 0x3339);  // addiu $25, $25, %lo(func)
1276:   write16(ctx, buf + 12, 0x0c00); // nop
1277:   ctx.target->relocateNoSym(buf, R_MICROMIPS_HI16, s);
1278:   ctx.target->relocateNoSym(buf + 4, R_MICROMIPS_26_S1, s);
1279:   ctx.target->relocateNoSym(buf + 8, R_MICROMIPS_LO16, s);
1280: }
1281: 
```

- **L1259**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1260**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1261**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1262**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1263**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1264**: Defines function or method \`getTargetInputSection\`. / 定义函数或方法 \`getTargetInputSection\`。
- **L1265**: Declares function or method \`cast\`. / 声明函数或方法 \`cast\`。
- **L1266**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1267**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1268**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1269**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1270**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1271**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1272**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L1273**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1274**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1275**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1276**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1277**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1278**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1279**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1280**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1281**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1282-1306 / 第 1282-1306 行

```cpp
1282: void MicroMipsThunk::addSymbols(ThunkSection &isec) {
1283:   Defined *d =
1284:       addSymbol(ctx.saver.save("__microLA25Thunk_" + destination.getName()),
1285:                 STT_FUNC, 0, isec);
1286:   d->stOther |= STO_MIPS_MICROMIPS;
1287: }
1288: 
1289: InputSection *MicroMipsThunk::getTargetInputSection() const {
1290:   auto &dr = cast<Defined>(destination);
1291:   return dyn_cast<InputSection>(dr.section);
1292: }
1293: 
1294: // Write microMIPS R6 LA25 thunk code
1295: // to call PIC function from the non-PIC one.
1296: void MicroMipsR6Thunk::writeTo(uint8_t *buf) {
1297:   uint64_t s = destination.getVA(ctx);
1298:   uint64_t p = getThunkTargetSym()->getVA(ctx);
1299:   write16(ctx, buf, 0x1320);     // lui   $25, %hi(func)
1300:   write16(ctx, buf + 4, 0x3339); // addiu $25, $25, %lo(func)
1301:   write16(ctx, buf + 8, 0x9400); // bc    func
1302:   ctx.target->relocateNoSym(buf, R_MICROMIPS_HI16, s);
1303:   ctx.target->relocateNoSym(buf + 4, R_MICROMIPS_LO16, s);
1304:   ctx.target->relocateNoSym(buf + 8, R_MICROMIPS_PC26_S1, s - p - 12);
1305: }
1306: 
```

- **L1282**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1283**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1284**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1285**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1286**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1287**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1288**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1289**: Defines function or method \`getTargetInputSection\`. / 定义函数或方法 \`getTargetInputSection\`。
- **L1290**: Declares function or method \`cast\`. / 声明函数或方法 \`cast\`。
- **L1291**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1292**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1293**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1294**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1295**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1296**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1297**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L1298**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L1299**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1300**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1301**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1302**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1303**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1304**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1305**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1306**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1307-1336 / 第 1307-1336 行

```cpp
1307: void MicroMipsR6Thunk::addSymbols(ThunkSection &isec) {
1308:   Defined *d =
1309:       addSymbol(ctx.saver.save("__microLA25Thunk_" + destination.getName()),
1310:                 STT_FUNC, 0, isec);
1311:   d->stOther |= STO_MIPS_MICROMIPS;
1312: }
1313: 
1314: InputSection *MicroMipsR6Thunk::getTargetInputSection() const {
1315:   auto &dr = cast<Defined>(destination);
1316:   return dyn_cast<InputSection>(dr.section);
1317: }
1318: 
1319: void elf::writePPC32PltCallStub(Ctx &ctx, uint8_t *buf, uint64_t gotPltVA,
1320:                                 const InputFile *file, int64_t addend) {
1321:   if (!ctx.arg.isPic) {
1322:     write32(ctx, buf + 0, 0x3d600000 | (gotPltVA + 0x8000) >> 16); // lis r11,ha
1323:     write32(ctx, buf + 4, 0x816b0000 | (uint16_t)gotPltVA); // lwz r11,l(r11)
1324:     write32(ctx, buf + 8, 0x7d6903a6);                      // mtctr r11
1325:     write32(ctx, buf + 12, 0x4e800420);                     // bctr
1326:     return;
1327:   }
1328:   uint32_t offset;
1329:   if (addend >= 0x8000) {
1330:     // The stub loads an address relative to r30 (.got2+Addend). Addend is
1331:     // almost always 0x8000. The address of .got2 is different in another object
1332:     // file, so a stub cannot be shared.
1333:     offset = gotPltVA -
1334:              (ctx.in.ppc32Got2->getParent()->getVA() +
1335:               (file->ppc32Got2 ? file->ppc32Got2->outSecOff : 0) + addend);
1336:   } else {
```

- **L1307**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1308**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1309**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1310**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1311**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1312**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1313**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1314**: Defines function or method \`getTargetInputSection\`. / 定义函数或方法 \`getTargetInputSection\`。
- **L1315**: Declares function or method \`cast\`. / 声明函数或方法 \`cast\`。
- **L1316**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1317**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1318**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1319**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1320**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1321**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1322**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1323**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1324**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1325**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1326**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1327**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1328**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1329**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1330**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1331**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1332**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1333**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1334**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1335**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L1336**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1337-1354 / 第 1337-1354 行

```cpp
1337:     // The stub loads an address relative to _GLOBAL_OFFSET_TABLE_ (which is
1338:     // currently the address of .got).
1339:     offset = gotPltVA - ctx.in.got->getVA();
1340:   }
1341:   uint16_t ha = (offset + 0x8000) >> 16, l = (uint16_t)offset;
1342:   if (ha == 0) {
1343:     write32(ctx, buf + 0, 0x817e0000 | l); // lwz r11,l(r30)
1344:     write32(ctx, buf + 4, 0x7d6903a6);     // mtctr r11
1345:     write32(ctx, buf + 8, 0x4e800420);     // bctr
1346:     write32(ctx, buf + 12, 0x60000000);    // nop
1347:   } else {
1348:     write32(ctx, buf + 0, 0x3d7e0000 | ha); // addis r11,r30,ha
1349:     write32(ctx, buf + 4, 0x816b0000 | l);  // lwz r11,l(r11)
1350:     write32(ctx, buf + 8, 0x7d6903a6);      // mtctr r11
1351:     write32(ctx, buf + 12, 0x4e800420);     // bctr
1352:   }
1353: }
1354: 
```

- **L1337**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1338**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1339**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L1340**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1341**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1342**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1343**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1344**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1345**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1346**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1347**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1348**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1349**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1350**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1351**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1352**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1353**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1354**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1355-1372 / 第 1355-1372 行

```cpp
1355: void PPC32PltCallStub::writeTo(uint8_t *buf) {
1356:   writePPC32PltCallStub(ctx, buf, destination.getGotPltVA(ctx), file, addend);
1357: }
1358: 
1359: void PPC32PltCallStub::addSymbols(ThunkSection &isec) {
1360:   std::string buf;
1361:   raw_string_ostream os(buf);
1362:   os << format_hex_no_prefix(addend, 8);
1363:   if (!ctx.arg.isPic)
1364:     os << ".plt_call32.";
1365:   else if (addend >= 0x8000)
1366:     os << ".got2.plt_pic32.";
1367:   else
1368:     os << ".plt_pic32.";
1369:   os << destination.getName();
1370:   addSymbol(ctx.saver.save(buf), STT_FUNC, 0, isec);
1371: }
1372: 
```

- **L1355**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1356**: Declares function or method \`writePPC32PltCallStub\`. / 声明函数或方法 \`writePPC32PltCallStub\`。
- **L1357**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1358**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1359**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1360**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1361**: Declares function or method \`os\`. / 声明函数或方法 \`os\`。
- **L1362**: Declares function or method \`format_hex_no_prefix\`. / 声明函数或方法 \`format_hex_no_prefix\`。
- **L1363**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1364**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1365**: Adds another conditional branch for an alternative predicate. / 为另一种判定条件添加分支。
- **L1366**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1367**: Begins the fallback branch for the preceding conditional logic. / 开始前一个条件逻辑的后备分支。
- **L1368**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1369**: Declares function or method \`getName\`. / 声明函数或方法 \`getName\`。
- **L1370**: Declares function or method \`addSymbol\`. / 声明函数或方法 \`addSymbol\`。
- **L1371**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1372**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1373-1402 / 第 1373-1402 行

```cpp
1373: bool PPC32PltCallStub::isCompatibleWith(const InputSection &isec,
1374:                                         const Relocation &rel) const {
1375:   return !ctx.arg.isPic || (isec.file == file && rel.addend == addend);
1376: }
1377: 
1378: void PPC32LongThunk::addSymbols(ThunkSection &isec) {
1379:   addSymbol(ctx.saver.save("__LongThunk_" + destination.getName()), STT_FUNC, 0,
1380:             isec);
1381: }
1382: 
1383: void PPC32LongThunk::writeTo(uint8_t *buf) {
1384:   auto ha = [](uint32_t v) -> uint16_t { return (v + 0x8000) >> 16; };
1385:   auto lo = [](uint32_t v) -> uint16_t { return v; };
1386:   uint32_t d = destination.getVA(ctx, addend);
1387:   if (ctx.arg.isPic) {
1388:     uint32_t off = d - (getThunkTargetSym()->getVA(ctx) + 8);
1389:     write32(ctx, buf + 0, 0x7c0802a6);            // mflr r12,0
1390:     write32(ctx, buf + 4, 0x429f0005);            // bcl r20,r31,.+4
1391:     write32(ctx, buf + 8, 0x7d8802a6);            // mtctr r12
1392:     write32(ctx, buf + 12, 0x3d8c0000 | ha(off)); // addis r12,r12,off@ha
1393:     write32(ctx, buf + 16, 0x398c0000 | lo(off)); // addi r12,r12,off@l
1394:     write32(ctx, buf + 20, 0x7c0803a6);           // mtlr r0
1395:     buf += 24;
1396:   } else {
1397:     write32(ctx, buf + 0, 0x3d800000 | ha(d)); // lis r12,d@ha
1398:     write32(ctx, buf + 4, 0x398c0000 | lo(d)); // addi r12,r12,d@l
1399:     buf += 8;
1400:   }
1401:   write32(ctx, buf + 0, 0x7d8903a6); // mtctr r12
1402:   write32(ctx, buf + 4, 0x4e800420); // bctr
```

- **L1373**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1374**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1375**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1376**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1377**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1378**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1379**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1380**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1381**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1382**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1383**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1384**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L1385**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L1386**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L1387**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1388**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L1389**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1390**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1391**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1392**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1393**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1394**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1395**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1396**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1397**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1398**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1399**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1400**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1401**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1402**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 1403-1421 / 第 1403-1421 行

```cpp
1403: }
1404: 
1405: void elf::writePPC64LoadAndBranch(Ctx &ctx, uint8_t *buf, int64_t offset) {
1406:   uint16_t offHa = (offset + 0x8000) >> 16;
1407:   uint16_t offLo = offset & 0xffff;
1408: 
1409:   write32(ctx, buf + 0, 0x3d820000 | offHa); // addis r12, r2, OffHa
1410:   write32(ctx, buf + 4, 0xe98c0000 | offLo); // ld    r12, OffLo(r12)
1411:   write32(ctx, buf + 8, 0x7d8903a6);         // mtctr r12
1412:   write32(ctx, buf + 12, 0x4e800420);        // bctr
1413: }
1414: 
1415: void PPC64PltCallStub::writeTo(uint8_t *buf) {
1416:   int64_t offset = destination.getGotPltVA(ctx) - getPPC64TocBase(ctx);
1417:   // Save the TOC pointer to the save-slot reserved in the call frame.
1418:   write32(ctx, buf + 0, 0xf8410018); // std     r2,24(r1)
1419:   writePPC64LoadAndBranch(ctx, buf + 4, offset);
1420: }
1421: 
```

- **L1403**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1404**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1405**: Defines function or method \`writePPC64LoadAndBranch\`. / 定义函数或方法 \`writePPC64LoadAndBranch\`。
- **L1406**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1407**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1408**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1409**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1410**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1411**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1412**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1413**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1414**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1415**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1416**: Declares function or method \`getGotPltVA\`. / 声明函数或方法 \`getGotPltVA\`。
- **L1417**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1418**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1419**: Declares function or method \`writePPC64LoadAndBranch\`. / 声明函数或方法 \`writePPC64LoadAndBranch\`。
- **L1420**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1421**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1422-1451 / 第 1422-1451 行

```cpp
1422: void PPC64PltCallStub::addSymbols(ThunkSection &isec) {
1423:   Defined *s = addSymbol(ctx.saver.save("__plt_" + destination.getName()),
1424:                          STT_FUNC, 0, isec);
1425:   s->setNeedsTocRestore(true);
1426:   s->file = destination.file;
1427: }
1428: 
1429: bool PPC64PltCallStub::isCompatibleWith(const InputSection &isec,
1430:                                         const Relocation &rel) const {
1431:   return rel.type == R_PPC64_REL24 || rel.type == R_PPC64_REL14;
1432: }
1433: 
1434: void PPC64R2SaveStub::writeTo(uint8_t *buf) {
1435:   const int64_t offset = computeOffset();
1436:   write32(ctx, buf + 0, 0xf8410018); // std  r2,24(r1)
1437:   // The branch offset needs to fit in 26 bits.
1438:   if (getMayUseShortThunk()) {
1439:     write32(ctx, buf + 4, 0x48000000 | (offset & 0x03fffffc)); // b    <offset>
1440:   } else if (isInt<34>(offset)) {
1441:     int nextInstOffset;
1442:     uint64_t tocOffset = destination.getVA(ctx) - getPPC64TocBase(ctx);
1443:     if (tocOffset >> 16 > 0) {
1444:       const uint64_t addi = ADDI_R12_TO_R12_NO_DISP | (tocOffset & 0xffff);
1445:       const uint64_t addis =
1446:           ADDIS_R12_TO_R2_NO_DISP | ((tocOffset >> 16) & 0xffff);
1447:       write32(ctx, buf + 4, addis); // addis r12, r2 , top of offset
1448:       write32(ctx, buf + 8, addi);  // addi  r12, r12, bottom of offset
1449:       nextInstOffset = 12;
1450:     } else {
1451:       const uint64_t addi = ADDI_R12_TO_R2_NO_DISP | (tocOffset & 0xffff);
```

- **L1422**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1423**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1424**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1425**: Declares function or method \`setNeedsTocRestore\`. / 声明函数或方法 \`setNeedsTocRestore\`。
- **L1426**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1427**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1428**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1429**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1430**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1431**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1432**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1433**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1434**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1435**: Declares function or method \`computeOffset\`. / 声明函数或方法 \`computeOffset\`。
- **L1436**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1437**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1438**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1439**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1440**: Defines function or method \`if\`. / 定义函数或方法 \`if\`。
- **L1441**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1442**: Declares function or method \`getVA\`. / 声明函数或方法 \`getVA\`。
- **L1443**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1444**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1445**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1446**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L1447**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1448**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1449**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1450**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1451**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。

### Lines 1452-1471 / 第 1452-1471 行

```cpp
1452:       write32(ctx, buf + 4, addi); // addi r12, r2, offset
1453:       nextInstOffset = 8;
1454:     }
1455:     write32(ctx, buf + nextInstOffset, MTCTR_R12); // mtctr r12
1456:     write32(ctx, buf + nextInstOffset + 4, BCTR);  // bctr
1457:   } else {
1458:     ctx.in.ppc64LongBranchTarget->addEntry(&destination, addend);
1459:     const int64_t offsetFromTOC =
1460:         ctx.in.ppc64LongBranchTarget->getEntryVA(&destination, addend) -
1461:         getPPC64TocBase(ctx);
1462:     writePPC64LoadAndBranch(ctx, buf + 4, offsetFromTOC);
1463:   }
1464: }
1465: 
1466: void PPC64R2SaveStub::addSymbols(ThunkSection &isec) {
1467:   Defined *s = addSymbol(ctx.saver.save("__toc_save_" + destination.getName()),
1468:                          STT_FUNC, 0, isec);
1469:   s->setNeedsTocRestore(true);
1470: }
1471: 
```

- **L1452**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1453**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1454**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1455**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1456**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1457**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1458**: Declares function or method \`addEntry\`. / 声明函数或方法 \`addEntry\`。
- **L1459**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1460**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1461**: Declares function or method \`getPPC64TocBase\`. / 声明函数或方法 \`getPPC64TocBase\`。
- **L1462**: Declares function or method \`writePPC64LoadAndBranch\`. / 声明函数或方法 \`writePPC64LoadAndBranch\`。
- **L1463**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1464**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1465**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1466**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1467**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1468**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1469**: Declares function or method \`setNeedsTocRestore\`. / 声明函数或方法 \`setNeedsTocRestore\`。
- **L1470**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1471**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1472-1501 / 第 1472-1501 行

```cpp
1472: bool PPC64R2SaveStub::isCompatibleWith(const InputSection &isec,
1473:                                        const Relocation &rel) const {
1474:   return rel.type == R_PPC64_REL24 || rel.type == R_PPC64_REL14;
1475: }
1476: 
1477: void PPC64R12SetupStub::writeTo(uint8_t *buf) {
1478:   int64_t offset =
1479:       (gotPlt ? destination.getGotPltVA(ctx) : destination.getVA(ctx)) -
1480:       getThunkTargetSym()->getVA(ctx);
1481:   if (!isInt<34>(offset))
1482:     reportRangeError(ctx, buf, offset, 34, destination,
1483:                      "R12 setup stub offset");
1484: 
1485:   int nextInstOffset;
1486:   if (ctx.arg.power10Stubs) {
1487:     const uint64_t imm = (((offset >> 16) & 0x3ffff) << 32) | (offset & 0xffff);
1488:     // pld 12, func@plt@pcrel or  paddi r12, 0, func@pcrel
1489:     writePrefixedInst(ctx, buf,
1490:                       (gotPlt ? PLD_R12_NO_DISP : PADDI_R12_NO_DISP) | imm);
1491:     nextInstOffset = 8;
1492:   } else {
1493:     uint32_t off = offset - 8;
1494:     write32(ctx, buf + 0, 0x7d8802a6);  // mflr 12
1495:     write32(ctx, buf + 4, 0x429f0005);  // bcl 20,31,.+4
1496:     write32(ctx, buf + 8, 0x7d6802a6);  // mflr 11
1497:     write32(ctx, buf + 12, 0x7d8803a6); // mtlr 12
1498:     write32(ctx, buf + 16,
1499:             0x3d8b0000 | ((off + 0x8000) >> 16)); // addis 12,11,off@ha
1500:     if (gotPlt)
1501:       write32(ctx, buf + 20, 0xe98c0000 | (off & 0xffff)); // ld 12, off@l(12)
```

- **L1472**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1473**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1474**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1475**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1476**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1477**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1478**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1479**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1480**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L1481**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1482**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1483**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1484**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1485**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1486**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1487**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1488**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1489**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1490**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L1491**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1492**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1493**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1494**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1495**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1496**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1497**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1498**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1499**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1500**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1501**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 1502-1520 / 第 1502-1520 行

```cpp
1502:     else
1503:       write32(ctx, buf + 20, 0x398c0000 | (off & 0xffff)); // addi 12,12,off@l
1504:     nextInstOffset = 24;
1505:   }
1506:   write32(ctx, buf + nextInstOffset, MTCTR_R12); // mtctr r12
1507:   write32(ctx, buf + nextInstOffset + 4, BCTR);  // bctr
1508: }
1509: 
1510: void PPC64R12SetupStub::addSymbols(ThunkSection &isec) {
1511:   addSymbol(ctx.saver.save((gotPlt ? "__plt_pcrel_" : "__gep_setup_") +
1512:                            destination.getName()),
1513:             STT_FUNC, 0, isec);
1514: }
1515: 
1516: bool PPC64R12SetupStub::isCompatibleWith(const InputSection &isec,
1517:                                          const Relocation &rel) const {
1518:   return rel.type == R_PPC64_REL24_NOTOC;
1519: }
1520: 
```

- **L1502**: Begins the fallback branch for the preceding conditional logic. / 开始前一个条件逻辑的后备分支。
- **L1503**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1504**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1505**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1506**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1507**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1508**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1509**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1510**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1511**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1512**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1513**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1514**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1515**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1516**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1517**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1518**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1519**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1520**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1521-1537 / 第 1521-1537 行

```cpp
1521: void PPC64LongBranchThunk::writeTo(uint8_t *buf) {
1522:   int64_t offset =
1523:       ctx.in.ppc64LongBranchTarget->getEntryVA(&destination, addend) -
1524:       getPPC64TocBase(ctx);
1525:   writePPC64LoadAndBranch(ctx, buf, offset);
1526: }
1527: 
1528: void PPC64LongBranchThunk::addSymbols(ThunkSection &isec) {
1529:   addSymbol(ctx.saver.save("__long_branch_" + destination.getName()), STT_FUNC,
1530:             0, isec);
1531: }
1532: 
1533: bool PPC64LongBranchThunk::isCompatibleWith(const InputSection &isec,
1534:                                             const Relocation &rel) const {
1535:   return rel.type == R_PPC64_REL24 || rel.type == R_PPC64_REL14;
1536: }
1537: 
```

- **L1521**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1522**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1523**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1524**: Declares function or method \`getPPC64TocBase\`. / 声明函数或方法 \`getPPC64TocBase\`。
- **L1525**: Declares function or method \`writePPC64LoadAndBranch\`. / 声明函数或方法 \`writePPC64LoadAndBranch\`。
- **L1526**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1527**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1528**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1529**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1530**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1531**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1532**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1533**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1534**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1535**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1536**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1537**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1538-1553 / 第 1538-1553 行

```cpp
1538: // Hexagon Target Thunks
1539: static uint64_t getHexagonThunkDestVA(Ctx &ctx, const Symbol &s, int64_t a) {
1540:   uint64_t v = s.isInPlt(ctx) ? s.getPltVA(ctx) : s.getVA(ctx, a);
1541:   return SignExtend64<32>(v);
1542: }
1543: 
1544: void HexagonThunk::writeTo(uint8_t *buf) {
1545:   uint64_t s = getHexagonThunkDestVA(ctx, destination, addend);
1546:   uint64_t p = getThunkTargetSym()->getVA(ctx);
1547: 
1548:   if (ctx.arg.isPic) {
1549:     write32(ctx, buf + 0, 0x00004000); // {  immext(#0)
1550:     ctx.target->relocateNoSym(buf, R_HEX_B32_PCREL_X, s - p);
1551:     write32(ctx, buf + 4, 0x6a49c00e); //    r14 = add(pc,##0) }
1552:     ctx.target->relocateNoSym(buf + 4, R_HEX_6_PCREL_X, s - p);
1553: 
```

- **L1538**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1539**: Defines function or method \`getHexagonThunkDestVA\`. / 定义函数或方法 \`getHexagonThunkDestVA\`。
- **L1540**: Declares function or method \`isInPlt\`. / 声明函数或方法 \`isInPlt\`。
- **L1541**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1542**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1543**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1544**: Defines function or method \`writeTo\`. / 定义函数或方法 \`writeTo\`。
- **L1545**: Declares function or method \`getHexagonThunkDestVA\`. / 声明函数或方法 \`getHexagonThunkDestVA\`。
- **L1546**: Declares function or method \`getThunkTargetSym\`. / 声明函数或方法 \`getThunkTargetSym\`。
- **L1547**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1548**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1549**: Defines function or method \`write32\`. / 定义函数或方法 \`write32\`。
- **L1550**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1551**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1552**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1553**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1554-1570 / 第 1554-1570 行

```cpp
1554:     write32(ctx, buf + 8, 0x528ec000); // {  jumpr r14 }
1555:   } else {
1556:     write32(ctx, buf + 0, 0x00004000); //  { immext
1557:     ctx.target->relocateNoSym(buf, R_HEX_B32_PCREL_X, s - p);
1558:     write32(ctx, buf + 4, 0x5800c000); //    jump <> }
1559:     ctx.target->relocateNoSym(buf + 4, R_HEX_B22_PCREL_X, s - p);
1560:   }
1561: }
1562: void HexagonThunk::addSymbols(ThunkSection &isec) {
1563:   Symbol *enclosing = isec.getEnclosingSymbol(relOffset);
1564:   StringRef src = enclosing ? enclosing->getName() : isec.name;
1565: 
1566:   addSymbol(
1567:       saver().save("__hexagon_thunk_" + destination.getName() + "_from_" + src),
1568:       STT_FUNC, 0, isec);
1569: }
1570: 
```

- **L1554**: Defines function or method \`write32\`. / 定义函数或方法 \`write32\`。
- **L1555**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1556**: Defines function or method \`write32\`. / 定义函数或方法 \`write32\`。
- **L1557**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1558**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1559**: Declares function or method \`relocateNoSym\`. / 声明函数或方法 \`relocateNoSym\`。
- **L1560**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1561**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1562**: Defines function or method \`addSymbols\`. / 定义函数或方法 \`addSymbols\`。
- **L1563**: Declares function or method \`getEnclosingSymbol\`. / 声明函数或方法 \`getEnclosingSymbol\`。
- **L1564**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1565**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1566**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1567**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1568**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1569**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1570**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1571-1593 / 第 1571-1593 行

```cpp
1571: Thunk::Thunk(Ctx &ctx, Symbol &d, int64_t a)
1572:     : ctx(ctx), destination(d), addend(a), offset(0) {
1573:   destination.thunkAccessed = true;
1574: }
1575: 
1576: Thunk::~Thunk() = default;
1577: 
1578: static std::unique_ptr<Thunk> addThunkAArch64(Ctx &ctx, const InputSection &sec,
1579:                                               RelType type, Symbol &s,
1580:                                               int64_t a) {
1581:   assert(is_contained({R_AARCH64_CALL26, R_AARCH64_JUMP26, R_AARCH64_PLT32},
1582:                       type));
1583:   bool mayNeedLandingPad =
1584:       (ctx.arg.andFeatures & GNU_PROPERTY_AARCH64_FEATURE_1_BTI) &&
1585:       !isAArch64BTILandingPad(ctx, s, a);
1586:   if (ctx.arg.picThunk)
1587:     return std::make_unique<AArch64ADRPThunk>(ctx, s, a, mayNeedLandingPad);
1588:   if (sec.getParent()->flags & SHF_AARCH64_PURECODE)
1589:     return std::make_unique<AArch64ABSXOLongThunk>(ctx, s, a,
1590:                                                    mayNeedLandingPad);
1591:   return std::make_unique<AArch64ABSLongThunk>(ctx, s, a, mayNeedLandingPad);
1592: }
1593: 
```

- **L1571**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1572**: Defines function or method \`ctx\`. / 定义函数或方法 \`ctx\`。
- **L1573**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1574**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1575**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1576**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1577**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1578**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1579**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1580**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1581**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1582**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1583**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1584**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1585**: Declares function or method \`isAArch64BTILandingPad\`. / 声明函数或方法 \`isAArch64BTILandingPad\`。
- **L1586**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1587**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1588**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1589**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1590**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1591**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1592**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1593**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1594-1623 / 第 1594-1623 行

```cpp
1594: // Creates a thunk for long branches or Thumb-ARM interworking.
1595: // Arm Architectures v4t does not support Thumb2 technology, and does not
1596: // support BLX or LDR Arm/Thumb state switching. This means that
1597: // - MOVT and MOVW instructions cannot be used.
1598: // - We can't rewrite BL in place to BLX. We will need thunks.
1599: //
1600: // TODO: use B for short Thumb->Arm thunks instead of LDR (this doesn't work for
1601: //       Arm->Thumb, as in Arm state no BX PC trick; it doesn't switch state).
1602: static std::unique_ptr<Thunk> addThunkArmv4(Ctx &ctx, RelType reloc, Symbol &s,
1603:                                             int64_t a) {
1604:   bool thumb_target = s.getVA(ctx, a) & 1;
1605: 
1606:   switch (reloc) {
1607:   case R_ARM_PC24:
1608:   case R_ARM_PLT32:
1609:   case R_ARM_JUMP24:
1610:   case R_ARM_CALL:
1611:     if (ctx.arg.picThunk) {
1612:       if (thumb_target)
1613:         return std::make_unique<ARMV4PILongBXThunk>(ctx, s, a);
1614:       return std::make_unique<ARMV4PILongThunk>(ctx, s, a);
1615:     }
1616:     if (thumb_target)
1617:       return std::make_unique<ARMV4ABSLongBXThunk>(ctx, s, a);
1618:     return std::make_unique<ARMV5LongLdrPcThunk>(ctx, s, a);
1619:   case R_ARM_THM_CALL:
1620:     if (ctx.arg.picThunk) {
1621:       if (thumb_target)
1622:         return std::make_unique<ThumbV4PILongThunk>(ctx, s, a);
1623:       return std::make_unique<ThumbV4PILongBXThunk>(ctx, s, a);
```

- **L1594**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1595**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1596**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1597**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1598**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1599**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1600**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1601**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1602**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1603**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1604**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1605**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1606**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L1607**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1608**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1609**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1610**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1611**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1612**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1613**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1614**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1615**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1616**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1617**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1618**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1619**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1620**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1621**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1622**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1623**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。

### Lines 1624-1653 / 第 1624-1653 行

```cpp
1624:     }
1625:     if (thumb_target)
1626:       return std::make_unique<ThumbV4ABSLongThunk>(ctx, s, a);
1627:     return std::make_unique<ThumbV4ABSLongBXThunk>(ctx, s, a);
1628:   }
1629:   Fatal(ctx) << "relocation " << reloc << " to " << &s
1630:              << " not supported for Armv4 or Armv4T target";
1631:   llvm_unreachable("");
1632: }
1633: 
1634: // Creates a thunk for Thumb-ARM interworking compatible with Armv5 and Armv6.
1635: // Arm Architectures v5 and v6 do not support Thumb2 technology. This means that
1636: // - MOVT and MOVW instructions cannot be used
1637: // - Only Thumb relocation that can generate a Thunk is a BL, this can always
1638: //   be transformed into a BLX
1639: static std::unique_ptr<Thunk> addThunkArmv5v6(Ctx &ctx, RelType reloc,
1640:                                               Symbol &s, int64_t a) {
1641:   switch (reloc) {
1642:   case R_ARM_PC24:
1643:   case R_ARM_PLT32:
1644:   case R_ARM_JUMP24:
1645:   case R_ARM_CALL:
1646:   case R_ARM_THM_CALL:
1647:     if (ctx.arg.picThunk)
1648:       return std::make_unique<ARMV4PILongBXThunk>(ctx, s, a);
1649:     return std::make_unique<ARMV5LongLdrPcThunk>(ctx, s, a);
1650:   }
1651:   Fatal(ctx) << "relocation " << reloc << " to " << &s
1652:              << " not supported for Armv5 or Armv6 targets";
1653:   llvm_unreachable("");
```

- **L1624**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1625**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1626**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1627**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1628**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1629**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1630**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1631**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。
- **L1632**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1633**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1634**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1635**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1636**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1637**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1638**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1639**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1640**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1641**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L1642**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1643**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1644**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1645**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1646**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1647**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1648**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1649**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1650**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1651**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1652**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1653**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。

### Lines 1654-1671 / 第 1654-1671 行

```cpp
1654: }
1655: 
1656: // Create a thunk for Thumb long branch on V6-M.
1657: // Arm Architecture v6-M only supports Thumb instructions. This means
1658: // - MOVT and MOVW instructions cannot be used.
1659: // - Only a limited number of instructions can access registers r8 and above
1660: // - No interworking support is needed (all Thumb).
1661: static std::unique_ptr<Thunk> addThunkV6M(Ctx &ctx, const InputSection &isec,
1662:                                           RelType reloc, Symbol &s, int64_t a) {
1663:   const bool isPureCode = isec.getParent()->flags & SHF_ARM_PURECODE;
1664:   switch (reloc) {
1665:   case R_ARM_THM_JUMP19:
1666:   case R_ARM_THM_JUMP24:
1667:   case R_ARM_THM_CALL:
1668:     if (ctx.arg.isPic) {
1669:       if (!isPureCode)
1670:         return std::make_unique<ThumbV6MPILongThunk>(ctx, s, a);
1671: 
```

- **L1654**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1655**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1656**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1657**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1658**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1659**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1660**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1661**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1662**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1663**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1664**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L1665**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1666**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1667**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1668**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1669**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1670**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1671**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1672-1686 / 第 1672-1686 行

```cpp
1672:       Fatal(ctx)
1673:           << "relocation " << reloc << " to " << &s
1674:           << " not supported for Armv6-M targets for position independent"
1675:              " and execute only code";
1676:       llvm_unreachable("");
1677:     }
1678:     if (isPureCode)
1679:       return std::make_unique<ThumbV6MABSXOLongThunk>(ctx, s, a);
1680:     return std::make_unique<ThumbV6MABSLongThunk>(ctx, s, a);
1681:   }
1682:   Fatal(ctx) << "relocation " << reloc << " to " << &s
1683:              << " not supported for Armv6-M targets";
1684:   llvm_unreachable("");
1685: }
1686: 
```

- **L1672**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1673**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1674**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1675**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1676**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。
- **L1677**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1678**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1679**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1680**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1681**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1682**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1683**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1684**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。
- **L1685**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1686**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1687-1702 / 第 1687-1702 行

```cpp
1687: // Creates a thunk for Thumb-ARM interworking or branch range extension.
1688: static std::unique_ptr<Thunk> addThunkArm(Ctx &ctx, const InputSection &isec,
1689:                                           RelType reloc, Symbol &s, int64_t a) {
1690:   // Decide which Thunk is needed based on:
1691:   // Available instruction set
1692:   // - An Arm Thunk can only be used if Arm state is available.
1693:   // - A Thumb Thunk can only be used if Thumb state is available.
1694:   // - Can only use a Thunk if it uses instructions that the Target supports.
1695:   // Relocation is branch or branch and link
1696:   // - Branch instructions cannot change state, can only select Thunk that
1697:   //   starts in the same state as the caller.
1698:   // - Branch and link relocations can change state, can select Thunks from
1699:   //   either Arm or Thumb.
1700:   // Position independent Thunks if we require position independent code.
1701:   // Execute Only Thunks if the output section is execute only code.
1702: 
```

- **L1687**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1688**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1689**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1690**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1691**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1692**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1693**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1694**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1695**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1696**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1697**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1698**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1699**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1700**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1701**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1702**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1703-1732 / 第 1703-1732 行

```cpp
1703:   // Handle architectures that have restrictions on the instructions that they
1704:   // can use in Thunks. The flags below are set by reading the BuildAttributes
1705:   // of the input objects. InputFiles.cpp contains the mapping from ARM
1706:   // architecture to flag.
1707:   if (!ctx.arg.armHasMovtMovw) {
1708:     if (ctx.arg.armJ1J2BranchEncoding)
1709:       return addThunkV6M(ctx, isec, reloc, s, a);
1710:     if (ctx.arg.armHasBlx)
1711:       return addThunkArmv5v6(ctx, reloc, s, a);
1712:     return addThunkArmv4(ctx, reloc, s, a);
1713:   }
1714: 
1715:   switch (reloc) {
1716:   case R_ARM_PC24:
1717:   case R_ARM_PLT32:
1718:   case R_ARM_JUMP24:
1719:   case R_ARM_CALL:
1720:     if (ctx.arg.picThunk)
1721:       return std::make_unique<ARMV7PILongThunk>(ctx, s, a);
1722:     return std::make_unique<ARMV7ABSLongThunk>(ctx, s, a);
1723:   case R_ARM_THM_JUMP19:
1724:   case R_ARM_THM_JUMP24:
1725:   case R_ARM_THM_CALL:
1726:     if (ctx.arg.picThunk)
1727:       return std::make_unique<ThumbV7PILongThunk>(ctx, s, a);
1728:     return std::make_unique<ThumbV7ABSLongThunk>(ctx, s, a);
1729:   }
1730:   llvm_unreachable("");
1731: }
1732: 
```

- **L1703**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1704**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1705**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1706**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1707**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1708**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1709**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1710**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1711**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1712**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1713**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1714**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1715**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L1716**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1717**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1718**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1719**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1720**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1721**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1722**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1723**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1724**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1725**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1726**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1727**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1728**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1729**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1730**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。
- **L1731**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1732**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1733-1761 / 第 1733-1761 行

```cpp
1733: static std::unique_ptr<Thunk> addThunkAVR(Ctx &ctx, RelType type, Symbol &s,
1734:                                           int64_t a) {
1735:   switch (type) {
1736:   case R_AVR_LO8_LDI_GS:
1737:   case R_AVR_HI8_LDI_GS:
1738:     return std::make_unique<AVRThunk>(ctx, s, a);
1739:   default:
1740:     llvm_unreachable("");
1741:   }
1742: }
1743: 
1744: static std::unique_ptr<Thunk> addThunkHexagon(Ctx &ctx,
1745:                                               const InputSection &isec,
1746:                                               Relocation &rel, Symbol &s) {
1747:   switch (rel.type) {
1748:   case R_HEX_B9_PCREL:
1749:   case R_HEX_B13_PCREL:
1750:   case R_HEX_B15_PCREL:
1751:   case R_HEX_B22_PCREL:
1752:   case R_HEX_PLT_B22_PCREL:
1753:   case R_HEX_GD_PLT_B22_PCREL:
1754:     return std::make_unique<HexagonThunk>(ctx, isec, rel, s);
1755:   default:
1756:     Fatal(ctx) << "unrecognized relocation " << rel.type << " to " << &s
1757:                << " for hexagon target";
1758:     llvm_unreachable("");
1759:   }
1760: }
1761: 
```

- **L1733**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1734**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1735**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L1736**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1737**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1738**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1739**: Marks the default \`switch\` alternative. / 标记 \`switch\` 语句中的默认分支。
- **L1740**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。
- **L1741**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1742**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1743**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1744**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1745**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1746**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1747**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L1748**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1749**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1750**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1751**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1752**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1753**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1754**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1755**: Marks the default \`switch\` alternative. / 标记 \`switch\` 语句中的默认分支。
- **L1756**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1757**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1758**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。
- **L1759**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1760**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1761**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1762-1779 / 第 1762-1779 行

```cpp
1762: static std::unique_ptr<Thunk> addThunkMips(Ctx &ctx, RelType type, Symbol &s) {
1763:   if ((s.stOther & STO_MIPS_MICROMIPS) && isMipsR6(ctx))
1764:     return std::make_unique<MicroMipsR6Thunk>(ctx, s);
1765:   if (s.stOther & STO_MIPS_MICROMIPS)
1766:     return std::make_unique<MicroMipsThunk>(ctx, s);
1767:   return std::make_unique<MipsThunk>(ctx, s);
1768: }
1769: 
1770: static std::unique_ptr<Thunk> addThunkPPC32(Ctx &ctx, const InputSection &isec,
1771:                                             const Relocation &rel, Symbol &s) {
1772:   assert((rel.type == R_PPC_LOCAL24PC || rel.type == R_PPC_REL24 ||
1773:           rel.type == R_PPC_PLTREL24) &&
1774:          "unexpected relocation type for thunk");
1775:   if (s.isInPlt(ctx))
1776:     return std::make_unique<PPC32PltCallStub>(ctx, isec, rel, s);
1777:   return std::make_unique<PPC32LongThunk>(ctx, s, rel.addend);
1778: }
1779: 
```

- **L1762**: Defines function or method \`addThunkMips\`. / 定义函数或方法 \`addThunkMips\`。
- **L1763**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1764**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1765**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1766**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1767**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1768**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1769**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1770**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1771**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1772**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1773**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1774**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1775**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1776**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1777**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1778**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1779**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1780-1797 / 第 1780-1797 行

```cpp
1780: static std::unique_ptr<Thunk> addThunkPPC64(Ctx &ctx, RelType type, Symbol &s,
1781:                                             int64_t a) {
1782:   assert((type == R_PPC64_REL14 || type == R_PPC64_REL24 ||
1783:           type == R_PPC64_REL24_NOTOC) &&
1784:          "unexpected relocation type for thunk");
1785: 
1786:   // If we are emitting stubs for NOTOC relocations, we need to tell
1787:   // the PLT resolver that there can be multiple TOCs.
1788:   if (type == R_PPC64_REL24_NOTOC)
1789:     ctx.target->ppc64DynamicSectionOpt = 0x2;
1790: 
1791:   if (s.isInPlt(ctx)) {
1792:     if (type == R_PPC64_REL24_NOTOC)
1793:       return std::make_unique<PPC64R12SetupStub>(ctx, s,
1794:                                                  /*gotPlt=*/true);
1795:     return std::make_unique<PPC64PltCallStub>(ctx, s);
1796:   }
1797: 
```

- **L1780**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1781**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1782**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1783**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1784**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1785**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1786**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1787**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1788**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1789**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1790**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1791**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1792**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1793**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1794**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1795**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1796**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1797**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1798-1812 / 第 1798-1812 行

```cpp
1798:   // This check looks at the st_other bits of the callee. If the value is 1
1799:   // then the callee clobbers the TOC and we need an R2 save stub when RelType
1800:   // is R_PPC64_REL14 or R_PPC64_REL24.
1801:   if ((type == R_PPC64_REL14 || type == R_PPC64_REL24) && (s.stOther >> 5) == 1)
1802:     return std::make_unique<PPC64R2SaveStub>(ctx, s, a);
1803: 
1804:   if (type == R_PPC64_REL24_NOTOC)
1805:     return std::make_unique<PPC64R12SetupStub>(ctx, s, /*gotPlt=*/false);
1806: 
1807:   if (ctx.arg.picThunk)
1808:     return std::make_unique<PPC64PILongBranchThunk>(ctx, s, a);
1809: 
1810:   return std::make_unique<PPC64PDLongBranchThunk>(ctx, s, a);
1811: }
1812: 
```

- **L1798**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1799**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1800**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1801**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1802**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1803**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1804**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1805**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1806**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1807**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L1808**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1809**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1810**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1811**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1812**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1813-1838 / 第 1813-1838 行

```cpp
1813: std::unique_ptr<Thunk> elf::addThunk(Ctx &ctx, const InputSection &isec,
1814:                                      Relocation &rel) {
1815:   Symbol &s = *rel.sym;
1816:   int64_t a = rel.addend;
1817: 
1818:   switch (ctx.arg.emachine) {
1819:   case EM_AARCH64:
1820:     return addThunkAArch64(ctx, isec, rel.type, s, a);
1821:   case EM_ARM:
1822:     return addThunkArm(ctx, isec, rel.type, s, a);
1823:   case EM_AVR:
1824:     return addThunkAVR(ctx, rel.type, s, a);
1825:   case EM_MIPS:
1826:     return addThunkMips(ctx, rel.type, s);
1827:   case EM_PPC:
1828:     return addThunkPPC32(ctx, isec, rel, s);
1829:   case EM_PPC64:
1830:     return addThunkPPC64(ctx, rel.type, s, a);
1831:   case EM_HEXAGON:
1832:     return addThunkHexagon(ctx, isec, rel, s);
1833:   default:
1834:     llvm_unreachable(
1835:         "add Thunk only supported for ARM, AVR, Hexagon, Mips and PowerPC");
1836:   }
1837: }
1838: 
```

- **L1813**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L1814**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1815**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1816**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L1817**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1818**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L1819**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1820**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1821**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1822**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1823**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1824**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1825**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1826**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1827**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1828**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1829**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1830**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1831**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1832**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1833**: Marks the default \`switch\` alternative. / 标记 \`switch\` 语句中的默认分支。
- **L1834**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1835**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1836**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1837**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1838**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1839-1846 / 第 1839-1846 行

```cpp
1839: std::unique_ptr<Thunk> elf::addLandingPadThunk(Ctx &ctx, Symbol &s, int64_t a) {
1840:   switch (ctx.arg.emachine) {
1841:   case EM_AARCH64:
1842:     return std::make_unique<AArch64BTILandingPadThunk>(ctx, s, a);
1843:   default:
1844:     llvm_unreachable("add landing pad only supported for AArch64");
1845:   }
1846: }
```

- **L1839**: Defines function or method \`addLandingPadThunk\`. / 定义函数或方法 \`addLandingPadThunk\`。
- **L1840**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L1841**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L1842**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L1843**: Marks the default \`switch\` alternative. / 标记 \`switch\` 语句中的默认分支。
- **L1844**: Declares function or method \`llvm_unreachable\`. / 声明函数或方法 \`llvm_unreachable\`。
- **L1845**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1846**: Closes the current scope or body. / 关闭当前作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This file contains Thunk subclasses. / 实现 ELF 链接器逻辑，例如符号解析、节布局、重定位以及驱动行为。
- **Scale / 规模**: 1846 lines, 15 direct includes, 24 named types, and 40 detected routines. / 共 1846 行，含 15 个直接包含、24 个具名类型、40 个检测到的例程。
- **ELF linking / ELF 链接**: The code works on ELF symbols, sections, relocations, or output layout. / 该代码处理 ELF 符号、节、重定位或输出布局。
- **Relocation processing / 重定位处理**: The implementation reasons about relocation records, fixups, or address adjustments. / 该实现处理重定位记录、修正项或地址调整。
- **Symbol management / 符号管理**: The code tracks declarations, references, visibility, or resolution of symbols. / 该代码跟踪符号的声明、引用、可见性或解析过程。
- **Section/segment layout / 节/段布局**: The file manipulates input or output sections, segments, or related layout state. / 该文件操作输入/输出节、段或相关布局状态。
- **Device/plugin integration / 设备/插件集成**: The implementation coordinates backend plugins, device state, or host-device interaction. / 该实现协调后端插件、设备状态或主机-设备交互。
- **Pass orchestration / Pass 编排**: The file defines or wires analysis/transform passes into a larger pipeline. / 该文件定义或接入分析/变换 Pass 到更大的流水线中。
- **Rich type modeling / 丰富的类型建模**: The file introduces or manipulates named types that capture subsystem state. / 该文件引入或操作用于表达子系统状态的具名类型。

## Dependencies / 依赖关系

- **LLVM / LLVM**: `llvm/BinaryFormat/ELF.h`, `llvm/Support/Casting.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/MathExtras.h`.
- **lld / lld**: `lld/Common/CommonLinkerContext.h`.
- **System or local / 系统或本地**: `Thunks.h`, `Config.h`, `InputFiles.h`, `InputSection.h`, `OutputSections.h`, `Symbols.h`, `SyntheticSections.h`, `Target.h`, `cstdint`, `cstring`.
- **Header roles / 头文件角色**: standard-library or local support header / 标准库或本地支持头文件 (10), support-library helpers / Support 库辅助功能 (3), lld shared linker infrastructure / lld 共享链接基础设施 (1), binary-format constants and helpers / 二进制格式常量与辅助工具 (1).
- **Core types / 核心类型**: `for`, `tries`, `AArch64Thunk`, `AArch64ABSLongThunk`, `AArch64ABSXOLongThunk`, `AArch64ADRPThunk`, `AArch64BTILandingPadThunk`, `ARMThunk`, `is`, `ThumbThunk`, `ARMV7ABSLongThunk`, `ARMV7PILongThunk`.
- **Visible routines / 可见例程**: `Thunk`, `getMayUseShortThunk`, `addLongMapSyms`, `AArch64Thunk`, `writeLong`, `ARMThunk`, `ThumbThunk`, `AVRThunk`, `MipsThunk`, `MicroMipsThunk`, `MicroMipsR6Thunk`, `PPC64PltCallStub`.
