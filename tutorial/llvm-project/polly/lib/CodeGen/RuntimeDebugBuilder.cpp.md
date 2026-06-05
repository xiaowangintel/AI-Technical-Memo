# RuntimeDebugBuilder.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/CodeGen/RuntimeDebugBuilder.cpp` | `polly/lib/CodeGen/RuntimeDebugBuilder.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements Polly code generation, AST lowering, and runtime emission logic. The leading comment describes it as: Helper to insert prints into LLVM-IR. | 实现 Polly 的代码生成、AST 降级与运行时生成逻辑。 文件开头注释将其概括为：Helper to insert prints into LLVM-IR。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

````cpp
//===--- RuntimeDebugBuilder.cpp - Helper to insert prints into LLVM-IR ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//===----------------------------------------------------------------------===//

#include "polly/CodeGen/RuntimeDebugBuilder.h"
#include "llvm/IR/Module.h"
#include <string>
#include <vector>

````
- **EN**: This block records the standard LLVM file banner and license metadata; imports Polly, LLVM-family, system/standard headers needed by the surrounding code; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 引入周边逻辑所需的 Polly、LLVM-family、system/standard 头文件; 保留用于解释意图、用法或算法背景的注释.

### Lines 16-28

````cpp
using namespace llvm;
using namespace polly;

llvm::Value *RuntimeDebugBuilder::getPrintableString(PollyIRBuilder &Builder,
                                                     llvm::StringRef Str) {
  // FIXME: addressspace(4) is a marker for a string (for the %s conversion
  // specifier) but should be using the default address space. This only works
  // because CPU backends typically ignore the address space. For constant
  // strings as returned by getPrintableString, the format string should instead
  // directly spell out the string.
  return Builder.CreateGlobalString(Str, "", 4);
}

````
- **EN**: This block declares or defines routines around `getPrintableString`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getPrintableString` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 29-40

````cpp
Function *RuntimeDebugBuilder::getVPrintF(PollyIRBuilder &Builder) {
  Module *M = Builder.GetInsertBlock()->getParent()->getParent();
  const char *Name = "vprintf";
  Function *F = M->getFunction(Name);

  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    FunctionType *Ty = FunctionType::get(
        Builder.getInt32Ty(), {Builder.getPtrTy(), Builder.getPtrTy()}, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

````
- **EN**: This block declares or defines routines around `getVPrintF`, `GetInsertBlock`, `getFunction`, `get` (+2 more); contains control flow with 1 conditional check(s).
- **CN**: 该代码块 声明或定义与 `getVPrintF`, `GetInsertBlock`, `getFunction`, `get` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断.

### Lines 41-52

````cpp
  return F;
}

void RuntimeDebugBuilder::createPrinter(PollyIRBuilder &Builder,
                                        ArrayRef<Value *> Values) {
  createCPUPrinterT(Builder, Values);
}

bool RuntimeDebugBuilder::isPrintable(Type *Ty) {
  if (Ty->isFloatingPointTy())
    return true;

````
- **EN**: This block declares or defines routines around `createPrinter`, `createCPUPrinterT`, `isPrintable`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `createPrinter`, `createCPUPrinterT`, `isPrintable` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 53-66

````cpp
  if (Ty->isIntegerTy())
    return Ty->getIntegerBitWidth() <= 64;

  if (isa<PointerType>(Ty))
    return true;

  return false;
}

static std::tuple<std::string, std::vector<Value *>>
prepareValuesForPrinting(PollyIRBuilder &Builder, ArrayRef<Value *> Values) {
  std::string FormatString;
  std::vector<Value *> ValuesToPrint;

````
- **EN**: This block declares or defines routines around `prepareValuesForPrinting`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `prepareValuesForPrinting` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 67-82

````cpp
  for (auto Val : Values) {
    Type *Ty = Val->getType();

    if (Ty->isFloatingPointTy()) {
      if (!Ty->isDoubleTy())
        Val = Builder.CreateFPExt(Val, Builder.getDoubleTy());
    } else if (Ty->isIntegerTy()) {
      if (Ty->getIntegerBitWidth() < 64)
        Val = Builder.CreateSExt(Val, Builder.getInt64Ty());
      else
        assert(Ty->getIntegerBitWidth() &&
               "Integer types larger 64 bit not supported");
    } else if (isa<PointerType>(Ty)) {
      if (Ty == Builder.getPtrTy(4)) {
        Val = Builder.CreateGEP(Builder.getInt8Ty(), Val, Builder.getInt64(0));
      } else {
````
- **EN**: This block declares or defines routines around `getType`, `CreateFPExt`, `CreateSExt`, `CreateGEP`; contains control flow with 1 loop construct(s), 6 conditional check(s); adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `getType`, `CreateFPExt`, `CreateSExt`, `CreateGEP` 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 加入用于保护不变量或异常状态的断言/检查.

### Lines 83-97

````cpp
        Val = Builder.CreatePtrToInt(Val, Builder.getInt64Ty());
      }
    } else {
      llvm_unreachable("Unknown type");
    }

    Ty = Val->getType();

    if (Ty->isFloatingPointTy())
      FormatString += "%f";
    else if (Ty->isIntegerTy())
      FormatString += "%ld";
    else
      FormatString += "%s";

````
- **EN**: This block declares or defines routines around `CreatePtrToInt`, `getType`; contains control flow with 2 conditional check(s); adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `CreatePtrToInt`, `getType` 相关的例程; 包含控制流结构：2 处条件判断; 加入用于保护不变量或异常状态的断言/检查.

### Lines 98-109

````cpp
    ValuesToPrint.push_back(Val);
  }

  return std::make_tuple(FormatString, ValuesToPrint);
}

void RuntimeDebugBuilder::createCPUPrinterT(PollyIRBuilder &Builder,
                                            ArrayRef<Value *> Values) {

  std::string FormatString;
  std::vector<Value *> ValuesToPrint;

````
- **EN**: This block declares or defines routines around `push_back`, `createCPUPrinterT`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `push_back`, `createCPUPrinterT` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 110-121

````cpp
  std::tie(FormatString, ValuesToPrint) =
      prepareValuesForPrinting(Builder, Values);

  createPrintF(Builder, FormatString, ValuesToPrint);
  createFlush(Builder);
}

Function *RuntimeDebugBuilder::getPrintF(PollyIRBuilder &Builder) {
  Module *M = Builder.GetInsertBlock()->getParent()->getParent();
  const char *Name = "printf";
  Function *F = M->getFunction(Name);

````
- **EN**: This block declares or defines routines around `tie`, `prepareValuesForPrinting`, `createPrintF`, `createFlush` (+3 more).
- **CN**: 该代码块 声明或定义与 `tie`, `prepareValuesForPrinting`, `createPrintF`, `createFlush` (+3 more) 相关的例程.

### Lines 122-136

````cpp
  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    FunctionType *Ty = FunctionType::get(Builder.getInt32Ty(), true);
    F = Function::Create(Ty, Linkage, Name, M);
  }

  return F;
}

void RuntimeDebugBuilder::createPrintF(PollyIRBuilder &Builder,
                                       std::string Format,
                                       ArrayRef<Value *> Values) {
  Value *FormatString = Builder.CreateGlobalString(Format);
  std::vector<Value *> Arguments;

````
- **EN**: This block declares or defines routines around `get`, `Create`, `createPrintF`, `CreateGlobalString`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `get`, `Create`, `createPrintF`, `CreateGlobalString` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 137-153

````cpp
  Arguments.push_back(FormatString);
  Arguments.insert(Arguments.end(), Values.begin(), Values.end());
  Builder.CreateCall(getPrintF(Builder), Arguments);
}

void RuntimeDebugBuilder::createFlush(PollyIRBuilder &Builder) {
  Module *M = Builder.GetInsertBlock()->getParent()->getParent();
  const char *Name = "fflush";
  Function *F = M->getFunction(Name);

  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    FunctionType *Ty =
        FunctionType::get(Builder.getInt32Ty(), Builder.getPtrTy(), false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

````
- **EN**: This block declares or defines routines around `push_back`, `insert`, `CreateCall`, `createFlush` (+4 more); contains control flow with 1 conditional check(s).
- **CN**: 该代码块 声明或定义与 `push_back`, `insert`, `CreateCall`, `createFlush` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断.

### Lines 154-161

````cpp
  // fflush(NULL) flushes _all_ open output streams.
  //
  // fflush is declared as 'int fflush(FILE *stream)'. As we only pass on a NULL
  // pointer, the type we point to does conceptually not matter. However, if
  // fflush is already declared in this translation unit, we use the very same
  // type to ensure that LLVM does not complain about mismatching types.
  Builder.CreateCall(F, Constant::getNullValue(F->arg_begin()->getType()));
}
````
- **EN**: This block declares or defines routines around `CreateCall`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `CreateCall` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

## Key Concepts / 关键概念

- **Code generation**
  - **CN**: 代码生成
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持
- **Loop transformation**
  - **CN**: 循环变换

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/RuntimeDebugBuilder.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/RuntimeDebugBuilder.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/IR/Module.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/IR/Module.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
- **System/standard headers**: `string`, `vector` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`string`, `vector` —— 实现所需的标准库或系统声明。
