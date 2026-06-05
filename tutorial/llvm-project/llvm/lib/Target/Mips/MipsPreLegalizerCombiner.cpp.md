# MipsPreLegalizerCombiner.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsPreLegalizerCombiner.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsPreLegalizerCombiner` for the Mips backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 Mips 后端中的 `MipsPreLegalizerCombiner`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//=== lib/CodeGen/GlobalISel/MipsPreLegalizerCombiner.cpp --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass does combining of machine instructions at the generic MI level,
// before the legalizer.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-21
```cpp
#include "MipsLegalizerInfo.h"
#include "MipsTargetMachine.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/TargetPassConfig.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 23-23
```cpp
#define DEBUG_TYPE "mips-prelegalizer-combiner"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 25-25
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 27-34
```cpp
namespace {
struct MipsPreLegalizerCombinerInfo : public CombinerInfo {
public:
  MipsPreLegalizerCombinerInfo()
      : CombinerInfo(/*AllowIllegalOps*/ true, /*ShouldLegalizeIllegal*/ false,
                     /*LegalizerInfo*/ nullptr, /*EnableOpt*/ false,
                     /*EnableOptSize*/ false, /*EnableMinSize*/ false) {}
};
```
- EN: Declares `MipsPreLegalizerCombinerInfo`, packaging target-specific state and APIs around `MipsPreLegalizerCombiner`.
- CN: 这里声明 `MipsPreLegalizerCombinerInfo`，把与 `MipsPreLegalizerCombiner` 相关的目标特定状态和 API 组织在一起。

### Lines 36-39
```cpp
class MipsPreLegalizerCombinerImpl : public Combiner {
protected:
  const MipsSubtarget &STI;
  const CombinerHelper Helper;
```
- EN: Declares `MipsPreLegalizerCombinerImpl`, packaging target-specific state and APIs around `MipsPreLegalizerCombiner`.
- CN: 这里声明 `MipsPreLegalizerCombinerImpl`，把与 `MipsPreLegalizerCombiner` 相关的目标特定状态和 API 组织在一起。

### Lines 41-48
```cpp
public:
  MipsPreLegalizerCombinerImpl(MachineFunction &MF, CombinerInfo &CInfo,
                               GISelValueTracking &VT, GISelCSEInfo *CSEInfo,
                               const MipsSubtarget &STI,
                               MachineDominatorTree *MDT,
                               const LegalizerInfo *LI)
      : Combiner(MF, CInfo, &VT, CSEInfo), STI(STI),
        Helper(Observer, B, /*IsPreLegalize*/ true, &VT, MDT, LI) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 50-50
```cpp
  static const char *getName() { return "MipsPreLegalizerCombiner"; }
```
- EN: Implements `getName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 52-54
```cpp
  void setupGeneratedPerFunctionState(MachineFunction &MF) override {
    // TODO: TableGen-erate this class' impl.
  }
```
- EN: Implements `setupGeneratedPerFunctionState`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setupGeneratedPerFunctionState`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 56-70
```cpp
  bool tryCombineAll(MachineInstr &MI) const override {
    switch (MI.getOpcode()) {
    default:
      return false;
    case TargetOpcode::G_MEMCPY_INLINE:
      return Helper.tryEmitMemcpyInline(MI);
    case TargetOpcode::G_LOAD:
    case TargetOpcode::G_SEXTLOAD:
    case TargetOpcode::G_ZEXTLOAD: {
      // Don't attempt to combine non power of 2 loads or unaligned loads when
      // subtarget doesn't support them.
      auto MMO = *MI.memoperands_begin();
      const MipsSubtarget &STI = MI.getMF()->getSubtarget<MipsSubtarget>();
      if (!MMO->getSize().hasValue() ||
          !isPowerOf2_64(MMO->getSize().getValue()))
```
- EN: Implements `tryCombineAll`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `tryCombineAll`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-74
```cpp
        return false;
      bool isUnaligned = MMO->getAlign() < MMO->getSize().getValue();
      if (!STI.systemSupportsUnalignedAccess() && isUnaligned)
        return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 76-78
```cpp
      return Helper.tryCombineExtendingLoads(MI);
    }
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 80-82
```cpp
    return false;
  }
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 84-85
```cpp
// Pass boilerplate
// ================
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 87-89
```cpp
class MipsPreLegalizerCombiner : public MachineFunctionPass {
public:
  static char ID;
```
- EN: Declares `MipsPreLegalizerCombiner`, packaging target-specific state and APIs around `MipsPreLegalizerCombiner`.
- CN: 这里声明 `MipsPreLegalizerCombiner`，把与 `MipsPreLegalizerCombiner` 相关的目标特定状态和 API 组织在一起。

### Lines 91-91
```cpp
  MipsPreLegalizerCombiner();
```
- EN: Declares `MipsPreLegalizerCombiner`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MipsPreLegalizerCombiner`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-93
```cpp
  StringRef getPassName() const override { return "MipsPreLegalizerCombiner"; }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 95-95
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 97-99
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override;
};
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 101-107
```cpp
void MipsPreLegalizerCombiner::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<GISelValueTrackingAnalysisLegacy>();
  AU.addPreserved<GISelValueTrackingAnalysisLegacy>();
  AU.setPreservesCFG();
  getSelectionDAGFallbackAnalysisUsage(AU);
  MachineFunctionPass::getAnalysisUsage(AU);
}
```
- EN: Implements `MipsPreLegalizerCombiner::getAnalysisUsage`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPreLegalizerCombiner::getAnalysisUsage`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 109-110
```cpp
MipsPreLegalizerCombiner::MipsPreLegalizerCombiner()
    : MachineFunctionPass(ID) {}
```
- EN: Implements `MipsPreLegalizerCombiner::MipsPreLegalizerCombiner`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPreLegalizerCombiner::MipsPreLegalizerCombiner`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 112-114
```cpp
bool MipsPreLegalizerCombiner::runOnMachineFunction(MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
```
- EN: Implements `MipsPreLegalizerCombiner::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPreLegalizerCombiner::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 116-118
```cpp
  const MipsSubtarget &ST = MF.getSubtarget<MipsSubtarget>();
  const MipsLegalizerInfo *LI =
      static_cast<const MipsLegalizerInfo *>(ST.getLegalizerInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 120-126
```cpp
  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);
  MipsPreLegalizerCombinerInfo PCInfo;
  MipsPreLegalizerCombinerImpl Impl(MF, PCInfo, *VT, /*CSEInfo*/ nullptr, ST,
                                    /*MDT*/ nullptr, LI);
  return Impl.combineMachineInstrs();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 128-135
```cpp
char MipsPreLegalizerCombiner::ID = 0;
INITIALIZE_PASS_BEGIN(MipsPreLegalizerCombiner, DEBUG_TYPE,
                      "Combine Mips machine instrs before legalization", false,
                      false)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_END(MipsPreLegalizerCombiner, DEBUG_TYPE,
                    "Combine Mips machine instrs before legalization", false,
                    false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 137-139
```cpp
FunctionPass *llvm::createMipsPreLegalizeCombiner() {
  return new MipsPreLegalizerCombiner();
}
```
- EN: Implements `llvm::createMipsPreLegalizeCombiner`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMipsPreLegalizeCombiner`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsLegalizerInfo.h`, `MipsTargetMachine.h`.
  - CN: 后端本地头文件：`MipsLegalizerInfo.h`, `MipsTargetMachine.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/GlobalISel/Combiner.h`, `llvm/CodeGen/GlobalISel/CombinerHelper.h`, `llvm/CodeGen/GlobalISel/CombinerInfo.h`, `llvm/CodeGen/GlobalISel/GISelValueTracking.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/TargetPassConfig.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/GlobalISel/Combiner.h`, `llvm/CodeGen/GlobalISel/CombinerHelper.h`, `llvm/CodeGen/GlobalISel/CombinerInfo.h`, `llvm/CodeGen/GlobalISel/GISelValueTracking.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/TargetPassConfig.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
