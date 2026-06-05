# AArch64ExpandImm.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64ExpandImm.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file contains the AArch64 immediate expansion stuff. / 该文件实现 AArch64 后端中的立即数物化。
## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Documented code section
```cpp
//===- AArch64ExpandImm.h - AArch64 Immediate Expansion ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the AArch64 immediate expansion stuff.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64EXPANDIMM_H
#define LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64EXPANDIMM_H

#include "llvm/ADT/SmallVector.h"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 17-33: Namespace llvm
```cpp

namespace llvm {

namespace AArch64_IMM {

struct ImmInsnModel {
  unsigned Opcode;
  uint64_t Op1;
  uint64_t Op2;
};

struct AddrInsnModel {
  unsigned Opcode;
};

void expandMOVImm(uint64_t Imm, unsigned BitSize,
                  SmallVectorImpl<ImmInsnModel> &Insn);
```
**EN:** This block scopes the following declarations to the intended namespace boundary.  
**CN:** 该代码块将后续声明限定在目标命名空间边界内。
### Lines 34-42: Core AArch64 backend logic
```cpp

void expandMOVAddr(unsigned Opcode, unsigned TargetFlags, bool IsTargetMachO,
                   SmallVectorImpl<AddrInsnModel> &Insn);

} // end namespace AArch64_IMM

} // end namespace llvm

#endif
```
**EN:** This block continues the file's main immediate materialization logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的立即数物化主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Materializing immediates within AArch64 encoding limits **CN:** 在 AArch64 编码限制内物化立即数
## Dependencies / 依赖关系
- **EN:** Core LLVM interfaces: llvm/ADT/SmallVector.h **CN:** 核心 LLVM 接口：llvm/ADT/SmallVector.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for immediate materialization. **CN:** 与周边负责立即数物化的 AArch64 后端组件紧密协作。
