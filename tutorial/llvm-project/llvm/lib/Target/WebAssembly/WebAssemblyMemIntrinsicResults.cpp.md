# WebAssemblyMemIntrinsicResults.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyMemIntrinsicResults.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements an optimization pass using memory intrinsic results.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyMemIntrinsicResults.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//== WebAssemblyMemIntrinsicResults.cpp - Optimize memory intrinsic results ==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-26

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements an optimization pass using memory intrinsic results.
///
/// Calls to memory intrinsics (memcpy, memmove, memset) return the destination
/// address. They are in the form of
///   %dst_new = call @memcpy %dst, %src, %len
/// where %dst and %dst_new registers contain the same value.
///
/// This is to enable an optimization wherein uses of the %dst register used in
/// the parameter can be replaced by uses of the %dst_new register used in the
/// result, making the %dst register more likely to be single-use, thus more
/// likely to be useful to register stackifying, and potentially also exposing
/// the call instruction itself to register stackifying. These both can reduce
/// local.get/local.set traffic.
///
/// The LLVM intrinsics for these return void so they can't use the returned
/// attribute and consequently aren't handled by the OptimizeReturned pass.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Notable symbols in this range include `intrinsics`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 该区间中较显眼的符号包括 `intrinsics`。

### Lines 27-33

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblySubtarget.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 34-42

```cpp
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 43-80

```cpp
#define DEBUG_TYPE "wasm-mem-intrinsic-results"

namespace {
class WebAssemblyMemIntrinsicResults final : public MachineFunctionPass {
public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyMemIntrinsicResults() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "WebAssembly Memory Intrinsic Results";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();
    AU.addPreserved<MachineBlockFrequencyInfoWrapperPass>();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.addPreserved<SlotIndexesWrapperPass>();
    AU.addPreserved<LiveIntervalsWrapperPass>();
    AU.addRequired<TargetLibraryInfoWrapperPass>();
    AU.addRequired<LibcallLoweringInfoWrapper>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

private:
  MachineDominatorTree *MDT;
  LiveIntervals *LIS;
  const TargetLibraryInfo *LibInfo;

  StringRef MemcpyName, MemmoveName, MemsetName;

  bool optimizeCall(MachineBasicBlock &MBB, MachineInstr &MI,
                    const MachineRegisterInfo &MRI) const;
};
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 81-118

```cpp
} // end anonymous namespace

char WebAssemblyMemIntrinsicResults::ID = 0;
INITIALIZE_PASS(WebAssemblyMemIntrinsicResults, DEBUG_TYPE,
                "Optimize memory intrinsic result values for WebAssembly",
                false, false)

FunctionPass *llvm::createWebAssemblyMemIntrinsicResults() {
  return new WebAssemblyMemIntrinsicResults();
}

// Replace uses of FromReg with ToReg if they are dominated by MI.
static bool replaceDominatedUses(MachineBasicBlock &MBB, MachineInstr &MI,
                                 unsigned FromReg, unsigned ToReg,
                                 const MachineRegisterInfo &MRI,
                                 MachineDominatorTree &MDT,
                                 LiveIntervals &LIS) {
  bool Changed = false;

  LiveInterval *FromLI = &LIS.getInterval(FromReg);
  LiveInterval *ToLI = &LIS.getInterval(ToReg);

  SlotIndex FromIdx = LIS.getInstructionIndex(MI).getRegSlot();
  VNInfo *FromVNI = FromLI->getVNInfoAt(FromIdx);

  SmallVector<SlotIndex, 4> Indices;

  for (MachineOperand &O :
       llvm::make_early_inc_range(MRI.use_nodbg_operands(FromReg))) {
    MachineInstr *Where = O.getParent();

    // Check that MI dominates the instruction in the normal way.
    if (&MI == Where || !MDT.dominates(&MI, Where))
      continue;

    // If this use gets a different value, skip it.
    SlotIndex WhereIdx = LIS.getInstructionIndex(*Where);
    VNInfo *WhereVNI = FromLI->getVNInfoAt(WhereIdx);
```
- **EN**: Implements helper routine(s) `INITIALIZE_PASS`, `createWebAssemblyMemIntrinsicResults`, `WebAssemblyMemIntrinsicResults` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `INITIALIZE_PASS`, `createWebAssemblyMemIntrinsicResults`, `WebAssemblyMemIntrinsicResults`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 119-156

```cpp
    if (WhereVNI && WhereVNI != FromVNI)
      continue;

    // Make sure ToReg isn't clobbered before it gets there.
    VNInfo *ToVNI = ToLI->getVNInfoAt(WhereIdx);
    if (ToVNI && ToVNI != FromVNI)
      continue;

    Changed = true;
    LLVM_DEBUG(dbgs() << "Setting operand " << O << " in " << *Where << " from "
                      << MI << "\n");
    O.setReg(ToReg);

    // If the store's def was previously dead, it is no longer.
    if (!O.isUndef()) {
      MI.getOperand(0).setIsDead(false);

      Indices.push_back(WhereIdx.getRegSlot());
    }
  }

  if (Changed) {
    // Extend ToReg's liveness.
    LIS.extendToIndices(*ToLI, Indices);

    // Shrink FromReg's liveness.
    LIS.shrinkToUses(FromLI);

    // If we replaced all dominated uses, FromReg is now killed at MI.
    if (!FromLI->liveAt(FromIdx.getDeadSlot()))
      MI.addRegisterKilled(FromReg, MBB.getParent()
                                        ->getSubtarget<WebAssemblySubtarget>()
                                        .getRegisterInfo());
  }

  return Changed;
}
```
- **EN**: Implements helper routine(s) `getVNInfoAt`, `dbgs`, `setReg` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getVNInfoAt`, `dbgs`, `setReg`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 157-194

```cpp
bool WebAssemblyMemIntrinsicResults::optimizeCall(
    MachineBasicBlock &MBB, MachineInstr &MI,
    const MachineRegisterInfo &MRI) const {
  MachineOperand &Op1 = MI.getOperand(1);
  if (!Op1.isSymbol())
    return false;

  StringRef Name(Op1.getSymbolName());

  // TODO: Could generalize by parsing to LibcallImpl and checking signature
  // attributes
  bool CallReturnsInput =
      Name == MemcpyName || Name == MemmoveName || Name == MemsetName;
  if (!CallReturnsInput)
    return false;

  LibFunc Func;
  if (!LibInfo->getLibFunc(Name, Func))
    return false;

  Register FromReg = MI.getOperand(2).getReg();
  Register ToReg = MI.getOperand(0).getReg();
  if (MRI.getRegClass(FromReg) != MRI.getRegClass(ToReg))
    report_fatal_error("Memory Intrinsic results: call to builtin function "
                       "with wrong signature, from/to mismatch");
  return replaceDominatedUses(MBB, MI, FromReg, ToReg, MRI, *MDT, *LIS);
}

bool WebAssemblyMemIntrinsicResults::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG({
    dbgs() << "********** Memory Intrinsic Results **********\n"
           << "********** Function: " << MF.getName() << '\n';
  });

  MachineRegisterInfo &MRI = MF.getRegInfo();
  LIS = &getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  const WebAssemblySubtarget &Subtarget =
```
- **EN**: Implements helper routine(s) `optimizeCall`, `getOperand`, `isSymbol` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `optimizeCall`, `getOperand`, `isSymbol`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 195-230

```cpp
      MF.getSubtarget<WebAssemblySubtarget>();
  LibInfo =
      &getAnalysis<TargetLibraryInfoWrapperPass>().getTLI(MF.getFunction());
  const LibcallLoweringInfo &Libcalls =
      getAnalysis<LibcallLoweringInfoWrapper>().getLibcallLowering(
          *MF.getFunction().getParent(), Subtarget);

  MemcpyName = RTLIB::RuntimeLibcallsInfo::getLibcallImplName(
      Libcalls.getLibcallImpl(RTLIB::MEMCPY));
  MemmoveName = RTLIB::RuntimeLibcallsInfo::getLibcallImplName(
      Libcalls.getLibcallImpl(RTLIB::MEMMOVE));
  MemsetName = RTLIB::RuntimeLibcallsInfo::getLibcallImplName(
      Libcalls.getLibcallImpl(RTLIB::MEMSET));

  bool Changed = false;

  // We don't preserve SSA form.
  MRI.leaveSSA();

  assert(MRI.tracksLiveness() &&
         "MemIntrinsicResults expects liveness tracking");

  for (auto &MBB : MF) {
    LLVM_DEBUG(dbgs() << "Basic Block: " << MBB.getName() << '\n');
    for (auto &MI : MBB)
      switch (MI.getOpcode()) {
      default:
        break;
      case WebAssembly::CALL:
        Changed |= optimizeCall(MBB, MI, MRI);
        break;
      }
  }

  return Changed;
}
```
- **EN**: Implements helper routine(s) `getTLI`, `getFunction`, `getLibcallLowering` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getTLI`, `getFunction`, `getLibcallLowering`。 这里重点涉及寄存器分配与寄存器类约束。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `llvm/Analysis/TargetLibraryInfo.h`
- `llvm/CodeGen/LiveIntervals.h`
- `llvm/CodeGen/MachineBlockFrequencyInfo.h`
- `llvm/CodeGen/MachineDominators.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
