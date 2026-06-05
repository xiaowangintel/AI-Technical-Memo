# X86InstrFMA3Info.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrFMA3Info.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements instruction definitions and metadata for the core X86 backend. / 实现X86 后端核心中的指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86InstrFMA3Info.cpp - X86 FMA3 Instruction Information -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the implementation of the classes providing information
// about existing X86 FMA3 opcodes, classifying and grouping them.
//
//===----------------------------------------------------------------------===//

#include "X86InstrFMA3Info.h"
#include "X86InstrInfo.h"
#include <atomic>
#include <cassert>
#include <cstdint>

using namespace llvm;

#define FMA3GROUP(Name, Suf, Attrs) \
  { { X86::Name##132##Suf, X86::Name##213##Suf, X86::Name##231##Suf }, Attrs },

#define FMA3GROUP_MASKED(Name, Suf, Attrs) \
  FMA3GROUP(Name, Suf, Attrs) \
  FMA3GROUP(Name, Suf##k, Attrs | X86InstrFMA3Group::KMergeMasked) \
  FMA3GROUP(Name, Suf##kz, Attrs | X86InstrFMA3Group::KZeroMasked)

#define FMA3GROUP_MASKED_INT(Name, Suf, Attrs) \
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  FMA3GROUP(Name, Suf##_Int, Attrs) \
  FMA3GROUP(Name, Suf##k_Int, Attrs | X86InstrFMA3Group::KMergeMasked) \
  FMA3GROUP(Name, Suf##kz_Int, Attrs | X86InstrFMA3Group::KZeroMasked)

#define FMA3GROUP_PACKED_WIDTHS_Z(Name, Suf, Attrs) \
  FMA3GROUP_MASKED(Name, Suf##Z128m, Attrs) \
  FMA3GROUP_MASKED(Name, Suf##Z128r, Attrs) \
  FMA3GROUP_MASKED(Name, Suf##Z256m, Attrs) \
  FMA3GROUP_MASKED(Name, Suf##Z256r, Attrs) \
  FMA3GROUP_MASKED(Name, Suf##Zm, Attrs) \
  FMA3GROUP_MASKED(Name, Suf##Zr, Attrs) \

#define FMA3GROUP_PACKED_WIDTHS_ALL(Name, Suf, Attrs) \
  FMA3GROUP(Name, Suf##Ym, Attrs) \
  FMA3GROUP(Name, Suf##Yr, Attrs) \
  FMA3GROUP_PACKED_WIDTHS_Z(Name, Suf, Attrs) \
  FMA3GROUP(Name, Suf##m, Attrs) \
  FMA3GROUP(Name, Suf##r, Attrs)

#define FMA3GROUP_PACKED_DHS(Name, Attrs) \
  FMA3GROUP_PACKED_WIDTHS_ALL(Name, PD, Attrs) \
  FMA3GROUP_PACKED_WIDTHS_Z(Name, PH, Attrs) \
  FMA3GROUP_PACKED_WIDTHS_ALL(Name, PS, Attrs)

#define FMA3GROUP_PACKED_BF16(Name, Attrs)                                     \
  FMA3GROUP_PACKED_WIDTHS_Z(Name, BF16, Attrs)

#define FMA3GROUP_SCALAR_WIDTHS_Z(Name, Suf, Attrs) \
  FMA3GROUP(Name, Suf##Zm, Attrs) \
  FMA3GROUP_MASKED_INT(Name, Suf##Zm, Attrs | X86InstrFMA3Group::Intrinsic) \
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  FMA3GROUP(Name, Suf##Zr, Attrs) \
  FMA3GROUP_MASKED_INT(Name, Suf##Zr, Attrs | X86InstrFMA3Group::Intrinsic) \

#define FMA3GROUP_SCALAR_WIDTHS_ALL(Name, Suf, Attrs) \
  FMA3GROUP_SCALAR_WIDTHS_Z(Name, Suf, Attrs) \
  FMA3GROUP(Name, Suf##m, Attrs) \
  FMA3GROUP(Name, Suf##m_Int, Attrs | X86InstrFMA3Group::Intrinsic) \
  FMA3GROUP(Name, Suf##r, Attrs) \
  FMA3GROUP(Name, Suf##r_Int, Attrs | X86InstrFMA3Group::Intrinsic)

#define FMA3GROUP_SCALAR(Name, Attrs) \
  FMA3GROUP_SCALAR_WIDTHS_ALL(Name, SD, Attrs) \
  FMA3GROUP_SCALAR_WIDTHS_Z(Name, SH, Attrs) \
  FMA3GROUP_SCALAR_WIDTHS_ALL(Name, SS, Attrs)

#define FMA3GROUP_FULL(Name, Attrs) \
  FMA3GROUP_PACKED_BF16(Name, Attrs) \
  FMA3GROUP_PACKED_DHS(Name, Attrs) \
  FMA3GROUP_SCALAR(Name, Attrs)

static const X86InstrFMA3Group Groups[] = {
  FMA3GROUP_FULL(VFMADD, 0)
  FMA3GROUP_PACKED_DHS(VFMADDSUB, 0)
  FMA3GROUP_FULL(VFMSUB, 0)
  FMA3GROUP_PACKED_DHS(VFMSUBADD, 0)
  FMA3GROUP_FULL(VFNMADD, 0)
  FMA3GROUP_FULL(VFNMSUB, 0)
};

#define FMA3GROUP_PACKED_AVX512_WIDTHS(Name, Type, Suf, Attrs) \
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  FMA3GROUP_MASKED(Name, Type##Z128##Suf, Attrs) \
  FMA3GROUP_MASKED(Name, Type##Z256##Suf, Attrs) \
  FMA3GROUP_MASKED(Name, Type##Z##Suf, Attrs)

#define FMA3GROUP_PACKED_AVX512_ALL(Name, Suf, Attrs)                          \
  FMA3GROUP_PACKED_AVX512_WIDTHS(Name, BF16, Suf, Attrs)                       \
  FMA3GROUP_PACKED_AVX512_WIDTHS(Name, PD, Suf, Attrs)                         \
  FMA3GROUP_PACKED_AVX512_WIDTHS(Name, PH, Suf, Attrs)                         \
  FMA3GROUP_PACKED_AVX512_WIDTHS(Name, PS, Suf, Attrs)

#define FMA3GROUP_PACKED_AVX512_DHS(Name, Suf, Attrs) \
  FMA3GROUP_PACKED_AVX512_WIDTHS(Name, PD, Suf, Attrs) \
  FMA3GROUP_PACKED_AVX512_WIDTHS(Name, PH, Suf, Attrs) \
  FMA3GROUP_PACKED_AVX512_WIDTHS(Name, PS, Suf, Attrs)

#define FMA3GROUP_PACKED_AVX512_ROUND(Name, Suf, Attrs) \
  FMA3GROUP_MASKED(Name, PDZ##Suf, Attrs) \
  FMA3GROUP_MASKED(Name, PHZ##Suf, Attrs) \
  FMA3GROUP_MASKED(Name, PSZ##Suf, Attrs)

#define FMA3GROUP_SCALAR_AVX512_ROUND(Name, Suf, Attrs) \
  FMA3GROUP(Name, SDZ##Suf, Attrs) \
  FMA3GROUP_MASKED_INT(Name, SDZ##Suf, Attrs) \
  FMA3GROUP(Name, SHZ##Suf, Attrs) \
  FMA3GROUP_MASKED_INT(Name, SHZ##Suf, Attrs) \
  FMA3GROUP(Name, SSZ##Suf, Attrs) \
  FMA3GROUP_MASKED_INT(Name, SSZ##Suf, Attrs)

static const X86InstrFMA3Group BroadcastGroups[] = {
  FMA3GROUP_PACKED_AVX512_ALL(VFMADD, mb, 0)
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  FMA3GROUP_PACKED_AVX512_DHS(VFMADDSUB, mb, 0)
  FMA3GROUP_PACKED_AVX512_ALL(VFMSUB, mb, 0)
  FMA3GROUP_PACKED_AVX512_DHS(VFMSUBADD, mb, 0)
  FMA3GROUP_PACKED_AVX512_ALL(VFNMADD, mb, 0)
  FMA3GROUP_PACKED_AVX512_ALL(VFNMSUB, mb, 0)
};

static const X86InstrFMA3Group RoundGroups[] = {
  FMA3GROUP_PACKED_AVX512_ROUND(VFMADD, rb, 0)
  FMA3GROUP_SCALAR_AVX512_ROUND(VFMADD, rb, X86InstrFMA3Group::Intrinsic)
  FMA3GROUP_PACKED_AVX512_ROUND(VFMADDSUB, rb, 0)
  FMA3GROUP_PACKED_AVX512_ROUND(VFMSUB, rb, 0)
  FMA3GROUP_SCALAR_AVX512_ROUND(VFMSUB, rb, X86InstrFMA3Group::Intrinsic)
  FMA3GROUP_PACKED_AVX512_ROUND(VFMSUBADD, rb, 0)
  FMA3GROUP_PACKED_AVX512_ROUND(VFNMADD, rb, 0)
  FMA3GROUP_SCALAR_AVX512_ROUND(VFNMADD, rb, X86InstrFMA3Group::Intrinsic)
  FMA3GROUP_PACKED_AVX512_ROUND(VFNMSUB, rb, 0)
  FMA3GROUP_SCALAR_AVX512_ROUND(VFNMSUB, rb, X86InstrFMA3Group::Intrinsic)
};

static void verifyTables() {
#ifndef NDEBUG
  static std::atomic<bool> TableChecked(false);
  if (!TableChecked.load(std::memory_order_relaxed)) {
    assert(llvm::is_sorted(Groups) && llvm::is_sorted(RoundGroups) &&
           llvm::is_sorted(BroadcastGroups) && "FMA3 tables not sorted!");
    TableChecked.store(true, std::memory_order_relaxed);
  }
#endif
}
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp

/// Returns a reference to a group of FMA3 opcodes to where the given
/// \p Opcode is included. If the given \p Opcode is not recognized as FMA3
/// and not included into any FMA3 group, then nullptr is returned.
const X86InstrFMA3Group *llvm::getFMA3Group(unsigned Opcode, uint64_t TSFlags) {

  // FMA3 instructions have a well defined encoding pattern we can exploit.
  uint8_t BaseOpcode = X86II::getBaseOpcodeFor(TSFlags);
  bool IsFMA3Opcode = ((BaseOpcode >= 0x96 && BaseOpcode <= 0x9F) ||
                       (BaseOpcode >= 0xA6 && BaseOpcode <= 0xAF) ||
                       (BaseOpcode >= 0xB6 && BaseOpcode <= 0xBF));
  bool IsFMA3Encoding = ((TSFlags & X86II::EncodingMask) == X86II::VEX &&
                         (TSFlags & X86II::OpMapMask) == X86II::T8) ||
                        ((TSFlags & X86II::EncodingMask) == X86II::EVEX &&
                         ((TSFlags & X86II::OpMapMask) == X86II::T8 ||
                          (TSFlags & X86II::OpMapMask) == X86II::T_MAP6));
  bool IsFMA3Prefix = (TSFlags & X86II::OpPrefixMask) == X86II::PD ||
                      (TSFlags & X86II::OpPrefixMask) == 0; // X86II::PS
  if (!IsFMA3Opcode || !IsFMA3Encoding || !IsFMA3Prefix)
    return nullptr;

  verifyTables();

  ArrayRef<X86InstrFMA3Group> Table;
  if (TSFlags & X86II::EVEX_RC)
    Table = ArrayRef(RoundGroups);
  else if (TSFlags & X86II::EVEX_B)
    Table = ArrayRef(BroadcastGroups);
  else
    Table = ArrayRef(Groups);
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-193: Comments and explanatory notes / 注释与说明性文字
```cpp

  // FMA 132 instructions have an opcode of 0x96-0x9F
  // FMA 213 instructions have an opcode of 0xA6-0xAF
  // FMA 231 instructions have an opcode of 0xB6-0xBF
  unsigned FormIndex = ((BaseOpcode - 0x90) >> 4) & 0x3;

  auto I = partition_point(Table, [=](const X86InstrFMA3Group &Group) {
    return Group.Opcodes[FormIndex] < Opcode;
  });
  assert(I != Table.end() && I->Opcodes[FormIndex] == Opcode &&
         "Couldn't find FMA3 opcode!");
  return I;
}
```
**EN:** This section implements comments and explanatory notes for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的注释与说明性文字。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86InstrFMA3Info.h, X86InstrInfo.h, atomic, cassert, cstdint. / 直接包含：X86InstrFMA3Info.h, X86InstrInfo.h, atomic, cassert, cstdint。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
