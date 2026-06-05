# Target.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Target.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements shared target-level utilities and registration support used across LLVM backends.
  - **CN**: 实现 LLVM 后端共用的目标层工具与注册支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- Target.cpp --------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```cpp
//
// This file implements the common infrastructure (including C bindings) for
// libLLVMTarget.a, which implements target information.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```cpp

#include "llvm-c/Target.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Value.h"
#include "llvm/InitializePasses.h"
#include <cstring>
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm-c/Target.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/IR/DataLayout.h`, `llvm/IR/LLVMContext.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm-c/Target.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/IR/DataLayout.h`, `llvm/IR/LLVMContext.h`。

### Lines 23-29
```cpp

using namespace llvm;

inline TargetLibraryInfoImpl *unwrap(LLVMTargetLibraryInfoRef P) {
  return reinterpret_cast<TargetLibraryInfoImpl*>(P);
}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 30-34
```cpp
inline LLVMTargetLibraryInfoRef wrap(const TargetLibraryInfoImpl *P) {
  TargetLibraryInfoImpl *X = const_cast<TargetLibraryInfoImpl*>(P);
  return reinterpret_cast<LLVMTargetLibraryInfoRef>(X);
}

```
- **EN**: Implements logic around `wrap`, `reinterpret_cast<LLVMTargetLibraryInfoRef>`; this block returns target-specific results.
- **CN**: 围绕 `wrap`, `reinterpret_cast<LLVMTargetLibraryInfoRef>` 实现具体逻辑；这一段返回目标相关结果。

### Lines 35-40
```cpp
void llvm::initializeTarget(PassRegistry &Registry) {
  initializeTargetLibraryInfoWrapperPassPass(Registry);
  initializeRuntimeLibraryInfoWrapperPass(Registry);
  initializeTargetTransformInfoWrapperPassPass(Registry);
}

```
- **EN**: Implements logic around `initializeTarget`, `initializeTargetLibraryInfoWrapperPassPass`, `initializeRuntimeLibraryInfoWrapperPass`, `initializeTargetTransformInfoWrapperPassPass`.
- **CN**: 围绕 `initializeTarget`, `initializeTargetLibraryInfoWrapperPassPass`, `initializeRuntimeLibraryInfoWrapperPass`, `initializeTargetTransformInfoWrapperPassPass` 实现具体逻辑。

### Lines 41-48
```cpp
LLVMTargetDataRef LLVMGetModuleDataLayout(LLVMModuleRef M) {
  return wrap(&unwrap(M)->getDataLayout());
}

void LLVMSetModuleDataLayout(LLVMModuleRef M, LLVMTargetDataRef DL) {
  unwrap(M)->setDataLayout(*unwrap(DL));
}

```
- **EN**: Implements logic around `wrap`, `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `wrap`, `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 49-56
```cpp
LLVMTargetDataRef LLVMCreateTargetData(const char *StringRep) {
  return wrap(new DataLayout(StringRep));
}

void LLVMDisposeTargetData(LLVMTargetDataRef TD) {
  delete unwrap(TD);
}

```
- **EN**: Implements logic around `wrap`, `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `wrap`, `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 57-61
```cpp
void LLVMAddTargetLibraryInfo(LLVMTargetLibraryInfoRef TLI,
                              LLVMPassManagerRef PM) {
  unwrap(PM)->add(new TargetLibraryInfoWrapperPass(*unwrap(TLI)));
}

```
- **EN**: Implements logic around `unwrap`.
- **CN**: 围绕 `unwrap` 实现具体逻辑。

### Lines 62-66
```cpp
char *LLVMCopyStringRepOfTargetData(LLVMTargetDataRef TD) {
  std::string StringRep = unwrap(TD)->getStringRepresentation();
  return strdup(StringRep.c_str());
}

```
- **EN**: Implements logic around `unwrap`, `strdup`; this block returns target-specific results.
- **CN**: 围绕 `unwrap`, `strdup` 实现具体逻辑；这一段返回目标相关结果。

### Lines 67-74
```cpp
LLVMByteOrdering LLVMByteOrder(LLVMTargetDataRef TD) {
  return unwrap(TD)->isLittleEndian() ? LLVMLittleEndian : LLVMBigEndian;
}

unsigned LLVMPointerSize(LLVMTargetDataRef TD) {
  return unwrap(TD)->getPointerSize(0);
}

```
- **EN**: Implements logic around `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 75-82
```cpp
unsigned LLVMPointerSizeForAS(LLVMTargetDataRef TD, unsigned AS) {
  return unwrap(TD)->getPointerSize(AS);
}

LLVMTypeRef LLVMIntPtrType(LLVMTargetDataRef TD) {
  return wrap(unwrap(TD)->getIntPtrType(*unwrap(getGlobalContextForCAPI())));
}

```
- **EN**: Implements logic around `unwrap`, `wrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap`, `wrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 83-87
```cpp
LLVMTypeRef LLVMIntPtrTypeForAS(LLVMTargetDataRef TD, unsigned AS) {
  return wrap(
      unwrap(TD)->getIntPtrType(*unwrap(getGlobalContextForCAPI()), AS));
}

```
- **EN**: Implements logic around `wrap`, `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `wrap`, `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 88-95
```cpp
LLVMTypeRef LLVMIntPtrTypeInContext(LLVMContextRef C, LLVMTargetDataRef TD) {
  return wrap(unwrap(TD)->getIntPtrType(*unwrap(C)));
}

LLVMTypeRef LLVMIntPtrTypeForASInContext(LLVMContextRef C, LLVMTargetDataRef TD, unsigned AS) {
  return wrap(unwrap(TD)->getIntPtrType(*unwrap(C), AS));
}

```
- **EN**: Implements logic around `wrap`; this block returns target-specific results.
- **CN**: 围绕 `wrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 96-103
```cpp
unsigned long long LLVMSizeOfTypeInBits(LLVMTargetDataRef TD, LLVMTypeRef Ty) {
  return unwrap(TD)->getTypeSizeInBits(unwrap(Ty));
}

unsigned long long LLVMStoreSizeOfType(LLVMTargetDataRef TD, LLVMTypeRef Ty) {
  return unwrap(TD)->getTypeStoreSize(unwrap(Ty));
}

```
- **EN**: Implements logic around `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 104-111
```cpp
unsigned long long LLVMABISizeOfType(LLVMTargetDataRef TD, LLVMTypeRef Ty) {
  return unwrap(TD)->getTypeAllocSize(unwrap(Ty));
}

unsigned LLVMABIAlignmentOfType(LLVMTargetDataRef TD, LLVMTypeRef Ty) {
  return unwrap(TD)->getABITypeAlign(unwrap(Ty)).value();
}

```
- **EN**: Implements logic around `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 112-119
```cpp
unsigned LLVMCallFrameAlignmentOfType(LLVMTargetDataRef TD, LLVMTypeRef Ty) {
  return unwrap(TD)->getABITypeAlign(unwrap(Ty)).value();
}

unsigned LLVMPreferredAlignmentOfType(LLVMTargetDataRef TD, LLVMTypeRef Ty) {
  return unwrap(TD)->getPrefTypeAlign(unwrap(Ty)).value();
}

```
- **EN**: Implements logic around `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 120-126
```cpp
unsigned LLVMPreferredAlignmentOfGlobal(LLVMTargetDataRef TD,
                                        LLVMValueRef GlobalVar) {
  return unwrap(TD)
      ->getPreferredAlign(unwrap<GlobalVariable>(GlobalVar))
      .value();
}

```
- **EN**: Implements logic around `unwrap`, `getPreferredAlign`, `value`; this block returns target-specific results.
- **CN**: 围绕 `unwrap`, `getPreferredAlign`, `value` 实现具体逻辑；这一段返回目标相关结果。

### Lines 127-132
```cpp
unsigned LLVMElementAtOffset(LLVMTargetDataRef TD, LLVMTypeRef StructTy,
                             unsigned long long Offset) {
  StructType *STy = unwrap<StructType>(StructTy);
  return unwrap(TD)->getStructLayout(STy)->getElementContainingOffset(Offset);
}

```
- **EN**: Implements logic around `unwrap<StructType>`, `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap<StructType>`, `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 133-137
```cpp
unsigned long long LLVMOffsetOfElement(LLVMTargetDataRef TD, LLVMTypeRef StructTy,
                                       unsigned Element) {
  StructType *STy = unwrap<StructType>(StructTy);
  return unwrap(TD)->getStructLayout(STy)->getElementOffset(Element);
}
```
- **EN**: Implements logic around `unwrap<StructType>`, `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap<StructType>`, `unwrap` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Backend integration / 后端集成**:
  - **EN**: Connects this file to LLVM target infrastructure
  - **CN**: 把该文件接入 LLVM 目标基础设施

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm-c/Target.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/IR/DataLayout.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/LegacyPassManager.h`, `llvm/IR/Module.h`, `llvm/IR/Value.h`, `llvm/InitializePasses.h`, `cstring`
- **LLVM subsystems / LLVM 子系统**: IR
