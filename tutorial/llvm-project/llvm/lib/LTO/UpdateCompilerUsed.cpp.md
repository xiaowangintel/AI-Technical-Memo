# UpdateCompilerUsed.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/LTO/UpdateCompilerUsed.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: ==-LTOInternalize.cpp - LLVM Link Time Optimizer Internalization Utility -==//.
  - **CN**: 实现 LTO/ThinLTO 的编排、后端以及模块管理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//==-LTOInternalize.cpp - LLVM Link Time Optimizer Internalization Utility -==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-12
```cpp
//
// This file defines a helper to run the internalization part of LTO.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 13-21
```cpp
#include "llvm/LTO/legacy/UpdateCompilerUsed.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/Mangler.h"
#include "llvm/IR/Module.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/LTO/legacy/UpdateCompilerUsed.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/TargetSubtargetInfo.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/LTO/legacy/UpdateCompilerUsed.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/TargetSubtargetInfo.h`。

### Lines 22-27
```cpp
using namespace llvm;

namespace {

// Helper class that collects AsmUsed and user supplied libcalls.
class PreserveLibCallsAndAsmUsed {
```
- **EN**: Introduces declarations for `llvm`, `that`, `PreserveLibCallsAndAsmUsed`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm`, `that`, `PreserveLibCallsAndAsmUsed` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 28-33
```cpp
public:
  PreserveLibCallsAndAsmUsed(const StringSet<> &AsmUndefinedRefs,
                             const TargetMachine &TM,
                             std::vector<GlobalValue *> &LLVMUsed)
      : AsmUndefinedRefs(AsmUndefinedRefs), TM(TM), LLVMUsed(LLVMUsed) {}

```
- **EN**: Implements logic around `PreserveLibCallsAndAsmUsed`, `AsmUndefinedRefs`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `PreserveLibCallsAndAsmUsed`, `AsmUndefinedRefs` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 34-43
```cpp
  void findInModule(Module &TheModule) {
    initializeLibCalls(TheModule);
    for (Function &F : TheModule)
      findLibCallsAndAsm(F);
    for (GlobalVariable &GV : TheModule.globals())
      findLibCallsAndAsm(GV);
    for (GlobalAlias &GA : TheModule.aliases())
      findLibCallsAndAsm(GA);
  }

```
- **EN**: Implements logic around `findInModule`, `initializeLibCalls`, `findLibCallsAndAsm`, `globals`, and 1 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `findInModule`, `initializeLibCalls`, `findLibCallsAndAsm`, `globals`, and 1 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 44-48
```cpp
private:
  // Inputs
  const StringSet<> &AsmUndefinedRefs;
  const TargetMachine &TM;

```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 49-55
```cpp
  // Temps
  llvm::Mangler Mangler;
  StringSet<> Libcalls;

  // Output
  std::vector<GlobalValue *> &LLVMUsed;

```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 56-62
```cpp
  // Collect names of runtime library functions. User-defined functions with the
  // same names are added to llvm.compiler.used to prevent them from being
  // deleted by optimizations.
  void initializeLibCalls(const Module &TheModule) {
    TargetLibraryInfoImpl TLII(TM.getTargetTriple(), TM.Options.VecLib);
    TargetLibraryInfo TLI(TLII);

```
- **EN**: Implements logic around `initializeLibCalls`, `TLII`, `TLI`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `initializeLibCalls`, `TLII`, `TLI` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 63-71
```cpp
    // TargetLibraryInfo has info on C runtime library calls on the current
    // target.
    for (unsigned I = LibFunc::Begin_LibFunc, E = LibFunc::End_LibFunc; I != E;
         ++I) {
      LibFunc F = static_cast<LibFunc>(I);
      if (TLI.has(F))
        Libcalls.insert(TLI.getName(F));
    }

```
- **EN**: Implements logic around `static_cast`, `has`, `insert`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `static_cast`, `has`, `insert` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 72-77
```cpp
    SmallPtrSet<const TargetLowering *, 1> TLSet;

    for (const Function &F : TheModule) {
      const TargetLowering *Lowering =
          TM.getSubtargetImpl(F)->getTargetLowering();

```
- **EN**: Implements logic around `getSubtargetImpl`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getSubtargetImpl` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 78-87
```cpp
      if (Lowering && TLSet.insert(Lowering).second)
        // TargetLowering has info on library calls that CodeGen expects to be
        // available, both from the C runtime and compiler-rt.
        for (unsigned I = 0, E = static_cast<unsigned>(RTLIB::UNKNOWN_LIBCALL);
             I != E; ++I)
          if (const char *Name =
                  Lowering->getLibcallName(static_cast<RTLIB::Libcall>(I)))
            Libcalls.insert(Name);
    }
  }
```
- **EN**: Implements logic around `insert`, `static_cast`, `getLibcallName`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `insert`, `static_cast`, `getLibcallName` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 88-93
```cpp

  void findLibCallsAndAsm(GlobalValue &GV) {
    // There are no restrictions to apply to declarations.
    if (GV.isDeclaration())
      return;

```
- **EN**: Implements logic around `findLibCallsAndAsm`, `isDeclaration`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `findLibCallsAndAsm`, `isDeclaration` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 94-103
```cpp
    // There is nothing more restrictive than private linkage.
    if (GV.hasPrivateLinkage())
      return;

    // Conservatively append user-supplied runtime library functions (supplied
    // either directly, or via a function alias) to llvm.compiler.used.  These
    // could be internalized and deleted by optimizations like -globalopt,
    // causing problems when later optimizations add new library calls (e.g.,
    // llvm.memset => memset and printf => puts).
    // Leave it to the linker to remove any dead code (e.g. with -dead_strip).
```
- **EN**: Implements logic around `hasPrivateLinkage`; this block emits or serializes data to an external representation; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `hasPrivateLinkage` 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并协调跨模块链接或优化状态。

### Lines 104-113
```cpp
    GlobalValue *FuncAliasee = nullptr;
    if (isa<GlobalAlias>(GV)) {
      auto *A = cast<GlobalAlias>(&GV);
      FuncAliasee = dyn_cast<Function>(A->getAliasee());
    }
    if ((isa<Function>(GV) || FuncAliasee) && Libcalls.count(GV.getName())) {
      LLVMUsed.push_back(&GV);
      return;
    }

```
- **EN**: Implements logic around `isa`, `cast`, `dyn_cast`, `push_back`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `isa`, `cast`, `dyn_cast`, `push_back` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 114-120
```cpp
    SmallString<64> Buffer;
    TM.getNameWithPrefix(Buffer, &GV, Mangler);
    if (AsmUndefinedRefs.count(Buffer))
      LLVMUsed.push_back(&GV);
  }
};

```
- **EN**: Implements logic around `getNameWithPrefix`, `count`, `push_back`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getNameWithPrefix`, `count`, `push_back` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 121-128
```cpp
} // namespace anonymous

void llvm::updateCompilerUsed(Module &TheModule, const TargetMachine &TM,
                              const StringSet<> &AsmUndefinedRefs) {
  std::vector<GlobalValue *> UsedValues;
  PreserveLibCallsAndAsmUsed(AsmUndefinedRefs, TM, UsedValues)
      .findInModule(TheModule);

```
- **EN**: Introduces declarations for `anonymous`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `anonymous` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 129-133
```cpp
  if (UsedValues.empty())
    return;

  appendToCompilerUsed(TheModule, UsedValues);
}
```
- **EN**: Implements logic around `empty`, `appendToCompilerUsed`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `empty`, `appendToCompilerUsed` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

## Key Concepts / 关键概念

- **Link-time optimization / 链接时优化**:
  - **EN**: Coordinates summary-based and full-module optimization at link time.
  - **CN**: 在链接阶段协调基于摘要和全模块的优化。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/LTO/legacy/UpdateCompilerUsed.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/Mangler.h`, `llvm/IR/Module.h`, `llvm/Target/TargetMachine.h`, `llvm/Transforms/Utils/ModuleUtils.h`
- **Subsystem categories / 子系统类别**: code-generation support types / 代码生成支持类型 (2), LLVM IR core abstractions / LLVM IR 核心抽象 (2), link-time optimization interfaces / 链接时优化接口 (1), analysis interfaces and cached results / 分析接口与缓存结果 (1), target description interfaces / 目标描述接口 (1)
