# ASanStackFrameLayout.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/ASanStackFrameLayout.cpp` | `llvm/lib/Transforms/Utils/ASanStackFrameLayout.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements helper for AddressSanitizer within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 ASanStackFrameLayout 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===-- ASanStackFrameLayout.cpp - helper for AddressSanitizer ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Definition of ComputeASanStackFrameLayout (see ASanStackFrameLayout.h).
//
//===----------------------------------------------------------------------===//
#include "llvm/Transforms/Utils/ASanStackFrameLayout.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/ScopedPrinter.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。

### Lines 19-36

```cpp
namespace llvm {

// We sort the stack variables by alignment (largest first) to minimize
// unnecessary large gaps due to alignment.
// It is tempting to also sort variables by size so that larger variables
// have larger redzones at both ends. But reordering will make report analysis
// harder, especially when temporary unnamed variables are present.
// So, until we can provide more information (type, line number, etc)
// for the stack variables we avoid reordering them too much.
static inline bool CompareVars(const ASanStackVariableDescription &a,
                               const ASanStackVariableDescription &b) {
  return a.Alignment > b.Alignment;
}

// We also force minimal alignment for all vars to kMinAlignment so that vars
// with e.g. alignment 1 and alignment 16 do not get reordered by CompareVars.
static const uint64_t kMinAlignment = 16;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 37-51

```cpp
// We want to add a full redzone after every variable.
// The larger the variable Size the larger is the redzone.
// The resulting frame size is a multiple of Alignment.
static uint64_t VarAndRedzoneSize(uint64_t Size, uint64_t Granularity,
                                  uint64_t Alignment) {
  uint64_t Res = 0;
  if (Size <= 4)  Res = 16;
  else if (Size <= 16) Res = 32;
  else if (Size <= 128) Res = Size + 32;
  else if (Size <= 512) Res = Size + 64;
  else if (Size <= 4096) Res = Size + 128;
  else                   Res = Size + 256;
  return alignTo(std::max(Res, 2 * Granularity), Alignment);
}

```
- EN: This region continues the ASanStackFrameLayout implementation with local helper logic centered on The, Size, Alignment, VarAndRedzoneSize.
- CN: 这一段延续了 ASanStackFrameLayout 的主体实现，围绕 The, Size, Alignment, VarAndRedzoneSize 等局部辅助逻辑展开。

### Lines 52-65

```cpp
ASanStackFrameLayout
ComputeASanStackFrameLayout(SmallVectorImpl<ASanStackVariableDescription> &Vars,
                            uint64_t Granularity, uint64_t MinHeaderSize) {
  assert(Granularity >= 8 && Granularity <= 64 &&
         (Granularity & (Granularity - 1)) == 0);
  assert(MinHeaderSize >= 16 && (MinHeaderSize & (MinHeaderSize - 1)) == 0 &&
         MinHeaderSize >= Granularity);
  const size_t NumVars = Vars.size();
  assert(NumVars > 0);
  for (size_t i = 0; i < NumVars; i++)
    Vars[i].Alignment = std::max(Vars[i].Alignment, kMinAlignment);

  llvm::stable_sort(Vars, CompareVars);

```
- EN: This region continues the ASanStackFrameLayout implementation with local helper logic centered on ASanStackFrameLayout, ComputeASanStackFrameLayout, SmallVectorImpl, ASanStackVariableDescription.
- CN: 这一段延续了 ASanStackFrameLayout 的主体实现，围绕 ASanStackFrameLayout, ComputeASanStackFrameLayout, SmallVectorImpl, ASanStackVariableDescription 等局部辅助逻辑展开。

### Lines 66-81

```cpp
  ASanStackFrameLayout Layout;
  Layout.Granularity = Granularity;
  Layout.FrameAlignment = std::max(Granularity, Vars[0].Alignment);
  uint64_t Offset =
      std::max(std::max(MinHeaderSize, Granularity), Vars[0].Alignment);
  assert((Offset % Granularity) == 0);
  for (size_t i = 0; i < NumVars; i++) {
    bool IsLast = i == NumVars - 1;
    uint64_t Alignment = std::max(Granularity, Vars[i].Alignment);
    (void)Alignment;  // Used only in asserts.
    uint64_t Size = Vars[i].Size;
    assert((Alignment & (Alignment - 1)) == 0);
    assert(Layout.FrameAlignment >= Alignment);
    assert((Offset % Alignment) == 0);
    assert(Size > 0);
    uint64_t NextAlignment =
```
- EN: This region continues the ASanStackFrameLayout implementation with local helper logic centered on ASanStackFrameLayout, Layout, Granularity, FrameAlignment.
- CN: 这一段延续了 ASanStackFrameLayout 的主体实现，围绕 ASanStackFrameLayout, Layout, Granularity, FrameAlignment 等局部辅助逻辑展开。

### Lines 82-101

```cpp
        IsLast ? Granularity : std::max(Granularity, Vars[i + 1].Alignment);
    uint64_t SizeWithRedzone =
        VarAndRedzoneSize(Size, Granularity, NextAlignment);
    Vars[i].Offset = Offset;
    Offset += SizeWithRedzone;
  }
  if (Offset % MinHeaderSize) {
    Offset += MinHeaderSize - (Offset % MinHeaderSize);
  }
  Layout.FrameSize = Offset;
  assert((Layout.FrameSize % MinHeaderSize) == 0);
  return Layout;
}

SmallString<64> ComputeASanStackFrameDescription(
    const SmallVectorImpl<ASanStackVariableDescription> &Vars) {
  SmallString<2048> StackDescriptionStorage;
  raw_svector_ostream StackDescription(StackDescriptionStorage);
  StackDescription << Vars.size();

```
- EN: This region continues the ASanStackFrameLayout implementation with local helper logic centered on IsLast, Granularity, Vars, Alignment.
- CN: 这一段延续了 ASanStackFrameLayout 的主体实现，围绕 IsLast, Granularity, Vars, Alignment 等局部辅助逻辑展开。

### Lines 102-117

```cpp
  for (const auto &Var : Vars) {
    std::string Name = Var.Name;
    if (Var.Line) {
      Name += ":";
      Name += to_string(Var.Line);
    }
    StackDescription << " " << Var.Offset << " " << Var.Size << " "
                     << Name.size() << " " << Name;
  }
  return StackDescription.str();
}

SmallVector<uint8_t, 64>
GetShadowBytes(const SmallVectorImpl<ASanStackVariableDescription> &Vars,
               const ASanStackFrameLayout &Layout) {
  assert(Vars.size() > 0);
```
- EN: This region continues the ASanStackFrameLayout implementation with local helper logic centered on Var, Vars, Name, Line.
- CN: 这一段延续了 ASanStackFrameLayout 的主体实现，围绕 Var, Vars, Name, Line 等局部辅助逻辑展开。

### Lines 118-132

```cpp
  SmallVector<uint8_t, 64> SB;
  SB.clear();
  const uint64_t Granularity = Layout.Granularity;
  SB.resize(Vars[0].Offset / Granularity, kAsanStackLeftRedzoneMagic);
  for (const auto &Var : Vars) {
    SB.resize(Var.Offset / Granularity, kAsanStackMidRedzoneMagic);

    SB.resize(SB.size() + Var.Size / Granularity, 0);
    if (Var.Size % Granularity)
      SB.push_back(Var.Size % Granularity);
  }
  SB.resize(Layout.FrameSize / Granularity, kAsanStackRightRedzoneMagic);
  return SB;
}

```
- EN: This region continues the ASanStackFrameLayout implementation with local helper logic centered on SmallVector, Granularity, Layout, Vars.
- CN: 这一段延续了 ASanStackFrameLayout 的主体实现，围绕 SmallVector, Granularity, Layout, Vars 等局部辅助逻辑展开。

### Lines 133-150

```cpp
SmallVector<uint8_t, 64> GetShadowBytesAfterScope(
    const SmallVectorImpl<ASanStackVariableDescription> &Vars,
    const ASanStackFrameLayout &Layout) {
  SmallVector<uint8_t, 64> SB = GetShadowBytes(Vars, Layout);
  const uint64_t Granularity = Layout.Granularity;

  for (const auto &Var : Vars) {
    assert(Var.LifetimeSize <= Var.Size);
    const uint64_t LifetimeShadowSize =
        (Var.LifetimeSize + Granularity - 1) / Granularity;
    const uint64_t Offset = Var.Offset / Granularity;
    std::fill(SB.begin() + Offset, SB.begin() + Offset + LifetimeShadowSize,
              kAsanStackUseAfterScopeMagic);
  }

  return SB;
}

```
- EN: This region continues the ASanStackFrameLayout implementation with local helper logic centered on SmallVector, GetShadowBytesAfterScope, SmallVectorImpl, ASanStackVariableDescription.
- CN: 这一段延续了 ASanStackFrameLayout 的主体实现，围绕 SmallVector, GetShadowBytesAfterScope, SmallVectorImpl, ASanStackVariableDescription 等局部辅助逻辑展开。

### Lines 151-151

```cpp
} // llvm namespace
```
- EN: This region continues the ASanStackFrameLayout implementation with local helper logic centered on ASanStackFrameLayout.
- CN: 这一段延续了 ASanStackFrameLayout 的主体实现，围绕 ASanStackFrameLayout 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/Transforms/Utils/ASanStackFrameLayout.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Utils/ASanStackFrameLayout.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/MathExtras.h`, `llvm/Support/ScopedPrinter.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallString.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/MathExtras.h`, `llvm/Support/ScopedPrinter.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallString.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm` 为 LLVM API 之外的 C++ 语言工具提供支持。
