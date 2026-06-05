# TargetSelect.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/TargetSelect.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Target Chooser Code.
  - **CN**: 实现供执行引擎用户调用的辅助入口，用于初始化 LLVM 各目标。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- TargetSelect.cpp - Target Chooser Code ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-14
```cpp
//
// This just asks the TargetRegistry for the appropriate target to use, and
// allows the user to specify a specific one on the commandline with -march=x,
// -mcpu=y, and -mattr=a,-b,+c. Clients should initialize targets prior to
// calling selectTarget().
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declaration group or explains a subsystem-specific rule.
- **CN**: 为后续声明分组提供说明，或解释某个子系统的专用规则。

### Lines 15-23
```cpp

#include "llvm/ExecutionEngine/ExecutionEngine.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/TargetParser/Host.h"
#include "llvm/TargetParser/SubtargetFeature.h"
#include "llvm/TargetParser/Triple.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/ExecutionEngine.h`, `llvm/IR/Module.h`, `llvm/MC/TargetRegistry.h`, `llvm/Target/TargetMachine.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/ExecutionEngine.h`, `llvm/IR/Module.h`, `llvm/MC/TargetRegistry.h`, `llvm/Target/TargetMachine.h`。

### Lines 24-28
```cpp
using namespace llvm;

TargetMachine *EngineBuilder::selectTarget() {
  Triple TT;

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 29-33
```cpp
  // MCJIT can generate code for remote targets, but the old JIT and Interpreter
  // must use the host architecture.
  if (WhichEngine != EngineKind::Interpreter && M)
    TT = M->getTargetTriple();

```
- **EN**: Implements logic around `getTargetTriple`.
- **CN**: 围绕 `getTargetTriple` 实现具体逻辑。

### Lines 34-38
```cpp
  return selectTarget(TT, MArch, MCPU, MAttrs);
}

/// selectTarget - Pick a target either via -march or by guessing the native
/// arch.  Add any CPU features specified via -mcpu or -mattr.
```
- **EN**: Implements logic around `selectTarget`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `selectTarget` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 39-46
```cpp
TargetMachine *EngineBuilder::selectTarget(const Triple &TargetTriple,
                              StringRef MArch,
                              StringRef MCPU,
                              const SmallVectorImpl<std::string>& MAttrs) {
  Triple TheTriple(TargetTriple);
  if (TheTriple.getTriple().empty())
    TheTriple.setTriple(sys::getProcessTriple());

```
- **EN**: Implements logic around `selectTarget`, `TheTriple`, `setTriple`.
- **CN**: 围绕 `selectTarget`, `TheTriple`, `setTriple` 实现具体逻辑。

### Lines 47-52
```cpp
  // Adjust the triple to match what the user requested.
  const Target *TheTarget = nullptr;
  if (!MArch.empty()) {
    auto I = find_if(TargetRegistry::targets(),
                     [&](const Target &T) { return MArch == T.getName(); });

```
- **EN**: Implements logic around `find_if`, `getName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `find_if`, `getName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 53-59
```cpp
    if (I == TargetRegistry::targets().end()) {
      if (ErrorStr)
        *ErrorStr = "No available targets are compatible with this -march, "
                    "see -version for the available targets.\n";
      return nullptr;
    }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 60-69
```cpp
    TheTarget = &*I;

    // Adjust the triple to match (if known), otherwise stick with the
    // requested/host triple.
    Triple::ArchType Type = Triple::getArchTypeForLLVMName(MArch);
    if (Type != Triple::UnknownArch)
      TheTriple.setArch(Type);
  } else {
    std::string Error;
    TheTarget = TargetRegistry::lookupTarget(TheTriple, Error);
```
- **EN**: Implements logic around `getArchTypeForLLVMName`, `setArch`, `lookupTarget`.
- **CN**: 围绕 `getArchTypeForLLVMName`, `setArch`, `lookupTarget` 实现具体逻辑。

### Lines 70-76
```cpp
    if (!TheTarget) {
      if (ErrorStr)
        *ErrorStr = Error;
      return nullptr;
    }
  }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 77-85
```cpp
  // Package up features to be passed to target/subtarget
  std::string FeaturesStr;
  if (!MAttrs.empty()) {
    SubtargetFeatures Features;
    for (unsigned i = 0; i != MAttrs.size(); ++i)
      Features.AddFeature(MAttrs[i]);
    FeaturesStr = Features.getString();
  }

```
- **EN**: Implements logic around `AddFeature`, `getString`.
- **CN**: 围绕 `AddFeature`, `getString` 实现具体逻辑。

### Lines 86-91
```cpp
  // Allocate a target...
  TargetMachine *Target = TheTarget->createTargetMachine(
      TheTriple, MCPU, FeaturesStr, Options, RelocModel, CMModel, OptLevel,
      /*JIT=*/true);
  Target->Options.EmulatedTLS = EmulatedTLS;

```
- **EN**: Implements logic around `createTargetMachine`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `createTargetMachine` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 92-94
```cpp
  assert(Target && "Could not allocate target machine!");
  return Target;
}
```
- **EN**: Implements logic around `assert`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/ExecutionEngine.h`, `llvm/IR/Module.h`, `llvm/MC/TargetRegistry.h`, `llvm/Target/TargetMachine.h`, `llvm/TargetParser/Host.h`, `llvm/TargetParser/SubtargetFeature.h`, `llvm/TargetParser/Triple.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, MC, IR, Target/TargetParser
