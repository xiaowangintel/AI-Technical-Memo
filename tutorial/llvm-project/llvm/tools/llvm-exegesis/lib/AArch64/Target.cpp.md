# Target.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/tools/llvm-exegesis/lib/AArch64/Target.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This file belongs to `lib/AArch64` and implements benchmarking, target modeling, or analysis helpers for `Target`. / 该文件位于 `lib/AArch64`，主要实现与 `Target` 相关的基准分析、目标建模或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===-- Target.cpp ----------------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
#include "../Target.h"
#include "AArch64.h"
#include "AArch64RegisterInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"

#if defined(__aarch64__) && defined(__linux__)
#include <sys/prctl.h> // For PR_PAC_* constants
#ifndef PR_PAC_APIAKEY
#define PR_PAC_APIAKEY (1UL << 0)
#endif
#ifndef PR_PAC_APIBKEY
#define PR_PAC_APIBKEY (1UL << 1)
#endif
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used to visually divide sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment explains nearby logic or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic or intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually divide sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Includes `../Target.h` to access local declarations paired with this implementation file. / 引入 `../Target.h` 以使用与该实现文件配套的本地声明。
- **L9**: Includes `AArch64.h` to access local declarations paired with this implementation file. / 引入 `AArch64.h` 以使用与该实现文件配套的本地声明。
- **L10**: Includes `AArch64RegisterInfo.h` to access local declarations paired with this implementation file. / 引入 `AArch64RegisterInfo.h` 以使用与该实现文件配套的本地声明。
- **L11**: Includes `llvm/CodeGen/MachineInstrBuilder.h` to access code-generation infrastructure. / 引入 `llvm/CodeGen/MachineInstrBuilder.h` 以使用代码生成基础设施。
- **L12**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L13**: Preprocessor directive controls conditional compilation or build behavior: `#if defined(__aarch64__) && defined(__linux__)`. / 预处理指令控制条件编译或构建行为：`#if defined(__aarch64__) && defined(__linux__)`。
- **L14**: Includes `sys/prctl.h` to access local declarations paired with this implementation file. / 引入 `sys/prctl.h` 以使用与该实现文件配套的本地声明。
- **L15**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef PR_PAC_APIAKEY`. / 预处理指令控制条件编译或构建行为：`#ifndef PR_PAC_APIAKEY`。
- **L16**: Defines macro `PR_PAC_APIAKEY` for later conditional logic or annotations. / 定义宏 `PR_PAC_APIAKEY`，供后续条件逻辑或注解使用。
- **L17**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L18**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef PR_PAC_APIBKEY`. / 预处理指令控制条件编译或构建行为：`#ifndef PR_PAC_APIBKEY`。
- **L19**: Defines macro `PR_PAC_APIBKEY` for later conditional logic or annotations. / 定义宏 `PR_PAC_APIBKEY`，供后续条件逻辑或注解使用。
- **L20**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。

### Lines 21-40

```cpp
#ifndef PR_PAC_APDAKEY
#define PR_PAC_APDAKEY (1UL << 2)
#endif
#ifndef PR_PAC_APDBKEY
#define PR_PAC_APDBKEY (1UL << 3)
#endif
#endif

#define GET_AVAILABLE_OPCODE_CHECKER
#include "AArch64GenInstrInfo.inc"

namespace llvm {
namespace exegesis {

static unsigned getLoadImmediateOpcode(unsigned RegBitWidth) {
  switch (RegBitWidth) {
  case 32:
    return AArch64::MOVi32imm;
  case 64:
    return AArch64::MOVi64imm;
```

- **L21**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef PR_PAC_APDAKEY`. / 预处理指令控制条件编译或构建行为：`#ifndef PR_PAC_APDAKEY`。
- **L22**: Defines macro `PR_PAC_APDAKEY` for later conditional logic or annotations. / 定义宏 `PR_PAC_APDAKEY`，供后续条件逻辑或注解使用。
- **L23**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L24**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef PR_PAC_APDBKEY`. / 预处理指令控制条件编译或构建行为：`#ifndef PR_PAC_APDBKEY`。
- **L25**: Defines macro `PR_PAC_APDBKEY` for later conditional logic or annotations. / 定义宏 `PR_PAC_APDBKEY`，供后续条件逻辑或注解使用。
- **L26**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L27**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L28**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L29**: Defines macro `GET_AVAILABLE_OPCODE_CHECKER` for later conditional logic or annotations. / 定义宏 `GET_AVAILABLE_OPCODE_CHECKER`，供后续条件逻辑或注解使用。
- **L30**: Includes `AArch64GenInstrInfo.inc` to access supporting declarations required by this file. / 引入 `AArch64GenInstrInfo.inc` 以使用本文件所需的辅助声明。
- **L31**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L32**: Opens namespace scope `llvm`. / 打开命名空间作用域 `llvm`。
- **L33**: Opens namespace scope `exegesis`. / 打开命名空间作用域 `exegesis`。
- **L34**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L35**: Starts the definition of function or method `getLoadImmediateOpcode`. / 开始定义函数或方法 `getLoadImmediateOpcode`。
- **L36**: Starts a multi-way branch based on an expression: `switch (RegBitWidth) {`. / 开始基于表达式的多路分支：`switch (RegBitWidth) {`。
- **L37**: Introduces a switch dispatch label: `case 32:`. / 引入一个 switch 分发标签：`case 32:`。
- **L38**: Returns control, optionally with a value: `return AArch64::MOVi32imm;`. / 返回控制流，并可附带返回值：`return AArch64::MOVi32imm;`。
- **L39**: Introduces a switch dispatch label: `case 64:`. / 引入一个 switch 分发标签：`case 64:`。
- **L40**: Returns control, optionally with a value: `return AArch64::MOVi64imm;`. / 返回控制流，并可附带返回值：`return AArch64::MOVi64imm;`。

### Lines 41-60

```cpp
  }
  llvm_unreachable("Invalid Value Width");
}

// Generates instruction to load an immediate value into a register.
static MCInst loadImmediate(MCRegister Reg, unsigned RegBitWidth,
                            const APInt &Value) {
  assert(Value.getBitWidth() <= RegBitWidth &&
         "Value must fit in the Register");
  return MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))
      .addReg(Reg)
      .addImm(Value.getZExtValue());
}

static MCInst loadZPRImmediate(MCRegister Reg, unsigned RegBitWidth,
                               const APInt &Value) {
  assert(Value.getZExtValue() < (1 << 7) &&
         "Value must be in the range of the immediate opcode");
  return MCInstBuilder(AArch64::DUP_ZI_D)
      .addReg(Reg)
```

- **L41**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L42**: Declares or invokes `llvm_unreachable`. / 声明或调用 `llvm_unreachable`。
- **L43**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L44**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L45**: Comment explains nearby logic or intent: `Generates instruction to load an immediate value into a register.`. / 注释说明了附近代码的逻辑或设计意图：`Generates instruction to load an immediate value into a register.`。
- **L46**: Continues a multi-line argument list or initializer: `static MCInst loadImmediate(MCRegister Reg, unsigned RegBitWidth,`. / 继续一个多行参数列表或初始化器：`static MCInst loadImmediate(MCRegister Reg, unsigned RegBitWidth,`。
- **L47**: Continues the surrounding expression or declaration: `const APInt &Value) {`. / 继续构造周围的表达式或声明：`const APInt &Value) {`。
- **L48**: Checks an internal invariant with an assertion: `assert(Value.getBitWidth() <= RegBitWidth &&`. / 通过断言检查内部不变式：`assert(Value.getBitWidth() <= RegBitWidth &&`。
- **L49**: Executes a standalone statement or declaration: `"Value must fit in the Register");`. / 执行一条独立语句或声明：`"Value must fit in the Register");`。
- **L50**: Returns control, optionally with a value: `return MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`. / 返回控制流，并可附带返回值：`return MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`。
- **L51**: Continues the surrounding expression or declaration: `.addReg(Reg)`. / 继续构造周围的表达式或声明：`.addReg(Reg)`。
- **L52**: Declares or invokes `.addImm`. / 声明或调用 `.addImm`。
- **L53**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L54**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L55**: Continues a multi-line argument list or initializer: `static MCInst loadZPRImmediate(MCRegister Reg, unsigned RegBitWidth,`. / 继续一个多行参数列表或初始化器：`static MCInst loadZPRImmediate(MCRegister Reg, unsigned RegBitWidth,`。
- **L56**: Continues the surrounding expression or declaration: `const APInt &Value) {`. / 继续构造周围的表达式或声明：`const APInt &Value) {`。
- **L57**: Checks an internal invariant with an assertion: `assert(Value.getZExtValue() < (1 << 7) &&`. / 通过断言检查内部不变式：`assert(Value.getZExtValue() < (1 << 7) &&`。
- **L58**: Executes a standalone statement or declaration: `"Value must be in the range of the immediate opcode");`. / 执行一条独立语句或声明：`"Value must be in the range of the immediate opcode");`。
- **L59**: Returns control, optionally with a value: `return MCInstBuilder(AArch64::DUP_ZI_D)`. / 返回控制流，并可附带返回值：`return MCInstBuilder(AArch64::DUP_ZI_D)`。
- **L60**: Continues the surrounding expression or declaration: `.addReg(Reg)`. / 继续构造周围的表达式或声明：`.addReg(Reg)`。

### Lines 61-80

```cpp
      .addImm(Value.getZExtValue())
      .addImm(0);
}

static MCInst loadPPRImmediate(MCRegister Reg, unsigned RegBitWidth,
                               const APInt &Value) {
  // For PPR, we typically use PTRUE instruction to set predicate registers
  return MCInstBuilder(AArch64::PTRUE_B)
      .addReg(Reg)
      .addImm(31); // All lanes true for 16 bits
}

static MCInst loadFFRImmediate(MCRegister Reg, unsigned RegBitWidth,
                               const APInt &Value) {
  assert(Value.getZExtValue() == 0 && "Expected initialisation value 0");
  // For first-fault register FFR, we set it to a true
  return MCInstBuilder(AArch64::SETFFR);
}

// Generates instructions to load an immediate value into an FPCR register.
```

- **L61**: Continues the surrounding expression or declaration: `.addImm(Value.getZExtValue())`. / 继续构造周围的表达式或声明：`.addImm(Value.getZExtValue())`。
- **L62**: Declares or invokes `.addImm`. / 声明或调用 `.addImm`。
- **L63**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L64**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L65**: Continues a multi-line argument list or initializer: `static MCInst loadPPRImmediate(MCRegister Reg, unsigned RegBitWidth,`. / 继续一个多行参数列表或初始化器：`static MCInst loadPPRImmediate(MCRegister Reg, unsigned RegBitWidth,`。
- **L66**: Continues the surrounding expression or declaration: `const APInt &Value) {`. / 继续构造周围的表达式或声明：`const APInt &Value) {`。
- **L67**: Comment explains nearby logic or intent: `For PPR, we typically use PTRUE instruction to set predicate registers`. / 注释说明了附近代码的逻辑或设计意图：`For PPR, we typically use PTRUE instruction to set predicate registers`。
- **L68**: Returns control, optionally with a value: `return MCInstBuilder(AArch64::PTRUE_B)`. / 返回控制流，并可附带返回值：`return MCInstBuilder(AArch64::PTRUE_B)`。
- **L69**: Continues the surrounding expression or declaration: `.addReg(Reg)`. / 继续构造周围的表达式或声明：`.addReg(Reg)`。
- **L70**: Continues the surrounding expression or declaration: `.addImm(31); // All lanes true for 16 bits`. / 继续构造周围的表达式或声明：`.addImm(31); // All lanes true for 16 bits`。
- **L71**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L72**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L73**: Continues a multi-line argument list or initializer: `static MCInst loadFFRImmediate(MCRegister Reg, unsigned RegBitWidth,`. / 继续一个多行参数列表或初始化器：`static MCInst loadFFRImmediate(MCRegister Reg, unsigned RegBitWidth,`。
- **L74**: Continues the surrounding expression or declaration: `const APInt &Value) {`. / 继续构造周围的表达式或声明：`const APInt &Value) {`。
- **L75**: Checks an internal invariant with an assertion: `assert(Value.getZExtValue() == 0 && "Expected initialisation value 0");`. / 通过断言检查内部不变式：`assert(Value.getZExtValue() == 0 && "Expected initialisation value 0");`。
- **L76**: Comment explains nearby logic or intent: `For first-fault register FFR, we set it to a true`. / 注释说明了附近代码的逻辑或设计意图：`For first-fault register FFR, we set it to a true`。
- **L77**: Returns control, optionally with a value: `return MCInstBuilder(AArch64::SETFFR);`. / 返回控制流，并可附带返回值：`return MCInstBuilder(AArch64::SETFFR);`。
- **L78**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L79**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L80**: Comment explains nearby logic or intent: `Generates instructions to load an immediate value into an FPCR register.`. / 注释说明了附近代码的逻辑或设计意图：`Generates instructions to load an immediate value into an FPCR register.`。

### Lines 81-100

```cpp
static std::vector<MCInst>
loadFPCRImmediate(MCRegister Reg, unsigned RegBitWidth, const APInt &Value) {
  MCRegister TempReg = AArch64::X8;
  MCInst LoadImm = MCInstBuilder(AArch64::MOVi64imm).addReg(TempReg).addImm(0);
  MCInst MoveToFPCR =
      MCInstBuilder(AArch64::MSR).addImm(AArch64SysReg::FPCR).addReg(TempReg);
  return {LoadImm, MoveToFPCR};
}

// Generates instructions to load an immediate value into a pair of W registers
static std::vector<MCInst> loadWSeqPairImmediate(MCRegister Reg,
                                                 unsigned RegBitWidth,
                                                 const APInt &Value) {
  MCRegister EvenReg = (Reg - AArch64::W0_W1) * 2 + AArch64::W0 + 0;
  MCRegister OddReg = (Reg - AArch64::W0_W1) * 2 + AArch64::W0 + 1;
  assert(Value.getBitWidth() <= RegBitWidth &&
         "Value must fit in the Register");

  MCInst LoadEven = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))
                        .addReg(EvenReg)
```

- **L81**: Continues the surrounding expression or declaration: `static std::vector<MCInst>`. / 继续构造周围的表达式或声明：`static std::vector<MCInst>`。
- **L82**: Starts the definition of function or method `loadFPCRImmediate`. / 开始定义函数或方法 `loadFPCRImmediate`。
- **L83**: Initializes or updates `MCRegister TempReg` from the right-hand expression. / 使用右侧表达式初始化或更新 `MCRegister TempReg`。
- **L84**: Declares or invokes `MCInstBuilder`. / 声明或调用 `MCInstBuilder`。
- **L85**: Continues the surrounding expression or declaration: `MCInst MoveToFPCR =`. / 继续构造周围的表达式或声明：`MCInst MoveToFPCR =`。
- **L86**: Declares or invokes `MCInstBuilder`. / 声明或调用 `MCInstBuilder`。
- **L87**: Returns control, optionally with a value: `return {LoadImm, MoveToFPCR};`. / 返回控制流，并可附带返回值：`return {LoadImm, MoveToFPCR};`。
- **L88**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L89**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L90**: Comment explains nearby logic or intent: `Generates instructions to load an immediate value into a pair of W registers`. / 注释说明了附近代码的逻辑或设计意图：`Generates instructions to load an immediate value into a pair of W registers`。
- **L91**: Continues a multi-line argument list or initializer: `static std::vector<MCInst> loadWSeqPairImmediate(MCRegister Reg,`. / 继续一个多行参数列表或初始化器：`static std::vector<MCInst> loadWSeqPairImmediate(MCRegister Reg,`。
- **L92**: Continues a multi-line argument list or initializer: `unsigned RegBitWidth,`. / 继续一个多行参数列表或初始化器：`unsigned RegBitWidth,`。
- **L93**: Continues the surrounding expression or declaration: `const APInt &Value) {`. / 继续构造周围的表达式或声明：`const APInt &Value) {`。
- **L94**: Declares or invokes `=`. / 声明或调用 `=`。
- **L95**: Declares or invokes `=`. / 声明或调用 `=`。
- **L96**: Checks an internal invariant with an assertion: `assert(Value.getBitWidth() <= RegBitWidth &&`. / 通过断言检查内部不变式：`assert(Value.getBitWidth() <= RegBitWidth &&`。
- **L97**: Executes a standalone statement or declaration: `"Value must fit in the Register");`. / 执行一条独立语句或声明：`"Value must fit in the Register");`。
- **L98**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L99**: Continues the surrounding expression or declaration: `MCInst LoadEven = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`. / 继续构造周围的表达式或声明：`MCInst LoadEven = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`。
- **L100**: Continues the surrounding expression or declaration: `.addReg(EvenReg)`. / 继续构造周围的表达式或声明：`.addReg(EvenReg)`。

### Lines 101-120

```cpp
                        .addImm(Value.getZExtValue());
  MCInst LoadOdd = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))
                       .addReg(OddReg)
                       .addImm(Value.getZExtValue());
  return {LoadEven, LoadOdd};
}

// Generates instructions to load an immediate value into a pair of X registers
static std::vector<MCInst> loadXSeqPairImmediate(MCRegister Reg,
                                                 unsigned RegBitWidth,
                                                 const APInt &Value) {
  MCRegister EvenReg = (Reg - AArch64::X0_X1) * 2 + AArch64::X0 + 0;
  MCRegister OddReg = (Reg - AArch64::X0_X1) * 2 + AArch64::X0 + 1;
  assert(Value.getBitWidth() <= RegBitWidth &&
         "Value must fit in the Register");

  MCInst LoadEven = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))
                        .addReg(EvenReg)
                        .addImm(Value.getZExtValue());
  MCInst LoadOdd = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))
```

- **L101**: Declares or invokes `.addImm`. / 声明或调用 `.addImm`。
- **L102**: Continues the surrounding expression or declaration: `MCInst LoadOdd = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`. / 继续构造周围的表达式或声明：`MCInst LoadOdd = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`。
- **L103**: Continues the surrounding expression or declaration: `.addReg(OddReg)`. / 继续构造周围的表达式或声明：`.addReg(OddReg)`。
- **L104**: Declares or invokes `.addImm`. / 声明或调用 `.addImm`。
- **L105**: Returns control, optionally with a value: `return {LoadEven, LoadOdd};`. / 返回控制流，并可附带返回值：`return {LoadEven, LoadOdd};`。
- **L106**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L107**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L108**: Comment explains nearby logic or intent: `Generates instructions to load an immediate value into a pair of X registers`. / 注释说明了附近代码的逻辑或设计意图：`Generates instructions to load an immediate value into a pair of X registers`。
- **L109**: Continues a multi-line argument list or initializer: `static std::vector<MCInst> loadXSeqPairImmediate(MCRegister Reg,`. / 继续一个多行参数列表或初始化器：`static std::vector<MCInst> loadXSeqPairImmediate(MCRegister Reg,`。
- **L110**: Continues a multi-line argument list or initializer: `unsigned RegBitWidth,`. / 继续一个多行参数列表或初始化器：`unsigned RegBitWidth,`。
- **L111**: Continues the surrounding expression or declaration: `const APInt &Value) {`. / 继续构造周围的表达式或声明：`const APInt &Value) {`。
- **L112**: Declares or invokes `=`. / 声明或调用 `=`。
- **L113**: Declares or invokes `=`. / 声明或调用 `=`。
- **L114**: Checks an internal invariant with an assertion: `assert(Value.getBitWidth() <= RegBitWidth &&`. / 通过断言检查内部不变式：`assert(Value.getBitWidth() <= RegBitWidth &&`。
- **L115**: Executes a standalone statement or declaration: `"Value must fit in the Register");`. / 执行一条独立语句或声明：`"Value must fit in the Register");`。
- **L116**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L117**: Continues the surrounding expression or declaration: `MCInst LoadEven = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`. / 继续构造周围的表达式或声明：`MCInst LoadEven = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`。
- **L118**: Continues the surrounding expression or declaration: `.addReg(EvenReg)`. / 继续构造周围的表达式或声明：`.addReg(EvenReg)`。
- **L119**: Declares or invokes `.addImm`. / 声明或调用 `.addImm`。
- **L120**: Continues the surrounding expression or declaration: `MCInst LoadOdd = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`. / 继续构造周围的表达式或声明：`MCInst LoadOdd = MCInstBuilder(getLoadImmediateOpcode(RegBitWidth))`。

### Lines 121-140

```cpp
                       .addReg(OddReg)
                       .addImm(Value.getZExtValue());
  return {LoadEven, LoadOdd};
}

// Fetch base-instruction to load an FP immediate value into a register.
static unsigned getLoadFPImmediateOpcode(unsigned RegBitWidth) {
  switch (RegBitWidth) {
  case 16:
    return AArch64::FMOVH0; // FMOVHi;
  case 32:
    return AArch64::FMOVS0; // FMOVSi;
  case 64:
    return AArch64::MOVID; // FMOVDi;
  case 128:
    return AArch64::MOVIv2d_ns;
  }
  llvm_unreachable("Invalid Value Width");
}

```

- **L121**: Continues the surrounding expression or declaration: `.addReg(OddReg)`. / 继续构造周围的表达式或声明：`.addReg(OddReg)`。
- **L122**: Declares or invokes `.addImm`. / 声明或调用 `.addImm`。
- **L123**: Returns control, optionally with a value: `return {LoadEven, LoadOdd};`. / 返回控制流，并可附带返回值：`return {LoadEven, LoadOdd};`。
- **L124**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L125**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L126**: Comment explains nearby logic or intent: `Fetch base-instruction to load an FP immediate value into a register.`. / 注释说明了附近代码的逻辑或设计意图：`Fetch base-instruction to load an FP immediate value into a register.`。
- **L127**: Starts the definition of function or method `getLoadFPImmediateOpcode`. / 开始定义函数或方法 `getLoadFPImmediateOpcode`。
- **L128**: Starts a multi-way branch based on an expression: `switch (RegBitWidth) {`. / 开始基于表达式的多路分支：`switch (RegBitWidth) {`。
- **L129**: Introduces a switch dispatch label: `case 16:`. / 引入一个 switch 分发标签：`case 16:`。
- **L130**: Returns control, optionally with a value: `return AArch64::FMOVH0; // FMOVHi;`. / 返回控制流，并可附带返回值：`return AArch64::FMOVH0; // FMOVHi;`。
- **L131**: Introduces a switch dispatch label: `case 32:`. / 引入一个 switch 分发标签：`case 32:`。
- **L132**: Returns control, optionally with a value: `return AArch64::FMOVS0; // FMOVSi;`. / 返回控制流，并可附带返回值：`return AArch64::FMOVS0; // FMOVSi;`。
- **L133**: Introduces a switch dispatch label: `case 64:`. / 引入一个 switch 分发标签：`case 64:`。
- **L134**: Returns control, optionally with a value: `return AArch64::MOVID; // FMOVDi;`. / 返回控制流，并可附带返回值：`return AArch64::MOVID; // FMOVDi;`。
- **L135**: Introduces a switch dispatch label: `case 128:`. / 引入一个 switch 分发标签：`case 128:`。
- **L136**: Returns control, optionally with a value: `return AArch64::MOVIv2d_ns;`. / 返回控制流，并可附带返回值：`return AArch64::MOVIv2d_ns;`。
- **L137**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L138**: Declares or invokes `llvm_unreachable`. / 声明或调用 `llvm_unreachable`。
- **L139**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L140**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 141-160

```cpp
// Generates instruction to load an FP immediate value into a register.
static MCInst loadFPImmediate(MCRegister Reg, unsigned RegBitWidth,
                              const APInt &Value) {
  assert(Value.getZExtValue() == 0 && "Expected initialisation value 0");
  MCInst Instructions =
      MCInstBuilder(getLoadFPImmediateOpcode(RegBitWidth)).addReg(Reg);
  if (RegBitWidth >= 64)
    Instructions.addOperand(MCOperand::createImm(Value.getZExtValue()));
  return Instructions;
}

// Generates instructions to load an immediate value into a DD, DDD, DDDD,
// QQ, QQQ or QQQQ Reg
static std::vector<MCInst>
loadDQ234RegImmediate(MCRegister Reg, unsigned RegBitWidth, const APInt &Value,
                      MCRegister BaseReg, unsigned RegCount) {
  MCRegister RegDorQ0 = AArch64::D0;
  if (RegBitWidth == 128)
    RegDorQ0 = AArch64::Q0;

```

- **L141**: Comment explains nearby logic or intent: `Generates instruction to load an FP immediate value into a register.`. / 注释说明了附近代码的逻辑或设计意图：`Generates instruction to load an FP immediate value into a register.`。
- **L142**: Continues a multi-line argument list or initializer: `static MCInst loadFPImmediate(MCRegister Reg, unsigned RegBitWidth,`. / 继续一个多行参数列表或初始化器：`static MCInst loadFPImmediate(MCRegister Reg, unsigned RegBitWidth,`。
- **L143**: Continues the surrounding expression or declaration: `const APInt &Value) {`. / 继续构造周围的表达式或声明：`const APInt &Value) {`。
- **L144**: Checks an internal invariant with an assertion: `assert(Value.getZExtValue() == 0 && "Expected initialisation value 0");`. / 通过断言检查内部不变式：`assert(Value.getZExtValue() == 0 && "Expected initialisation value 0");`。
- **L145**: Continues the surrounding expression or declaration: `MCInst Instructions =`. / 继续构造周围的表达式或声明：`MCInst Instructions =`。
- **L146**: Declares or invokes `MCInstBuilder`. / 声明或调用 `MCInstBuilder`。
- **L147**: Introduces a conditional branch: `if (RegBitWidth >= 64)`. / 引入条件分支：`if (RegBitWidth >= 64)`。
- **L148**: Declares or invokes `Instructions.addOperand`. / 声明或调用 `Instructions.addOperand`。
- **L149**: Returns control, optionally with a value: `return Instructions;`. / 返回控制流，并可附带返回值：`return Instructions;`。
- **L150**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L151**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L152**: Comment explains nearby logic or intent: `Generates instructions to load an immediate value into a DD, DDD, DDDD,`. / 注释说明了附近代码的逻辑或设计意图：`Generates instructions to load an immediate value into a DD, DDD, DDDD,`。
- **L153**: Comment explains nearby logic or intent: `QQ, QQQ or QQQQ Reg`. / 注释说明了附近代码的逻辑或设计意图：`QQ, QQQ or QQQQ Reg`。
- **L154**: Continues the surrounding expression or declaration: `static std::vector<MCInst>`. / 继续构造周围的表达式或声明：`static std::vector<MCInst>`。
- **L155**: Continues a multi-line argument list or initializer: `loadDQ234RegImmediate(MCRegister Reg, unsigned RegBitWidth, const APInt &Value,`. / 继续一个多行参数列表或初始化器：`loadDQ234RegImmediate(MCRegister Reg, unsigned RegBitWidth, const APInt &Value,`。
- **L156**: Continues the surrounding expression or declaration: `MCRegister BaseReg, unsigned RegCount) {`. / 继续构造周围的表达式或声明：`MCRegister BaseReg, unsigned RegCount) {`。
- **L157**: Initializes or updates `MCRegister RegDorQ0` from the right-hand expression. / 使用右侧表达式初始化或更新 `MCRegister RegDorQ0`。
- **L158**: Introduces a conditional branch: `if (RegBitWidth == 128)`. / 引入条件分支：`if (RegBitWidth == 128)`。
- **L159**: Initializes or updates `RegDorQ0` from the right-hand expression. / 使用右侧表达式初始化或更新 `RegDorQ0`。
- **L160**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 161-180

```cpp
  MCRegister RegDQ0 = RegDorQ0 + ((Reg - BaseReg + 0) % 32);
  MCRegister RegDQ1 = RegDorQ0 + ((Reg - BaseReg + 1) % 32);
  MCRegister RegDQ2 = RegDorQ0 + ((Reg - BaseReg + 2) % 32);
  MCRegister RegDQ3 = RegDorQ0 + ((Reg - BaseReg + 3) % 32);

  MCInst LoadDQ0 = loadFPImmediate(RegDQ0, RegBitWidth, Value);
  MCInst LoadDQ1 = loadFPImmediate(RegDQ1, RegBitWidth, Value);
  if (RegCount == 2)
    return {LoadDQ0, LoadDQ1};
  MCInst LoadDQ2 = loadFPImmediate(RegDQ2, RegBitWidth, Value);
  if (RegCount == 3)
    return {LoadDQ0, LoadDQ1, LoadDQ2};
  MCInst LoadDQ3 = loadFPImmediate(RegDQ3, RegBitWidth, Value);
  assert((RegCount == 4) && "ExpectedRegCount 2, 3 or 4");
  return {LoadDQ0, LoadDQ1, LoadDQ2, LoadDQ3};
}

// Generates instructions to load immediate in the flags register
static std::vector<MCInst>
loadNZCVImmediate(MCRegister Reg, unsigned RegBitWidth, const APInt &Value) {
```

- **L161**: Declares or invokes `+`. / 声明或调用 `+`。
- **L162**: Declares or invokes `+`. / 声明或调用 `+`。
- **L163**: Declares or invokes `+`. / 声明或调用 `+`。
- **L164**: Declares or invokes `+`. / 声明或调用 `+`。
- **L165**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L166**: Declares or invokes `loadFPImmediate`. / 声明或调用 `loadFPImmediate`。
- **L167**: Declares or invokes `loadFPImmediate`. / 声明或调用 `loadFPImmediate`。
- **L168**: Introduces a conditional branch: `if (RegCount == 2)`. / 引入条件分支：`if (RegCount == 2)`。
- **L169**: Returns control, optionally with a value: `return {LoadDQ0, LoadDQ1};`. / 返回控制流，并可附带返回值：`return {LoadDQ0, LoadDQ1};`。
- **L170**: Declares or invokes `loadFPImmediate`. / 声明或调用 `loadFPImmediate`。
- **L171**: Introduces a conditional branch: `if (RegCount == 3)`. / 引入条件分支：`if (RegCount == 3)`。
- **L172**: Returns control, optionally with a value: `return {LoadDQ0, LoadDQ1, LoadDQ2};`. / 返回控制流，并可附带返回值：`return {LoadDQ0, LoadDQ1, LoadDQ2};`。
- **L173**: Declares or invokes `loadFPImmediate`. / 声明或调用 `loadFPImmediate`。
- **L174**: Checks an internal invariant with an assertion: `assert((RegCount == 4) && "ExpectedRegCount 2, 3 or 4");`. / 通过断言检查内部不变式：`assert((RegCount == 4) && "ExpectedRegCount 2, 3 or 4");`。
- **L175**: Returns control, optionally with a value: `return {LoadDQ0, LoadDQ1, LoadDQ2, LoadDQ3};`. / 返回控制流，并可附带返回值：`return {LoadDQ0, LoadDQ1, LoadDQ2, LoadDQ3};`。
- **L176**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L177**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L178**: Comment explains nearby logic or intent: `Generates instructions to load immediate in the flags register`. / 注释说明了附近代码的逻辑或设计意图：`Generates instructions to load immediate in the flags register`。
- **L179**: Continues the surrounding expression or declaration: `static std::vector<MCInst>`. / 继续构造周围的表达式或声明：`static std::vector<MCInst>`。
- **L180**: Starts the definition of function or method `loadNZCVImmediate`. / 开始定义函数或方法 `loadNZCVImmediate`。

### Lines 181-200

```cpp
  MCRegister TempReg1 = AArch64::X8;
  MCRegister TempReg2 = AArch64::X9;

  MCInst MoveFromNZCV =
      MCInstBuilder(AArch64::MRS).addReg(TempReg1).addImm(AArch64SysReg::NZCV);
  MCInst LoadMask =
      MCInstBuilder(AArch64::MOVi64imm).addReg(TempReg2).addImm(0xf0000000);
  MCInst BitClear = MCInstBuilder(AArch64::BICXrr)
                        .addReg(TempReg1)
                        .addReg(TempReg1)
                        .addReg(TempReg2);
  MCInst MoveToNZCV =
      MCInstBuilder(AArch64::MSR).addImm(AArch64SysReg::NZCV).addReg(TempReg1);

  if (Value.getZExtValue() == 0)
    return {MoveFromNZCV, LoadMask, BitClear, MoveToNZCV};

  MCInst OrrMask = MCInstBuilder(AArch64::ORRXrr)
                       .addReg(TempReg1)
                       .addReg(TempReg1)
```

- **L181**: Initializes or updates `MCRegister TempReg1` from the right-hand expression. / 使用右侧表达式初始化或更新 `MCRegister TempReg1`。
- **L182**: Initializes or updates `MCRegister TempReg2` from the right-hand expression. / 使用右侧表达式初始化或更新 `MCRegister TempReg2`。
- **L183**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L184**: Continues the surrounding expression or declaration: `MCInst MoveFromNZCV =`. / 继续构造周围的表达式或声明：`MCInst MoveFromNZCV =`。
- **L185**: Declares or invokes `MCInstBuilder`. / 声明或调用 `MCInstBuilder`。
- **L186**: Continues the surrounding expression or declaration: `MCInst LoadMask =`. / 继续构造周围的表达式或声明：`MCInst LoadMask =`。
- **L187**: Declares or invokes `MCInstBuilder`. / 声明或调用 `MCInstBuilder`。
- **L188**: Continues the surrounding expression or declaration: `MCInst BitClear = MCInstBuilder(AArch64::BICXrr)`. / 继续构造周围的表达式或声明：`MCInst BitClear = MCInstBuilder(AArch64::BICXrr)`。
- **L189**: Continues the surrounding expression or declaration: `.addReg(TempReg1)`. / 继续构造周围的表达式或声明：`.addReg(TempReg1)`。
- **L190**: Continues the surrounding expression or declaration: `.addReg(TempReg1)`. / 继续构造周围的表达式或声明：`.addReg(TempReg1)`。
- **L191**: Declares or invokes `.addReg`. / 声明或调用 `.addReg`。
- **L192**: Continues the surrounding expression or declaration: `MCInst MoveToNZCV =`. / 继续构造周围的表达式或声明：`MCInst MoveToNZCV =`。
- **L193**: Declares or invokes `MCInstBuilder`. / 声明或调用 `MCInstBuilder`。
- **L194**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L195**: Introduces a conditional branch: `if (Value.getZExtValue() == 0)`. / 引入条件分支：`if (Value.getZExtValue() == 0)`。
- **L196**: Returns control, optionally with a value: `return {MoveFromNZCV, LoadMask, BitClear, MoveToNZCV};`. / 返回控制流，并可附带返回值：`return {MoveFromNZCV, LoadMask, BitClear, MoveToNZCV};`。
- **L197**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L198**: Continues the surrounding expression or declaration: `MCInst OrrMask = MCInstBuilder(AArch64::ORRXrr)`. / 继续构造周围的表达式或声明：`MCInst OrrMask = MCInstBuilder(AArch64::ORRXrr)`。
- **L199**: Continues the surrounding expression or declaration: `.addReg(TempReg1)`. / 继续构造周围的表达式或声明：`.addReg(TempReg1)`。
- **L200**: Continues the surrounding expression or declaration: `.addReg(TempReg1)`. / 继续构造周围的表达式或声明：`.addReg(TempReg1)`。

### Lines 201-220

```cpp
                       .addImm(Value.getZExtValue());
  return {MoveFromNZCV, LoadMask, BitClear, OrrMask, MoveToNZCV};
}

#include "AArch64GenExegesis.inc"

namespace {

// Use X19 as the loop counter register since it's a callee-saved register
// that's available for temporary use.
constexpr MCPhysReg kDefaultLoopCounterReg = AArch64::X19;

class ExegesisAArch64Target : public ExegesisTarget {
public:
  ExegesisAArch64Target()
      : ExegesisTarget(AArch64CpuPfmCounters, AArch64_MC::isOpcodeAvailable) {}

  Error randomizeTargetMCOperand(const Instruction &Instr, const Variable &Var,
                                 MCOperand &AssignedValue,
                                 const BitVector &ForbiddenRegs) const override;
```

- **L201**: Declares or invokes `.addImm`. / 声明或调用 `.addImm`。
- **L202**: Returns control, optionally with a value: `return {MoveFromNZCV, LoadMask, BitClear, OrrMask, MoveToNZCV};`. / 返回控制流，并可附带返回值：`return {MoveFromNZCV, LoadMask, BitClear, OrrMask, MoveToNZCV};`。
- **L203**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L204**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L205**: Includes `AArch64GenExegesis.inc` to access supporting declarations required by this file. / 引入 `AArch64GenExegesis.inc` 以使用本文件所需的辅助声明。
- **L206**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L207**: Opens namespace scope `(anonymous)`. / 打开命名空间作用域 `(anonymous)`。
- **L208**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L209**: Comment explains nearby logic or intent: `Use X19 as the loop counter register since it's a callee-saved register`. / 注释说明了附近代码的逻辑或设计意图：`Use X19 as the loop counter register since it's a callee-saved register`。
- **L210**: Comment explains nearby logic or intent: `that's available for temporary use.`. / 注释说明了附近代码的逻辑或设计意图：`that's available for temporary use.`。
- **L211**: Initializes or updates `constexpr MCPhysReg kDefaultLoopCounterReg` from the right-hand expression. / 使用右侧表达式初始化或更新 `constexpr MCPhysReg kDefaultLoopCounterReg`。
- **L212**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L213**: Declares class `ExegesisTarget`. / 声明 class `ExegesisTarget`。
- **L214**: Sets the following members to `public` access. / 将后续成员的访问级别设为 `public`。
- **L215**: Continues the surrounding expression or declaration: `ExegesisAArch64Target()`. / 继续构造周围的表达式或声明：`ExegesisAArch64Target()`。
- **L216**: Continues a multi-line argument list or initializer: `: ExegesisTarget(AArch64CpuPfmCounters, AArch64_MC::isOpcodeAvailable) {}`. / 继续一个多行参数列表或初始化器：`: ExegesisTarget(AArch64CpuPfmCounters, AArch64_MC::isOpcodeAvailable) {}`。
- **L217**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L218**: Continues a multi-line argument list or initializer: `Error randomizeTargetMCOperand(const Instruction &Instr, const Variable &Var,`. / 继续一个多行参数列表或初始化器：`Error randomizeTargetMCOperand(const Instruction &Instr, const Variable &Var,`。
- **L219**: Continues a multi-line argument list or initializer: `MCOperand &AssignedValue,`. / 继续一个多行参数列表或初始化器：`MCOperand &AssignedValue,`。
- **L220**: Executes a standalone statement or declaration: `const BitVector &ForbiddenRegs) const override;`. / 执行一条独立语句或声明：`const BitVector &ForbiddenRegs) const override;`。

### Lines 221-240

```cpp

private:
  std::vector<MCInst> setRegTo(const MCSubtargetInfo &STI, MCRegister Reg,
                               const APInt &Value) const override {
    if (AArch64::GPR32RegClass.contains(Reg))
      return {loadImmediate(Reg, 32, Value)};
    if (AArch64::GPR64RegClass.contains(Reg))
      return {loadImmediate(Reg, 64, Value)};
    if (AArch64::PPRRegClass.contains(Reg))
      return {loadPPRImmediate(Reg, 16, Value)};
    if (AArch64::FPR8RegClass.contains(Reg))
      return {loadFPImmediate(Reg - AArch64::B0 + AArch64::D0, 64, Value)};
    if (AArch64::FPR16RegClass.contains(Reg))
      return {loadFPImmediate(Reg, 16, Value)};
    if (AArch64::FPR32RegClass.contains(Reg))
      return {loadFPImmediate(Reg, 32, Value)};
    if (AArch64::FPR64RegClass.contains(Reg))
      return {loadFPImmediate(Reg, 64, Value)};
    if (AArch64::FPR128RegClass.contains(Reg))
      return {loadFPImmediate(Reg, 128, Value)};
```

- **L221**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L222**: Sets the following members to `private` access. / 将后续成员的访问级别设为 `private`。
- **L223**: Continues a multi-line argument list or initializer: `std::vector<MCInst> setRegTo(const MCSubtargetInfo &STI, MCRegister Reg,`. / 继续一个多行参数列表或初始化器：`std::vector<MCInst> setRegTo(const MCSubtargetInfo &STI, MCRegister Reg,`。
- **L224**: Continues the surrounding expression or declaration: `const APInt &Value) const override {`. / 继续构造周围的表达式或声明：`const APInt &Value) const override {`。
- **L225**: Introduces a conditional branch: `if (AArch64::GPR32RegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::GPR32RegClass.contains(Reg))`。
- **L226**: Returns control, optionally with a value: `return {loadImmediate(Reg, 32, Value)};`. / 返回控制流，并可附带返回值：`return {loadImmediate(Reg, 32, Value)};`。
- **L227**: Introduces a conditional branch: `if (AArch64::GPR64RegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::GPR64RegClass.contains(Reg))`。
- **L228**: Returns control, optionally with a value: `return {loadImmediate(Reg, 64, Value)};`. / 返回控制流，并可附带返回值：`return {loadImmediate(Reg, 64, Value)};`。
- **L229**: Introduces a conditional branch: `if (AArch64::PPRRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::PPRRegClass.contains(Reg))`。
- **L230**: Returns control, optionally with a value: `return {loadPPRImmediate(Reg, 16, Value)};`. / 返回控制流，并可附带返回值：`return {loadPPRImmediate(Reg, 16, Value)};`。
- **L231**: Introduces a conditional branch: `if (AArch64::FPR8RegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::FPR8RegClass.contains(Reg))`。
- **L232**: Returns control, optionally with a value: `return {loadFPImmediate(Reg - AArch64::B0 + AArch64::D0, 64, Value)};`. / 返回控制流，并可附带返回值：`return {loadFPImmediate(Reg - AArch64::B0 + AArch64::D0, 64, Value)};`。
- **L233**: Introduces a conditional branch: `if (AArch64::FPR16RegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::FPR16RegClass.contains(Reg))`。
- **L234**: Returns control, optionally with a value: `return {loadFPImmediate(Reg, 16, Value)};`. / 返回控制流，并可附带返回值：`return {loadFPImmediate(Reg, 16, Value)};`。
- **L235**: Introduces a conditional branch: `if (AArch64::FPR32RegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::FPR32RegClass.contains(Reg))`。
- **L236**: Returns control, optionally with a value: `return {loadFPImmediate(Reg, 32, Value)};`. / 返回控制流，并可附带返回值：`return {loadFPImmediate(Reg, 32, Value)};`。
- **L237**: Introduces a conditional branch: `if (AArch64::FPR64RegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::FPR64RegClass.contains(Reg))`。
- **L238**: Returns control, optionally with a value: `return {loadFPImmediate(Reg, 64, Value)};`. / 返回控制流，并可附带返回值：`return {loadFPImmediate(Reg, 64, Value)};`。
- **L239**: Introduces a conditional branch: `if (AArch64::FPR128RegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::FPR128RegClass.contains(Reg))`。
- **L240**: Returns control, optionally with a value: `return {loadFPImmediate(Reg, 128, Value)};`. / 返回控制流，并可附带返回值：`return {loadFPImmediate(Reg, 128, Value)};`。

### Lines 241-260

```cpp
    if (AArch64::ZPRRegClass.contains(Reg))
      return {loadZPRImmediate(Reg, 128, Value)};
    if (Reg == AArch64::FPCR)
      return {loadFPCRImmediate(Reg, 32, Value)};
    if (Reg == AArch64::NZCV)
      return {loadNZCVImmediate(Reg, 32, Value)};
    if (Reg == AArch64::FFR)
      return {loadFFRImmediate(Reg, 32, Value)};
    if (AArch64::WSeqPairsClassRegClass.contains(Reg))
      return {loadWSeqPairImmediate(Reg, 32, Value)};
    if (AArch64::XSeqPairsClassRegClass.contains(Reg))
      return {loadXSeqPairImmediate(Reg, 64, Value)};
    if (AArch64::DDRegClass.contains(Reg))
      return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1, 2);
    if (AArch64::DDDRegClass.contains(Reg))
      return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1_D2, 3);
    if (AArch64::DDDDRegClass.contains(Reg))
      return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1_D2_D3, 4);
    if (AArch64::QQRegClass.contains(Reg))
      return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1, 2);
```

- **L241**: Introduces a conditional branch: `if (AArch64::ZPRRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::ZPRRegClass.contains(Reg))`。
- **L242**: Returns control, optionally with a value: `return {loadZPRImmediate(Reg, 128, Value)};`. / 返回控制流，并可附带返回值：`return {loadZPRImmediate(Reg, 128, Value)};`。
- **L243**: Introduces a conditional branch: `if (Reg == AArch64::FPCR)`. / 引入条件分支：`if (Reg == AArch64::FPCR)`。
- **L244**: Returns control, optionally with a value: `return {loadFPCRImmediate(Reg, 32, Value)};`. / 返回控制流，并可附带返回值：`return {loadFPCRImmediate(Reg, 32, Value)};`。
- **L245**: Introduces a conditional branch: `if (Reg == AArch64::NZCV)`. / 引入条件分支：`if (Reg == AArch64::NZCV)`。
- **L246**: Returns control, optionally with a value: `return {loadNZCVImmediate(Reg, 32, Value)};`. / 返回控制流，并可附带返回值：`return {loadNZCVImmediate(Reg, 32, Value)};`。
- **L247**: Introduces a conditional branch: `if (Reg == AArch64::FFR)`. / 引入条件分支：`if (Reg == AArch64::FFR)`。
- **L248**: Returns control, optionally with a value: `return {loadFFRImmediate(Reg, 32, Value)};`. / 返回控制流，并可附带返回值：`return {loadFFRImmediate(Reg, 32, Value)};`。
- **L249**: Introduces a conditional branch: `if (AArch64::WSeqPairsClassRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::WSeqPairsClassRegClass.contains(Reg))`。
- **L250**: Returns control, optionally with a value: `return {loadWSeqPairImmediate(Reg, 32, Value)};`. / 返回控制流，并可附带返回值：`return {loadWSeqPairImmediate(Reg, 32, Value)};`。
- **L251**: Introduces a conditional branch: `if (AArch64::XSeqPairsClassRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::XSeqPairsClassRegClass.contains(Reg))`。
- **L252**: Returns control, optionally with a value: `return {loadXSeqPairImmediate(Reg, 64, Value)};`. / 返回控制流，并可附带返回值：`return {loadXSeqPairImmediate(Reg, 64, Value)};`。
- **L253**: Introduces a conditional branch: `if (AArch64::DDRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::DDRegClass.contains(Reg))`。
- **L254**: Returns control, optionally with a value: `return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1, 2);`. / 返回控制流，并可附带返回值：`return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1, 2);`。
- **L255**: Introduces a conditional branch: `if (AArch64::DDDRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::DDDRegClass.contains(Reg))`。
- **L256**: Returns control, optionally with a value: `return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1_D2, 3);`. / 返回控制流，并可附带返回值：`return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1_D2, 3);`。
- **L257**: Introduces a conditional branch: `if (AArch64::DDDDRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::DDDDRegClass.contains(Reg))`。
- **L258**: Returns control, optionally with a value: `return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1_D2_D3, 4);`. / 返回控制流，并可附带返回值：`return loadDQ234RegImmediate(Reg, 64, Value, AArch64::D0_D1_D2_D3, 4);`。
- **L259**: Introduces a conditional branch: `if (AArch64::QQRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::QQRegClass.contains(Reg))`。
- **L260**: Returns control, optionally with a value: `return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1, 2);`. / 返回控制流，并可附带返回值：`return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1, 2);`。

### Lines 261-280

```cpp
    if (AArch64::QQQRegClass.contains(Reg))
      return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1_Q2, 3);
    if (AArch64::QQQQRegClass.contains(Reg))
      return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1_Q2_Q3, 4);
    // TODO if (AArch64::PNRRegClass.contains(Reg))
    // TODO if (AArch64::ZPRRegClass.contains(Reg))
    // TODO if (AArch64::ZPR2RegClass.contains(Reg))
    // TODO if (AArch64::ZPR2StridedOrContiguousRegClass.contains(Reg))

    errs() << "setRegTo is not implemented, results will be unreliable\n";
    return {};
  }
  MCRegister getDefaultLoopCounterRegister(const Triple &) const override {
    return kDefaultLoopCounterReg;
  }

  void decrementLoopCounterAndJump(MachineBasicBlock &MBB,
                                   MachineBasicBlock &TargetMBB,
                                   const MCInstrInfo &MII,
                                   MCRegister LoopRegister) const override {
```

- **L261**: Introduces a conditional branch: `if (AArch64::QQQRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::QQQRegClass.contains(Reg))`。
- **L262**: Returns control, optionally with a value: `return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1_Q2, 3);`. / 返回控制流，并可附带返回值：`return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1_Q2, 3);`。
- **L263**: Introduces a conditional branch: `if (AArch64::QQQQRegClass.contains(Reg))`. / 引入条件分支：`if (AArch64::QQQQRegClass.contains(Reg))`。
- **L264**: Returns control, optionally with a value: `return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1_Q2_Q3, 4);`. / 返回控制流，并可附带返回值：`return loadDQ234RegImmediate(Reg, 128, Value, AArch64::Q0_Q1_Q2_Q3, 4);`。
- **L265**: Comment records an implementation note or caution: `TODO if (AArch64::PNRRegClass.contains(Reg))`. / 注释记录了一条实现说明或注意事项：`TODO if (AArch64::PNRRegClass.contains(Reg))`。
- **L266**: Comment records an implementation note or caution: `TODO if (AArch64::ZPRRegClass.contains(Reg))`. / 注释记录了一条实现说明或注意事项：`TODO if (AArch64::ZPRRegClass.contains(Reg))`。
- **L267**: Comment records an implementation note or caution: `TODO if (AArch64::ZPR2RegClass.contains(Reg))`. / 注释记录了一条实现说明或注意事项：`TODO if (AArch64::ZPR2RegClass.contains(Reg))`。
- **L268**: Comment records an implementation note or caution: `TODO if (AArch64::ZPR2StridedOrContiguousRegClass.contains(Reg))`. / 注释记录了一条实现说明或注意事项：`TODO if (AArch64::ZPR2StridedOrContiguousRegClass.contains(Reg))`。
- **L269**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L270**: Declares or invokes `errs`. / 声明或调用 `errs`。
- **L271**: Returns control, optionally with a value: `return {};`. / 返回控制流，并可附带返回值：`return {};`。
- **L272**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L273**: Starts the definition of function or method `getDefaultLoopCounterRegister`. / 开始定义函数或方法 `getDefaultLoopCounterRegister`。
- **L274**: Returns control, optionally with a value: `return kDefaultLoopCounterReg;`. / 返回控制流，并可附带返回值：`return kDefaultLoopCounterReg;`。
- **L275**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L276**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L277**: Continues a multi-line argument list or initializer: `void decrementLoopCounterAndJump(MachineBasicBlock &MBB,`. / 继续一个多行参数列表或初始化器：`void decrementLoopCounterAndJump(MachineBasicBlock &MBB,`。
- **L278**: Continues a multi-line argument list or initializer: `MachineBasicBlock &TargetMBB,`. / 继续一个多行参数列表或初始化器：`MachineBasicBlock &TargetMBB,`。
- **L279**: Continues a multi-line argument list or initializer: `const MCInstrInfo &MII,`. / 继续一个多行参数列表或初始化器：`const MCInstrInfo &MII,`。
- **L280**: Continues the surrounding expression or declaration: `MCRegister LoopRegister) const override {`. / 继续构造周围的表达式或声明：`MCRegister LoopRegister) const override {`。

### Lines 281-300

```cpp
    // subs LoopRegister, LoopRegister, #1
    BuildMI(&MBB, DebugLoc(), MII.get(AArch64::SUBSXri))
        .addDef(LoopRegister)
        .addUse(LoopRegister)
        .addImm(1)  // Subtract 1
        .addImm(0); // No shift amount
    // b.ne TargetMBB
    BuildMI(&MBB, DebugLoc(), MII.get(AArch64::Bcc))
        .addImm(AArch64CC::NE)
        .addMBB(&TargetMBB);
  }

  // Registers that should not be selected for use in snippets.
  const MCPhysReg UnavailableRegisters[1] = {kDefaultLoopCounterReg};
  ArrayRef<MCPhysReg> getUnavailableRegisters() const override {
    return UnavailableRegisters;
  }

  bool matchesArch(Triple::ArchType Arch) const override {
    return Arch == Triple::aarch64 || Arch == Triple::aarch64_be;
```

- **L281**: Comment explains nearby logic or intent: `subs LoopRegister, LoopRegister, #1`. / 注释说明了附近代码的逻辑或设计意图：`subs LoopRegister, LoopRegister, #1`。
- **L282**: Continues the surrounding expression or declaration: `BuildMI(&MBB, DebugLoc(), MII.get(AArch64::SUBSXri))`. / 继续构造周围的表达式或声明：`BuildMI(&MBB, DebugLoc(), MII.get(AArch64::SUBSXri))`。
- **L283**: Continues the surrounding expression or declaration: `.addDef(LoopRegister)`. / 继续构造周围的表达式或声明：`.addDef(LoopRegister)`。
- **L284**: Continues the surrounding expression or declaration: `.addUse(LoopRegister)`. / 继续构造周围的表达式或声明：`.addUse(LoopRegister)`。
- **L285**: Continues the surrounding expression or declaration: `.addImm(1) // Subtract 1`. / 继续构造周围的表达式或声明：`.addImm(1) // Subtract 1`。
- **L286**: Continues the surrounding expression or declaration: `.addImm(0); // No shift amount`. / 继续构造周围的表达式或声明：`.addImm(0); // No shift amount`。
- **L287**: Comment explains nearby logic or intent: `b.ne TargetMBB`. / 注释说明了附近代码的逻辑或设计意图：`b.ne TargetMBB`。
- **L288**: Continues the surrounding expression or declaration: `BuildMI(&MBB, DebugLoc(), MII.get(AArch64::Bcc))`. / 继续构造周围的表达式或声明：`BuildMI(&MBB, DebugLoc(), MII.get(AArch64::Bcc))`。
- **L289**: Continues the surrounding expression or declaration: `.addImm(AArch64CC::NE)`. / 继续构造周围的表达式或声明：`.addImm(AArch64CC::NE)`。
- **L290**: Declares or invokes `.addMBB`. / 声明或调用 `.addMBB`。
- **L291**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L292**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L293**: Comment explains nearby logic or intent: `Registers that should not be selected for use in snippets.`. / 注释说明了附近代码的逻辑或设计意图：`Registers that should not be selected for use in snippets.`。
- **L294**: Initializes or updates `const MCPhysReg UnavailableRegisters[1]` from the right-hand expression. / 使用右侧表达式初始化或更新 `const MCPhysReg UnavailableRegisters[1]`。
- **L295**: Starts the definition of function or method `getUnavailableRegisters`. / 开始定义函数或方法 `getUnavailableRegisters`。
- **L296**: Returns control, optionally with a value: `return UnavailableRegisters;`. / 返回控制流，并可附带返回值：`return UnavailableRegisters;`。
- **L297**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L298**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L299**: Starts the definition of function or method `matchesArch`. / 开始定义函数或方法 `matchesArch`。
- **L300**: Returns control, optionally with a value: `return Arch == Triple::aarch64 || Arch == Triple::aarch64_be;`. / 返回控制流，并可附带返回值：`return Arch == Triple::aarch64 || Arch == Triple::aarch64_be;`。

### Lines 301-320

```cpp
  }

  void addTargetSpecificPasses(PassManagerBase &PM) const override {
    // Function return is a pseudo-instruction that needs to be expanded
    PM.add(createAArch64ExpandPseudoLegacyPass());
  }
};

Error ExegesisAArch64Target::randomizeTargetMCOperand(
    const Instruction &Instr, const Variable &Var, MCOperand &AssignedValue,
    const BitVector &ForbiddenRegs) const {
  const Operand &Op = Instr.getPrimaryOperand(Var);
  const auto OperandType = Op.getExplicitOperandInfo().OperandType;
  // NOTE: To resolve "Not all operands were initialized by snippet generator"
  // Requires OperandType to be defined for such opcode's operands in AArch64
  // tablegen files. And omit introduced OperandType(s).

  // Hacky Fix: Defaulting all OPERAND_UNKNOWN to immediate value 0 works with a
  // limitation that it introduces illegal instruction error for system
  // instructions. System instructions will need to be omitted with OperandType
```

- **L301**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L302**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L303**: Starts the definition of function or method `addTargetSpecificPasses`. / 开始定义函数或方法 `addTargetSpecificPasses`。
- **L304**: Comment explains nearby logic or intent: `Function return is a pseudo-instruction that needs to be expanded`. / 注释说明了附近代码的逻辑或设计意图：`Function return is a pseudo-instruction that needs to be expanded`。
- **L305**: Declares or invokes `PM.add`. / 声明或调用 `PM.add`。
- **L306**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L307**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L308**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L309**: Continues a multi-line argument list or initializer: `Error ExegesisAArch64Target::randomizeTargetMCOperand(`. / 继续一个多行参数列表或初始化器：`Error ExegesisAArch64Target::randomizeTargetMCOperand(`。
- **L310**: Continues a multi-line argument list or initializer: `const Instruction &Instr, const Variable &Var, MCOperand &AssignedValue,`. / 继续一个多行参数列表或初始化器：`const Instruction &Instr, const Variable &Var, MCOperand &AssignedValue,`。
- **L311**: Continues the surrounding expression or declaration: `const BitVector &ForbiddenRegs) const {`. / 继续构造周围的表达式或声明：`const BitVector &ForbiddenRegs) const {`。
- **L312**: Declares or invokes `Instr.getPrimaryOperand`. / 声明或调用 `Instr.getPrimaryOperand`。
- **L313**: Declares or invokes `Op.getExplicitOperandInfo`. / 声明或调用 `Op.getExplicitOperandInfo`。
- **L314**: Comment records an implementation note or caution: `NOTE: To resolve "Not all operands were initialized by snippet generator"`. / 注释记录了一条实现说明或注意事项：`NOTE: To resolve "Not all operands were initialized by snippet generator"`。
- **L315**: Comment explains nearby logic or intent: `Requires OperandType to be defined for such opcode's operands in AArch64`. / 注释说明了附近代码的逻辑或设计意图：`Requires OperandType to be defined for such opcode's operands in AArch64`。
- **L316**: Comment explains nearby logic or intent: `tablegen files. And omit introduced OperandType(s).`. / 注释说明了附近代码的逻辑或设计意图：`tablegen files. And omit introduced OperandType(s).`。
- **L317**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L318**: Comment explains nearby logic or intent: `Hacky Fix: Defaulting all OPERAND_UNKNOWN to immediate value 0 works with a`. / 注释说明了附近代码的逻辑或设计意图：`Hacky Fix: Defaulting all OPERAND_UNKNOWN to immediate value 0 works with a`。
- **L319**: Comment explains nearby logic or intent: `limitation that it introduces illegal instruction error for system`. / 注释说明了附近代码的逻辑或设计意图：`limitation that it introduces illegal instruction error for system`。
- **L320**: Comment explains nearby logic or intent: `instructions. System instructions will need to be omitted with OperandType`. / 注释说明了附近代码的逻辑或设计意图：`instructions. System instructions will need to be omitted with OperandType`。

### Lines 321-340

```cpp
  // or opcode specific values to avoid generating invalid encodings or
  // unreliable benchmark results for these system-level instructions.
  //  Implement opcode-specific immediate value handling for system instrs:
  //   - MRS/MSR: Use valid system register encodings (e.g., NZCV, FPCR, FPSR)
  //   - MSRpstatesvcrImm1: Use valid PSTATE field encodings (e.g., SPSel,
  //   DAIFSet)
  //   - SYSLxt/SYSxt: Use valid system instruction encodings with proper
  //   CRn/CRm/op values
  //   - UDF: Use valid undefined instruction immediate ranges (0-65535)

  switch (OperandType) {
  // MSL (Masking Shift Left) imm operand for 32-bit splatted SIMD constants
  // Correspond to AArch64InstructionSelector::tryAdvSIMDModImm321s()
  case llvm::AArch64::OPERAND_SHIFT_MSL: {
    // There are two valid encodings:
    //   - Type 7: imm at [15:8], [47:40], shift = 264 (0x108) → msl #8
    //   - Type 8: imm at [23:16], [55:48], shift = 272 (0x110) → msl #16
    //     Corresponds AArch64_AM::encodeAdvSIMDModImmType7()
    // But, v2s_msl and v4s_msl instructions accept either form,
    // Thus, Arbitrarily chosing 264 (msl #8) for simplicity.
```

- **L321**: Comment explains nearby logic or intent: `or opcode specific values to avoid generating invalid encodings or`. / 注释说明了附近代码的逻辑或设计意图：`or opcode specific values to avoid generating invalid encodings or`。
- **L322**: Comment explains nearby logic or intent: `unreliable benchmark results for these system-level instructions.`. / 注释说明了附近代码的逻辑或设计意图：`unreliable benchmark results for these system-level instructions.`。
- **L323**: Comment explains nearby logic or intent: `Implement opcode-specific immediate value handling for system instrs:`. / 注释说明了附近代码的逻辑或设计意图：`Implement opcode-specific immediate value handling for system instrs:`。
- **L324**: Comment explains nearby logic or intent: `- MRS/MSR: Use valid system register encodings (e.g., NZCV, FPCR, FPSR)`. / 注释说明了附近代码的逻辑或设计意图：`- MRS/MSR: Use valid system register encodings (e.g., NZCV, FPCR, FPSR)`。
- **L325**: Comment explains nearby logic or intent: `- MSRpstatesvcrImm1: Use valid PSTATE field encodings (e.g., SPSel,`. / 注释说明了附近代码的逻辑或设计意图：`- MSRpstatesvcrImm1: Use valid PSTATE field encodings (e.g., SPSel,`。
- **L326**: Comment explains nearby logic or intent: `DAIFSet)`. / 注释说明了附近代码的逻辑或设计意图：`DAIFSet)`。
- **L327**: Comment explains nearby logic or intent: `- SYSLxt/SYSxt: Use valid system instruction encodings with proper`. / 注释说明了附近代码的逻辑或设计意图：`- SYSLxt/SYSxt: Use valid system instruction encodings with proper`。
- **L328**: Comment explains nearby logic or intent: `CRn/CRm/op values`. / 注释说明了附近代码的逻辑或设计意图：`CRn/CRm/op values`。
- **L329**: Comment explains nearby logic or intent: `- UDF: Use valid undefined instruction immediate ranges (0-65535)`. / 注释说明了附近代码的逻辑或设计意图：`- UDF: Use valid undefined instruction immediate ranges (0-65535)`。
- **L330**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L331**: Starts a multi-way branch based on an expression: `switch (OperandType) {`. / 开始基于表达式的多路分支：`switch (OperandType) {`。
- **L332**: Comment explains nearby logic or intent: `MSL (Masking Shift Left) imm operand for 32-bit splatted SIMD constants`. / 注释说明了附近代码的逻辑或设计意图：`MSL (Masking Shift Left) imm operand for 32-bit splatted SIMD constants`。
- **L333**: Comment explains nearby logic or intent: `Correspond to AArch64InstructionSelector::tryAdvSIMDModImm321s()`. / 注释说明了附近代码的逻辑或设计意图：`Correspond to AArch64InstructionSelector::tryAdvSIMDModImm321s()`。
- **L334**: Introduces a switch dispatch label: `case llvm::AArch64::OPERAND_SHIFT_MSL: {`. / 引入一个 switch 分发标签：`case llvm::AArch64::OPERAND_SHIFT_MSL: {`。
- **L335**: Comment explains nearby logic or intent: `There are two valid encodings:`. / 注释说明了附近代码的逻辑或设计意图：`There are two valid encodings:`。
- **L336**: Comment explains nearby logic or intent: `- Type 7: imm at [15:8], [47:40], shift 264 (0x108) → msl #8`. / 注释说明了附近代码的逻辑或设计意图：`- Type 7: imm at [15:8], [47:40], shift 264 (0x108) → msl #8`。
- **L337**: Comment explains nearby logic or intent: `- Type 8: imm at [23:16], [55:48], shift 272 (0x110) → msl #16`. / 注释说明了附近代码的逻辑或设计意图：`- Type 8: imm at [23:16], [55:48], shift 272 (0x110) → msl #16`。
- **L338**: Comment explains nearby logic or intent: `Corresponds AArch64_AM::encodeAdvSIMDModImmType7()`. / 注释说明了附近代码的逻辑或设计意图：`Corresponds AArch64_AM::encodeAdvSIMDModImmType7()`。
- **L339**: Comment explains nearby logic or intent: `But, v2s_msl and v4s_msl instructions accept either form,`. / 注释说明了附近代码的逻辑或设计意图：`But, v2s_msl and v4s_msl instructions accept either form,`。
- **L340**: Comment explains nearby logic or intent: `Thus, Arbitrarily chosing 264 (msl #8) for simplicity.`. / 注释说明了附近代码的逻辑或设计意图：`Thus, Arbitrarily chosing 264 (msl #8) for simplicity.`。

### Lines 341-360

```cpp
    AssignedValue = MCOperand::createImm(264);
    return Error::success();
  }
  case llvm::AArch64::OPERAND_IMPLICIT_IMM_0:
    AssignedValue = MCOperand::createImm(0);
    return Error::success();
  case llvm::AArch64::OPERAND_SHIFTED_REGISTER:
    // TODO it would be better if these operands were randomized
    AssignedValue = MCOperand::createReg(0);
    return Error::success();
  case llvm::AArch64::OPERAND_SHIFTED_IMMEDIATE:
    AssignedValue = MCOperand::createImm(0);
    return Error::success();
  case MCOI::OperandType::OPERAND_PCREL:
    AssignedValue = MCOperand::createImm(8);
    return Error::success();
  default:
    break;
  }

```

- **L341**: Declares or invokes `MCOperand::createImm`. / 声明或调用 `MCOperand::createImm`。
- **L342**: Returns control, optionally with a value: `return Error::success();`. / 返回控制流，并可附带返回值：`return Error::success();`。
- **L343**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L344**: Introduces a switch dispatch label: `case llvm::AArch64::OPERAND_IMPLICIT_IMM_0:`. / 引入一个 switch 分发标签：`case llvm::AArch64::OPERAND_IMPLICIT_IMM_0:`。
- **L345**: Declares or invokes `MCOperand::createImm`. / 声明或调用 `MCOperand::createImm`。
- **L346**: Returns control, optionally with a value: `return Error::success();`. / 返回控制流，并可附带返回值：`return Error::success();`。
- **L347**: Introduces a switch dispatch label: `case llvm::AArch64::OPERAND_SHIFTED_REGISTER:`. / 引入一个 switch 分发标签：`case llvm::AArch64::OPERAND_SHIFTED_REGISTER:`。
- **L348**: Comment records an implementation note or caution: `TODO it would be better if these operands were randomized`. / 注释记录了一条实现说明或注意事项：`TODO it would be better if these operands were randomized`。
- **L349**: Declares or invokes `MCOperand::createReg`. / 声明或调用 `MCOperand::createReg`。
- **L350**: Returns control, optionally with a value: `return Error::success();`. / 返回控制流，并可附带返回值：`return Error::success();`。
- **L351**: Introduces a switch dispatch label: `case llvm::AArch64::OPERAND_SHIFTED_IMMEDIATE:`. / 引入一个 switch 分发标签：`case llvm::AArch64::OPERAND_SHIFTED_IMMEDIATE:`。
- **L352**: Declares or invokes `MCOperand::createImm`. / 声明或调用 `MCOperand::createImm`。
- **L353**: Returns control, optionally with a value: `return Error::success();`. / 返回控制流，并可附带返回值：`return Error::success();`。
- **L354**: Introduces a switch dispatch label: `case MCOI::OperandType::OPERAND_PCREL:`. / 引入一个 switch 分发标签：`case MCOI::OperandType::OPERAND_PCREL:`。
- **L355**: Declares or invokes `MCOperand::createImm`. / 声明或调用 `MCOperand::createImm`。
- **L356**: Returns control, optionally with a value: `return Error::success();`. / 返回控制流，并可附带返回值：`return Error::success();`。
- **L357**: Introduces the default switch branch: `default:`. / 引入 switch 的默认分支：`default:`。
- **L358**: Executes a standalone statement or declaration: `break;`. / 执行一条独立语句或声明：`break;`。
- **L359**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L360**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 361-378

```cpp
  return make_error<Failure>(
      Twine("Unimplemented operand type: MCOI::OperandType:")
          .concat(Twine(static_cast<int>(OperandType))));
}

} // namespace

static ExegesisTarget *getTheExegesisAArch64Target() {
  static ExegesisAArch64Target Target;
  return &Target;
}

void InitializeAArch64ExegesisTarget() {
  ExegesisTarget::registerTarget(getTheExegesisAArch64Target());
}

} // namespace exegesis
} // namespace llvm
```

- **L361**: Returns control, optionally with a value: `return make_error<Failure>(`. / 返回控制流，并可附带返回值：`return make_error<Failure>(`。
- **L362**: Continues the surrounding expression or declaration: `Twine("Unimplemented operand type: MCOI::OperandType:")`. / 继续构造周围的表达式或声明：`Twine("Unimplemented operand type: MCOI::OperandType:")`。
- **L363**: Declares or invokes `.concat`. / 声明或调用 `.concat`。
- **L364**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L365**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L366**: Closes a namespace scope with a trailing comment: `} // namespace`. / 结束一个带尾注释的命名空间作用域：`} // namespace`。
- **L367**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L368**: Starts the definition of function or method `getTheExegesisAArch64Target`. / 开始定义函数或方法 `getTheExegesisAArch64Target`。
- **L369**: Executes a standalone statement or declaration: `static ExegesisAArch64Target Target;`. / 执行一条独立语句或声明：`static ExegesisAArch64Target Target;`。
- **L370**: Returns control, optionally with a value: `return &Target;`. / 返回控制流，并可附带返回值：`return &Target;`。
- **L371**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L372**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L373**: Starts the definition of function or method `InitializeAArch64ExegesisTarget`. / 开始定义函数或方法 `InitializeAArch64ExegesisTarget`。
- **L374**: Declares or invokes `ExegesisTarget::registerTarget`. / 声明或调用 `ExegesisTarget::registerTarget`。
- **L375**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L376**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L377**: Closes a namespace scope with a trailing comment: `} // namespace exegesis`. / 结束一个带尾注释的命名空间作用域：`} // namespace exegesis`。
- **L378**: Closes a namespace scope with a trailing comment: `} // namespace llvm`. / 结束一个带尾注释的命名空间作用域：`} // namespace llvm`。

## Key Concepts / 关键概念

- **Benchmarking and target-specific measurements / 基准测试与目标相关测量**
- **Module/file-scoped coordination / 模块/文件级协调**
- **`Target` focused implementation / 围绕 `Target` 的实现逻辑**

## Dependencies / 依赖关系

- **Include / 包含** `../Target.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `AArch64.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `AArch64RegisterInfo.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `llvm/CodeGen/MachineInstrBuilder.h`: Provides code-generation infrastructure. / 提供代码生成基础设施。
- **Include / 包含** `sys/prctl.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `AArch64GenInstrInfo.inc`: Provides supporting declarations required by this file. / 提供本文件所需的辅助声明。
- **Include / 包含** `AArch64GenExegesis.inc`: Provides supporting declarations required by this file. / 提供本文件所需的辅助声明。
