# Character.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Builder/Character.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements FIR/MLIR optimizer-side utilities and infrastructure for Character.
- **Purpose (CN)**: 实现 Character 相关的 FIR/MLIR 优化器辅助工具与基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- Character.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide/
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Builder/Character.h"
#include "flang/Optimizer/Builder/DoLoopHelper.h"
#include "flang/Optimizer/Builder/FIRBuilder.h"
#include "flang/Optimizer/Builder/Todo.h"
#include "flang/Optimizer/Dialect/FIROpsSupport.h"
#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "llvm/Support/Debug.h"
#include <optional>

#define DEBUG_TYPE "flang-lower-character"

//===----------------------------------------------------------------------===//
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, intent, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的逻辑、意图或元数据：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, intent, or metadata: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的逻辑、意图或元数据：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, intent, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的逻辑、意图或元数据：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes "flang/Optimizer/Builder/Character.h" to access FIR builder helpers and runtime-construction utilities.
  **L13 CN**: 引入 "flang/Optimizer/Builder/Character.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L14 EN**: Includes "flang/Optimizer/Builder/DoLoopHelper.h" to access FIR builder helpers and runtime-construction utilities.
  **L14 CN**: 引入 "flang/Optimizer/Builder/DoLoopHelper.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L15 EN**: Includes "flang/Optimizer/Builder/FIRBuilder.h" to access FIR builder helpers and runtime-construction utilities.
  **L15 CN**: 引入 "flang/Optimizer/Builder/FIRBuilder.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L16 EN**: Includes "flang/Optimizer/Builder/Todo.h" to access FIR builder helpers and runtime-construction utilities.
  **L16 CN**: 引入 "flang/Optimizer/Builder/Todo.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L17 EN**: Includes "flang/Optimizer/Dialect/FIROpsSupport.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L17 CN**: 引入 "flang/Optimizer/Dialect/FIROpsSupport.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L18 EN**: Includes "mlir/Dialect/LLVMIR/LLVMDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L18 CN**: 引入 "mlir/Dialect/LLVMIR/LLVMDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L19 EN**: Includes "llvm/Support/Debug.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L19 CN**: 引入 "llvm/Support/Debug.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L20 EN**: Includes <optional> to access supporting declarations used by this translation unit.
  **L20 CN**: 引入 <optional> 以使用当前编译单元使用的辅助声明。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Defines macro `DEBUG_TYPE` for conditional compilation or local shorthand.
  **L22 CN**: 定义宏 `DEBUG_TYPE`，用于条件编译或本地简写。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Banner comment marking a file or section boundary.
  **L24 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 25-48

````cpp
// CharacterExprHelper implementation
//===----------------------------------------------------------------------===//

/// Unwrap all the ref and box types and return the inner element type.
static mlir::Type unwrapBoxAndRef(mlir::Type type) {
  if (auto boxType = mlir::dyn_cast<fir::BoxCharType>(type))
    return boxType.getEleTy();
  while (true) {
    type = fir::unwrapRefType(type);
    if (auto boxTy = mlir::dyn_cast<fir::BoxType>(type))
      type = boxTy.getEleTy();
    else
      break;
  }
  return type;
}

/// Unwrap base fir.char<kind,len> type.
static fir::CharacterType recoverCharacterType(mlir::Type type) {
  type = fir::unwrapSequenceType(unwrapBoxAndRef(type));
  if (auto charTy = mlir::dyn_cast<fir::CharacterType>(type))
    return charTy;
  llvm::report_fatal_error("expected a character type");
}
````
- **L25 EN**: Comment explains nearby logic, intent, or metadata: `CharacterExprHelper implementation`.
  **L25 CN**: 注释说明附近代码的逻辑、意图或元数据：`CharacterExprHelper implementation`。
- **L26 EN**: Banner comment marking a file or section boundary.
  **L26 CN**: 横幅注释，用于标记文件或章节边界。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Comment explains nearby logic, intent, or metadata: `Unwrap all the ref and box types and return the inner element type.`.
  **L28 CN**: 注释说明附近代码的逻辑、意图或元数据：`Unwrap all the ref and box types and return the inner element type.`。
- **L29 EN**: Starts a function, method, lambda, or structured scope: `static mlir::Type unwrapBoxAndRef(mlir::Type type) {`.
  **L29 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static mlir::Type unwrapBoxAndRef(mlir::Type type) {`。
- **L30 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L30 CN**: 开始 `if` 控制流语句并计算其条件。
- **L31 EN**: Returns from the current function with `boxType.getEleTy()`.
  **L31 CN**: 以 `boxType.getEleTy()` 从当前函数返回。
- **L32 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L32 CN**: 开始 `while` 控制流语句并计算其条件。
- **L33 EN**: Executes a call or declaration centered on `fir::unwrapRefType`.
  **L33 CN**: 执行以 `fir::unwrapRefType` 为核心的调用或声明。
- **L34 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L34 CN**: 开始 `if` 控制流语句并计算其条件。
- **L35 EN**: Executes a call or declaration centered on `boxTy.getEleTy`.
  **L35 CN**: 执行以 `boxTy.getEleTy` 为核心的调用或声明。
- **L36 EN**: Transitions from the previous branch into the alternative path.
  **L36 CN**: 从前一个分支过渡到备选路径。
- **L37 EN**: Exits the nearest loop or switch statement.
  **L37 CN**: 退出最近的循环或 switch 语句。
- **L38 EN**: Closes the current lexical scope or compound statement.
  **L38 CN**: 结束当前词法作用域或复合语句块。
- **L39 EN**: Returns from the current function with `type`.
  **L39 CN**: 以 `type` 从当前函数返回。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Comment explains nearby logic, intent, or metadata: `Unwrap base fir.char<kind,len> type.`.
  **L42 CN**: 注释说明附近代码的逻辑、意图或元数据：`Unwrap base fir.char<kind,len> type.`。
- **L43 EN**: Starts a function, method, lambda, or structured scope: `static fir::CharacterType recoverCharacterType(mlir::Type type) {`.
  **L43 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static fir::CharacterType recoverCharacterType(mlir::Type type) {`。
- **L44 EN**: Executes a call or declaration centered on `fir::unwrapSequenceType`.
  **L44 CN**: 执行以 `fir::unwrapSequenceType` 为核心的调用或声明。
- **L45 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L45 CN**: 开始 `if` 控制流语句并计算其条件。
- **L46 EN**: Returns from the current function with `charTy`.
  **L46 CN**: 以 `charTy` 从当前函数返回。
- **L47 EN**: Executes a call or declaration centered on `llvm::report_fatal_error`.
  **L47 CN**: 执行以 `llvm::report_fatal_error` 为核心的调用或声明。
- **L48 EN**: Closes the current lexical scope or compound statement.
  **L48 CN**: 结束当前词法作用域或复合语句块。

### Lines 49-72

````cpp

bool fir::factory::CharacterExprHelper::isCharacterScalar(mlir::Type type) {
  type = unwrapBoxAndRef(type);
  return !mlir::isa<fir::SequenceType>(type) && fir::isa_char(type);
}

bool fir::factory::CharacterExprHelper::isArray(mlir::Type type) {
  type = unwrapBoxAndRef(type);
  if (auto seqTy = mlir::dyn_cast<fir::SequenceType>(type))
    return fir::isa_char(seqTy.getEleTy());
  return false;
}

fir::CharacterType
fir::factory::CharacterExprHelper::getCharacterType(mlir::Type type) {
  assert(isCharacterScalar(type) && "expected scalar character");
  return recoverCharacterType(type);
}

fir::CharacterType
fir::factory::CharacterExprHelper::getCharType(mlir::Type type) {
  return recoverCharacterType(type);
}

````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Starts a function, method, lambda, or structured scope: `bool fir::factory::CharacterExprHelper::isCharacterScalar(mlir::Type type) {`.
  **L50 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool fir::factory::CharacterExprHelper::isCharacterScalar(mlir::Type type) {`。
- **L51 EN**: Executes a call or declaration centered on `unwrapBoxAndRef`.
  **L51 CN**: 执行以 `unwrapBoxAndRef` 为核心的调用或声明。
- **L52 EN**: Returns from the current function with `!mlir::isa<fir::SequenceType>(type) && fir::isa_char(type)`.
  **L52 CN**: 以 `!mlir::isa<fir::SequenceType>(type) && fir::isa_char(type)` 从当前函数返回。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L55 EN**: Starts a function, method, lambda, or structured scope: `bool fir::factory::CharacterExprHelper::isArray(mlir::Type type) {`.
  **L55 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool fir::factory::CharacterExprHelper::isArray(mlir::Type type) {`。
- **L56 EN**: Executes a call or declaration centered on `unwrapBoxAndRef`.
  **L56 CN**: 执行以 `unwrapBoxAndRef` 为核心的调用或声明。
- **L57 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L57 CN**: 开始 `if` 控制流语句并计算其条件。
- **L58 EN**: Returns from the current function with `fir::isa_char(seqTy.getEleTy())`.
  **L58 CN**: 以 `fir::isa_char(seqTy.getEleTy())` 从当前函数返回。
- **L59 EN**: Returns from the current function with `false`.
  **L59 CN**: 以 `false` 从当前函数返回。
- **L60 EN**: Closes the current lexical scope or compound statement.
  **L60 CN**: 结束当前词法作用域或复合语句块。
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L62 EN**: Continues the surrounding expression or declaration: `fir::CharacterType`.
  **L62 CN**: 继续构造周围的表达式或声明：`fir::CharacterType`。
- **L63 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::getCharacterType(mlir::Type type) {`.
  **L63 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::getCharacterType(mlir::Type type) {`。
- **L64 EN**: Checks an internal invariant in debug builds.
  **L64 CN**: 在调试构建中检查内部不变式。
- **L65 EN**: Returns from the current function with `recoverCharacterType(type)`.
  **L65 CN**: 以 `recoverCharacterType(type)` 从当前函数返回。
- **L66 EN**: Closes the current lexical scope or compound statement.
  **L66 CN**: 结束当前词法作用域或复合语句块。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Continues the surrounding expression or declaration: `fir::CharacterType`.
  **L68 CN**: 继续构造周围的表达式或声明：`fir::CharacterType`。
- **L69 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::getCharType(mlir::Type type) {`.
  **L69 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::getCharType(mlir::Type type) {`。
- **L70 EN**: Returns from the current function with `recoverCharacterType(type)`.
  **L70 CN**: 以 `recoverCharacterType(type)` 从当前函数返回。
- **L71 EN**: Closes the current lexical scope or compound statement.
  **L71 CN**: 结束当前词法作用域或复合语句块。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 73-96

````cpp
fir::CharacterType fir::factory::CharacterExprHelper::getCharacterType(
    const fir::CharBoxValue &box) {
  return getCharacterType(box.getBuffer().getType());
}

fir::CharacterType
fir::factory::CharacterExprHelper::getCharacterType(mlir::Value str) {
  return getCharacterType(str.getType());
}

/// Determine the static size of the character. Returns the computed size, not
/// an IR Value.
static std::optional<fir::CharacterType::LenType>
getCompileTimeLength(const fir::CharBoxValue &box) {
  auto len = recoverCharacterType(box.getBuffer().getType()).getLen();
  if (len == fir::CharacterType::unknownLen())
    return {};
  return len;
}

/// Detect the precondition that the value `str` does not reside in memory. Such
/// values will have a type `!fir.array<...x!fir.char<N>>` or `!fir.char<N>`.
[[maybe_unused]] static bool needToMaterialize(mlir::Value str) {
  return mlir::isa<fir::SequenceType>(str.getType()) ||
````
- **L73 EN**: Continues logic associated with callable symbol `getCharacterType`.
  **L73 CN**: 继续与可调用符号 `getCharacterType` 相关的逻辑。
- **L74 EN**: Continues the surrounding expression or declaration: `const fir::CharBoxValue &box) {`.
  **L74 CN**: 继续构造周围的表达式或声明：`const fir::CharBoxValue &box) {`。
- **L75 EN**: Returns from the current function with `getCharacterType(box.getBuffer().getType())`.
  **L75 CN**: 以 `getCharacterType(box.getBuffer().getType())` 从当前函数返回。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L78 EN**: Continues the surrounding expression or declaration: `fir::CharacterType`.
  **L78 CN**: 继续构造周围的表达式或声明：`fir::CharacterType`。
- **L79 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::getCharacterType(mlir::Value str) {`.
  **L79 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::getCharacterType(mlir::Value str) {`。
- **L80 EN**: Returns from the current function with `getCharacterType(str.getType())`.
  **L80 CN**: 以 `getCharacterType(str.getType())` 从当前函数返回。
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Comment explains nearby logic, intent, or metadata: `Determine the static size of the character. Returns the computed size, not`.
  **L83 CN**: 注释说明附近代码的逻辑、意图或元数据：`Determine the static size of the character. Returns the computed size, not`。
- **L84 EN**: Comment explains nearby logic, intent, or metadata: `an IR Value.`.
  **L84 CN**: 注释说明附近代码的逻辑、意图或元数据：`an IR Value.`。
- **L85 EN**: Continues the surrounding expression or declaration: `static std::optional<fir::CharacterType::LenType>`.
  **L85 CN**: 继续构造周围的表达式或声明：`static std::optional<fir::CharacterType::LenType>`。
- **L86 EN**: Starts a function, method, lambda, or structured scope: `getCompileTimeLength(const fir::CharBoxValue &box) {`.
  **L86 CN**: 开始一个函数、方法、lambda 或结构化作用域：`getCompileTimeLength(const fir::CharBoxValue &box) {`。
- **L87 EN**: Initializes variable `len` from the right-hand expression.
  **L87 CN**: 使用右侧表达式初始化变量 `len`。
- **L88 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L88 CN**: 开始 `if` 控制流语句并计算其条件。
- **L89 EN**: Returns from the current function with `{}`.
  **L89 CN**: 以 `{}` 从当前函数返回。
- **L90 EN**: Returns from the current function with `len`.
  **L90 CN**: 以 `len` 从当前函数返回。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Comment explains nearby logic, intent, or metadata: `Detect the precondition that the value `str` does not reside in memory. Such`.
  **L93 CN**: 注释说明附近代码的逻辑、意图或元数据：`Detect the precondition that the value `str` does not reside in memory. Such`。
- **L94 EN**: Comment explains nearby logic, intent, or metadata: `values will have a type `!fir.array<...x!fir.char<N>>` or `!fir.char<N>`.`.
  **L94 CN**: 注释说明附近代码的逻辑、意图或元数据：`values will have a type `!fir.array<...x!fir.char<N>>` or `!fir.char<N>`.`。
- **L95 EN**: Starts a function, method, lambda, or structured scope: `[[maybe_unused]] static bool needToMaterialize(mlir::Value str) {`.
  **L95 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[[maybe_unused]] static bool needToMaterialize(mlir::Value str) {`。
- **L96 EN**: Returns from the current function with `mlir::isa<fir::SequenceType>(str.getType()) ||`.
  **L96 CN**: 以 `mlir::isa<fir::SequenceType>(str.getType()) ||` 从当前函数返回。

### Lines 97-120

````cpp
         fir::isa_char(str.getType());
}

/// This is called only if `str` does not reside in memory. Such a bare string
/// value will be converted into a memory-based temporary and an extended
/// boxchar value returned.
fir::CharBoxValue
fir::factory::CharacterExprHelper::materializeValue(mlir::Value str) {
  assert(needToMaterialize(str));
  auto ty = str.getType();
  assert(isCharacterScalar(ty) && "expected scalar character");
  auto charTy = mlir::dyn_cast<fir::CharacterType>(ty);
  if (!charTy || charTy.getLen() == fir::CharacterType::unknownLen()) {
    LLVM_DEBUG(llvm::dbgs() << "cannot materialize: " << str << '\n');
    llvm_unreachable("must be a !fir.char<N> type");
  }
  auto len = builder.createIntegerConstant(
      loc, builder.getCharacterLengthType(), charTy.getLen());
  auto temp = fir::AllocaOp::create(builder, loc, charTy);
  fir::StoreOp::create(builder, loc, str, temp);
  LLVM_DEBUG(llvm::dbgs() << "materialized as local: " << str << " -> (" << temp
                          << ", " << len << ")\n");
  return {temp, len};
}
````
- **L97 EN**: Executes a call or declaration centered on `fir::isa_char`.
  **L97 CN**: 执行以 `fir::isa_char` 为核心的调用或声明。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L100 EN**: Comment explains nearby logic, intent, or metadata: `This is called only if `str` does not reside in memory. Such a bare string`.
  **L100 CN**: 注释说明附近代码的逻辑、意图或元数据：`This is called only if `str` does not reside in memory. Such a bare string`。
- **L101 EN**: Comment explains nearby logic, intent, or metadata: `value will be converted into a memory-based temporary and an extended`.
  **L101 CN**: 注释说明附近代码的逻辑、意图或元数据：`value will be converted into a memory-based temporary and an extended`。
- **L102 EN**: Comment explains nearby logic, intent, or metadata: `boxchar value returned.`.
  **L102 CN**: 注释说明附近代码的逻辑、意图或元数据：`boxchar value returned.`。
- **L103 EN**: Continues the surrounding expression or declaration: `fir::CharBoxValue`.
  **L103 CN**: 继续构造周围的表达式或声明：`fir::CharBoxValue`。
- **L104 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::materializeValue(mlir::Value str) {`.
  **L104 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::materializeValue(mlir::Value str) {`。
- **L105 EN**: Checks an internal invariant in debug builds.
  **L105 CN**: 在调试构建中检查内部不变式。
- **L106 EN**: Initializes variable `ty` from the right-hand expression.
  **L106 CN**: 使用右侧表达式初始化变量 `ty`。
- **L107 EN**: Checks an internal invariant in debug builds.
  **L107 CN**: 在调试构建中检查内部不变式。
- **L108 EN**: Initializes variable `charTy` from the right-hand expression.
  **L108 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L109 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L109 CN**: 开始 `if` 控制流语句并计算其条件。
- **L110 EN**: Executes a call or declaration centered on `LLVM_DEBUG`.
  **L110 CN**: 执行以 `LLVM_DEBUG` 为核心的调用或声明。
- **L111 EN**: Marks this control path as unreachable to LLVM.
  **L111 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L112 EN**: Closes the current lexical scope or compound statement.
  **L112 CN**: 结束当前词法作用域或复合语句块。
- **L113 EN**: Continues logic associated with callable symbol `createIntegerConstant`.
  **L113 CN**: 继续与可调用符号 `createIntegerConstant` 相关的逻辑。
- **L114 EN**: Executes a call or declaration centered on `builder.getCharacterLengthType`.
  **L114 CN**: 执行以 `builder.getCharacterLengthType` 为核心的调用或声明。
- **L115 EN**: Initializes variable `temp` from the right-hand expression.
  **L115 CN**: 使用右侧表达式初始化变量 `temp`。
- **L116 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L116 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L117 EN**: Continues logic associated with callable symbol `LLVM_DEBUG`.
  **L117 CN**: 继续与可调用符号 `LLVM_DEBUG` 相关的逻辑。
- **L118 EN**: Executes a standalone statement or declaration: `<< ", " << len << ")\n");`.
  **L118 CN**: 执行一条独立语句或声明：`<< ", " << len << ")\n");`。
- **L119 EN**: Returns from the current function with `{temp, len}`.
  **L119 CN**: 以 `{temp, len}` 从当前函数返回。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。

### Lines 121-144

````cpp

fir::ExtendedValue
fir::factory::CharacterExprHelper::toExtendedValue(mlir::Value character,
                                                   mlir::Value len) {
  auto lenType = builder.getCharacterLengthType();
  auto type = character.getType();
  auto base = fir::isa_passbyref_type(type) ? character : mlir::Value{};
  auto resultLen = len;
  llvm::SmallVector<mlir::Value> extents;

  if (auto eleType = fir::dyn_cast_ptrEleTy(type))
    type = eleType;

  if (auto arrayType = mlir::dyn_cast<fir::SequenceType>(type)) {
    type = arrayType.getEleTy();
    auto indexType = builder.getIndexType();
    for (auto extent : arrayType.getShape()) {
      if (extent == fir::SequenceType::getUnknownExtent())
        break;
      extents.emplace_back(
          builder.createIntegerConstant(loc, indexType, extent));
    }
    // Last extent might be missing in case of assumed-size. If more extents
    // could not be deduced from type, that's an error (a fir.box should
````
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L122 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L122 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::CharacterExprHelper::toExtendedValue(mlir::Value character,`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::CharacterExprHelper::toExtendedValue(mlir::Value character,`。
- **L124 EN**: Continues the surrounding expression or declaration: `mlir::Value len) {`.
  **L124 CN**: 继续构造周围的表达式或声明：`mlir::Value len) {`。
- **L125 EN**: Initializes variable `lenType` from the right-hand expression.
  **L125 CN**: 使用右侧表达式初始化变量 `lenType`。
- **L126 EN**: Initializes variable `type` from the right-hand expression.
  **L126 CN**: 使用右侧表达式初始化变量 `type`。
- **L127 EN**: Initializes variable `base` from the right-hand expression.
  **L127 CN**: 使用右侧表达式初始化变量 `base`。
- **L128 EN**: Initializes variable `resultLen` from the right-hand expression.
  **L128 CN**: 使用右侧表达式初始化变量 `resultLen`。
- **L129 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> extents;`.
  **L129 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> extents;`。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L131 CN**: 开始 `if` 控制流语句并计算其条件。
- **L132 EN**: Executes a standalone statement or declaration: `type = eleType;`.
  **L132 CN**: 执行一条独立语句或声明：`type = eleType;`。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L134 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L134 CN**: 开始 `if` 控制流语句并计算其条件。
- **L135 EN**: Executes a call or declaration centered on `arrayType.getEleTy`.
  **L135 CN**: 执行以 `arrayType.getEleTy` 为核心的调用或声明。
- **L136 EN**: Initializes variable `indexType` from the right-hand expression.
  **L136 CN**: 使用右侧表达式初始化变量 `indexType`。
- **L137 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L137 CN**: 开始 `for` 控制流语句并计算其条件。
- **L138 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L138 CN**: 开始 `if` 控制流语句并计算其条件。
- **L139 EN**: Exits the nearest loop or switch statement.
  **L139 CN**: 退出最近的循环或 switch 语句。
- **L140 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L140 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L141 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L141 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Comment explains nearby logic, intent, or metadata: `Last extent might be missing in case of assumed-size. If more extents`.
  **L143 CN**: 注释说明附近代码的逻辑、意图或元数据：`Last extent might be missing in case of assumed-size. If more extents`。
- **L144 EN**: Comment explains nearby logic, intent, or metadata: `could not be deduced from type, that's an error (a fir.box should`.
  **L144 CN**: 注释说明附近代码的逻辑、意图或元数据：`could not be deduced from type, that's an error (a fir.box should`。

### Lines 145-168

````cpp
    // have been used in the interface).
    if (extents.size() + 1 < arrayType.getShape().size())
      mlir::emitError(loc, "cannot retrieve array extents from type");
  }

  if (auto charTy = mlir::dyn_cast<fir::CharacterType>(type)) {
    if (!resultLen && charTy.getLen() != fir::CharacterType::unknownLen())
      resultLen = builder.createIntegerConstant(loc, lenType, charTy.getLen());
  } else if (auto boxCharType = mlir::dyn_cast<fir::BoxCharType>(type)) {
    auto refType = builder.getRefType(boxCharType.getEleTy());
    // If the embox is accessible, use its operand to avoid filling
    // the generated fir with embox/unbox.
    mlir::Value boxCharLen;
    if (auto definingOp = character.getDefiningOp()) {
      if (auto box = mlir::dyn_cast<fir::EmboxCharOp>(definingOp)) {
        base = box.getMemref();
        boxCharLen = box.getLen();
      }
    }
    if (!boxCharLen) {
      auto unboxed =
          fir::UnboxCharOp::create(builder, loc, refType, lenType, character);
      base = builder.createConvert(loc, refType, unboxed.getResult(0));
      boxCharLen = unboxed.getResult(1);
````
- **L145 EN**: Comment explains nearby logic, intent, or metadata: `have been used in the interface).`.
  **L145 CN**: 注释说明附近代码的逻辑、意图或元数据：`have been used in the interface).`。
- **L146 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L146 CN**: 开始 `if` 控制流语句并计算其条件。
- **L147 EN**: Executes a call or declaration centered on `mlir::emitError`.
  **L147 CN**: 执行以 `mlir::emitError` 为核心的调用或声明。
- **L148 EN**: Closes the current lexical scope or compound statement.
  **L148 CN**: 结束当前词法作用域或复合语句块。
- **L149 EN**: Blank line separating nearby declarations or logic blocks.
  **L149 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L150 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L150 CN**: 开始 `if` 控制流语句并计算其条件。
- **L151 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L151 CN**: 开始 `if` 控制流语句并计算其条件。
- **L152 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L152 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L153 EN**: Transitions from the previous branch into an `else if` condition.
  **L153 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L154 EN**: Initializes variable `refType` from the right-hand expression.
  **L154 CN**: 使用右侧表达式初始化变量 `refType`。
- **L155 EN**: Comment explains nearby logic, intent, or metadata: `If the embox is accessible, use its operand to avoid filling`.
  **L155 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the embox is accessible, use its operand to avoid filling`。
- **L156 EN**: Comment explains nearby logic, intent, or metadata: `the generated fir with embox/unbox.`.
  **L156 CN**: 注释说明附近代码的逻辑、意图或元数据：`the generated fir with embox/unbox.`。
- **L157 EN**: Executes a standalone statement or declaration: `mlir::Value boxCharLen;`.
  **L157 CN**: 执行一条独立语句或声明：`mlir::Value boxCharLen;`。
- **L158 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L158 CN**: 开始 `if` 控制流语句并计算其条件。
- **L159 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L159 CN**: 开始 `if` 控制流语句并计算其条件。
- **L160 EN**: Executes a call or declaration centered on `box.getMemref`.
  **L160 CN**: 执行以 `box.getMemref` 为核心的调用或声明。
- **L161 EN**: Executes a call or declaration centered on `box.getLen`.
  **L161 CN**: 执行以 `box.getLen` 为核心的调用或声明。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L164 CN**: 开始 `if` 控制流语句并计算其条件。
- **L165 EN**: Continues the surrounding expression or declaration: `auto unboxed =`.
  **L165 CN**: 继续构造周围的表达式或声明：`auto unboxed =`。
- **L166 EN**: Executes a call or declaration centered on `fir::UnboxCharOp::create`.
  **L166 CN**: 执行以 `fir::UnboxCharOp::create` 为核心的调用或声明。
- **L167 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L167 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L168 EN**: Executes a call or declaration centered on `unboxed.getResult`.
  **L168 CN**: 执行以 `unboxed.getResult` 为核心的调用或声明。

### Lines 169-192

````cpp
    }
    if (!resultLen) {
      resultLen = boxCharLen;
    }
  } else if (mlir::isa<fir::BoxType>(type)) {
    mlir::emitError(loc, "descriptor or derived type not yet handled");
  } else {
    llvm_unreachable("Cannot translate mlir::Value to character ExtendedValue");
  }

  if (!base) {
    if (auto load =
            mlir::dyn_cast_or_null<fir::LoadOp>(character.getDefiningOp())) {
      base = load.getOperand();
    } else {
      return materializeValue(fir::getBase(character));
    }
  }
  if (!resultLen)
    llvm::report_fatal_error("no dynamic length found for character");
  if (!extents.empty())
    return fir::CharArrayBoxValue{base, resultLen, extents};
  return fir::CharBoxValue{base, resultLen};
}
````
- **L169 EN**: Closes the current lexical scope or compound statement.
  **L169 CN**: 结束当前词法作用域或复合语句块。
- **L170 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L170 CN**: 开始 `if` 控制流语句并计算其条件。
- **L171 EN**: Executes a standalone statement or declaration: `resultLen = boxCharLen;`.
  **L171 CN**: 执行一条独立语句或声明：`resultLen = boxCharLen;`。
- **L172 EN**: Closes the current lexical scope or compound statement.
  **L172 CN**: 结束当前词法作用域或复合语句块。
- **L173 EN**: Transitions from the previous branch into an `else if` condition.
  **L173 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L174 EN**: Executes a call or declaration centered on `mlir::emitError`.
  **L174 CN**: 执行以 `mlir::emitError` 为核心的调用或声明。
- **L175 EN**: Transitions from the previous branch into the alternative path.
  **L175 CN**: 从前一个分支过渡到备选路径。
- **L176 EN**: Marks this control path as unreachable to LLVM.
  **L176 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L177 EN**: Closes the current lexical scope or compound statement.
  **L177 CN**: 结束当前词法作用域或复合语句块。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L179 CN**: 开始 `if` 控制流语句并计算其条件。
- **L180 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L180 CN**: 开始 `if` 控制流语句并计算其条件。
- **L181 EN**: Starts a function, method, lambda, or structured scope: `mlir::dyn_cast_or_null<fir::LoadOp>(character.getDefiningOp())) {`.
  **L181 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::dyn_cast_or_null<fir::LoadOp>(character.getDefiningOp())) {`。
- **L182 EN**: Executes a call or declaration centered on `load.getOperand`.
  **L182 CN**: 执行以 `load.getOperand` 为核心的调用或声明。
- **L183 EN**: Transitions from the previous branch into the alternative path.
  **L183 CN**: 从前一个分支过渡到备选路径。
- **L184 EN**: Returns from the current function with `materializeValue(fir::getBase(character))`.
  **L184 CN**: 以 `materializeValue(fir::getBase(character))` 从当前函数返回。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L187 CN**: 开始 `if` 控制流语句并计算其条件。
- **L188 EN**: Executes a call or declaration centered on `llvm::report_fatal_error`.
  **L188 CN**: 执行以 `llvm::report_fatal_error` 为核心的调用或声明。
- **L189 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L189 CN**: 开始 `if` 控制流语句并计算其条件。
- **L190 EN**: Returns from the current function with `fir::CharArrayBoxValue{base, resultLen, extents}`.
  **L190 CN**: 以 `fir::CharArrayBoxValue{base, resultLen, extents}` 从当前函数返回。
- **L191 EN**: Returns from the current function with `fir::CharBoxValue{base, resultLen}`.
  **L191 CN**: 以 `fir::CharBoxValue{base, resultLen}` 从当前函数返回。
- **L192 EN**: Closes the current lexical scope or compound statement.
  **L192 CN**: 结束当前词法作用域或复合语句块。

### Lines 193-216

````cpp

static mlir::Type getSingletonCharType(mlir::MLIRContext *ctxt, int kind) {
  return fir::CharacterType::getSingleton(ctxt, kind);
}

mlir::Value
fir::factory::CharacterExprHelper::createEmbox(const fir::CharBoxValue &box) {
  // Base CharBoxValue of CharArrayBoxValue are ok here (do not require a scalar
  // type)
  auto charTy = recoverCharacterType(box.getBuffer().getType());
  auto boxCharType =
      fir::BoxCharType::get(builder.getContext(), charTy.getFKind());
  auto refType = fir::ReferenceType::get(boxCharType.getEleTy());
  mlir::Value buff = box.getBuffer();
  // fir.boxchar requires a memory reference. Allocate temp if the character is
  // not in memory.
  if (!fir::isa_ref_type(buff.getType())) {
    auto temp = builder.createTemporary(loc, buff.getType());
    fir::StoreOp::create(builder, loc, buff, temp);
    buff = temp;
  }
  // fir.emboxchar only accepts scalar, cast array buffer to a scalar buffer.
  if (mlir::isa<fir::SequenceType>(fir::dyn_cast_ptrEleTy(buff.getType())))
    buff = builder.createConvert(loc, refType, buff);
````
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L194 EN**: Starts a function, method, lambda, or structured scope: `static mlir::Type getSingletonCharType(mlir::MLIRContext *ctxt, int kind) {`.
  **L194 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static mlir::Type getSingletonCharType(mlir::MLIRContext *ctxt, int kind) {`。
- **L195 EN**: Returns from the current function with `fir::CharacterType::getSingleton(ctxt, kind)`.
  **L195 CN**: 以 `fir::CharacterType::getSingleton(ctxt, kind)` 从当前函数返回。
- **L196 EN**: Closes the current lexical scope or compound statement.
  **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L198 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L199 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::createEmbox(const fir::CharBoxValue &box) {`.
  **L199 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::createEmbox(const fir::CharBoxValue &box) {`。
- **L200 EN**: Comment explains nearby logic, intent, or metadata: `Base CharBoxValue of CharArrayBoxValue are ok here (do not require a scalar`.
  **L200 CN**: 注释说明附近代码的逻辑、意图或元数据：`Base CharBoxValue of CharArrayBoxValue are ok here (do not require a scalar`。
- **L201 EN**: Comment explains nearby logic, intent, or metadata: `type)`.
  **L201 CN**: 注释说明附近代码的逻辑、意图或元数据：`type)`。
- **L202 EN**: Initializes variable `charTy` from the right-hand expression.
  **L202 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L203 EN**: Continues the surrounding expression or declaration: `auto boxCharType =`.
  **L203 CN**: 继续构造周围的表达式或声明：`auto boxCharType =`。
- **L204 EN**: Executes a call or declaration centered on `fir::BoxCharType::get`.
  **L204 CN**: 执行以 `fir::BoxCharType::get` 为核心的调用或声明。
- **L205 EN**: Initializes variable `refType` from the right-hand expression.
  **L205 CN**: 使用右侧表达式初始化变量 `refType`。
- **L206 EN**: Initializes variable `buff` from the right-hand expression.
  **L206 CN**: 使用右侧表达式初始化变量 `buff`。
- **L207 EN**: Comment explains nearby logic, intent, or metadata: `fir.boxchar requires a memory reference. Allocate temp if the character is`.
  **L207 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.boxchar requires a memory reference. Allocate temp if the character is`。
- **L208 EN**: Comment explains nearby logic, intent, or metadata: `not in memory.`.
  **L208 CN**: 注释说明附近代码的逻辑、意图或元数据：`not in memory.`。
- **L209 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L209 CN**: 开始 `if` 控制流语句并计算其条件。
- **L210 EN**: Initializes variable `temp` from the right-hand expression.
  **L210 CN**: 使用右侧表达式初始化变量 `temp`。
- **L211 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L211 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L212 EN**: Executes a standalone statement or declaration: `buff = temp;`.
  **L212 CN**: 执行一条独立语句或声明：`buff = temp;`。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Comment explains nearby logic, intent, or metadata: `fir.emboxchar only accepts scalar, cast array buffer to a scalar buffer.`.
  **L214 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.emboxchar only accepts scalar, cast array buffer to a scalar buffer.`。
- **L215 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L215 CN**: 开始 `if` 控制流语句并计算其条件。
- **L216 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L216 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。

### Lines 217-240

````cpp
  // Convert in case the provided length is not of the integer type that must
  // be used in boxchar.
  auto len = builder.createConvert(loc, builder.getCharacterLengthType(),
                                   box.getLen());
  return fir::EmboxCharOp::create(builder, loc, boxCharType, buff, len);
}

fir::CharBoxValue fir::factory::CharacterExprHelper::toScalarCharacter(
    const fir::CharArrayBoxValue &box) {
  if (mlir::isa<fir::PointerType>(box.getBuffer().getType()))
    TODO(loc, "concatenating non contiguous character array into a scalar");

  // TODO: add a fast path multiplying new length at compile time if the info is
  // in the array type.
  auto lenType = builder.getCharacterLengthType();
  auto len = builder.createConvert(loc, lenType, box.getLen());
  for (auto extent : box.getExtents())
    len = mlir::arith::MulIOp::create(
        builder, loc, len, builder.createConvert(loc, lenType, extent));

  // TODO: typeLen can be improved in compiled constant cases
  // TODO: allow bare fir.array<> (no ref) conversion here ?
  auto typeLen = fir::CharacterType::unknownLen();
  auto kind = recoverCharacterType(box.getBuffer().getType()).getFKind();
````
- **L217 EN**: Comment explains nearby logic, intent, or metadata: `Convert in case the provided length is not of the integer type that must`.
  **L217 CN**: 注释说明附近代码的逻辑、意图或元数据：`Convert in case the provided length is not of the integer type that must`。
- **L218 EN**: Comment explains nearby logic, intent, or metadata: `be used in boxchar.`.
  **L218 CN**: 注释说明附近代码的逻辑、意图或元数据：`be used in boxchar.`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto len = builder.createConvert(loc, builder.getCharacterLengthType(),`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto len = builder.createConvert(loc, builder.getCharacterLengthType(),`。
- **L220 EN**: Executes a call or declaration centered on `box.getLen`.
  **L220 CN**: 执行以 `box.getLen` 为核心的调用或声明。
- **L221 EN**: Returns from the current function with `fir::EmboxCharOp::create(builder, loc, boxCharType, buff, len)`.
  **L221 CN**: 以 `fir::EmboxCharOp::create(builder, loc, boxCharType, buff, len)` 从当前函数返回。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Continues logic associated with callable symbol `toScalarCharacter`.
  **L224 CN**: 继续与可调用符号 `toScalarCharacter` 相关的逻辑。
- **L225 EN**: Continues the surrounding expression or declaration: `const fir::CharArrayBoxValue &box) {`.
  **L225 CN**: 继续构造周围的表达式或声明：`const fir::CharArrayBoxValue &box) {`。
- **L226 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L226 CN**: 开始 `if` 控制流语句并计算其条件。
- **L227 EN**: Executes a call or declaration centered on `TODO`.
  **L227 CN**: 执行以 `TODO` 为核心的调用或声明。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L229 EN**: Comment records a pending task or caution: `TODO: add a fast path multiplying new length at compile time if the info is`.
  **L229 CN**: 注释记录待办事项或注意点：`TODO: add a fast path multiplying new length at compile time if the info is`。
- **L230 EN**: Comment explains nearby logic, intent, or metadata: `in the array type.`.
  **L230 CN**: 注释说明附近代码的逻辑、意图或元数据：`in the array type.`。
- **L231 EN**: Initializes variable `lenType` from the right-hand expression.
  **L231 CN**: 使用右侧表达式初始化变量 `lenType`。
- **L232 EN**: Initializes variable `len` from the right-hand expression.
  **L232 CN**: 使用右侧表达式初始化变量 `len`。
- **L233 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L233 CN**: 开始 `for` 控制流语句并计算其条件。
- **L234 EN**: Continues logic associated with callable symbol `create`.
  **L234 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L235 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L235 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L237 EN**: Comment records a pending task or caution: `TODO: typeLen can be improved in compiled constant cases`.
  **L237 CN**: 注释记录待办事项或注意点：`TODO: typeLen can be improved in compiled constant cases`。
- **L238 EN**: Comment records a pending task or caution: `TODO: allow bare fir.array<> (no ref) conversion here ?`.
  **L238 CN**: 注释记录待办事项或注意点：`TODO: allow bare fir.array<> (no ref) conversion here ?`。
- **L239 EN**: Initializes variable `typeLen` from the right-hand expression.
  **L239 CN**: 使用右侧表达式初始化变量 `typeLen`。
- **L240 EN**: Initializes variable `kind` from the right-hand expression.
  **L240 CN**: 使用右侧表达式初始化变量 `kind`。

### Lines 241-264

````cpp
  auto charTy = fir::CharacterType::get(builder.getContext(), kind, typeLen);
  auto type = fir::ReferenceType::get(charTy);
  auto buffer = builder.createConvert(loc, type, box.getBuffer());
  return {buffer, len};
}

mlir::Value fir::factory::CharacterExprHelper::createEmbox(
    const fir::CharArrayBoxValue &box) {
  // Use same embox as for scalar. It's losing the actual data size information
  // (We do not multiply the length by the array size), but that is what Fortran
  // call interfaces using boxchar expect.
  return createEmbox(static_cast<const fir::CharBoxValue &>(box));
}

/// Get the address of the element at position \p index of the scalar character
/// \p buffer.
/// \p buffer must be of type !fir.ref<fir.char<k, len>>. The length may be
/// unknown. \p index must have any integer type, and is zero based. The return
/// value is a singleton address (!fir.ref<!fir.char<kind>>)
mlir::Value
fir::factory::CharacterExprHelper::createElementAddr(mlir::Value buffer,
                                                     mlir::Value index) {
  // The only way to address an element of a fir.ref<char<kind, len>> is to cast
  // it to a fir.array<len x fir.char<kind>> and use fir.coordinate_of.
````
- **L241 EN**: Initializes variable `charTy` from the right-hand expression.
  **L241 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L242 EN**: Initializes variable `type` from the right-hand expression.
  **L242 CN**: 使用右侧表达式初始化变量 `type`。
- **L243 EN**: Initializes variable `buffer` from the right-hand expression.
  **L243 CN**: 使用右侧表达式初始化变量 `buffer`。
- **L244 EN**: Returns from the current function with `{buffer, len}`.
  **L244 CN**: 以 `{buffer, len}` 从当前函数返回。
- **L245 EN**: Closes the current lexical scope or compound statement.
  **L245 CN**: 结束当前词法作用域或复合语句块。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L247 EN**: Continues logic associated with callable symbol `createEmbox`.
  **L247 CN**: 继续与可调用符号 `createEmbox` 相关的逻辑。
- **L248 EN**: Continues the surrounding expression or declaration: `const fir::CharArrayBoxValue &box) {`.
  **L248 CN**: 继续构造周围的表达式或声明：`const fir::CharArrayBoxValue &box) {`。
- **L249 EN**: Comment explains nearby logic, intent, or metadata: `Use same embox as for scalar. It's losing the actual data size information`.
  **L249 CN**: 注释说明附近代码的逻辑、意图或元数据：`Use same embox as for scalar. It's losing the actual data size information`。
- **L250 EN**: Comment explains nearby logic, intent, or metadata: `(We do not multiply the length by the array size), but that is what Fortran`.
  **L250 CN**: 注释说明附近代码的逻辑、意图或元数据：`(We do not multiply the length by the array size), but that is what Fortran`。
- **L251 EN**: Comment explains nearby logic, intent, or metadata: `call interfaces using boxchar expect.`.
  **L251 CN**: 注释说明附近代码的逻辑、意图或元数据：`call interfaces using boxchar expect.`。
- **L252 EN**: Returns from the current function with `createEmbox(static_cast<const fir::CharBoxValue &>(box))`.
  **L252 CN**: 以 `createEmbox(static_cast<const fir::CharBoxValue &>(box))` 从当前函数返回。
- **L253 EN**: Closes the current lexical scope or compound statement.
  **L253 CN**: 结束当前词法作用域或复合语句块。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L255 EN**: Comment explains nearby logic, intent, or metadata: `Get the address of the element at position \p index of the scalar character`.
  **L255 CN**: 注释说明附近代码的逻辑、意图或元数据：`Get the address of the element at position \p index of the scalar character`。
- **L256 EN**: Comment explains nearby logic, intent, or metadata: `\p buffer.`.
  **L256 CN**: 注释说明附近代码的逻辑、意图或元数据：`\p buffer.`。
- **L257 EN**: Comment explains nearby logic, intent, or metadata: `\p buffer must be of type !fir.ref<fir.char<k, len>>. The length may be`.
  **L257 CN**: 注释说明附近代码的逻辑、意图或元数据：`\p buffer must be of type !fir.ref<fir.char<k, len>>. The length may be`。
- **L258 EN**: Comment explains nearby logic, intent, or metadata: `unknown. \p index must have any integer type, and is zero based. The return`.
  **L258 CN**: 注释说明附近代码的逻辑、意图或元数据：`unknown. \p index must have any integer type, and is zero based. The return`。
- **L259 EN**: Comment explains nearby logic, intent, or metadata: `value is a singleton address (!fir.ref<!fir.char<kind>>)`.
  **L259 CN**: 注释说明附近代码的逻辑、意图或元数据：`value is a singleton address (!fir.ref<!fir.char<kind>>)`。
- **L260 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L260 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::CharacterExprHelper::createElementAddr(mlir::Value buffer,`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::CharacterExprHelper::createElementAddr(mlir::Value buffer,`。
- **L262 EN**: Continues the surrounding expression or declaration: `mlir::Value index) {`.
  **L262 CN**: 继续构造周围的表达式或声明：`mlir::Value index) {`。
- **L263 EN**: Comment explains nearby logic, intent, or metadata: `The only way to address an element of a fir.ref<char<kind, len>> is to cast`.
  **L263 CN**: 注释说明附近代码的逻辑、意图或元数据：`The only way to address an element of a fir.ref<char<kind, len>> is to cast`。
- **L264 EN**: Comment explains nearby logic, intent, or metadata: `it to a fir.array<len x fir.char<kind>> and use fir.coordinate_of.`.
  **L264 CN**: 注释说明附近代码的逻辑、意图或元数据：`it to a fir.array<len x fir.char<kind>> and use fir.coordinate_of.`。

### Lines 265-288

````cpp
  auto bufferType = buffer.getType();
  assert(fir::isa_ref_type(bufferType));
  assert(isCharacterScalar(bufferType));
  auto charTy = recoverCharacterType(bufferType);
  auto singleTy = getSingletonCharType(builder.getContext(), charTy.getFKind());
  auto singleRefTy = builder.getRefType(singleTy);
  auto extent = fir::SequenceType::getUnknownExtent();
  if (charTy.getLen() != fir::CharacterType::unknownLen())
    extent = charTy.getLen();
  const bool isVolatile = fir::isa_volatile_type(buffer.getType());
  auto sequenceType = fir::SequenceType::get({extent}, singleTy);
  auto coorTy = builder.getRefType(sequenceType, isVolatile);

  auto coor = builder.createConvert(loc, coorTy, buffer);
  auto i = builder.createConvert(loc, builder.getIndexType(), index);
  return fir::CoordinateOp::create(builder, loc, singleRefTy, coor, i);
}

/// Load a character out of `buff` from offset `index`.
/// `buff` must be a reference to memory.
mlir::Value
fir::factory::CharacterExprHelper::createLoadCharAt(mlir::Value buff,
                                                    mlir::Value index) {
  LLVM_DEBUG(llvm::dbgs() << "load a char: " << buff << " type: "
````
- **L265 EN**: Initializes variable `bufferType` from the right-hand expression.
  **L265 CN**: 使用右侧表达式初始化变量 `bufferType`。
- **L266 EN**: Checks an internal invariant in debug builds.
  **L266 CN**: 在调试构建中检查内部不变式。
- **L267 EN**: Checks an internal invariant in debug builds.
  **L267 CN**: 在调试构建中检查内部不变式。
- **L268 EN**: Initializes variable `charTy` from the right-hand expression.
  **L268 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L269 EN**: Initializes variable `singleTy` from the right-hand expression.
  **L269 CN**: 使用右侧表达式初始化变量 `singleTy`。
- **L270 EN**: Initializes variable `singleRefTy` from the right-hand expression.
  **L270 CN**: 使用右侧表达式初始化变量 `singleRefTy`。
- **L271 EN**: Initializes variable `extent` from the right-hand expression.
  **L271 CN**: 使用右侧表达式初始化变量 `extent`。
- **L272 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L272 CN**: 开始 `if` 控制流语句并计算其条件。
- **L273 EN**: Executes a call or declaration centered on `charTy.getLen`.
  **L273 CN**: 执行以 `charTy.getLen` 为核心的调用或声明。
- **L274 EN**: Initializes variable `isVolatile` from the right-hand expression.
  **L274 CN**: 使用右侧表达式初始化变量 `isVolatile`。
- **L275 EN**: Initializes variable `sequenceType` from the right-hand expression.
  **L275 CN**: 使用右侧表达式初始化变量 `sequenceType`。
- **L276 EN**: Initializes variable `coorTy` from the right-hand expression.
  **L276 CN**: 使用右侧表达式初始化变量 `coorTy`。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L278 EN**: Initializes variable `coor` from the right-hand expression.
  **L278 CN**: 使用右侧表达式初始化变量 `coor`。
- **L279 EN**: Initializes variable `i` from the right-hand expression.
  **L279 CN**: 使用右侧表达式初始化变量 `i`。
- **L280 EN**: Returns from the current function with `fir::CoordinateOp::create(builder, loc, singleRefTy, coor, i)`.
  **L280 CN**: 以 `fir::CoordinateOp::create(builder, loc, singleRefTy, coor, i)` 从当前函数返回。
- **L281 EN**: Closes the current lexical scope or compound statement.
  **L281 CN**: 结束当前词法作用域或复合语句块。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L283 EN**: Comment explains nearby logic, intent, or metadata: `Load a character out of `buff` from offset `index`.`.
  **L283 CN**: 注释说明附近代码的逻辑、意图或元数据：`Load a character out of `buff` from offset `index`.`。
- **L284 EN**: Comment explains nearby logic, intent, or metadata: ``buff` must be a reference to memory.`.
  **L284 CN**: 注释说明附近代码的逻辑、意图或元数据：``buff` must be a reference to memory.`。
- **L285 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L285 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::CharacterExprHelper::createLoadCharAt(mlir::Value buff,`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::CharacterExprHelper::createLoadCharAt(mlir::Value buff,`。
- **L287 EN**: Continues the surrounding expression or declaration: `mlir::Value index) {`.
  **L287 CN**: 继续构造周围的表达式或声明：`mlir::Value index) {`。
- **L288 EN**: Continues logic associated with callable symbol `LLVM_DEBUG`.
  **L288 CN**: 继续与可调用符号 `LLVM_DEBUG` 相关的逻辑。

### Lines 289-312

````cpp
                          << buff.getType() << " at: " << index << '\n');
  return fir::LoadOp::create(builder, loc, createElementAddr(buff, index));
}

/// Store the singleton character `c` to `str` at offset `index`.
/// `str` must be a reference to memory.
void fir::factory::CharacterExprHelper::createStoreCharAt(mlir::Value str,
                                                          mlir::Value index,
                                                          mlir::Value c) {
  LLVM_DEBUG(llvm::dbgs() << "store the char: " << c << " into: " << str
                          << " type: " << str.getType() << " at: " << index
                          << '\n');
  auto addr = createElementAddr(str, index);
  fir::StoreOp::create(builder, loc, c, addr);
}

// FIXME: this temp is useless... either fir.coordinate_of needs to
// work on "loaded" characters (!fir.array<len x fir.char<kind>>) or
// character should never be loaded.
// If this is a fir.array<>, allocate and store the value so that
// fir.cooridnate_of can be use on the value.
mlir::Value fir::factory::CharacterExprHelper::getCharBoxBuffer(
    const fir::CharBoxValue &box) {
  auto buff = box.getBuffer();
````
- **L289 EN**: Executes a call or declaration centered on `buff.getType`.
  **L289 CN**: 执行以 `buff.getType` 为核心的调用或声明。
- **L290 EN**: Returns from the current function with `fir::LoadOp::create(builder, loc, createElementAddr(buff, index))`.
  **L290 CN**: 以 `fir::LoadOp::create(builder, loc, createElementAddr(buff, index))` 从当前函数返回。
- **L291 EN**: Closes the current lexical scope or compound statement.
  **L291 CN**: 结束当前词法作用域或复合语句块。
- **L292 EN**: Blank line separating nearby declarations or logic blocks.
  **L292 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L293 EN**: Comment explains nearby logic, intent, or metadata: `Store the singleton character `c` to `str` at offset `index`.`.
  **L293 CN**: 注释说明附近代码的逻辑、意图或元数据：`Store the singleton character `c` to `str` at offset `index`.`。
- **L294 EN**: Comment explains nearby logic, intent, or metadata: ``str` must be a reference to memory.`.
  **L294 CN**: 注释说明附近代码的逻辑、意图或元数据：``str` must be a reference to memory.`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::factory::CharacterExprHelper::createStoreCharAt(mlir::Value str,`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::factory::CharacterExprHelper::createStoreCharAt(mlir::Value str,`。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value index,`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value index,`。
- **L297 EN**: Continues the surrounding expression or declaration: `mlir::Value c) {`.
  **L297 CN**: 继续构造周围的表达式或声明：`mlir::Value c) {`。
- **L298 EN**: Continues logic associated with callable symbol `LLVM_DEBUG`.
  **L298 CN**: 继续与可调用符号 `LLVM_DEBUG` 相关的逻辑。
- **L299 EN**: Continues logic associated with callable symbol `getType`.
  **L299 CN**: 继续与可调用符号 `getType` 相关的逻辑。
- **L300 EN**: Executes a standalone statement or declaration: `<< '\n');`.
  **L300 CN**: 执行一条独立语句或声明：`<< '\n');`。
- **L301 EN**: Initializes variable `addr` from the right-hand expression.
  **L301 CN**: 使用右侧表达式初始化变量 `addr`。
- **L302 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L302 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L303 EN**: Closes the current lexical scope or compound statement.
  **L303 CN**: 结束当前词法作用域或复合语句块。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L305 EN**: Comment records a pending task or caution: `FIXME: this temp is useless... either fir.coordinate_of needs to`.
  **L305 CN**: 注释记录待办事项或注意点：`FIXME: this temp is useless... either fir.coordinate_of needs to`。
- **L306 EN**: Comment explains nearby logic, intent, or metadata: `work on "loaded" characters (!fir.array<len x fir.char<kind>>) or`.
  **L306 CN**: 注释说明附近代码的逻辑、意图或元数据：`work on "loaded" characters (!fir.array<len x fir.char<kind>>) or`。
- **L307 EN**: Comment explains nearby logic, intent, or metadata: `character should never be loaded.`.
  **L307 CN**: 注释说明附近代码的逻辑、意图或元数据：`character should never be loaded.`。
- **L308 EN**: Comment explains nearby logic, intent, or metadata: `If this is a fir.array<>, allocate and store the value so that`.
  **L308 CN**: 注释说明附近代码的逻辑、意图或元数据：`If this is a fir.array<>, allocate and store the value so that`。
- **L309 EN**: Comment explains nearby logic, intent, or metadata: `fir.cooridnate_of can be use on the value.`.
  **L309 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.cooridnate_of can be use on the value.`。
- **L310 EN**: Continues logic associated with callable symbol `getCharBoxBuffer`.
  **L310 CN**: 继续与可调用符号 `getCharBoxBuffer` 相关的逻辑。
- **L311 EN**: Continues the surrounding expression or declaration: `const fir::CharBoxValue &box) {`.
  **L311 CN**: 继续构造周围的表达式或声明：`const fir::CharBoxValue &box) {`。
- **L312 EN**: Initializes variable `buff` from the right-hand expression.
  **L312 CN**: 使用右侧表达式初始化变量 `buff`。

### Lines 313-336

````cpp
  if (fir::isa_char(buff.getType())) {
    auto newBuff = fir::AllocaOp::create(builder, loc, buff.getType());
    fir::StoreOp::create(builder, loc, buff, newBuff);
    return newBuff;
  }
  return buff;
}

/// Create a loop to copy `count` characters from `src` to `dest`. Note that the
/// KIND indicates the number of bits in a code point. (ASCII, UCS-2, or UCS-4.)
void fir::factory::CharacterExprHelper::createCopy(
    const fir::CharBoxValue &dest, const fir::CharBoxValue &src,
    mlir::Value count) {
  auto fromBuff = getCharBoxBuffer(src);
  auto toBuff = getCharBoxBuffer(dest);
  LLVM_DEBUG(llvm::dbgs() << "create char copy from: "; src.dump();
             llvm::dbgs() << " to: "; dest.dump();
             llvm::dbgs() << " count: " << count << '\n');
  auto kind = getCharacterKind(src.getBuffer().getType());
  // If the src and dest are the same KIND, then use memmove to move the bits.
  // We don't have to worry about overlapping ranges with memmove.
  if (getCharacterKind(dest.getBuffer().getType()) == kind) {
    const bool isVolatile = fir::isa_volatile_type(fromBuff.getType()) ||
                            fir::isa_volatile_type(toBuff.getType());
````
- **L313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L314 EN**: Initializes variable `newBuff` from the right-hand expression.
  **L314 CN**: 使用右侧表达式初始化变量 `newBuff`。
- **L315 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L315 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L316 EN**: Returns from the current function with `newBuff`.
  **L316 CN**: 以 `newBuff` 从当前函数返回。
- **L317 EN**: Closes the current lexical scope or compound statement.
  **L317 CN**: 结束当前词法作用域或复合语句块。
- **L318 EN**: Returns from the current function with `buff`.
  **L318 CN**: 以 `buff` 从当前函数返回。
- **L319 EN**: Closes the current lexical scope or compound statement.
  **L319 CN**: 结束当前词法作用域或复合语句块。
- **L320 EN**: Blank line separating nearby declarations or logic blocks.
  **L320 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L321 EN**: Comment explains nearby logic, intent, or metadata: `Create a loop to copy `count` characters from `src` to `dest`. Note that the`.
  **L321 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create a loop to copy `count` characters from `src` to `dest`. Note that the`。
- **L322 EN**: Comment explains nearby logic, intent, or metadata: `KIND indicates the number of bits in a code point. (ASCII, UCS-2, or UCS-4.)`.
  **L322 CN**: 注释说明附近代码的逻辑、意图或元数据：`KIND indicates the number of bits in a code point. (ASCII, UCS-2, or UCS-4.)`。
- **L323 EN**: Continues logic associated with callable symbol `createCopy`.
  **L323 CN**: 继续与可调用符号 `createCopy` 相关的逻辑。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::CharBoxValue &dest, const fir::CharBoxValue &src,`.
  **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::CharBoxValue &dest, const fir::CharBoxValue &src,`。
- **L325 EN**: Continues the surrounding expression or declaration: `mlir::Value count) {`.
  **L325 CN**: 继续构造周围的表达式或声明：`mlir::Value count) {`。
- **L326 EN**: Initializes variable `fromBuff` from the right-hand expression.
  **L326 CN**: 使用右侧表达式初始化变量 `fromBuff`。
- **L327 EN**: Initializes variable `toBuff` from the right-hand expression.
  **L327 CN**: 使用右侧表达式初始化变量 `toBuff`。
- **L328 EN**: Executes a call or declaration centered on `LLVM_DEBUG`.
  **L328 CN**: 执行以 `LLVM_DEBUG` 为核心的调用或声明。
- **L329 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L329 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L330 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L330 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L331 EN**: Initializes variable `kind` from the right-hand expression.
  **L331 CN**: 使用右侧表达式初始化变量 `kind`。
- **L332 EN**: Comment explains nearby logic, intent, or metadata: `If the src and dest are the same KIND, then use memmove to move the bits.`.
  **L332 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the src and dest are the same KIND, then use memmove to move the bits.`。
- **L333 EN**: Comment explains nearby logic, intent, or metadata: `We don't have to worry about overlapping ranges with memmove.`.
  **L333 CN**: 注释说明附近代码的逻辑、意图或元数据：`We don't have to worry about overlapping ranges with memmove.`。
- **L334 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L334 CN**: 开始 `if` 控制流语句并计算其条件。
- **L335 EN**: Continues logic associated with callable symbol `isa_volatile_type`.
  **L335 CN**: 继续与可调用符号 `isa_volatile_type` 相关的逻辑。
- **L336 EN**: Executes a call or declaration centered on `fir::isa_volatile_type`.
  **L336 CN**: 执行以 `fir::isa_volatile_type` 为核心的调用或声明。

### Lines 337-360

````cpp
    auto bytes = builder.getKindMap().getCharacterBitsize(kind) / 8;
    auto i64Ty = builder.getI64Type();
    auto kindBytes = builder.createIntegerConstant(loc, i64Ty, bytes);
    auto castCount = builder.createConvert(loc, i64Ty, count);
    auto totalBytes =
        mlir::arith::MulIOp::create(builder, loc, kindBytes, castCount);
    auto llvmPointerType =
        mlir::LLVM::LLVMPointerType::get(builder.getContext());
    auto toPtr = builder.createConvert(loc, llvmPointerType, toBuff);
    auto fromPtr = builder.createConvert(loc, llvmPointerType, fromBuff);
    mlir::LLVM::MemmoveOp::create(builder, loc, toPtr, fromPtr, totalBytes,
                                  isVolatile);
    return;
  }

  // Convert a CHARACTER of one KIND into a CHARACTER of another KIND.
  fir::CharConvertOp::create(builder, loc, src.getBuffer(), count,
                             dest.getBuffer());
}

void fir::factory::CharacterExprHelper::createPadding(
    const fir::CharBoxValue &str, mlir::Value lower, mlir::Value upper) {
  auto blank = createBlankConstant(getCharacterType(str));
  // Always create the loop, if upper < lower, no iteration will be
````
- **L337 EN**: Initializes variable `bytes` from the right-hand expression.
  **L337 CN**: 使用右侧表达式初始化变量 `bytes`。
- **L338 EN**: Initializes variable `i64Ty` from the right-hand expression.
  **L338 CN**: 使用右侧表达式初始化变量 `i64Ty`。
- **L339 EN**: Initializes variable `kindBytes` from the right-hand expression.
  **L339 CN**: 使用右侧表达式初始化变量 `kindBytes`。
- **L340 EN**: Initializes variable `castCount` from the right-hand expression.
  **L340 CN**: 使用右侧表达式初始化变量 `castCount`。
- **L341 EN**: Continues the surrounding expression or declaration: `auto totalBytes =`.
  **L341 CN**: 继续构造周围的表达式或声明：`auto totalBytes =`。
- **L342 EN**: Executes a call or declaration centered on `mlir::arith::MulIOp::create`.
  **L342 CN**: 执行以 `mlir::arith::MulIOp::create` 为核心的调用或声明。
- **L343 EN**: Continues the surrounding expression or declaration: `auto llvmPointerType =`.
  **L343 CN**: 继续构造周围的表达式或声明：`auto llvmPointerType =`。
- **L344 EN**: Executes a call or declaration centered on `mlir::LLVM::LLVMPointerType::get`.
  **L344 CN**: 执行以 `mlir::LLVM::LLVMPointerType::get` 为核心的调用或声明。
- **L345 EN**: Initializes variable `toPtr` from the right-hand expression.
  **L345 CN**: 使用右侧表达式初始化变量 `toPtr`。
- **L346 EN**: Initializes variable `fromPtr` from the right-hand expression.
  **L346 CN**: 使用右侧表达式初始化变量 `fromPtr`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::LLVM::MemmoveOp::create(builder, loc, toPtr, fromPtr, totalBytes,`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::LLVM::MemmoveOp::create(builder, loc, toPtr, fromPtr, totalBytes,`。
- **L348 EN**: Executes a standalone statement or declaration: `isVolatile);`.
  **L348 CN**: 执行一条独立语句或声明：`isVolatile);`。
- **L349 EN**: Returns from the current function with `void`.
  **L349 CN**: 以 `void` 从当前函数返回。
- **L350 EN**: Closes the current lexical scope or compound statement.
  **L350 CN**: 结束当前词法作用域或复合语句块。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L352 EN**: Comment explains nearby logic, intent, or metadata: `Convert a CHARACTER of one KIND into a CHARACTER of another KIND.`.
  **L352 CN**: 注释说明附近代码的逻辑、意图或元数据：`Convert a CHARACTER of one KIND into a CHARACTER of another KIND.`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::CharConvertOp::create(builder, loc, src.getBuffer(), count,`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::CharConvertOp::create(builder, loc, src.getBuffer(), count,`。
- **L354 EN**: Executes a call or declaration centered on `dest.getBuffer`.
  **L354 CN**: 执行以 `dest.getBuffer` 为核心的调用或声明。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L357 EN**: Continues logic associated with callable symbol `createPadding`.
  **L357 CN**: 继续与可调用符号 `createPadding` 相关的逻辑。
- **L358 EN**: Continues the surrounding expression or declaration: `const fir::CharBoxValue &str, mlir::Value lower, mlir::Value upper) {`.
  **L358 CN**: 继续构造周围的表达式或声明：`const fir::CharBoxValue &str, mlir::Value lower, mlir::Value upper) {`。
- **L359 EN**: Initializes variable `blank` from the right-hand expression.
  **L359 CN**: 使用右侧表达式初始化变量 `blank`。
- **L360 EN**: Comment explains nearby logic, intent, or metadata: `Always create the loop, if upper < lower, no iteration will be`.
  **L360 CN**: 注释说明附近代码的逻辑、意图或元数据：`Always create the loop, if upper < lower, no iteration will be`。

### Lines 361-384

````cpp
  // executed.
  auto toBuff = getCharBoxBuffer(str);
  fir::factory::DoLoopHelper{builder, loc}.createLoop(
      lower, upper, [&](fir::FirOpBuilder &, mlir::Value index) {
        createStoreCharAt(toBuff, index, blank);
      });
}

fir::CharBoxValue
fir::factory::CharacterExprHelper::createCharacterTemp(mlir::Type type,
                                                       mlir::Value len) {
  auto kind = recoverCharacterType(type).getFKind();
  auto typeLen = fir::CharacterType::unknownLen();
  // If len is a constant, reflect the length in the type.
  if (auto cstLen = getIntIfConstant(len))
    typeLen = *cstLen;
  auto *ctxt = builder.getContext();
  auto charTy = fir::CharacterType::get(ctxt, kind, typeLen);
  llvm::SmallVector<mlir::Value> lenParams;
  if (typeLen == fir::CharacterType::unknownLen())
    lenParams.push_back(len);
  auto ref = builder.allocateLocal(loc, charTy, "", ".chrtmp",
                                   /*shape=*/{}, lenParams);
  return {ref, len};
````
- **L361 EN**: Comment explains nearby logic, intent, or metadata: `executed.`.
  **L361 CN**: 注释说明附近代码的逻辑、意图或元数据：`executed.`。
- **L362 EN**: Initializes variable `toBuff` from the right-hand expression.
  **L362 CN**: 使用右侧表达式初始化变量 `toBuff`。
- **L363 EN**: Continues logic associated with callable symbol `createLoop`.
  **L363 CN**: 继续与可调用符号 `createLoop` 相关的逻辑。
- **L364 EN**: Starts a function, method, lambda, or structured scope: `lower, upper, [&](fir::FirOpBuilder &, mlir::Value index) {`.
  **L364 CN**: 开始一个函数、方法、lambda 或结构化作用域：`lower, upper, [&](fir::FirOpBuilder &, mlir::Value index) {`。
- **L365 EN**: Executes a call or declaration centered on `createStoreCharAt`.
  **L365 CN**: 执行以 `createStoreCharAt` 为核心的调用或声明。
- **L366 EN**: Executes a standalone statement or declaration: `});`.
  **L366 CN**: 执行一条独立语句或声明：`});`。
- **L367 EN**: Closes the current lexical scope or compound statement.
  **L367 CN**: 结束当前词法作用域或复合语句块。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L369 EN**: Continues the surrounding expression or declaration: `fir::CharBoxValue`.
  **L369 CN**: 继续构造周围的表达式或声明：`fir::CharBoxValue`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::CharacterExprHelper::createCharacterTemp(mlir::Type type,`.
  **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::CharacterExprHelper::createCharacterTemp(mlir::Type type,`。
- **L371 EN**: Continues the surrounding expression or declaration: `mlir::Value len) {`.
  **L371 CN**: 继续构造周围的表达式或声明：`mlir::Value len) {`。
- **L372 EN**: Initializes variable `kind` from the right-hand expression.
  **L372 CN**: 使用右侧表达式初始化变量 `kind`。
- **L373 EN**: Initializes variable `typeLen` from the right-hand expression.
  **L373 CN**: 使用右侧表达式初始化变量 `typeLen`。
- **L374 EN**: Comment explains nearby logic, intent, or metadata: `If len is a constant, reflect the length in the type.`.
  **L374 CN**: 注释说明附近代码的逻辑、意图或元数据：`If len is a constant, reflect the length in the type.`。
- **L375 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L375 CN**: 开始 `if` 控制流语句并计算其条件。
- **L376 EN**: Executes a standalone statement or declaration: `typeLen = *cstLen;`.
  **L376 CN**: 执行一条独立语句或声明：`typeLen = *cstLen;`。
- **L377 EN**: Executes a call or declaration centered on `builder.getContext`.
  **L377 CN**: 执行以 `builder.getContext` 为核心的调用或声明。
- **L378 EN**: Initializes variable `charTy` from the right-hand expression.
  **L378 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L379 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> lenParams;`.
  **L379 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> lenParams;`。
- **L380 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L380 CN**: 开始 `if` 控制流语句并计算其条件。
- **L381 EN**: Executes a call or declaration centered on `lenParams.push_back`.
  **L381 CN**: 执行以 `lenParams.push_back` 为核心的调用或声明。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto ref = builder.allocateLocal(loc, charTy, "", ".chrtmp",`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto ref = builder.allocateLocal(loc, charTy, "", ".chrtmp",`。
- **L383 EN**: Comment explains nearby logic, intent, or metadata: `shape=*/{}, lenParams);`.
  **L383 CN**: 注释说明附近代码的逻辑、意图或元数据：`shape=*/{}, lenParams);`。
- **L384 EN**: Returns from the current function with `{ref, len}`.
  **L384 CN**: 以 `{ref, len}` 从当前函数返回。

### Lines 385-408

````cpp
}

fir::CharBoxValue fir::factory::CharacterExprHelper::createTempFrom(
    const fir::ExtendedValue &source) {
  const auto *charBox = source.getCharBox();
  if (!charBox)
    fir::emitFatalError(loc, "source must be a fir::CharBoxValue");
  auto len = charBox->getLen();
  auto sourceTy = charBox->getBuffer().getType();
  auto temp = createCharacterTemp(sourceTy, len);
  if (fir::isa_ref_type(sourceTy)) {
    createCopy(temp, *charBox, len);
  } else {
    auto ref = builder.createConvert(loc, builder.getRefType(sourceTy),
                                     temp.getBuffer());
    fir::StoreOp::create(builder, loc, charBox->getBuffer(), ref);
  }
  return temp;
}

// Simple length one character assignment without loops.
void fir::factory::CharacterExprHelper::createLengthOneAssign(
    const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {
  auto addr = lhs.getBuffer();
````
- **L385 EN**: Closes the current lexical scope or compound statement.
  **L385 CN**: 结束当前词法作用域或复合语句块。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L387 EN**: Continues logic associated with callable symbol `createTempFrom`.
  **L387 CN**: 继续与可调用符号 `createTempFrom` 相关的逻辑。
- **L388 EN**: Continues the surrounding expression or declaration: `const fir::ExtendedValue &source) {`.
  **L388 CN**: 继续构造周围的表达式或声明：`const fir::ExtendedValue &source) {`。
- **L389 EN**: Executes a call or declaration centered on `source.getCharBox`.
  **L389 CN**: 执行以 `source.getCharBox` 为核心的调用或声明。
- **L390 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L390 CN**: 开始 `if` 控制流语句并计算其条件。
- **L391 EN**: Executes a call or declaration centered on `fir::emitFatalError`.
  **L391 CN**: 执行以 `fir::emitFatalError` 为核心的调用或声明。
- **L392 EN**: Initializes variable `len` from the right-hand expression.
  **L392 CN**: 使用右侧表达式初始化变量 `len`。
- **L393 EN**: Initializes variable `sourceTy` from the right-hand expression.
  **L393 CN**: 使用右侧表达式初始化变量 `sourceTy`。
- **L394 EN**: Initializes variable `temp` from the right-hand expression.
  **L394 CN**: 使用右侧表达式初始化变量 `temp`。
- **L395 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L395 CN**: 开始 `if` 控制流语句并计算其条件。
- **L396 EN**: Executes a call or declaration centered on `createCopy`.
  **L396 CN**: 执行以 `createCopy` 为核心的调用或声明。
- **L397 EN**: Transitions from the previous branch into the alternative path.
  **L397 CN**: 从前一个分支过渡到备选路径。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto ref = builder.createConvert(loc, builder.getRefType(sourceTy),`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto ref = builder.createConvert(loc, builder.getRefType(sourceTy),`。
- **L399 EN**: Executes a call or declaration centered on `temp.getBuffer`.
  **L399 CN**: 执行以 `temp.getBuffer` 为核心的调用或声明。
- **L400 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L400 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L401 EN**: Closes the current lexical scope or compound statement.
  **L401 CN**: 结束当前词法作用域或复合语句块。
- **L402 EN**: Returns from the current function with `temp`.
  **L402 CN**: 以 `temp` 从当前函数返回。
- **L403 EN**: Closes the current lexical scope or compound statement.
  **L403 CN**: 结束当前词法作用域或复合语句块。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L405 EN**: Comment explains nearby logic, intent, or metadata: `Simple length one character assignment without loops.`.
  **L405 CN**: 注释说明附近代码的逻辑、意图或元数据：`Simple length one character assignment without loops.`。
- **L406 EN**: Continues logic associated with callable symbol `createLengthOneAssign`.
  **L406 CN**: 继续与可调用符号 `createLengthOneAssign` 相关的逻辑。
- **L407 EN**: Continues the surrounding expression or declaration: `const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {`.
  **L407 CN**: 继续构造周围的表达式或声明：`const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {`。
- **L408 EN**: Initializes variable `addr` from the right-hand expression.
  **L408 CN**: 使用右侧表达式初始化变量 `addr`。

### Lines 409-432

````cpp
  auto toTy = fir::unwrapRefType(addr.getType());
  mlir::Value val = rhs.getBuffer();
  if (fir::isa_ref_type(val.getType())) {
    auto fromCharLen1RefTy = builder.getRefType(getSingletonCharType(
        builder.getContext(),
        getCharacterKind(fir::unwrapRefType(val.getType()))));
    val = fir::LoadOp::create(
        builder, loc, builder.createConvert(loc, fromCharLen1RefTy, val));
  }
  auto toCharLen1Ty =
      getSingletonCharType(builder.getContext(), getCharacterKind(toTy));
  val = builder.createConvert(loc, toCharLen1Ty, val);
  fir::StoreOp::create(
      builder, loc, val,
      builder.createConvert(loc, builder.getRefType(toCharLen1Ty), addr));
}

/// Returns the minimum of integer mlir::Value \p a and \b.
mlir::Value genMin(fir::FirOpBuilder &builder, mlir::Location loc,
                   mlir::Value a, mlir::Value b) {
  auto cmp = mlir::arith::CmpIOp::create(builder, loc,
                                         mlir::arith::CmpIPredicate::slt, a, b);
  return mlir::arith::SelectOp::create(builder, loc, cmp, a, b);
}
````
- **L409 EN**: Initializes variable `toTy` from the right-hand expression.
  **L409 CN**: 使用右侧表达式初始化变量 `toTy`。
- **L410 EN**: Initializes variable `val` from the right-hand expression.
  **L410 CN**: 使用右侧表达式初始化变量 `val`。
- **L411 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L411 CN**: 开始 `if` 控制流语句并计算其条件。
- **L412 EN**: Continues logic associated with callable symbol `getRefType`.
  **L412 CN**: 继续与可调用符号 `getRefType` 相关的逻辑。
- **L413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder.getContext(),`.
  **L413 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder.getContext(),`。
- **L414 EN**: Executes a call or declaration centered on `getCharacterKind`.
  **L414 CN**: 执行以 `getCharacterKind` 为核心的调用或声明。
- **L415 EN**: Continues logic associated with callable symbol `create`.
  **L415 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L416 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L416 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Continues the surrounding expression or declaration: `auto toCharLen1Ty =`.
  **L418 CN**: 继续构造周围的表达式或声明：`auto toCharLen1Ty =`。
- **L419 EN**: Executes a call or declaration centered on `getSingletonCharType`.
  **L419 CN**: 执行以 `getSingletonCharType` 为核心的调用或声明。
- **L420 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L420 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L421 EN**: Continues logic associated with callable symbol `create`.
  **L421 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, val,`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, val,`。
- **L423 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L423 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L424 EN**: Closes the current lexical scope or compound statement.
  **L424 CN**: 结束当前词法作用域或复合语句块。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L426 EN**: Comment explains nearby logic, intent, or metadata: `Returns the minimum of integer mlir::Value \p a and \b.`.
  **L426 CN**: 注释说明附近代码的逻辑、意图或元数据：`Returns the minimum of integer mlir::Value \p a and \b.`。
- **L427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value genMin(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L427 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value genMin(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L428 EN**: Continues the surrounding expression or declaration: `mlir::Value a, mlir::Value b) {`.
  **L428 CN**: 继续构造周围的表达式或声明：`mlir::Value a, mlir::Value b) {`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto cmp = mlir::arith::CmpIOp::create(builder, loc,`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto cmp = mlir::arith::CmpIOp::create(builder, loc,`。
- **L430 EN**: Executes a standalone statement or declaration: `mlir::arith::CmpIPredicate::slt, a, b);`.
  **L430 CN**: 执行一条独立语句或声明：`mlir::arith::CmpIPredicate::slt, a, b);`。
- **L431 EN**: Returns from the current function with `mlir::arith::SelectOp::create(builder, loc, cmp, a, b)`.
  **L431 CN**: 以 `mlir::arith::SelectOp::create(builder, loc, cmp, a, b)` 从当前函数返回。
- **L432 EN**: Closes the current lexical scope or compound statement.
  **L432 CN**: 结束当前词法作用域或复合语句块。

### Lines 433-456

````cpp

void fir::factory::CharacterExprHelper::createAssign(
    const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {
  auto rhsCstLen = getCompileTimeLength(rhs);
  auto lhsCstLen = getCompileTimeLength(lhs);
  bool compileTimeSameLength = false;
  bool isLengthOneAssign = false;

  if (lhsCstLen && rhsCstLen && *lhsCstLen == *rhsCstLen) {
    compileTimeSameLength = true;
    if (*lhsCstLen == 1)
      isLengthOneAssign = true;
  } else if (rhs.getLen() == lhs.getLen()) {
    compileTimeSameLength = true;

    // If the length values are the same for LHS and RHS,
    // then we can rely on the constant length deduced from
    // any of the two types.
    if (lhsCstLen && *lhsCstLen == 1)
      isLengthOneAssign = true;
    if (rhsCstLen && *rhsCstLen == 1)
      isLengthOneAssign = true;

    // We could have recognized constant operations here (e.g.
````
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L434 EN**: Continues logic associated with callable symbol `createAssign`.
  **L434 CN**: 继续与可调用符号 `createAssign` 相关的逻辑。
- **L435 EN**: Continues the surrounding expression or declaration: `const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {`.
  **L435 CN**: 继续构造周围的表达式或声明：`const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {`。
- **L436 EN**: Initializes variable `rhsCstLen` from the right-hand expression.
  **L436 CN**: 使用右侧表达式初始化变量 `rhsCstLen`。
- **L437 EN**: Initializes variable `lhsCstLen` from the right-hand expression.
  **L437 CN**: 使用右侧表达式初始化变量 `lhsCstLen`。
- **L438 EN**: Initializes variable `compileTimeSameLength` from the right-hand expression.
  **L438 CN**: 使用右侧表达式初始化变量 `compileTimeSameLength`。
- **L439 EN**: Initializes variable `isLengthOneAssign` from the right-hand expression.
  **L439 CN**: 使用右侧表达式初始化变量 `isLengthOneAssign`。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L441 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L441 CN**: 开始 `if` 控制流语句并计算其条件。
- **L442 EN**: Executes a standalone statement or declaration: `compileTimeSameLength = true;`.
  **L442 CN**: 执行一条独立语句或声明：`compileTimeSameLength = true;`。
- **L443 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L443 CN**: 开始 `if` 控制流语句并计算其条件。
- **L444 EN**: Executes a standalone statement or declaration: `isLengthOneAssign = true;`.
  **L444 CN**: 执行一条独立语句或声明：`isLengthOneAssign = true;`。
- **L445 EN**: Transitions from the previous branch into an `else if` condition.
  **L445 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L446 EN**: Executes a standalone statement or declaration: `compileTimeSameLength = true;`.
  **L446 CN**: 执行一条独立语句或声明：`compileTimeSameLength = true;`。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L448 EN**: Comment explains nearby logic, intent, or metadata: `If the length values are the same for LHS and RHS,`.
  **L448 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the length values are the same for LHS and RHS,`。
- **L449 EN**: Comment explains nearby logic, intent, or metadata: `then we can rely on the constant length deduced from`.
  **L449 CN**: 注释说明附近代码的逻辑、意图或元数据：`then we can rely on the constant length deduced from`。
- **L450 EN**: Comment explains nearby logic, intent, or metadata: `any of the two types.`.
  **L450 CN**: 注释说明附近代码的逻辑、意图或元数据：`any of the two types.`。
- **L451 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L451 CN**: 开始 `if` 控制流语句并计算其条件。
- **L452 EN**: Executes a standalone statement or declaration: `isLengthOneAssign = true;`.
  **L452 CN**: 执行一条独立语句或声明：`isLengthOneAssign = true;`。
- **L453 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L453 CN**: 开始 `if` 控制流语句并计算其条件。
- **L454 EN**: Executes a standalone statement or declaration: `isLengthOneAssign = true;`.
  **L454 CN**: 执行一条独立语句或声明：`isLengthOneAssign = true;`。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L456 EN**: Comment explains nearby logic, intent, or metadata: `We could have recognized constant operations here (e.g.`.
  **L456 CN**: 注释说明附近代码的逻辑、意图或元数据：`We could have recognized constant operations here (e.g.`。

### Lines 457-480

````cpp
    // two different arith.constant ops may produce the same value),
    // but for now leave it to CSE to get rid of the duplicates.
  }
  if (isLengthOneAssign) {
    createLengthOneAssign(lhs, rhs);
    return;
  }

  // Copy the minimum of the lhs and rhs lengths and pad the lhs remainder
  // if needed.
  auto copyCount = lhs.getLen();
  auto idxTy = builder.getIndexType();
  if (!compileTimeSameLength) {
    auto lhsLen = builder.createConvert(loc, idxTy, lhs.getLen());
    auto rhsLen = builder.createConvert(loc, idxTy, rhs.getLen());
    copyCount = genMin(builder, loc, lhsLen, rhsLen);
  }

  // Actual copy
  createCopy(lhs, rhs, copyCount);

  // Pad if needed.
  if (!compileTimeSameLength) {
    auto one = builder.createIntegerConstant(loc, lhs.getLen().getType(), 1);
````
- **L457 EN**: Comment explains nearby logic, intent, or metadata: `two different arith.constant ops may produce the same value),`.
  **L457 CN**: 注释说明附近代码的逻辑、意图或元数据：`two different arith.constant ops may produce the same value),`。
- **L458 EN**: Comment explains nearby logic, intent, or metadata: `but for now leave it to CSE to get rid of the duplicates.`.
  **L458 CN**: 注释说明附近代码的逻辑、意图或元数据：`but for now leave it to CSE to get rid of the duplicates.`。
- **L459 EN**: Closes the current lexical scope or compound statement.
  **L459 CN**: 结束当前词法作用域或复合语句块。
- **L460 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L460 CN**: 开始 `if` 控制流语句并计算其条件。
- **L461 EN**: Executes a call or declaration centered on `createLengthOneAssign`.
  **L461 CN**: 执行以 `createLengthOneAssign` 为核心的调用或声明。
- **L462 EN**: Returns from the current function with `void`.
  **L462 CN**: 以 `void` 从当前函数返回。
- **L463 EN**: Closes the current lexical scope or compound statement.
  **L463 CN**: 结束当前词法作用域或复合语句块。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L465 EN**: Comment explains nearby logic, intent, or metadata: `Copy the minimum of the lhs and rhs lengths and pad the lhs remainder`.
  **L465 CN**: 注释说明附近代码的逻辑、意图或元数据：`Copy the minimum of the lhs and rhs lengths and pad the lhs remainder`。
- **L466 EN**: Comment explains nearby logic, intent, or metadata: `if needed.`.
  **L466 CN**: 注释说明附近代码的逻辑、意图或元数据：`if needed.`。
- **L467 EN**: Initializes variable `copyCount` from the right-hand expression.
  **L467 CN**: 使用右侧表达式初始化变量 `copyCount`。
- **L468 EN**: Initializes variable `idxTy` from the right-hand expression.
  **L468 CN**: 使用右侧表达式初始化变量 `idxTy`。
- **L469 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L469 CN**: 开始 `if` 控制流语句并计算其条件。
- **L470 EN**: Initializes variable `lhsLen` from the right-hand expression.
  **L470 CN**: 使用右侧表达式初始化变量 `lhsLen`。
- **L471 EN**: Initializes variable `rhsLen` from the right-hand expression.
  **L471 CN**: 使用右侧表达式初始化变量 `rhsLen`。
- **L472 EN**: Executes a call or declaration centered on `genMin`.
  **L472 CN**: 执行以 `genMin` 为核心的调用或声明。
- **L473 EN**: Closes the current lexical scope or compound statement.
  **L473 CN**: 结束当前词法作用域或复合语句块。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L475 EN**: Comment explains nearby logic, intent, or metadata: `Actual copy`.
  **L475 CN**: 注释说明附近代码的逻辑、意图或元数据：`Actual copy`。
- **L476 EN**: Executes a call or declaration centered on `createCopy`.
  **L476 CN**: 执行以 `createCopy` 为核心的调用或声明。
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L478 EN**: Comment explains nearby logic, intent, or metadata: `Pad if needed.`.
  **L478 CN**: 注释说明附近代码的逻辑、意图或元数据：`Pad if needed.`。
- **L479 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L479 CN**: 开始 `if` 控制流语句并计算其条件。
- **L480 EN**: Initializes variable `one` from the right-hand expression.
  **L480 CN**: 使用右侧表达式初始化变量 `one`。

### Lines 481-504

````cpp
    auto maxPadding =
        mlir::arith::SubIOp::create(builder, loc, lhs.getLen(), one);
    createPadding(lhs, copyCount, maxPadding);
  }
}

fir::CharBoxValue fir::factory::CharacterExprHelper::createConcatenate(
    const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {
  auto lhsLen = builder.createConvert(loc, builder.getCharacterLengthType(),
                                      lhs.getLen());
  auto rhsLen = builder.createConvert(loc, builder.getCharacterLengthType(),
                                      rhs.getLen());
  mlir::Value len = mlir::arith::AddIOp::create(builder, loc, lhsLen, rhsLen);
  auto temp = createCharacterTemp(getCharacterType(rhs), len);
  createCopy(temp, lhs, lhsLen);
  auto one = builder.createIntegerConstant(loc, len.getType(), 1);
  auto upperBound = mlir::arith::SubIOp::create(builder, loc, len, one);
  auto lhsLenIdx = builder.createConvert(loc, builder.getIndexType(), lhsLen);
  auto fromBuff = getCharBoxBuffer(rhs);
  auto toBuff = getCharBoxBuffer(temp);
  fir::factory::DoLoopHelper{builder, loc}.createLoop(
      lhsLenIdx, upperBound, one,
      [&](fir::FirOpBuilder &bldr, mlir::Value index) {
        auto rhsIndex =
````
- **L481 EN**: Continues the surrounding expression or declaration: `auto maxPadding =`.
  **L481 CN**: 继续构造周围的表达式或声明：`auto maxPadding =`。
- **L482 EN**: Executes a call or declaration centered on `mlir::arith::SubIOp::create`.
  **L482 CN**: 执行以 `mlir::arith::SubIOp::create` 为核心的调用或声明。
- **L483 EN**: Executes a call or declaration centered on `createPadding`.
  **L483 CN**: 执行以 `createPadding` 为核心的调用或声明。
- **L484 EN**: Closes the current lexical scope or compound statement.
  **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Closes the current lexical scope or compound statement.
  **L485 CN**: 结束当前词法作用域或复合语句块。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L487 EN**: Continues logic associated with callable symbol `createConcatenate`.
  **L487 CN**: 继续与可调用符号 `createConcatenate` 相关的逻辑。
- **L488 EN**: Continues the surrounding expression or declaration: `const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {`.
  **L488 CN**: 继续构造周围的表达式或声明：`const fir::CharBoxValue &lhs, const fir::CharBoxValue &rhs) {`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto lhsLen = builder.createConvert(loc, builder.getCharacterLengthType(),`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto lhsLen = builder.createConvert(loc, builder.getCharacterLengthType(),`。
- **L490 EN**: Executes a call or declaration centered on `lhs.getLen`.
  **L490 CN**: 执行以 `lhs.getLen` 为核心的调用或声明。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto rhsLen = builder.createConvert(loc, builder.getCharacterLengthType(),`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto rhsLen = builder.createConvert(loc, builder.getCharacterLengthType(),`。
- **L492 EN**: Executes a call or declaration centered on `rhs.getLen`.
  **L492 CN**: 执行以 `rhs.getLen` 为核心的调用或声明。
- **L493 EN**: Initializes variable `len` from the right-hand expression.
  **L493 CN**: 使用右侧表达式初始化变量 `len`。
- **L494 EN**: Initializes variable `temp` from the right-hand expression.
  **L494 CN**: 使用右侧表达式初始化变量 `temp`。
- **L495 EN**: Executes a call or declaration centered on `createCopy`.
  **L495 CN**: 执行以 `createCopy` 为核心的调用或声明。
- **L496 EN**: Initializes variable `one` from the right-hand expression.
  **L496 CN**: 使用右侧表达式初始化变量 `one`。
- **L497 EN**: Initializes variable `upperBound` from the right-hand expression.
  **L497 CN**: 使用右侧表达式初始化变量 `upperBound`。
- **L498 EN**: Initializes variable `lhsLenIdx` from the right-hand expression.
  **L498 CN**: 使用右侧表达式初始化变量 `lhsLenIdx`。
- **L499 EN**: Initializes variable `fromBuff` from the right-hand expression.
  **L499 CN**: 使用右侧表达式初始化变量 `fromBuff`。
- **L500 EN**: Initializes variable `toBuff` from the right-hand expression.
  **L500 CN**: 使用右侧表达式初始化变量 `toBuff`。
- **L501 EN**: Continues logic associated with callable symbol `createLoop`.
  **L501 CN**: 继续与可调用符号 `createLoop` 相关的逻辑。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lhsLenIdx, upperBound, one,`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`lhsLenIdx, upperBound, one,`。
- **L503 EN**: Starts a function, method, lambda, or structured scope: `[&](fir::FirOpBuilder &bldr, mlir::Value index) {`.
  **L503 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](fir::FirOpBuilder &bldr, mlir::Value index) {`。
- **L504 EN**: Continues the surrounding expression or declaration: `auto rhsIndex =`.
  **L504 CN**: 继续构造周围的表达式或声明：`auto rhsIndex =`。

### Lines 505-528

````cpp
            mlir::arith::SubIOp::create(bldr, loc, index, lhsLenIdx);
        auto charVal = createLoadCharAt(fromBuff, rhsIndex);
        createStoreCharAt(toBuff, index, charVal);
      });
  return temp;
}

mlir::Value fir::factory::CharacterExprHelper::genSubstringBase(
    mlir::Value stringRawAddr, mlir::Value lowerBound,
    mlir::Type substringAddrType, mlir::Value one) {
  if (!one)
    one = builder.createIntegerConstant(loc, lowerBound.getType(), 1);
  auto offset =
      mlir::arith::SubIOp::create(builder, loc, lowerBound, one).getResult();
  auto addr = createElementAddr(stringRawAddr, offset);
  return builder.createConvert(loc, substringAddrType, addr);
}

fir::CharBoxValue fir::factory::CharacterExprHelper::createSubstring(
    const fir::CharBoxValue &box, llvm::ArrayRef<mlir::Value> bounds) {
  // Constant need to be materialize in memory to use fir.coordinate_of.
  auto nbounds = bounds.size();
  if (nbounds < 1 || nbounds > 2) {
    mlir::emitError(loc, "Incorrect number of bounds in substring");
````
- **L505 EN**: Executes a call or declaration centered on `mlir::arith::SubIOp::create`.
  **L505 CN**: 执行以 `mlir::arith::SubIOp::create` 为核心的调用或声明。
- **L506 EN**: Initializes variable `charVal` from the right-hand expression.
  **L506 CN**: 使用右侧表达式初始化变量 `charVal`。
- **L507 EN**: Executes a call or declaration centered on `createStoreCharAt`.
  **L507 CN**: 执行以 `createStoreCharAt` 为核心的调用或声明。
- **L508 EN**: Executes a standalone statement or declaration: `});`.
  **L508 CN**: 执行一条独立语句或声明：`});`。
- **L509 EN**: Returns from the current function with `temp`.
  **L509 CN**: 以 `temp` 从当前函数返回。
- **L510 EN**: Closes the current lexical scope or compound statement.
  **L510 CN**: 结束当前词法作用域或复合语句块。
- **L511 EN**: Blank line separating nearby declarations or logic blocks.
  **L511 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L512 EN**: Continues logic associated with callable symbol `genSubstringBase`.
  **L512 CN**: 继续与可调用符号 `genSubstringBase` 相关的逻辑。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringRawAddr, mlir::Value lowerBound,`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringRawAddr, mlir::Value lowerBound,`。
- **L514 EN**: Continues the surrounding expression or declaration: `mlir::Type substringAddrType, mlir::Value one) {`.
  **L514 CN**: 继续构造周围的表达式或声明：`mlir::Type substringAddrType, mlir::Value one) {`。
- **L515 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L515 CN**: 开始 `if` 控制流语句并计算其条件。
- **L516 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L516 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L517 EN**: Continues the surrounding expression or declaration: `auto offset =`.
  **L517 CN**: 继续构造周围的表达式或声明：`auto offset =`。
- **L518 EN**: Executes a call or declaration centered on `mlir::arith::SubIOp::create`.
  **L518 CN**: 执行以 `mlir::arith::SubIOp::create` 为核心的调用或声明。
- **L519 EN**: Initializes variable `addr` from the right-hand expression.
  **L519 CN**: 使用右侧表达式初始化变量 `addr`。
- **L520 EN**: Returns from the current function with `builder.createConvert(loc, substringAddrType, addr)`.
  **L520 CN**: 以 `builder.createConvert(loc, substringAddrType, addr)` 从当前函数返回。
- **L521 EN**: Closes the current lexical scope or compound statement.
  **L521 CN**: 结束当前词法作用域或复合语句块。
- **L522 EN**: Blank line separating nearby declarations or logic blocks.
  **L522 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L523 EN**: Continues logic associated with callable symbol `createSubstring`.
  **L523 CN**: 继续与可调用符号 `createSubstring` 相关的逻辑。
- **L524 EN**: Continues the surrounding expression or declaration: `const fir::CharBoxValue &box, llvm::ArrayRef<mlir::Value> bounds) {`.
  **L524 CN**: 继续构造周围的表达式或声明：`const fir::CharBoxValue &box, llvm::ArrayRef<mlir::Value> bounds) {`。
- **L525 EN**: Comment explains nearby logic, intent, or metadata: `Constant need to be materialize in memory to use fir.coordinate_of.`.
  **L525 CN**: 注释说明附近代码的逻辑、意图或元数据：`Constant need to be materialize in memory to use fir.coordinate_of.`。
- **L526 EN**: Initializes variable `nbounds` from the right-hand expression.
  **L526 CN**: 使用右侧表达式初始化变量 `nbounds`。
- **L527 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L527 CN**: 开始 `if` 控制流语句并计算其条件。
- **L528 EN**: Executes a call or declaration centered on `mlir::emitError`.
  **L528 CN**: 执行以 `mlir::emitError` 为核心的调用或声明。

### Lines 529-552

````cpp
    return {mlir::Value{}, mlir::Value{}};
  }
  mlir::SmallVector<mlir::Value> castBounds;
  // Convert bounds to length type to do safe arithmetic on it.
  for (auto bound : bounds)
    castBounds.push_back(
        builder.createConvert(loc, builder.getCharacterLengthType(), bound));
  auto lowerBound = castBounds[0];
  // FIR CoordinateOp is zero based but Fortran substring are one based.
  auto kind = getCharacterKind(box.getBuffer().getType());
  auto charTy = fir::CharacterType::getUnknownLen(builder.getContext(), kind);
  auto resultType = builder.getRefType(charTy);
  auto one = builder.createIntegerConstant(loc, lowerBound.getType(), 1);
  auto substringRef =
      genSubstringBase(box.getBuffer(), lowerBound, resultType, one);

  // Compute the length.
  mlir::Value substringLen;
  if (nbounds < 2) {
    substringLen =
        mlir::arith::SubIOp::create(builder, loc, box.getLen(), castBounds[0]);
  } else {
    substringLen =
        mlir::arith::SubIOp::create(builder, loc, castBounds[1], castBounds[0]);
````
- **L529 EN**: Returns from the current function with `{mlir::Value{}, mlir::Value{}}`.
  **L529 CN**: 以 `{mlir::Value{}, mlir::Value{}}` 从当前函数返回。
- **L530 EN**: Closes the current lexical scope or compound statement.
  **L530 CN**: 结束当前词法作用域或复合语句块。
- **L531 EN**: Executes a standalone statement or declaration: `mlir::SmallVector<mlir::Value> castBounds;`.
  **L531 CN**: 执行一条独立语句或声明：`mlir::SmallVector<mlir::Value> castBounds;`。
- **L532 EN**: Comment explains nearby logic, intent, or metadata: `Convert bounds to length type to do safe arithmetic on it.`.
  **L532 CN**: 注释说明附近代码的逻辑、意图或元数据：`Convert bounds to length type to do safe arithmetic on it.`。
- **L533 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L533 CN**: 开始 `for` 控制流语句并计算其条件。
- **L534 EN**: Continues logic associated with callable symbol `push_back`.
  **L534 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L535 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L535 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L536 EN**: Initializes variable `lowerBound` from the right-hand expression.
  **L536 CN**: 使用右侧表达式初始化变量 `lowerBound`。
- **L537 EN**: Comment explains nearby logic, intent, or metadata: `FIR CoordinateOp is zero based but Fortran substring are one based.`.
  **L537 CN**: 注释说明附近代码的逻辑、意图或元数据：`FIR CoordinateOp is zero based but Fortran substring are one based.`。
- **L538 EN**: Initializes variable `kind` from the right-hand expression.
  **L538 CN**: 使用右侧表达式初始化变量 `kind`。
- **L539 EN**: Initializes variable `charTy` from the right-hand expression.
  **L539 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L540 EN**: Initializes variable `resultType` from the right-hand expression.
  **L540 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L541 EN**: Initializes variable `one` from the right-hand expression.
  **L541 CN**: 使用右侧表达式初始化变量 `one`。
- **L542 EN**: Continues the surrounding expression or declaration: `auto substringRef =`.
  **L542 CN**: 继续构造周围的表达式或声明：`auto substringRef =`。
- **L543 EN**: Executes a call or declaration centered on `genSubstringBase`.
  **L543 CN**: 执行以 `genSubstringBase` 为核心的调用或声明。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L545 EN**: Comment explains nearby logic, intent, or metadata: `Compute the length.`.
  **L545 CN**: 注释说明附近代码的逻辑、意图或元数据：`Compute the length.`。
- **L546 EN**: Executes a standalone statement or declaration: `mlir::Value substringLen;`.
  **L546 CN**: 执行一条独立语句或声明：`mlir::Value substringLen;`。
- **L547 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L547 CN**: 开始 `if` 控制流语句并计算其条件。
- **L548 EN**: Continues the surrounding expression or declaration: `substringLen =`.
  **L548 CN**: 继续构造周围的表达式或声明：`substringLen =`。
- **L549 EN**: Executes a call or declaration centered on `mlir::arith::SubIOp::create`.
  **L549 CN**: 执行以 `mlir::arith::SubIOp::create` 为核心的调用或声明。
- **L550 EN**: Transitions from the previous branch into the alternative path.
  **L550 CN**: 从前一个分支过渡到备选路径。
- **L551 EN**: Continues the surrounding expression or declaration: `substringLen =`.
  **L551 CN**: 继续构造周围的表达式或声明：`substringLen =`。
- **L552 EN**: Executes a call or declaration centered on `mlir::arith::SubIOp::create`.
  **L552 CN**: 执行以 `mlir::arith::SubIOp::create` 为核心的调用或声明。

### Lines 553-576

````cpp
  }
  substringLen = mlir::arith::AddIOp::create(builder, loc, substringLen, one);

  // Set length to zero if bounds were reversed (Fortran 2018 9.4.1)
  auto zero = builder.createIntegerConstant(loc, substringLen.getType(), 0);
  auto cdt = mlir::arith::CmpIOp::create(
      builder, loc, mlir::arith::CmpIPredicate::slt, substringLen, zero);
  substringLen =
      mlir::arith::SelectOp::create(builder, loc, cdt, zero, substringLen);

  return {substringRef, substringLen};
}

mlir::Value
fir::factory::CharacterExprHelper::createLenTrim(const fir::CharBoxValue &str) {
  // Note: Runtime for LEN_TRIM should also be available at some
  // point. For now use an inlined implementation.
  auto indexType = builder.getIndexType();
  auto len = builder.createConvert(loc, indexType, str.getLen());
  auto one = builder.createIntegerConstant(loc, indexType, 1);
  auto minusOne = builder.createIntegerConstant(loc, indexType, -1);
  auto zero = builder.createIntegerConstant(loc, indexType, 0);
  auto trueVal = builder.createIntegerConstant(loc, builder.getI1Type(), 1);
  auto blank = createBlankConstantCode(getCharacterType(str));
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Executes a call or declaration centered on `mlir::arith::AddIOp::create`.
  **L554 CN**: 执行以 `mlir::arith::AddIOp::create` 为核心的调用或声明。
- **L555 EN**: Blank line separating nearby declarations or logic blocks.
  **L555 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L556 EN**: Comment explains nearby logic, intent, or metadata: `Set length to zero if bounds were reversed (Fortran 2018 9.4.1)`.
  **L556 CN**: 注释说明附近代码的逻辑、意图或元数据：`Set length to zero if bounds were reversed (Fortran 2018 9.4.1)`。
- **L557 EN**: Initializes variable `zero` from the right-hand expression.
  **L557 CN**: 使用右侧表达式初始化变量 `zero`。
- **L558 EN**: Continues logic associated with callable symbol `create`.
  **L558 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L559 EN**: Executes a standalone statement or declaration: `builder, loc, mlir::arith::CmpIPredicate::slt, substringLen, zero);`.
  **L559 CN**: 执行一条独立语句或声明：`builder, loc, mlir::arith::CmpIPredicate::slt, substringLen, zero);`。
- **L560 EN**: Continues the surrounding expression or declaration: `substringLen =`.
  **L560 CN**: 继续构造周围的表达式或声明：`substringLen =`。
- **L561 EN**: Executes a call or declaration centered on `mlir::arith::SelectOp::create`.
  **L561 CN**: 执行以 `mlir::arith::SelectOp::create` 为核心的调用或声明。
- **L562 EN**: Blank line separating nearby declarations or logic blocks.
  **L562 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L563 EN**: Returns from the current function with `{substringRef, substringLen}`.
  **L563 CN**: 以 `{substringRef, substringLen}` 从当前函数返回。
- **L564 EN**: Closes the current lexical scope or compound statement.
  **L564 CN**: 结束当前词法作用域或复合语句块。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L566 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L566 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L567 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::createLenTrim(const fir::CharBoxValue &str) {`.
  **L567 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::createLenTrim(const fir::CharBoxValue &str) {`。
- **L568 EN**: Comment explains nearby logic, intent, or metadata: `Note: Runtime for LEN_TRIM should also be available at some`.
  **L568 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note: Runtime for LEN_TRIM should also be available at some`。
- **L569 EN**: Comment explains nearby logic, intent, or metadata: `point. For now use an inlined implementation.`.
  **L569 CN**: 注释说明附近代码的逻辑、意图或元数据：`point. For now use an inlined implementation.`。
- **L570 EN**: Initializes variable `indexType` from the right-hand expression.
  **L570 CN**: 使用右侧表达式初始化变量 `indexType`。
- **L571 EN**: Initializes variable `len` from the right-hand expression.
  **L571 CN**: 使用右侧表达式初始化变量 `len`。
- **L572 EN**: Initializes variable `one` from the right-hand expression.
  **L572 CN**: 使用右侧表达式初始化变量 `one`。
- **L573 EN**: Initializes variable `minusOne` from the right-hand expression.
  **L573 CN**: 使用右侧表达式初始化变量 `minusOne`。
- **L574 EN**: Initializes variable `zero` from the right-hand expression.
  **L574 CN**: 使用右侧表达式初始化变量 `zero`。
- **L575 EN**: Initializes variable `trueVal` from the right-hand expression.
  **L575 CN**: 使用右侧表达式初始化变量 `trueVal`。
- **L576 EN**: Initializes variable `blank` from the right-hand expression.
  **L576 CN**: 使用右侧表达式初始化变量 `blank`。

### Lines 577-600

````cpp
  mlir::Value lastChar = mlir::arith::SubIOp::create(builder, loc, len, one);

  auto iterWhile =
      fir::IterWhileOp::create(builder, loc, lastChar, zero, minusOne, trueVal,
                               /*returnFinalCount=*/false, lastChar);
  auto insPt = builder.saveInsertionPoint();
  builder.setInsertionPointToStart(iterWhile.getBody());
  auto index = iterWhile.getInductionVar();
  // Look for first non-blank from the right of the character.
  auto fromBuff = getCharBoxBuffer(str);
  auto elemAddr = createElementAddr(fromBuff, index);
  auto codeAddr =
      builder.createConvert(loc, builder.getRefType(blank.getType()), elemAddr);
  auto c = fir::LoadOp::create(builder, loc, codeAddr);
  auto isBlank = mlir::arith::CmpIOp::create(
      builder, loc, mlir::arith::CmpIPredicate::eq, blank, c);
  llvm::SmallVector<mlir::Value> results = {isBlank, index};
  fir::ResultOp::create(builder, loc, results);
  builder.restoreInsertionPoint(insPt);
  // Compute length after iteration (zero if all blanks)
  mlir::Value newLen =
      mlir::arith::AddIOp::create(builder, loc, iterWhile.getResult(1), one);
  auto result = mlir::arith::SelectOp::create(
      builder, loc, iterWhile.getResult(0), zero, newLen);
````
- **L577 EN**: Initializes variable `lastChar` from the right-hand expression.
  **L577 CN**: 使用右侧表达式初始化变量 `lastChar`。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L579 EN**: Continues the surrounding expression or declaration: `auto iterWhile =`.
  **L579 CN**: 继续构造周围的表达式或声明：`auto iterWhile =`。
- **L580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::IterWhileOp::create(builder, loc, lastChar, zero, minusOne, trueVal,`.
  **L580 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::IterWhileOp::create(builder, loc, lastChar, zero, minusOne, trueVal,`。
- **L581 EN**: Comment explains nearby logic, intent, or metadata: `returnFinalCount=*/false, lastChar);`.
  **L581 CN**: 注释说明附近代码的逻辑、意图或元数据：`returnFinalCount=*/false, lastChar);`。
- **L582 EN**: Initializes variable `insPt` from the right-hand expression.
  **L582 CN**: 使用右侧表达式初始化变量 `insPt`。
- **L583 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L583 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L584 EN**: Initializes variable `index` from the right-hand expression.
  **L584 CN**: 使用右侧表达式初始化变量 `index`。
- **L585 EN**: Comment explains nearby logic, intent, or metadata: `Look for first non-blank from the right of the character.`.
  **L585 CN**: 注释说明附近代码的逻辑、意图或元数据：`Look for first non-blank from the right of the character.`。
- **L586 EN**: Initializes variable `fromBuff` from the right-hand expression.
  **L586 CN**: 使用右侧表达式初始化变量 `fromBuff`。
- **L587 EN**: Initializes variable `elemAddr` from the right-hand expression.
  **L587 CN**: 使用右侧表达式初始化变量 `elemAddr`。
- **L588 EN**: Continues the surrounding expression or declaration: `auto codeAddr =`.
  **L588 CN**: 继续构造周围的表达式或声明：`auto codeAddr =`。
- **L589 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L589 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L590 EN**: Initializes variable `c` from the right-hand expression.
  **L590 CN**: 使用右侧表达式初始化变量 `c`。
- **L591 EN**: Continues logic associated with callable symbol `create`.
  **L591 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L592 EN**: Executes a standalone statement or declaration: `builder, loc, mlir::arith::CmpIPredicate::eq, blank, c);`.
  **L592 CN**: 执行一条独立语句或声明：`builder, loc, mlir::arith::CmpIPredicate::eq, blank, c);`。
- **L593 EN**: Initializes variable `results` from the right-hand expression.
  **L593 CN**: 使用右侧表达式初始化变量 `results`。
- **L594 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L594 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L595 EN**: Executes a call or declaration centered on `builder.restoreInsertionPoint`.
  **L595 CN**: 执行以 `builder.restoreInsertionPoint` 为核心的调用或声明。
- **L596 EN**: Comment explains nearby logic, intent, or metadata: `Compute length after iteration (zero if all blanks)`.
  **L596 CN**: 注释说明附近代码的逻辑、意图或元数据：`Compute length after iteration (zero if all blanks)`。
- **L597 EN**: Continues the surrounding expression or declaration: `mlir::Value newLen =`.
  **L597 CN**: 继续构造周围的表达式或声明：`mlir::Value newLen =`。
- **L598 EN**: Executes a call or declaration centered on `mlir::arith::AddIOp::create`.
  **L598 CN**: 执行以 `mlir::arith::AddIOp::create` 为核心的调用或声明。
- **L599 EN**: Continues logic associated with callable symbol `create`.
  **L599 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L600 EN**: Executes a call or declaration centered on `iterWhile.getResult`.
  **L600 CN**: 执行以 `iterWhile.getResult` 为核心的调用或声明。

### Lines 601-624

````cpp
  return builder.createConvert(loc, builder.getCharacterLengthType(), result);
}

fir::CharBoxValue
fir::factory::CharacterExprHelper::createCharacterTemp(mlir::Type type,
                                                       int len) {
  assert(len >= 0 && "expected positive length");
  auto kind = recoverCharacterType(type).getFKind();
  auto charType = fir::CharacterType::get(builder.getContext(), kind, len);
  auto addr = fir::AllocaOp::create(builder, loc, charType);
  auto mlirLen =
      builder.createIntegerConstant(loc, builder.getCharacterLengthType(), len);
  return {addr, mlirLen};
}

// Returns integer with code for blank. The integer has the same
// size as the character. Blank has ascii space code for all kinds.
mlir::Value fir::factory::CharacterExprHelper::createBlankConstantCode(
    fir::CharacterType type) {
  auto bits = builder.getKindMap().getCharacterBitsize(type.getFKind());
  auto intType = builder.getIntegerType(bits);
  return builder.createIntegerConstant(loc, intType, ' ');
}

````
- **L601 EN**: Returns from the current function with `builder.createConvert(loc, builder.getCharacterLengthType(), result)`.
  **L601 CN**: 以 `builder.createConvert(loc, builder.getCharacterLengthType(), result)` 从当前函数返回。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L604 EN**: Continues the surrounding expression or declaration: `fir::CharBoxValue`.
  **L604 CN**: 继续构造周围的表达式或声明：`fir::CharBoxValue`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::CharacterExprHelper::createCharacterTemp(mlir::Type type,`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::CharacterExprHelper::createCharacterTemp(mlir::Type type,`。
- **L606 EN**: Continues the surrounding expression or declaration: `int len) {`.
  **L606 CN**: 继续构造周围的表达式或声明：`int len) {`。
- **L607 EN**: Checks an internal invariant in debug builds.
  **L607 CN**: 在调试构建中检查内部不变式。
- **L608 EN**: Initializes variable `kind` from the right-hand expression.
  **L608 CN**: 使用右侧表达式初始化变量 `kind`。
- **L609 EN**: Initializes variable `charType` from the right-hand expression.
  **L609 CN**: 使用右侧表达式初始化变量 `charType`。
- **L610 EN**: Initializes variable `addr` from the right-hand expression.
  **L610 CN**: 使用右侧表达式初始化变量 `addr`。
- **L611 EN**: Continues the surrounding expression or declaration: `auto mlirLen =`.
  **L611 CN**: 继续构造周围的表达式或声明：`auto mlirLen =`。
- **L612 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L612 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L613 EN**: Returns from the current function with `{addr, mlirLen}`.
  **L613 CN**: 以 `{addr, mlirLen}` 从当前函数返回。
- **L614 EN**: Closes the current lexical scope or compound statement.
  **L614 CN**: 结束当前词法作用域或复合语句块。
- **L615 EN**: Blank line separating nearby declarations or logic blocks.
  **L615 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L616 EN**: Comment explains nearby logic, intent, or metadata: `Returns integer with code for blank. The integer has the same`.
  **L616 CN**: 注释说明附近代码的逻辑、意图或元数据：`Returns integer with code for blank. The integer has the same`。
- **L617 EN**: Comment explains nearby logic, intent, or metadata: `size as the character. Blank has ascii space code for all kinds.`.
  **L617 CN**: 注释说明附近代码的逻辑、意图或元数据：`size as the character. Blank has ascii space code for all kinds.`。
- **L618 EN**: Continues logic associated with callable symbol `createBlankConstantCode`.
  **L618 CN**: 继续与可调用符号 `createBlankConstantCode` 相关的逻辑。
- **L619 EN**: Continues the surrounding expression or declaration: `fir::CharacterType type) {`.
  **L619 CN**: 继续构造周围的表达式或声明：`fir::CharacterType type) {`。
- **L620 EN**: Initializes variable `bits` from the right-hand expression.
  **L620 CN**: 使用右侧表达式初始化变量 `bits`。
- **L621 EN**: Initializes variable `intType` from the right-hand expression.
  **L621 CN**: 使用右侧表达式初始化变量 `intType`。
- **L622 EN**: Returns from the current function with `builder.createIntegerConstant(loc, intType, ' ')`.
  **L622 CN**: 以 `builder.createIntegerConstant(loc, intType, ' ')` 从当前函数返回。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 625-648

````cpp
mlir::Value fir::factory::CharacterExprHelper::createBlankConstant(
    fir::CharacterType type) {
  return createSingletonFromCode(createBlankConstantCode(type),
                                 type.getFKind());
}

void fir::factory::CharacterExprHelper::createAssign(
    const fir::ExtendedValue &lhs, const fir::ExtendedValue &rhs) {
  if (auto *str = rhs.getBoxOf<fir::CharBoxValue>()) {
    if (auto *to = lhs.getBoxOf<fir::CharBoxValue>()) {
      createAssign(*to, *str);
      return;
    }
  }
  TODO(loc, "character array assignment");
  // Note that it is not sure the array aspect should be handled
  // by this utility.
}

mlir::Value
fir::factory::CharacterExprHelper::createEmboxChar(mlir::Value addr,
                                                   mlir::Value len) {
  return createEmbox(fir::CharBoxValue{addr, len});
}
````
- **L625 EN**: Continues logic associated with callable symbol `createBlankConstant`.
  **L625 CN**: 继续与可调用符号 `createBlankConstant` 相关的逻辑。
- **L626 EN**: Continues the surrounding expression or declaration: `fir::CharacterType type) {`.
  **L626 CN**: 继续构造周围的表达式或声明：`fir::CharacterType type) {`。
- **L627 EN**: Returns from the current function with `createSingletonFromCode(createBlankConstantCode(type),`.
  **L627 CN**: 以 `createSingletonFromCode(createBlankConstantCode(type),` 从当前函数返回。
- **L628 EN**: Executes a call or declaration centered on `type.getFKind`.
  **L628 CN**: 执行以 `type.getFKind` 为核心的调用或声明。
- **L629 EN**: Closes the current lexical scope or compound statement.
  **L629 CN**: 结束当前词法作用域或复合语句块。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L631 EN**: Continues logic associated with callable symbol `createAssign`.
  **L631 CN**: 继续与可调用符号 `createAssign` 相关的逻辑。
- **L632 EN**: Continues the surrounding expression or declaration: `const fir::ExtendedValue &lhs, const fir::ExtendedValue &rhs) {`.
  **L632 CN**: 继续构造周围的表达式或声明：`const fir::ExtendedValue &lhs, const fir::ExtendedValue &rhs) {`。
- **L633 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L633 CN**: 开始 `if` 控制流语句并计算其条件。
- **L634 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L634 CN**: 开始 `if` 控制流语句并计算其条件。
- **L635 EN**: Executes a call or declaration centered on `createAssign`.
  **L635 CN**: 执行以 `createAssign` 为核心的调用或声明。
- **L636 EN**: Returns from the current function with `void`.
  **L636 CN**: 以 `void` 从当前函数返回。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Executes a call or declaration centered on `TODO`.
  **L639 CN**: 执行以 `TODO` 为核心的调用或声明。
- **L640 EN**: Comment explains nearby logic, intent, or metadata: `Note that it is not sure the array aspect should be handled`.
  **L640 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that it is not sure the array aspect should be handled`。
- **L641 EN**: Comment explains nearby logic, intent, or metadata: `by this utility.`.
  **L641 CN**: 注释说明附近代码的逻辑、意图或元数据：`by this utility.`。
- **L642 EN**: Closes the current lexical scope or compound statement.
  **L642 CN**: 结束当前词法作用域或复合语句块。
- **L643 EN**: Blank line separating nearby declarations or logic blocks.
  **L643 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L644 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L644 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::CharacterExprHelper::createEmboxChar(mlir::Value addr,`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::CharacterExprHelper::createEmboxChar(mlir::Value addr,`。
- **L646 EN**: Continues the surrounding expression or declaration: `mlir::Value len) {`.
  **L646 CN**: 继续构造周围的表达式或声明：`mlir::Value len) {`。
- **L647 EN**: Returns from the current function with `createEmbox(fir::CharBoxValue{addr, len})`.
  **L647 CN**: 以 `createEmbox(fir::CharBoxValue{addr, len})` 从当前函数返回。
- **L648 EN**: Closes the current lexical scope or compound statement.
  **L648 CN**: 结束当前词法作用域或复合语句块。

### Lines 649-672

````cpp

std::pair<mlir::Value, mlir::Value>
fir::factory::CharacterExprHelper::createUnboxChar(mlir::Value boxChar) {
  using T = std::pair<mlir::Value, mlir::Value>;
  return toExtendedValue(boxChar).match(
      [](const fir::CharBoxValue &b) -> T {
        return {b.getBuffer(), b.getLen()};
      },
      [](const fir::CharArrayBoxValue &b) -> T {
        return {b.getBuffer(), b.getLen()};
      },
      [](const auto &) -> T { llvm::report_fatal_error("not a character"); });
}

bool fir::factory::CharacterExprHelper::isCharacterLiteral(mlir::Type type) {
  if (auto seqType = mlir::dyn_cast<fir::SequenceType>(type))
    return (seqType.getShape().size() == 1) &&
           fir::isa_char(seqType.getEleTy());
  return false;
}

fir::KindTy
fir::factory::CharacterExprHelper::getCharacterKind(mlir::Type type) {
  assert(isCharacterScalar(type) && "expected scalar character");
````
- **L649 EN**: Blank line separating nearby declarations or logic blocks.
  **L649 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L650 EN**: Continues the surrounding expression or declaration: `std::pair<mlir::Value, mlir::Value>`.
  **L650 CN**: 继续构造周围的表达式或声明：`std::pair<mlir::Value, mlir::Value>`。
- **L651 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::createUnboxChar(mlir::Value boxChar) {`.
  **L651 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::createUnboxChar(mlir::Value boxChar) {`。
- **L652 EN**: Defines alias `T` to simplify later code.
  **L652 CN**: 定义别名 `T` 以简化后续代码。
- **L653 EN**: Returns from the current function with `toExtendedValue(boxChar).match(`.
  **L653 CN**: 以 `toExtendedValue(boxChar).match(` 从当前函数返回。
- **L654 EN**: Starts a function, method, lambda, or structured scope: `[](const fir::CharBoxValue &b) -> T {`.
  **L654 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](const fir::CharBoxValue &b) -> T {`。
- **L655 EN**: Returns from the current function with `{b.getBuffer(), b.getLen()}`.
  **L655 CN**: 以 `{b.getBuffer(), b.getLen()}` 从当前函数返回。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L657 EN**: Starts a function, method, lambda, or structured scope: `[](const fir::CharArrayBoxValue &b) -> T {`.
  **L657 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](const fir::CharArrayBoxValue &b) -> T {`。
- **L658 EN**: Returns from the current function with `{b.getBuffer(), b.getLen()}`.
  **L658 CN**: 以 `{b.getBuffer(), b.getLen()}` 从当前函数返回。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L660 EN**: Executes a call or declaration centered on `[]`.
  **L660 CN**: 执行以 `[]` 为核心的调用或声明。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Blank line separating nearby declarations or logic blocks.
  **L662 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L663 EN**: Starts a function, method, lambda, or structured scope: `bool fir::factory::CharacterExprHelper::isCharacterLiteral(mlir::Type type) {`.
  **L663 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool fir::factory::CharacterExprHelper::isCharacterLiteral(mlir::Type type) {`。
- **L664 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L664 CN**: 开始 `if` 控制流语句并计算其条件。
- **L665 EN**: Returns from the current function with `(seqType.getShape().size() == 1) &&`.
  **L665 CN**: 以 `(seqType.getShape().size() == 1) &&` 从当前函数返回。
- **L666 EN**: Executes a call or declaration centered on `fir::isa_char`.
  **L666 CN**: 执行以 `fir::isa_char` 为核心的调用或声明。
- **L667 EN**: Returns from the current function with `false`.
  **L667 CN**: 以 `false` 从当前函数返回。
- **L668 EN**: Closes the current lexical scope or compound statement.
  **L668 CN**: 结束当前词法作用域或复合语句块。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L670 EN**: Continues the surrounding expression or declaration: `fir::KindTy`.
  **L670 CN**: 继续构造周围的表达式或声明：`fir::KindTy`。
- **L671 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::getCharacterKind(mlir::Type type) {`.
  **L671 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::getCharacterKind(mlir::Type type) {`。
- **L672 EN**: Checks an internal invariant in debug builds.
  **L672 CN**: 在调试构建中检查内部不变式。

### Lines 673-696

````cpp
  return recoverCharacterType(type).getFKind();
}

fir::KindTy
fir::factory::CharacterExprHelper::getCharacterOrSequenceKind(mlir::Type type) {
  return recoverCharacterType(type).getFKind();
}

bool fir::factory::CharacterExprHelper::hasConstantLengthInType(
    const fir::ExtendedValue &exv) {
  auto charTy = recoverCharacterType(fir::getBase(exv).getType());
  return charTy.hasConstantLen();
}

mlir::Value
fir::factory::CharacterExprHelper::createSingletonFromCode(mlir::Value code,
                                                           int kind) {
  auto charType = fir::CharacterType::get(builder.getContext(), kind, 1);
  auto bits = builder.getKindMap().getCharacterBitsize(kind);
  auto intType = builder.getIntegerType(bits);
  auto cast = builder.createConvert(loc, intType, code);
  auto undef = fir::UndefOp::create(builder, loc, charType);
  auto zero = builder.getIntegerAttr(builder.getIndexType(), 0);
  return fir::InsertValueOp::create(builder, loc, charType, undef, cast,
````
- **L673 EN**: Returns from the current function with `recoverCharacterType(type).getFKind()`.
  **L673 CN**: 以 `recoverCharacterType(type).getFKind()` 从当前函数返回。
- **L674 EN**: Closes the current lexical scope or compound statement.
  **L674 CN**: 结束当前词法作用域或复合语句块。
- **L675 EN**: Blank line separating nearby declarations or logic blocks.
  **L675 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L676 EN**: Continues the surrounding expression or declaration: `fir::KindTy`.
  **L676 CN**: 继续构造周围的表达式或声明：`fir::KindTy`。
- **L677 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::getCharacterOrSequenceKind(mlir::Type type) {`.
  **L677 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::getCharacterOrSequenceKind(mlir::Type type) {`。
- **L678 EN**: Returns from the current function with `recoverCharacterType(type).getFKind()`.
  **L678 CN**: 以 `recoverCharacterType(type).getFKind()` 从当前函数返回。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L681 EN**: Continues logic associated with callable symbol `hasConstantLengthInType`.
  **L681 CN**: 继续与可调用符号 `hasConstantLengthInType` 相关的逻辑。
- **L682 EN**: Continues the surrounding expression or declaration: `const fir::ExtendedValue &exv) {`.
  **L682 CN**: 继续构造周围的表达式或声明：`const fir::ExtendedValue &exv) {`。
- **L683 EN**: Initializes variable `charTy` from the right-hand expression.
  **L683 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L684 EN**: Returns from the current function with `charTy.hasConstantLen()`.
  **L684 CN**: 以 `charTy.hasConstantLen()` 从当前函数返回。
- **L685 EN**: Closes the current lexical scope or compound statement.
  **L685 CN**: 结束当前词法作用域或复合语句块。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L687 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L687 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::CharacterExprHelper::createSingletonFromCode(mlir::Value code,`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::CharacterExprHelper::createSingletonFromCode(mlir::Value code,`。
- **L689 EN**: Continues the surrounding expression or declaration: `int kind) {`.
  **L689 CN**: 继续构造周围的表达式或声明：`int kind) {`。
- **L690 EN**: Initializes variable `charType` from the right-hand expression.
  **L690 CN**: 使用右侧表达式初始化变量 `charType`。
- **L691 EN**: Initializes variable `bits` from the right-hand expression.
  **L691 CN**: 使用右侧表达式初始化变量 `bits`。
- **L692 EN**: Initializes variable `intType` from the right-hand expression.
  **L692 CN**: 使用右侧表达式初始化变量 `intType`。
- **L693 EN**: Initializes variable `cast` from the right-hand expression.
  **L693 CN**: 使用右侧表达式初始化变量 `cast`。
- **L694 EN**: Initializes variable `undef` from the right-hand expression.
  **L694 CN**: 使用右侧表达式初始化变量 `undef`。
- **L695 EN**: Initializes variable `zero` from the right-hand expression.
  **L695 CN**: 使用右侧表达式初始化变量 `zero`。
- **L696 EN**: Returns from the current function with `fir::InsertValueOp::create(builder, loc, charType, undef, cast,`.
  **L696 CN**: 以 `fir::InsertValueOp::create(builder, loc, charType, undef, cast,` 从当前函数返回。

### Lines 697-720

````cpp
                                    builder.getArrayAttr(zero));
}

mlir::Value fir::factory::CharacterExprHelper::extractCodeFromSingleton(
    mlir::Value singleton) {
  auto type = getCharacterType(singleton);
  assert(type.getLen() == 1);
  auto bits = builder.getKindMap().getCharacterBitsize(type.getFKind());
  auto intType = builder.getIntegerType(bits);
  auto zero = builder.getIntegerAttr(builder.getIndexType(), 0);
  return fir::ExtractValueOp::create(builder, loc, intType, singleton,
                                     builder.getArrayAttr(zero));
}

mlir::Value
fir::factory::CharacterExprHelper::readLengthFromBox(mlir::Value box) {
  auto charTy = recoverCharacterType(box.getType());
  return readLengthFromBox(box, charTy);
}

mlir::Value fir::factory::CharacterExprHelper::readLengthFromBox(
    mlir::Value box, fir::CharacterType charTy) {
  auto lenTy = builder.getCharacterLengthType();
  auto size = fir::BoxEleSizeOp::create(builder, loc, lenTy, box);
````
- **L697 EN**: Executes a call or declaration centered on `builder.getArrayAttr`.
  **L697 CN**: 执行以 `builder.getArrayAttr` 为核心的调用或声明。
- **L698 EN**: Closes the current lexical scope or compound statement.
  **L698 CN**: 结束当前词法作用域或复合语句块。
- **L699 EN**: Blank line separating nearby declarations or logic blocks.
  **L699 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L700 EN**: Continues logic associated with callable symbol `extractCodeFromSingleton`.
  **L700 CN**: 继续与可调用符号 `extractCodeFromSingleton` 相关的逻辑。
- **L701 EN**: Continues the surrounding expression or declaration: `mlir::Value singleton) {`.
  **L701 CN**: 继续构造周围的表达式或声明：`mlir::Value singleton) {`。
- **L702 EN**: Initializes variable `type` from the right-hand expression.
  **L702 CN**: 使用右侧表达式初始化变量 `type`。
- **L703 EN**: Checks an internal invariant in debug builds.
  **L703 CN**: 在调试构建中检查内部不变式。
- **L704 EN**: Initializes variable `bits` from the right-hand expression.
  **L704 CN**: 使用右侧表达式初始化变量 `bits`。
- **L705 EN**: Initializes variable `intType` from the right-hand expression.
  **L705 CN**: 使用右侧表达式初始化变量 `intType`。
- **L706 EN**: Initializes variable `zero` from the right-hand expression.
  **L706 CN**: 使用右侧表达式初始化变量 `zero`。
- **L707 EN**: Returns from the current function with `fir::ExtractValueOp::create(builder, loc, intType, singleton,`.
  **L707 CN**: 以 `fir::ExtractValueOp::create(builder, loc, intType, singleton,` 从当前函数返回。
- **L708 EN**: Executes a call or declaration centered on `builder.getArrayAttr`.
  **L708 CN**: 执行以 `builder.getArrayAttr` 为核心的调用或声明。
- **L709 EN**: Closes the current lexical scope or compound statement.
  **L709 CN**: 结束当前词法作用域或复合语句块。
- **L710 EN**: Blank line separating nearby declarations or logic blocks.
  **L710 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L711 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L711 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L712 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::CharacterExprHelper::readLengthFromBox(mlir::Value box) {`.
  **L712 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::CharacterExprHelper::readLengthFromBox(mlir::Value box) {`。
- **L713 EN**: Initializes variable `charTy` from the right-hand expression.
  **L713 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L714 EN**: Returns from the current function with `readLengthFromBox(box, charTy)`.
  **L714 CN**: 以 `readLengthFromBox(box, charTy)` 从当前函数返回。
- **L715 EN**: Closes the current lexical scope or compound statement.
  **L715 CN**: 结束当前词法作用域或复合语句块。
- **L716 EN**: Blank line separating nearby declarations or logic blocks.
  **L716 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L717 EN**: Continues logic associated with callable symbol `readLengthFromBox`.
  **L717 CN**: 继续与可调用符号 `readLengthFromBox` 相关的逻辑。
- **L718 EN**: Continues the surrounding expression or declaration: `mlir::Value box, fir::CharacterType charTy) {`.
  **L718 CN**: 继续构造周围的表达式或声明：`mlir::Value box, fir::CharacterType charTy) {`。
- **L719 EN**: Initializes variable `lenTy` from the right-hand expression.
  **L719 CN**: 使用右侧表达式初始化变量 `lenTy`。
- **L720 EN**: Initializes variable `size` from the right-hand expression.
  **L720 CN**: 使用右侧表达式初始化变量 `size`。

### Lines 721-744

````cpp
  auto bits = builder.getKindMap().getCharacterBitsize(charTy.getFKind());
  auto width = bits / 8;
  if (width > 1) {
    auto widthVal = builder.createIntegerConstant(loc, lenTy, width);
    return mlir::arith::DivSIOp::create(builder, loc, size, widthVal);
  }
  return size;
}

mlir::Value fir::factory::CharacterExprHelper::getLength(mlir::Value memref) {
  auto memrefType = memref.getType();
  auto charType = recoverCharacterType(memrefType);
  assert(charType && "must be a character type");
  if (charType.hasConstantLen())
    return builder.createIntegerConstant(loc, builder.getCharacterLengthType(),
                                         charType.getLen());
  if (mlir::isa<fir::BoxType>(memrefType))
    return readLengthFromBox(memref);
  if (mlir::isa<fir::BoxCharType>(memrefType))
    return createUnboxChar(memref).second;

  // Length cannot be deduced from memref.
  return {};
}
````
- **L721 EN**: Initializes variable `bits` from the right-hand expression.
  **L721 CN**: 使用右侧表达式初始化变量 `bits`。
- **L722 EN**: Initializes variable `width` from the right-hand expression.
  **L722 CN**: 使用右侧表达式初始化变量 `width`。
- **L723 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L723 CN**: 开始 `if` 控制流语句并计算其条件。
- **L724 EN**: Initializes variable `widthVal` from the right-hand expression.
  **L724 CN**: 使用右侧表达式初始化变量 `widthVal`。
- **L725 EN**: Returns from the current function with `mlir::arith::DivSIOp::create(builder, loc, size, widthVal)`.
  **L725 CN**: 以 `mlir::arith::DivSIOp::create(builder, loc, size, widthVal)` 从当前函数返回。
- **L726 EN**: Closes the current lexical scope or compound statement.
  **L726 CN**: 结束当前词法作用域或复合语句块。
- **L727 EN**: Returns from the current function with `size`.
  **L727 CN**: 以 `size` 从当前函数返回。
- **L728 EN**: Closes the current lexical scope or compound statement.
  **L728 CN**: 结束当前词法作用域或复合语句块。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L730 EN**: Starts a function, method, lambda, or structured scope: `mlir::Value fir::factory::CharacterExprHelper::getLength(mlir::Value memref) {`.
  **L730 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::Value fir::factory::CharacterExprHelper::getLength(mlir::Value memref) {`。
- **L731 EN**: Initializes variable `memrefType` from the right-hand expression.
  **L731 CN**: 使用右侧表达式初始化变量 `memrefType`。
- **L732 EN**: Initializes variable `charType` from the right-hand expression.
  **L732 CN**: 使用右侧表达式初始化变量 `charType`。
- **L733 EN**: Checks an internal invariant in debug builds.
  **L733 CN**: 在调试构建中检查内部不变式。
- **L734 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L734 CN**: 开始 `if` 控制流语句并计算其条件。
- **L735 EN**: Returns from the current function with `builder.createIntegerConstant(loc, builder.getCharacterLengthType(),`.
  **L735 CN**: 以 `builder.createIntegerConstant(loc, builder.getCharacterLengthType(),` 从当前函数返回。
- **L736 EN**: Executes a call or declaration centered on `charType.getLen`.
  **L736 CN**: 执行以 `charType.getLen` 为核心的调用或声明。
- **L737 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L737 CN**: 开始 `if` 控制流语句并计算其条件。
- **L738 EN**: Returns from the current function with `readLengthFromBox(memref)`.
  **L738 CN**: 以 `readLengthFromBox(memref)` 从当前函数返回。
- **L739 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L739 CN**: 开始 `if` 控制流语句并计算其条件。
- **L740 EN**: Returns from the current function with `createUnboxChar(memref).second`.
  **L740 CN**: 以 `createUnboxChar(memref).second` 从当前函数返回。
- **L741 EN**: Blank line separating nearby declarations or logic blocks.
  **L741 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L742 EN**: Comment explains nearby logic, intent, or metadata: `Length cannot be deduced from memref.`.
  **L742 CN**: 注释说明附近代码的逻辑、意图或元数据：`Length cannot be deduced from memref.`。
- **L743 EN**: Returns from the current function with `{}`.
  **L743 CN**: 以 `{}` 从当前函数返回。
- **L744 EN**: Closes the current lexical scope or compound statement.
  **L744 CN**: 结束当前词法作用域或复合语句块。

### Lines 745-768

````cpp

std::pair<mlir::Value, mlir::Value>
fir::factory::extractCharacterProcedureTuple(fir::FirOpBuilder &builder,
                                             mlir::Location loc,
                                             mlir::Value tuple,
                                             bool openBoxProc) {
  mlir::TupleType tupleType = mlir::cast<mlir::TupleType>(tuple.getType());
  mlir::Value addr = fir::ExtractValueOp::create(
      builder, loc, tupleType.getType(0), tuple,
      builder.getArrayAttr(
          {builder.getIntegerAttr(builder.getIndexType(), 0)}));
  mlir::Value proc = [&]() -> mlir::Value {
    if (openBoxProc)
      if (auto addrTy = mlir::dyn_cast<fir::BoxProcType>(addr.getType()))
        return fir::BoxAddrOp::create(builder, loc, addrTy.getEleTy(), addr);
    return addr;
  }();
  mlir::Value len = fir::ExtractValueOp::create(
      builder, loc, tupleType.getType(1), tuple,
      builder.getArrayAttr(
          {builder.getIntegerAttr(builder.getIndexType(), 1)}));
  return {proc, len};
}

````
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L746 EN**: Continues the surrounding expression or declaration: `std::pair<mlir::Value, mlir::Value>`.
  **L746 CN**: 继续构造周围的表达式或声明：`std::pair<mlir::Value, mlir::Value>`。
- **L747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::extractCharacterProcedureTuple(fir::FirOpBuilder &builder,`.
  **L747 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::extractCharacterProcedureTuple(fir::FirOpBuilder &builder,`。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc,`.
  **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc,`。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value tuple,`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value tuple,`。
- **L750 EN**: Continues the surrounding expression or declaration: `bool openBoxProc) {`.
  **L750 CN**: 继续构造周围的表达式或声明：`bool openBoxProc) {`。
- **L751 EN**: Initializes variable `tupleType` from the right-hand expression.
  **L751 CN**: 使用右侧表达式初始化变量 `tupleType`。
- **L752 EN**: Continues logic associated with callable symbol `create`.
  **L752 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, tupleType.getType(0), tuple,`.
  **L753 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, tupleType.getType(0), tuple,`。
- **L754 EN**: Continues logic associated with callable symbol `getArrayAttr`.
  **L754 CN**: 继续与可调用符号 `getArrayAttr` 相关的逻辑。
- **L755 EN**: Executes a call or declaration centered on `{builder.getIntegerAttr`.
  **L755 CN**: 执行以 `{builder.getIntegerAttr` 为核心的调用或声明。
- **L756 EN**: Starts a function, method, lambda, or structured scope: `mlir::Value proc = [&]() -> mlir::Value {`.
  **L756 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::Value proc = [&]() -> mlir::Value {`。
- **L757 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L757 CN**: 开始 `if` 控制流语句并计算其条件。
- **L758 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L758 CN**: 开始 `if` 控制流语句并计算其条件。
- **L759 EN**: Returns from the current function with `fir::BoxAddrOp::create(builder, loc, addrTy.getEleTy(), addr)`.
  **L759 CN**: 以 `fir::BoxAddrOp::create(builder, loc, addrTy.getEleTy(), addr)` 从当前函数返回。
- **L760 EN**: Returns from the current function with `addr`.
  **L760 CN**: 以 `addr` 从当前函数返回。
- **L761 EN**: Executes a call or declaration centered on `}`.
  **L761 CN**: 执行以 `}` 为核心的调用或声明。
- **L762 EN**: Continues logic associated with callable symbol `create`.
  **L762 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, tupleType.getType(1), tuple,`.
  **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, tupleType.getType(1), tuple,`。
- **L764 EN**: Continues logic associated with callable symbol `getArrayAttr`.
  **L764 CN**: 继续与可调用符号 `getArrayAttr` 相关的逻辑。
- **L765 EN**: Executes a call or declaration centered on `{builder.getIntegerAttr`.
  **L765 CN**: 执行以 `{builder.getIntegerAttr` 为核心的调用或声明。
- **L766 EN**: Returns from the current function with `{proc, len}`.
  **L766 CN**: 以 `{proc, len}` 从当前函数返回。
- **L767 EN**: Closes the current lexical scope or compound statement.
  **L767 CN**: 结束当前词法作用域或复合语句块。
- **L768 EN**: Blank line separating nearby declarations or logic blocks.
  **L768 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 769-792

````cpp
mlir::Value fir::factory::createCharacterProcedureTuple(
    fir::FirOpBuilder &builder, mlir::Location loc, mlir::Type argTy,
    mlir::Value addr, mlir::Value len) {
  mlir::TupleType tupleType = mlir::cast<mlir::TupleType>(argTy);
  addr = builder.createConvert(loc, tupleType.getType(0), addr);
  if (len)
    len = builder.createConvert(loc, tupleType.getType(1), len);
  else
    len = fir::UndefOp::create(builder, loc, tupleType.getType(1));
  mlir::Value tuple = fir::UndefOp::create(builder, loc, tupleType);
  tuple = fir::InsertValueOp::create(
      builder, loc, tupleType, tuple, addr,
      builder.getArrayAttr(
          {builder.getIntegerAttr(builder.getIndexType(), 0)}));
  tuple = fir::InsertValueOp::create(
      builder, loc, tupleType, tuple, len,
      builder.getArrayAttr(
          {builder.getIntegerAttr(builder.getIndexType(), 1)}));
  return tuple;
}

mlir::Type
fir::factory::getCharacterProcedureTupleType(mlir::Type funcPointerType) {
  mlir::MLIRContext *context = funcPointerType.getContext();
````
- **L769 EN**: Continues logic associated with callable symbol `createCharacterProcedureTuple`.
  **L769 CN**: 继续与可调用符号 `createCharacterProcedureTuple` 相关的逻辑。
- **L770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::FirOpBuilder &builder, mlir::Location loc, mlir::Type argTy,`.
  **L770 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::FirOpBuilder &builder, mlir::Location loc, mlir::Type argTy,`。
- **L771 EN**: Continues the surrounding expression or declaration: `mlir::Value addr, mlir::Value len) {`.
  **L771 CN**: 继续构造周围的表达式或声明：`mlir::Value addr, mlir::Value len) {`。
- **L772 EN**: Initializes variable `tupleType` from the right-hand expression.
  **L772 CN**: 使用右侧表达式初始化变量 `tupleType`。
- **L773 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L773 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L774 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L774 CN**: 开始 `if` 控制流语句并计算其条件。
- **L775 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L775 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L776 EN**: Transitions from the previous branch into the alternative path.
  **L776 CN**: 从前一个分支过渡到备选路径。
- **L777 EN**: Executes a call or declaration centered on `fir::UndefOp::create`.
  **L777 CN**: 执行以 `fir::UndefOp::create` 为核心的调用或声明。
- **L778 EN**: Initializes variable `tuple` from the right-hand expression.
  **L778 CN**: 使用右侧表达式初始化变量 `tuple`。
- **L779 EN**: Continues logic associated with callable symbol `create`.
  **L779 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, tupleType, tuple, addr,`.
  **L780 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, tupleType, tuple, addr,`。
- **L781 EN**: Continues logic associated with callable symbol `getArrayAttr`.
  **L781 CN**: 继续与可调用符号 `getArrayAttr` 相关的逻辑。
- **L782 EN**: Executes a call or declaration centered on `{builder.getIntegerAttr`.
  **L782 CN**: 执行以 `{builder.getIntegerAttr` 为核心的调用或声明。
- **L783 EN**: Continues logic associated with callable symbol `create`.
  **L783 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, tupleType, tuple, len,`.
  **L784 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, tupleType, tuple, len,`。
- **L785 EN**: Continues logic associated with callable symbol `getArrayAttr`.
  **L785 CN**: 继续与可调用符号 `getArrayAttr` 相关的逻辑。
- **L786 EN**: Executes a call or declaration centered on `{builder.getIntegerAttr`.
  **L786 CN**: 执行以 `{builder.getIntegerAttr` 为核心的调用或声明。
- **L787 EN**: Returns from the current function with `tuple`.
  **L787 CN**: 以 `tuple` 从当前函数返回。
- **L788 EN**: Closes the current lexical scope or compound statement.
  **L788 CN**: 结束当前词法作用域或复合语句块。
- **L789 EN**: Blank line separating nearby declarations or logic blocks.
  **L789 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L790 EN**: Continues the surrounding expression or declaration: `mlir::Type`.
  **L790 CN**: 继续构造周围的表达式或声明：`mlir::Type`。
- **L791 EN**: Starts a function, method, lambda, or structured scope: `fir::factory::getCharacterProcedureTupleType(mlir::Type funcPointerType) {`.
  **L791 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::factory::getCharacterProcedureTupleType(mlir::Type funcPointerType) {`。
- **L792 EN**: Executes a call or declaration centered on `funcPointerType.getContext`.
  **L792 CN**: 执行以 `funcPointerType.getContext` 为核心的调用或声明。

### Lines 793-816

````cpp
  mlir::Type lenType = mlir::IntegerType::get(context, 64);
  return mlir::TupleType::get(context, {funcPointerType, lenType});
}

fir::CharBoxValue fir::factory::CharacterExprHelper::createCharExtremum(
    bool predIsMin, llvm::ArrayRef<fir::CharBoxValue> opCBVs) {
  // inputs: we are given a vector of all of the charboxes of the arguments
  // passed to hlfir.char_extremum, as well as the predicate for whether we
  // want llt or lgt
  //
  // note: we know that, regardless of whether we're looking at smallest or
  // largest char, the size of the output buffer will be the same size as the
  // largest character out of all of the operands. so, we find the biggest
  // length first. It's okay if these char lengths are not known at compile
  // time.

  fir::CharBoxValue firstCBV = opCBVs[0];
  mlir::Value firstBuf = getCharBoxBuffer(firstCBV);
  auto firstLen = builder.createConvert(loc, builder.getCharacterLengthType(),
                                        firstCBV.getLen());

  mlir::Value resultBuf = firstBuf;
  mlir::Value resultLen = firstLen;
  mlir::Value biggestLen = firstLen;
````
- **L793 EN**: Initializes variable `lenType` from the right-hand expression.
  **L793 CN**: 使用右侧表达式初始化变量 `lenType`。
- **L794 EN**: Returns from the current function with `mlir::TupleType::get(context, {funcPointerType, lenType})`.
  **L794 CN**: 以 `mlir::TupleType::get(context, {funcPointerType, lenType})` 从当前函数返回。
- **L795 EN**: Closes the current lexical scope or compound statement.
  **L795 CN**: 结束当前词法作用域或复合语句块。
- **L796 EN**: Blank line separating nearby declarations or logic blocks.
  **L796 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L797 EN**: Continues logic associated with callable symbol `createCharExtremum`.
  **L797 CN**: 继续与可调用符号 `createCharExtremum` 相关的逻辑。
- **L798 EN**: Continues the surrounding expression or declaration: `bool predIsMin, llvm::ArrayRef<fir::CharBoxValue> opCBVs) {`.
  **L798 CN**: 继续构造周围的表达式或声明：`bool predIsMin, llvm::ArrayRef<fir::CharBoxValue> opCBVs) {`。
- **L799 EN**: Comment explains nearby logic, intent, or metadata: `inputs: we are given a vector of all of the charboxes of the arguments`.
  **L799 CN**: 注释说明附近代码的逻辑、意图或元数据：`inputs: we are given a vector of all of the charboxes of the arguments`。
- **L800 EN**: Comment explains nearby logic, intent, or metadata: `passed to hlfir.char_extremum, as well as the predicate for whether we`.
  **L800 CN**: 注释说明附近代码的逻辑、意图或元数据：`passed to hlfir.char_extremum, as well as the predicate for whether we`。
- **L801 EN**: Comment explains nearby logic, intent, or metadata: `want llt or lgt`.
  **L801 CN**: 注释说明附近代码的逻辑、意图或元数据：`want llt or lgt`。
- **L802 EN**: Separator comment used for visual grouping.
  **L802 CN**: 用于视觉分组的分隔注释。
- **L803 EN**: Comment explains nearby logic, intent, or metadata: `note: we know that, regardless of whether we're looking at smallest or`.
  **L803 CN**: 注释说明附近代码的逻辑、意图或元数据：`note: we know that, regardless of whether we're looking at smallest or`。
- **L804 EN**: Comment explains nearby logic, intent, or metadata: `largest char, the size of the output buffer will be the same size as the`.
  **L804 CN**: 注释说明附近代码的逻辑、意图或元数据：`largest char, the size of the output buffer will be the same size as the`。
- **L805 EN**: Comment explains nearby logic, intent, or metadata: `largest character out of all of the operands. so, we find the biggest`.
  **L805 CN**: 注释说明附近代码的逻辑、意图或元数据：`largest character out of all of the operands. so, we find the biggest`。
- **L806 EN**: Comment explains nearby logic, intent, or metadata: `length first. It's okay if these char lengths are not known at compile`.
  **L806 CN**: 注释说明附近代码的逻辑、意图或元数据：`length first. It's okay if these char lengths are not known at compile`。
- **L807 EN**: Comment explains nearby logic, intent, or metadata: `time.`.
  **L807 CN**: 注释说明附近代码的逻辑、意图或元数据：`time.`。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L809 EN**: Initializes variable `firstCBV` from the right-hand expression.
  **L809 CN**: 使用右侧表达式初始化变量 `firstCBV`。
- **L810 EN**: Initializes variable `firstBuf` from the right-hand expression.
  **L810 CN**: 使用右侧表达式初始化变量 `firstBuf`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto firstLen = builder.createConvert(loc, builder.getCharacterLengthType(),`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto firstLen = builder.createConvert(loc, builder.getCharacterLengthType(),`。
- **L812 EN**: Executes a call or declaration centered on `firstCBV.getLen`.
  **L812 CN**: 执行以 `firstCBV.getLen` 为核心的调用或声明。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L814 EN**: Initializes variable `resultBuf` from the right-hand expression.
  **L814 CN**: 使用右侧表达式初始化变量 `resultBuf`。
- **L815 EN**: Initializes variable `resultLen` from the right-hand expression.
  **L815 CN**: 使用右侧表达式初始化变量 `resultLen`。
- **L816 EN**: Initializes variable `biggestLen` from the right-hand expression.
  **L816 CN**: 使用右侧表达式初始化变量 `biggestLen`。

### Lines 817-840

````cpp

  // values for casting buf type and len type
  auto typeLen = fir::CharacterType::unknownLen();
  auto kind = recoverCharacterType(firstBuf.getType()).getFKind();
  auto charTy = fir::CharacterType::get(builder.getContext(), kind, typeLen);
  auto type = fir::ReferenceType::get(charTy);

  size_t numOperands = opCBVs.size();
  for (size_t cbv_idx = 1; cbv_idx < numOperands; ++cbv_idx) {
    auto currChar = opCBVs[cbv_idx];
    auto currBuf = getCharBoxBuffer(currChar);
    auto currLen = builder.createConvert(loc, builder.getCharacterLengthType(),
                                         currChar.getLen());
    // biggest len result
    mlir::Value lhsBigger = mlir::arith::CmpIOp::create(
        builder, loc, mlir::arith::CmpIPredicate::uge, biggestLen, currLen);
    biggestLen = mlir::arith::SelectOp::create(builder, loc, lhsBigger,
                                               biggestLen, currLen);

    auto cmp = predIsMin ? mlir::arith::CmpIPredicate::slt
                         : mlir::arith::CmpIPredicate::sgt;

    // lexical compare result
    mlir::Value resultCmp = fir::runtime::genCharCompare(
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L818 EN**: Comment explains nearby logic, intent, or metadata: `values for casting buf type and len type`.
  **L818 CN**: 注释说明附近代码的逻辑、意图或元数据：`values for casting buf type and len type`。
- **L819 EN**: Initializes variable `typeLen` from the right-hand expression.
  **L819 CN**: 使用右侧表达式初始化变量 `typeLen`。
- **L820 EN**: Initializes variable `kind` from the right-hand expression.
  **L820 CN**: 使用右侧表达式初始化变量 `kind`。
- **L821 EN**: Initializes variable `charTy` from the right-hand expression.
  **L821 CN**: 使用右侧表达式初始化变量 `charTy`。
- **L822 EN**: Initializes variable `type` from the right-hand expression.
  **L822 CN**: 使用右侧表达式初始化变量 `type`。
- **L823 EN**: Blank line separating nearby declarations or logic blocks.
  **L823 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L824 EN**: Initializes variable `numOperands` from the right-hand expression.
  **L824 CN**: 使用右侧表达式初始化变量 `numOperands`。
- **L825 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L825 CN**: 开始 `for` 控制流语句并计算其条件。
- **L826 EN**: Initializes variable `currChar` from the right-hand expression.
  **L826 CN**: 使用右侧表达式初始化变量 `currChar`。
- **L827 EN**: Initializes variable `currBuf` from the right-hand expression.
  **L827 CN**: 使用右侧表达式初始化变量 `currBuf`。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto currLen = builder.createConvert(loc, builder.getCharacterLengthType(),`.
  **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto currLen = builder.createConvert(loc, builder.getCharacterLengthType(),`。
- **L829 EN**: Executes a call or declaration centered on `currChar.getLen`.
  **L829 CN**: 执行以 `currChar.getLen` 为核心的调用或声明。
- **L830 EN**: Comment explains nearby logic, intent, or metadata: `biggest len result`.
  **L830 CN**: 注释说明附近代码的逻辑、意图或元数据：`biggest len result`。
- **L831 EN**: Continues logic associated with callable symbol `create`.
  **L831 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L832 EN**: Executes a standalone statement or declaration: `builder, loc, mlir::arith::CmpIPredicate::uge, biggestLen, currLen);`.
  **L832 CN**: 执行一条独立语句或声明：`builder, loc, mlir::arith::CmpIPredicate::uge, biggestLen, currLen);`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `biggestLen = mlir::arith::SelectOp::create(builder, loc, lhsBigger,`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`biggestLen = mlir::arith::SelectOp::create(builder, loc, lhsBigger,`。
- **L834 EN**: Executes a standalone statement or declaration: `biggestLen, currLen);`.
  **L834 CN**: 执行一条独立语句或声明：`biggestLen, currLen);`。
- **L835 EN**: Blank line separating nearby declarations or logic blocks.
  **L835 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L836 EN**: Continues the surrounding expression or declaration: `auto cmp = predIsMin ? mlir::arith::CmpIPredicate::slt`.
  **L836 CN**: 继续构造周围的表达式或声明：`auto cmp = predIsMin ? mlir::arith::CmpIPredicate::slt`。
- **L837 EN**: Executes a standalone statement or declaration: `: mlir::arith::CmpIPredicate::sgt;`.
  **L837 CN**: 执行一条独立语句或声明：`: mlir::arith::CmpIPredicate::sgt;`。
- **L838 EN**: Blank line separating nearby declarations or logic blocks.
  **L838 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L839 EN**: Comment explains nearby logic, intent, or metadata: `lexical compare result`.
  **L839 CN**: 注释说明附近代码的逻辑、意图或元数据：`lexical compare result`。
- **L840 EN**: Continues logic associated with callable symbol `genCharCompare`.
  **L840 CN**: 继续与可调用符号 `genCharCompare` 相关的逻辑。

### Lines 841-864

````cpp
        builder, loc, cmp, currBuf, currLen, resultBuf, resultLen);

    // it's casting (to unknown size) time!
    resultBuf = builder.createConvert(loc, type, resultBuf);
    currBuf = builder.createConvert(loc, type, currBuf);

    resultBuf = mlir::arith::SelectOp::create(builder, loc, resultCmp, currBuf,
                                              resultBuf);
    resultLen = mlir::arith::SelectOp::create(builder, loc, resultCmp, currLen,
                                              resultLen);
  }

  // now that we know the lexicographically biggest/smallest char and which char
  // had the biggest len, we can populate a temp CBV and return it
  fir::CharBoxValue temp = createCharacterTemp(resultBuf.getType(), biggestLen);
  auto toBuf = temp;
  fir::CharBoxValue fromBuf{resultBuf, resultLen};
  createAssign(toBuf, fromBuf);
  return temp;
}

fir::CharBoxValue
fir::factory::convertCharacterKind(fir::FirOpBuilder &builder,
                                   mlir::Location loc,
````
- **L841 EN**: Executes a standalone statement or declaration: `builder, loc, cmp, currBuf, currLen, resultBuf, resultLen);`.
  **L841 CN**: 执行一条独立语句或声明：`builder, loc, cmp, currBuf, currLen, resultBuf, resultLen);`。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L843 EN**: Comment explains nearby logic, intent, or metadata: `it's casting (to unknown size) time!`.
  **L843 CN**: 注释说明附近代码的逻辑、意图或元数据：`it's casting (to unknown size) time!`。
- **L844 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L844 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L845 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L845 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L846 EN**: Blank line separating nearby declarations or logic blocks.
  **L846 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `resultBuf = mlir::arith::SelectOp::create(builder, loc, resultCmp, currBuf,`.
  **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`resultBuf = mlir::arith::SelectOp::create(builder, loc, resultCmp, currBuf,`。
- **L848 EN**: Executes a standalone statement or declaration: `resultBuf);`.
  **L848 CN**: 执行一条独立语句或声明：`resultBuf);`。
- **L849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `resultLen = mlir::arith::SelectOp::create(builder, loc, resultCmp, currLen,`.
  **L849 CN**: 继续一个多行参数列表、初始化器或聚合项：`resultLen = mlir::arith::SelectOp::create(builder, loc, resultCmp, currLen,`。
- **L850 EN**: Executes a standalone statement or declaration: `resultLen);`.
  **L850 CN**: 执行一条独立语句或声明：`resultLen);`。
- **L851 EN**: Closes the current lexical scope or compound statement.
  **L851 CN**: 结束当前词法作用域或复合语句块。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L853 EN**: Comment explains nearby logic, intent, or metadata: `now that we know the lexicographically biggest/smallest char and which char`.
  **L853 CN**: 注释说明附近代码的逻辑、意图或元数据：`now that we know the lexicographically biggest/smallest char and which char`。
- **L854 EN**: Comment explains nearby logic, intent, or metadata: `had the biggest len, we can populate a temp CBV and return it`.
  **L854 CN**: 注释说明附近代码的逻辑、意图或元数据：`had the biggest len, we can populate a temp CBV and return it`。
- **L855 EN**: Initializes variable `temp` from the right-hand expression.
  **L855 CN**: 使用右侧表达式初始化变量 `temp`。
- **L856 EN**: Initializes variable `toBuf` from the right-hand expression.
  **L856 CN**: 使用右侧表达式初始化变量 `toBuf`。
- **L857 EN**: Executes a standalone statement or declaration: `fir::CharBoxValue fromBuf{resultBuf, resultLen};`.
  **L857 CN**: 执行一条独立语句或声明：`fir::CharBoxValue fromBuf{resultBuf, resultLen};`。
- **L858 EN**: Executes a call or declaration centered on `createAssign`.
  **L858 CN**: 执行以 `createAssign` 为核心的调用或声明。
- **L859 EN**: Returns from the current function with `temp`.
  **L859 CN**: 以 `temp` 从当前函数返回。
- **L860 EN**: Closes the current lexical scope or compound statement.
  **L860 CN**: 结束当前词法作用域或复合语句块。
- **L861 EN**: Blank line separating nearby declarations or logic blocks.
  **L861 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L862 EN**: Continues the surrounding expression or declaration: `fir::CharBoxValue`.
  **L862 CN**: 继续构造周围的表达式或声明：`fir::CharBoxValue`。
- **L863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::convertCharacterKind(fir::FirOpBuilder &builder,`.
  **L863 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::convertCharacterKind(fir::FirOpBuilder &builder,`。
- **L864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc,`.
  **L864 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc,`。

### Lines 865-888

````cpp
                                   fir::CharBoxValue srcBoxChar, int toKind) {
  // Use char_convert. Each code point is translated from a
  // narrower/wider encoding to the target encoding. For example, 'A'
  // may be translated from 0x41 : i8 to 0x0041 : i16. The symbol
  // for euro (0x20AC : i16) may be translated from a wide character
  // to "0xE2 0x82 0xAC" : UTF-8.
  mlir::Value bufferSize = srcBoxChar.getLen();
  auto kindMap = builder.getKindMap();
  mlir::Value boxCharAddr = srcBoxChar.getAddr();
  auto fromTy = boxCharAddr.getType();
  if (auto charTy = mlir::dyn_cast<fir::CharacterType>(fromTy)) {
    // boxchar is a value, not a variable. Turn it into a temporary.
    // As a value, it ought to have a constant LEN value.
    assert(charTy.hasConstantLen() && "must have constant length");
    mlir::Value tmp = builder.createTemporary(loc, charTy);
    fir::StoreOp::create(builder, loc, boxCharAddr, tmp);
    boxCharAddr = tmp;
  }
  auto fromBits = kindMap.getCharacterBitsize(
      mlir::cast<fir::CharacterType>(fir::unwrapRefType(fromTy)).getFKind());
  auto toBits = kindMap.getCharacterBitsize(toKind);
  if (toBits < fromBits) {
    // Scale by relative ratio to give a buffer of the same length.
    auto ratio = builder.createIntegerConstant(loc, bufferSize.getType(),
````
- **L865 EN**: Continues the surrounding expression or declaration: `fir::CharBoxValue srcBoxChar, int toKind) {`.
  **L865 CN**: 继续构造周围的表达式或声明：`fir::CharBoxValue srcBoxChar, int toKind) {`。
- **L866 EN**: Comment explains nearby logic, intent, or metadata: `Use char_convert. Each code point is translated from a`.
  **L866 CN**: 注释说明附近代码的逻辑、意图或元数据：`Use char_convert. Each code point is translated from a`。
- **L867 EN**: Comment explains nearby logic, intent, or metadata: `narrower/wider encoding to the target encoding. For example, 'A'`.
  **L867 CN**: 注释说明附近代码的逻辑、意图或元数据：`narrower/wider encoding to the target encoding. For example, 'A'`。
- **L868 EN**: Comment explains nearby logic, intent, or metadata: `may be translated from 0x41 : i8 to 0x0041 : i16. The symbol`.
  **L868 CN**: 注释说明附近代码的逻辑、意图或元数据：`may be translated from 0x41 : i8 to 0x0041 : i16. The symbol`。
- **L869 EN**: Comment explains nearby logic, intent, or metadata: `for euro (0x20AC : i16) may be translated from a wide character`.
  **L869 CN**: 注释说明附近代码的逻辑、意图或元数据：`for euro (0x20AC : i16) may be translated from a wide character`。
- **L870 EN**: Comment explains nearby logic, intent, or metadata: `to "0xE2 0x82 0xAC" : UTF-8.`.
  **L870 CN**: 注释说明附近代码的逻辑、意图或元数据：`to "0xE2 0x82 0xAC" : UTF-8.`。
- **L871 EN**: Initializes variable `bufferSize` from the right-hand expression.
  **L871 CN**: 使用右侧表达式初始化变量 `bufferSize`。
- **L872 EN**: Initializes variable `kindMap` from the right-hand expression.
  **L872 CN**: 使用右侧表达式初始化变量 `kindMap`。
- **L873 EN**: Initializes variable `boxCharAddr` from the right-hand expression.
  **L873 CN**: 使用右侧表达式初始化变量 `boxCharAddr`。
- **L874 EN**: Initializes variable `fromTy` from the right-hand expression.
  **L874 CN**: 使用右侧表达式初始化变量 `fromTy`。
- **L875 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L875 CN**: 开始 `if` 控制流语句并计算其条件。
- **L876 EN**: Comment explains nearby logic, intent, or metadata: `boxchar is a value, not a variable. Turn it into a temporary.`.
  **L876 CN**: 注释说明附近代码的逻辑、意图或元数据：`boxchar is a value, not a variable. Turn it into a temporary.`。
- **L877 EN**: Comment explains nearby logic, intent, or metadata: `As a value, it ought to have a constant LEN value.`.
  **L877 CN**: 注释说明附近代码的逻辑、意图或元数据：`As a value, it ought to have a constant LEN value.`。
- **L878 EN**: Checks an internal invariant in debug builds.
  **L878 CN**: 在调试构建中检查内部不变式。
- **L879 EN**: Initializes variable `tmp` from the right-hand expression.
  **L879 CN**: 使用右侧表达式初始化变量 `tmp`。
- **L880 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L880 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L881 EN**: Executes a standalone statement or declaration: `boxCharAddr = tmp;`.
  **L881 CN**: 执行一条独立语句或声明：`boxCharAddr = tmp;`。
- **L882 EN**: Closes the current lexical scope or compound statement.
  **L882 CN**: 结束当前词法作用域或复合语句块。
- **L883 EN**: Continues logic associated with callable symbol `getCharacterBitsize`.
  **L883 CN**: 继续与可调用符号 `getCharacterBitsize` 相关的逻辑。
- **L884 EN**: Executes a call or declaration centered on `mlir::cast<fir::CharacterType>`.
  **L884 CN**: 执行以 `mlir::cast<fir::CharacterType>` 为核心的调用或声明。
- **L885 EN**: Initializes variable `toBits` from the right-hand expression.
  **L885 CN**: 使用右侧表达式初始化变量 `toBits`。
- **L886 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L886 CN**: 开始 `if` 控制流语句并计算其条件。
- **L887 EN**: Comment explains nearby logic, intent, or metadata: `Scale by relative ratio to give a buffer of the same length.`.
  **L887 CN**: 注释说明附近代码的逻辑、意图或元数据：`Scale by relative ratio to give a buffer of the same length.`。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto ratio = builder.createIntegerConstant(loc, bufferSize.getType(),`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto ratio = builder.createIntegerConstant(loc, bufferSize.getType(),`。

### Lines 889-899

````cpp
                                               fromBits / toBits);
    bufferSize = mlir::arith::MulIOp::create(builder, loc, bufferSize, ratio);
  }
  mlir::Type toType =
      fir::CharacterType::getUnknownLen(builder.getContext(), toKind);
  auto dest = builder.createTemporary(loc, toType, /*name=*/{}, /*shape=*/{},
                                      mlir::ValueRange{bufferSize});
  fir::CharConvertOp::create(builder, loc, boxCharAddr, srcBoxChar.getLen(),
                             dest);
  return fir::CharBoxValue{dest, srcBoxChar.getLen()};
}
````
- **L889 EN**: Executes a standalone statement or declaration: `fromBits / toBits);`.
  **L889 CN**: 执行一条独立语句或声明：`fromBits / toBits);`。
- **L890 EN**: Executes a call or declaration centered on `mlir::arith::MulIOp::create`.
  **L890 CN**: 执行以 `mlir::arith::MulIOp::create` 为核心的调用或声明。
- **L891 EN**: Closes the current lexical scope or compound statement.
  **L891 CN**: 结束当前词法作用域或复合语句块。
- **L892 EN**: Continues the surrounding expression or declaration: `mlir::Type toType =`.
  **L892 CN**: 继续构造周围的表达式或声明：`mlir::Type toType =`。
- **L893 EN**: Executes a call or declaration centered on `fir::CharacterType::getUnknownLen`.
  **L893 CN**: 执行以 `fir::CharacterType::getUnknownLen` 为核心的调用或声明。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto dest = builder.createTemporary(loc, toType, /*name=*/{}, /*shape=*/{},`.
  **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto dest = builder.createTemporary(loc, toType, /*name=*/{}, /*shape=*/{},`。
- **L895 EN**: Executes a standalone statement or declaration: `mlir::ValueRange{bufferSize});`.
  **L895 CN**: 执行一条独立语句或声明：`mlir::ValueRange{bufferSize});`。
- **L896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::CharConvertOp::create(builder, loc, boxCharAddr, srcBoxChar.getLen(),`.
  **L896 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::CharConvertOp::create(builder, loc, boxCharAddr, srcBoxChar.getLen(),`。
- **L897 EN**: Executes a standalone statement or declaration: `dest);`.
  **L897 CN**: 执行一条独立语句或声明：`dest);`。
- **L898 EN**: Returns from the current function with `fir::CharBoxValue{dest, srcBoxChar.getLen()}`.
  **L898 CN**: 以 `fir::CharBoxValue{dest, srcBoxChar.getLen()}` 从当前函数返回。
- **L899 EN**: Closes the current lexical scope or compound statement.
  **L899 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **IR builder orchestration / IR Builder 编排**
- **Runtime call integration / 运行时调用集成**

## Dependencies / 依赖关系

- `flang/Optimizer/Builder/Character.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/DoLoopHelper.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/FIRBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/Todo.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Dialect/FIROpsSupport.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `mlir/Dialect/LLVMIR/LLVMDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `llvm/Support/Debug.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `optional`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
