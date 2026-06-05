# AMDGPUNextUseAnalysis.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUNextUseAnalysis.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUNextUseAnalysis in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUNextUseAnalysis 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, license, and overview
```cpp
//===---------------------- AMDGPUNextUseAnalysis.h  ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements Next Use Analysis.
//
// For each register it goes over all uses and returns the estimated distance of
// the nearest use. This will be used for selecting which registers to spill
// before register allocation.
//
// This is based on ideas from the paper:
// "Register Spilling and Live-Range Splitting for SSA-Form Programs"
// Matthias Braun and Sebastian Hack, CC'09
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUNEXTUSEANALYSIS_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUNEXTUSEANALYSIS_H

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 24-55: Header dependencies and setup
```cpp
#include "SIInstrInfo.h"
#include "SIRegisterInfo.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/LiveVariables.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/IR/PassManager.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/JSON.h"
#include <limits>
#include <optional>

namespace llvm {

class AMDGPUNextUseAnalysisImpl;

//==============================================================================
// NextUseDistance - Represents a distance in the next-use analysis. Currently
// wraps a 64-bit int with special encoding for loop depth and unreachable
// distances.
//==============================================================================
class NextUseDistance {
public:
  constexpr static NextUseDistance unreachable() {
    return NextUseDistance(std::numeric_limits<int64_t>::max());
  }

  constexpr static NextUseDistance fromSize(unsigned Size, unsigned Depth) {
    return NextUseDistance(Size).applyLoopWeight(Depth);
  }

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUNextUseAnalysisImpl`, `NextUseDistance`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUNextUseAnalysisImpl`, `NextUseDistance`。

### Lines 56-88: Defines NextUseDistance
```cpp
  constexpr NextUseDistance(unsigned V) : Value(V) {}
  constexpr NextUseDistance(int V) : Value(V) {}
  constexpr NextUseDistance(const NextUseDistance &B) : Value(B.Value) {}

  constexpr bool isUnreachable() const { return *this == unreachable(); }
  constexpr bool isReachable() const { return !isUnreachable(); }

  //----------------------------------------------------------------------------
  // Assignment
  //----------------------------------------------------------------------------
  constexpr NextUseDistance &operator=(const NextUseDistance &B) {
    Value = B.Value;
    return *this;
  }

  constexpr NextUseDistance &operator=(unsigned V) {
    Value = V;
    return *this;
  }

  constexpr NextUseDistance &operator=(int V) {
    Value = V;
    return *this;
  }

  //----------------------------------------------------------------------------
  // Arithmetic operators
  //----------------------------------------------------------------------------
  constexpr NextUseDistance &operator+=(const NextUseDistance &B) {
    Value += B.Value;
    return *this;
  }

```
**EN:** This section contains concrete logic for NextUseDistance. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 NextUseDistance 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 89-122: Defines applyLoopWeight
```cpp
  constexpr NextUseDistance &operator-=(const NextUseDistance &B) {
    Value -= B.Value;
    return *this;
  }

  constexpr NextUseDistance operator-() const {
    return NextUseDistance(-Value);
  }

  constexpr NextUseDistance applyLoopWeight() const {
    NextUseDistance W = fromLoopDepth(1);
    if (W.isUnreachable())
      return unreachable();
    constexpr int64_t MaxVal = std::numeric_limits<int64_t>::max();
    if (Value != 0 && W.Value > MaxVal / Value)
      return unreachable();
    return NextUseDistance(Value * W.Value);
  }

  //----------------------------------------------------------------------------
  // Comparison operators
  //----------------------------------------------------------------------------
  constexpr bool operator<(const NextUseDistance &B) const {
    return Value < B.Value;
  }

  constexpr bool operator>(const NextUseDistance &B) const {
    return Value > B.Value;
  }

  constexpr bool operator<=(const NextUseDistance &B) const {
    return Value <= B.Value;
  }

```
**EN:** This section contains concrete logic for applyLoopWeight. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 applyLoopWeight 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 123-152: Conditional logic and checks
```cpp
  constexpr bool operator>=(const NextUseDistance &B) const {
    return Value >= B.Value;
  }

  constexpr bool operator==(const NextUseDistance &B) const {
    return Value == B.Value;
  }

  constexpr bool operator!=(const NextUseDistance &B) const {
    return Value != B.Value;
  }

  //----------------------------------------------------------------------------
  // Debugging
  //----------------------------------------------------------------------------
  format_object<int64_t> fmt() const { return format("%ld", Value); }

  void print(raw_ostream &OS) const {
    if (isUnreachable())
      OS << "<unreachable>";
    else
      OS << fmt();
  }

  json::Value toJsonValue() const {
    if (isUnreachable())
      return "<unreachable>";
    return Value;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 153-179: Defines toString
```cpp
  std::string toString() const {
    std::string Str;
    llvm::raw_string_ostream OS(Str);
    print(OS);
    return OS.str();
  }

  constexpr int64_t getRawValue() const { return Value; }
  using RawValueType = int64_t;

private:
  friend class AMDGPUNextUseAnalysisImpl;
  int64_t Value;
  constexpr explicit NextUseDistance(int64_t V) : Value(V) {}

  constexpr static NextUseDistance fromLoopDepth(unsigned Depth) {
    const unsigned Shift = 7 * Depth;

    // Saturate?
    if (Shift >= 63)
      return unreachable();

    // This implementation is multiplicative (f(a+b) == f(a) * f(b)) which we
    // take advantage of below in applyLoopWeight(Depth).
    return NextUseDistance(int64_t(1) << Shift);
  }

```
**EN:** This section contains concrete logic for toString. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 toString 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 180-212: Conditional logic and checks
```cpp
  // Semantically: apply fromLoopDepth(1) Depth times (compositional).
  //
  // Optimized to take advantage of multiplicative implementation of
  // fromLoopDepth - a single multiply by fromLoopDepth(Depth) gives the same
  // result. If fromLoopDepth is changed to a non-multiplicative formula,
  // replace the body with something like:
  //
  //   NextUseDistance D = *this;
  //   for (unsigned I = 0; I < Depth; ++I) {
  //     D = D.applyLoopWeight();
  //     if (D.isUnreachable())
  //       return unreachable();
  //   }
  //   return D;
  //
  constexpr NextUseDistance applyLoopWeight(unsigned Depth) const {
    if (!Depth)
      return *this;
    NextUseDistance W = fromLoopDepth(Depth);
    if (W.isUnreachable())
      return unreachable();
    constexpr int64_t MaxVal = std::numeric_limits<int64_t>::max();
    if (Value != 0 && W.Value > MaxVal / Value)
      return unreachable();
    return NextUseDistance(Value * W.Value);
  }
};

constexpr inline NextUseDistance operator+(NextUseDistance A,
                                           const NextUseDistance &B) {
  return A += B;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 213-246: Declares class AMDGPUNextUseAnalysis
```cpp
constexpr inline NextUseDistance operator-(NextUseDistance A,
                                           const NextUseDistance &B) {
  return A -= B;
}

constexpr inline NextUseDistance min(NextUseDistance A, NextUseDistance B) {
  return A < B ? A : B;
}

constexpr inline NextUseDistance max(NextUseDistance A, NextUseDistance B) {
  return A > B ? A : B;
}

//==============================================================================
// AMDGPUNextUseAnalysis - Provides next-use distances for live registers or
// sub-registers at a given MachineInstruction suitable for making spilling
// decisions.
//==============================================================================
class AMDGPUNextUseAnalysis {
  friend class AMDGPUNextUseAnalysisLegacyPass;
  friend class AMDGPUNextUseAnalysisPrinterLegacyPass;
  friend class AMDGPUNextUseAnalysisPass;
  friend class AMDGPUNextUseAnalysisPrinterPass;

  std::unique_ptr<AMDGPUNextUseAnalysisImpl> Impl;

  AMDGPUNextUseAnalysis(const MachineFunction *, const MachineLoopInfo *);

public:
  AMDGPUNextUseAnalysis(AMDGPUNextUseAnalysis &&Other);
  ~AMDGPUNextUseAnalysis();

  AMDGPUNextUseAnalysis &operator=(AMDGPUNextUseAnalysis &&Other);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUNextUseAnalysis`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUNextUseAnalysis`。

### Lines 247-271: Declares struct Config
```cpp
  // Configuration flags for controlling the distance model. Defaults correspond
  // to the Graphics preset.
  struct Config {
    // Count PHI instructions as having non-zero cost (distance and block
    // size). When false, all PHIs share ID 0 and don't contribute to block
    // size.
    bool CountPhis = true;

    // Restrict inter-block distances to forward-reachable paths only.
    // When false, distances through back-edges are also considered.
    bool ForwardOnly = true;

    // Model PHI uses as belonging to their incoming edge's block, and apply
    // full loop-aware reachability filtering including intermediate-def
    // checks. When false, a simple same-block / forward-reachable check is
    // used.
    bool PreciseUseModeling = false;

    // Promote uses that are inside a loop not yet entered or inside a directly
    // nested inner loop to the end of that loop's preheader. This models the
    // assumption that a spilled value will be reloaded at the preheader rather
    // than at the actual use site. When false, direct shortest distance to the
    // use is used instead.
    bool PromoteToPreheader = false;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `Config`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`Config`。

### Lines 272-299: Result computation and returns
```cpp
    /// Named presets. See note in AMDGPUNextUseAnalysis.cpp associated with
    /// 'amdgpu-next-use-analysis-config' regarding the historical context for
    /// these.
    static Config Graphics() { return {}; }
    static Config Compute() {
      Config Cfg;
      Cfg.CountPhis = false;
      Cfg.ForwardOnly = false;
      Cfg.PreciseUseModeling = true;
      Cfg.PromoteToPreheader = true;
      return Cfg;
    }
  };

  Config getConfig() const;
  void setConfig(Config);

  void getReachableUses(Register LiveReg, LaneBitmask LaneMask,
                        const MachineInstr &MI,
                        SmallVector<const MachineOperand *> &Uses) const;

  /// \Returns the shortest next-use distance from \p CurMI for \p LiveReg.
  NextUseDistance
  getShortestDistance(Register LiveReg, const MachineInstr &CurMI,
                      const SmallVector<const MachineOperand *> &Uses,
                      const MachineOperand **ShortestUseOut = nullptr,
                      SmallVector<NextUseDistance> *Distances = nullptr) const;

```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

### Lines 300-333: Declares struct UseDistancePair
```cpp
  struct UseDistancePair {
    const MachineOperand *Use = nullptr;
    NextUseDistance Dist = 0;
    UseDistancePair() = default;
    UseDistancePair(const MachineOperand *Use, NextUseDistance Dist)
        : Use(Use), Dist(Dist) {}
  };

  void getNextUseDistances(const DenseMap<unsigned, LaneBitmask> &LiveRegs,
                           const MachineInstr &MI, UseDistancePair &Furthest,
                           UseDistancePair *FurthestSubreg = nullptr,
                           DenseMap<const MachineOperand *, UseDistancePair>
                               *RelevantUses = nullptr) const;
};

//==============================================================================
// AMDGPUNextUseAnalysisLegacyPass - Legacy and New pass wrapper around
// AMDGPUNextUseAnalysis
//==============================================================================
class AMDGPUNextUseAnalysisLegacyPass : public MachineFunctionPass {

public:
  static char ID;

  AMDGPUNextUseAnalysisLegacyPass();

  AMDGPUNextUseAnalysis &getNextUseAnalysis() { return *NUA; }
  const AMDGPUNextUseAnalysis &getNextUseAnalysis() const { return *NUA; }
  StringRef getPassName() const override;

protected:
  bool runOnMachineFunction(MachineFunction &) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `UseDistancePair`, `AMDGPUNextUseAnalysisLegacyPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`UseDistancePair`, `AMDGPUNextUseAnalysisLegacyPass`。

### Lines 334-365: Declares class AMDGPUNextUseAnalysisPass
```cpp
private:
  std::unique_ptr<AMDGPUNextUseAnalysis> NUA;
};

class AMDGPUNextUseAnalysisPass
    : public AnalysisInfoMixin<AMDGPUNextUseAnalysisPass> {
  friend AnalysisInfoMixin<AMDGPUNextUseAnalysisPass>;
  static AnalysisKey Key;

public:
  using Result = AMDGPUNextUseAnalysis;
  Result run(MachineFunction &MF, MachineFunctionAnalysisManager &MFAM);
};

//==============================================================================
// AMDGPUNextUseAnalysisPrinterLegacyPass - Legacy Pass for printing
// AMDGPUNextUseAnalysis results as JSON.
//==============================================================================
class AMDGPUNextUseAnalysisPrinterLegacyPass : public MachineFunctionPass {

public:
  static char ID;

  AMDGPUNextUseAnalysisPrinterLegacyPass();

  StringRef getPassName() const override;

protected:
  bool runOnMachineFunction(MachineFunction &) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUNextUseAnalysisPass`, `AMDGPUNextUseAnalysisPrinterLegacyPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUNextUseAnalysisPass`, `AMDGPUNextUseAnalysisPrinterLegacyPass`。

### Lines 366-377: Preprocessor guards and macros
```cpp
class AMDGPUNextUseAnalysisPrinterPass
    : public RequiredPassInfoMixin<AMDGPUNextUseAnalysisPrinterPass> {
  raw_ostream &OS;

public:
  explicit AMDGPUNextUseAnalysisPrinterPass(raw_ostream &OS) : OS(OS) {}
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

} // namespace llvm
#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUNEXTUSEANALYSIS_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPUNextUseAnalysisPrinterPass`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPUNextUseAnalysisPrinterPass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUNextUseAnalysisImpl`, `NextUseDistance`, `AMDGPUNextUseAnalysis`, `Config`, `UseDistancePair`, `AMDGPUNextUseAnalysisLegacyPass`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIInstrInfo.h"`
- `"SIRegisterInfo.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/LiveVariables.h"`
- `"llvm/CodeGen/MachineLoopInfo.h"`
- `"llvm/CodeGen/MachineRegisterInfo.h"`
- `"llvm/CodeGen/TargetRegisterInfo.h"`
- `"llvm/IR/PassManager.h"`
- `"llvm/Support/Format.h"`
- `"llvm/Support/JSON.h"`
- `<limits>`
- `<optional>`
