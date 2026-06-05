# AMDGPUWaitcntUtils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUWaitcntUtils.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUWaitcntUtils in the LLVM backend utilities. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM 后端工具中 AMDGPUWaitcntUtils 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: File banner, includes, and setup
```cpp
//===- AMDGPUWaitcntUtils.h -------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUWAITCNTUTILS_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUWAITCNTUTILS_H

#include "llvm/ADT/Sequence.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/TargetParser.h"

namespace llvm {

namespace AMDGPU {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 22-41: Type declarations and aliases
```cpp
enum InstCounterType {
  LOAD_CNT = 0, // VMcnt prior to gfx12.
  DS_CNT,       // LKGMcnt prior to gfx12.
  EXP_CNT,      //
  STORE_CNT,    // VScnt in gfx10/gfx11.
  NUM_NORMAL_INST_CNTS,
  SAMPLE_CNT = NUM_NORMAL_INST_CNTS, // gfx12+ only.
  BVH_CNT,                           // gfx12+ only.
  KM_CNT,                            // gfx12+ only.
  X_CNT,                             // gfx1250.
  ASYNC_CNT,                         // gfx1250.
  NUM_EXTENDED_INST_CNTS,
  VA_VDST = NUM_EXTENDED_INST_CNTS, // gfx12+ expert mode only.
  VM_VSRC,                          // gfx12+ expert mode only.
  NUM_EXPERT_INST_CNTS,
  NUM_INST_CNTS = NUM_EXPERT_INST_CNTS
};

StringLiteral getInstCounterName(InstCounterType T);

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `InstCounterType`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`InstCounterType`。

### Lines 42-62: Declares class Waitcnt
```cpp
// Return an iterator over all counters between LOAD_CNT (the first counter)
// and \c MaxCounter (exclusive, default value yields an enumeration over
// all counters).
iota_range<InstCounterType>
inst_counter_types(InstCounterType MaxCounter = NUM_INST_CNTS);

} // namespace AMDGPU

template <> struct enum_iteration_traits<AMDGPU::InstCounterType> {
  static constexpr bool is_iterable = true;
};

namespace AMDGPU {

/// Represents the counter values to wait for in an s_waitcnt instruction.
///
/// Large values (including the maximum possible integer) can be used to
/// represent "don't care" waits.
class Waitcnt {
  std::array<unsigned, NUM_INST_CNTS> Cnt;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `Waitcnt`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`Waitcnt`。

### Lines 63-76: Defines get
```cpp
public:
  unsigned get(InstCounterType T) const { return Cnt[T]; }
  void set(InstCounterType T, unsigned Val) { Cnt[T] = Val; }

  Waitcnt() { fill(Cnt, ~0u); }
  // Pre-gfx12 constructor.
  Waitcnt(unsigned VmCnt, unsigned ExpCnt, unsigned LgkmCnt, unsigned VsCnt)
      : Waitcnt() {
    Cnt[LOAD_CNT] = VmCnt;
    Cnt[EXP_CNT] = ExpCnt;
    Cnt[DS_CNT] = LgkmCnt;
    Cnt[STORE_CNT] = VsCnt;
  }

```
**EN:** This section contains concrete logic for get. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 get 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 77-98: Defines Waitcnt
```cpp
  // gfx12+ constructor.
  Waitcnt(unsigned LoadCnt, unsigned ExpCnt, unsigned DsCnt, unsigned StoreCnt,
          unsigned SampleCnt, unsigned BvhCnt, unsigned KmCnt, unsigned XCnt,
          unsigned AsyncCnt, unsigned VaVdst, unsigned VmVsrc)
      : Waitcnt() {
    Cnt[LOAD_CNT] = LoadCnt;
    Cnt[DS_CNT] = DsCnt;
    Cnt[EXP_CNT] = ExpCnt;
    Cnt[STORE_CNT] = StoreCnt;
    Cnt[SAMPLE_CNT] = SampleCnt;
    Cnt[BVH_CNT] = BvhCnt;
    Cnt[KM_CNT] = KmCnt;
    Cnt[X_CNT] = XCnt;
    Cnt[ASYNC_CNT] = AsyncCnt;
    Cnt[VA_VDST] = VaVdst;
    Cnt[VM_VSRC] = VmVsrc;
  }

  bool hasWait() const {
    return any_of(Cnt, [](unsigned Val) { return Val != ~0u; });
  }

```
**EN:** This section contains concrete logic for Waitcnt. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 Waitcnt 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 99-114: Defines hasWaitExceptStoreCnt
```cpp
  bool hasWaitExceptStoreCnt() const {
    for (InstCounterType T : inst_counter_types()) {
      if (T == STORE_CNT)
        continue;
      if (Cnt[T] != ~0u)
        return true;
    }
    return false;
  }

  bool hasWaitStoreCnt() const { return Cnt[STORE_CNT] != ~0u; }

  bool hasWaitDepctr() const {
    return Cnt[VA_VDST] != ~0u || Cnt[VM_VSRC] != ~0u;
  }

```
**EN:** This section contains concrete logic for hasWaitExceptStoreCnt. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 hasWaitExceptStoreCnt 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 115-136: Preprocessor guards and macros
```cpp
  Waitcnt combined(const Waitcnt &Other) const {
    // Does the right thing provided self and Other are either both pre-gfx12
    // or both gfx12+.
    Waitcnt Wait;
    for (InstCounterType T : inst_counter_types())
      Wait.Cnt[T] = std::min(Cnt[T], Other.Cnt[T]);
    return Wait;
  }

  void print(raw_ostream &OS) const {
    ListSeparator LS;
    for (InstCounterType T : inst_counter_types())
      OS << LS << getInstCounterName(T) << ": " << Cnt[T];
    if (LS.unused())
      OS << "none";
    OS << '\n';
  }

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  LLVM_DUMP_METHOD void dump() const;
#endif

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `std::min`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`std::min`。

### Lines 137-157: Declares decodeWaitcnt
```cpp
  friend raw_ostream &operator<<(raw_ostream &OS, const AMDGPU::Waitcnt &Wait) {
    Wait.print(OS);
    return OS;
  }
};

Waitcnt decodeWaitcnt(const IsaVersion &Version, unsigned Encoded);

unsigned encodeWaitcnt(const IsaVersion &Version, const Waitcnt &Decoded);

// The following are only meaningful on targets that support
// S_WAIT_LOADCNT_DSCNT and S_WAIT_STORECNT_DSCNT.

/// \returns Decoded Waitcnt structure from given \p LoadcntDscnt for given
/// isa \p Version.
Waitcnt decodeLoadcntDscnt(const IsaVersion &Version, unsigned LoadcntDscnt);

/// \returns Decoded Waitcnt structure from given \p StorecntDscnt for given
/// isa \p Version.
Waitcnt decodeStorecntDscnt(const IsaVersion &Version, unsigned StorecntDscnt);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 158-172: Preprocessor guards and macros
```cpp
/// \returns \p Loadcnt and \p Dscnt components of \p Decoded  encoded as an
/// immediate that can be used with S_WAIT_LOADCNT_DSCNT for given isa
/// \p Version.
unsigned encodeLoadcntDscnt(const IsaVersion &Version, const Waitcnt &Decoded);

/// \returns \p Storecnt and \p Dscnt components of \p Decoded  encoded as an
/// immediate that can be used with S_WAIT_STORECNT_DSCNT for given isa
/// \p Version.
unsigned encodeStorecntDscnt(const IsaVersion &Version, const Waitcnt &Decoded);

} // namespace AMDGPU

} // namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUWAITCNTUTILS_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `InstCounterType`, `Waitcnt`, `std::min`
- **Main themes / 核心主题**: instruction semantics / 指令语义
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/ADT/Sequence.h"`
- `"llvm/ADT/StringExtras.h"`
- `"llvm/Support/Debug.h"`
- `"llvm/Support/raw_ostream.h"`
- `"llvm/TargetParser/TargetParser.h"`
