# AMDGPUPreloadKernArgProlog.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUPreloadKernArgProlog.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUPreloadKernArgProlog for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUPreloadKernArgProlog 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: File banner, license, and overview
```cpp
//===- AMDGPUPreloadKernArgProlog.cpp - Preload KernArg Prolog ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This pass creates a backward compatibility layer for kernel argument
/// preloading in situations where code is compiled with kernel argument
/// preloading enabled but executed on hardware without firmware support for it.
///
/// To avoid recompilation, the pass inserts a block at the beginning of the
/// program that loads the kernel arguments into SGPRs using s_load
/// instructions. This sets up the registers exactly as they would be on systems
/// with compatible firmware.
///
/// This effectively creates two entry points for the kernel. Firmware that
/// supports the feature will automatically jump past the first 256 bytes of the
/// program, skipping the compatibility layer and directly starting execution on
/// the optimized code path.
///
/// This pass should be run as late as possible to prevent any optimizations
/// that might assume the padding is dead code or that the added prologue is a
/// true predecessor of the kernel entry block.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 29-62: Header dependencies and setup
```cpp
#include "AMDGPUPreloadKernArgProlog.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/TargetParser/TargetParser.h"

using namespace llvm;

#define DEBUG_TYPE "amdgpu-preload-kern-arg-prolog"

namespace {

// Used to build s_loads maping user SGPRs to kernel arguments
struct LoadConfig {
  unsigned Size;
  const TargetRegisterClass *RegClass;
  unsigned Opcode;
  Register LoadReg = Register();
};

class AMDGPUPreloadKernArgProlog {
public:
  AMDGPUPreloadKernArgProlog(MachineFunction &MF);

  bool run();

private:
  MachineFunction &MF;
  const GCNSubtarget &ST;
  const SIMachineFunctionInfo &MFI;
  const SIInstrInfo &TII;
  const TargetRegisterInfo &TRI;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `LoadConfig`, `AMDGPUPreloadKernArgProlog`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`LoadConfig`, `AMDGPUPreloadKernArgProlog`。

### Lines 63-96: Registers LLVM passes
```cpp
  // Create a new block before the entry point to the kernel. Firmware that
  // supports preloading kernel arguments will automatically jump past this
  // block to the alternative kernel entry point.
  void createBackCompatBlock(unsigned NumKernArgPreloadSGPRs);

  // Add instructions to load kernel arguments into SGPRs.
  void addBackCompatLoads(MachineBasicBlock *BackCompatMBB,
                          Register KernArgSegmentPtr,
                          unsigned NumKernArgPreloadSGPRs);
};

class AMDGPUPreloadKernArgPrologLegacy : public MachineFunctionPass {
public:
  static char ID;

  AMDGPUPreloadKernArgPrologLegacy() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "AMDGPU Preload Kernel Arguments Prolog";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
};

} // end anonymous namespace

char AMDGPUPreloadKernArgPrologLegacy::ID = 0;

INITIALIZE_PASS(AMDGPUPreloadKernArgPrologLegacy, DEBUG_TYPE,
                "AMDGPU Preload Kernel Arguments Prolog", false, false)

char &llvm::AMDGPUPreloadKernArgPrologLegacyID =
    AMDGPUPreloadKernArgPrologLegacy::ID;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `AMDGPUPreloadKernArgPrologLegacy`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`AMDGPUPreloadKernArgPrologLegacy`。

### Lines 97-128: Implements llvm::createAMDGPUPreloadKernArgPrologLegacyPass
```cpp
FunctionPass *llvm::createAMDGPUPreloadKernArgPrologLegacyPass() {
  return new AMDGPUPreloadKernArgPrologLegacy();
}

bool AMDGPUPreloadKernArgPrologLegacy::runOnMachineFunction(
    MachineFunction &MF) {
  return AMDGPUPreloadKernArgProlog(MF).run();
}

AMDGPUPreloadKernArgProlog::AMDGPUPreloadKernArgProlog(MachineFunction &MF)
    : MF(MF), ST(MF.getSubtarget<GCNSubtarget>()),
      MFI(*MF.getInfo<SIMachineFunctionInfo>()), TII(*ST.getInstrInfo()),
      TRI(*ST.getRegisterInfo()) {}

bool AMDGPUPreloadKernArgProlog::run() {
  if (!ST.needsKernArgPreloadProlog())
    return false;

  unsigned NumKernArgPreloadSGPRs = MFI.getNumKernargPreloadedSGPRs();
  if (!NumKernArgPreloadSGPRs)
    return false;

  createBackCompatBlock(NumKernArgPreloadSGPRs);
  return true;
}

void AMDGPUPreloadKernArgProlog::createBackCompatBlock(
    unsigned NumKernArgPreloadSGPRs) {
  auto KernelEntryMBB = MF.begin();
  MachineBasicBlock *BackCompatMBB = MF.CreateMachineBasicBlock();
  MF.insert(KernelEntryMBB, BackCompatMBB);

```
**EN:** This section contains concrete logic for llvm::createAMDGPUPreloadKernArgPrologLegacyPass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createAMDGPUPreloadKernArgPrologLegacyPass`, `AMDGPUPreloadKernArgPrologLegacy::runOnMachineFunction`, `AMDGPUPreloadKernArgProlog::AMDGPUPreloadKernArgProlog`.
**CN:** 本节包含与 llvm::createAMDGPUPreloadKernArgPrologLegacyPass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createAMDGPUPreloadKernArgPrologLegacyPass`, `AMDGPUPreloadKernArgPrologLegacy::runOnMachineFunction`, `AMDGPUPreloadKernArgProlog::AMDGPUPreloadKernArgProlog`。

### Lines 129-155: Defines assert
```cpp
  assert(MFI.getUserSGPRInfo().hasKernargSegmentPtr() &&
         "Kernel argument segment pointer register not set.");
  Register KernArgSegmentPtr = MFI.getArgInfo().KernargSegmentPtr.getRegister();
  BackCompatMBB->addLiveIn(KernArgSegmentPtr);

  // Load kernel arguments to SGPRs
  addBackCompatLoads(BackCompatMBB, KernArgSegmentPtr, NumKernArgPreloadSGPRs);

  // Wait for loads to complete
  AMDGPU::IsaVersion IV = AMDGPU::getIsaVersion(ST.getCPU());
  unsigned Waitcnt =
      AMDGPU::encodeWaitcnt(IV, getVmcntBitMask(IV), getExpcntBitMask(IV), 0);
  BuildMI(BackCompatMBB, DebugLoc(), TII.get(AMDGPU::S_WAITCNT))
      .addImm(Waitcnt);

  // Branch to kernel start
  BuildMI(BackCompatMBB, DebugLoc(), TII.get(AMDGPU::S_BRANCH))
      .addMBB(&*KernelEntryMBB);
  BackCompatMBB->addSuccessor(&*KernelEntryMBB);

  // Create a new basic block for padding to 256 bytes
  MachineBasicBlock *PadMBB = MF.CreateMachineBasicBlock();
  MF.insert(++BackCompatMBB->getIterator(), PadMBB);
  PadMBB->setAlignment(Align(256));
  PadMBB->addSuccessor(&*KernelEntryMBB);
}

```
**EN:** This section contains concrete logic for assert. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getIsaVersion`, `AMDGPU::encodeWaitcnt`.
**CN:** 本节包含与 assert 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getIsaVersion`, `AMDGPU::encodeWaitcnt`。

### Lines 156-181: Defines getLoadParameters
```cpp
/// Find the largest possible load size that fits with SGPR alignment
static LoadConfig getLoadParameters(const TargetRegisterInfo &TRI,
                                    Register KernArgPreloadSGPR,
                                    unsigned NumKernArgPreloadSGPRs) {
  static constexpr LoadConfig Configs[] = {
      {8, &AMDGPU::SReg_256RegClass, AMDGPU::S_LOAD_DWORDX8_IMM},
      {4, &AMDGPU::SReg_128RegClass, AMDGPU::S_LOAD_DWORDX4_IMM},
      {2, &AMDGPU::SReg_64RegClass, AMDGPU::S_LOAD_DWORDX2_IMM}};

  for (const auto &Config : Configs) {
    if (NumKernArgPreloadSGPRs >= Config.Size) {
      Register LoadReg = TRI.getMatchingSuperReg(KernArgPreloadSGPR,
                                                 AMDGPU::sub0, Config.RegClass);
      if (LoadReg) {
        LoadConfig C(Config);
        C.LoadReg = LoadReg;
        return C;
      }
    }
  }

  // Fallback to a single register
  return LoadConfig{1, &AMDGPU::SReg_32RegClass, AMDGPU::S_LOAD_DWORD_IMM,
                    KernArgPreloadSGPR};
}

```
**EN:** This section contains concrete logic for getLoadParameters. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getLoadParameters 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 182-211: Implements AMDGPUPreloadKernArgProlog::addBackCompatLoads
```cpp
void AMDGPUPreloadKernArgProlog::addBackCompatLoads(
    MachineBasicBlock *BackCompatMBB, Register KernArgSegmentPtr,
    unsigned NumKernArgPreloadSGPRs) {
  Register KernArgPreloadSGPR = MFI.getArgInfo().FirstKernArgPreloadReg;
  unsigned Offset = 0;
  // Fill all user SGPRs used for kernarg preloading with sequential data from
  // the kernarg segment
  while (NumKernArgPreloadSGPRs > 0) {
    LoadConfig Config =
        getLoadParameters(TRI, KernArgPreloadSGPR, NumKernArgPreloadSGPRs);

    BuildMI(BackCompatMBB, DebugLoc(), TII.get(Config.Opcode), Config.LoadReg)
        .addReg(KernArgSegmentPtr)
        .addImm(Offset)
        .addImm(0);

    Offset += 4 * Config.Size;
    KernArgPreloadSGPR = KernArgPreloadSGPR.asMCReg() + Config.Size;
    NumKernArgPreloadSGPRs -= Config.Size;
  }
}

PreservedAnalyses
AMDGPUPreloadKernArgPrologPass::run(MachineFunction &MF,
                                    MachineFunctionAnalysisManager &) {
  if (!AMDGPUPreloadKernArgProlog(MF).run())
    return PreservedAnalyses::all();

  return getMachineFunctionPassPreservedAnalyses();
}
```
**EN:** This section contains concrete logic for AMDGPUPreloadKernArgProlog::addBackCompatLoads. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPreloadKernArgProlog::addBackCompatLoads`, `AMDGPUPreloadKernArgPrologPass::run`, `PreservedAnalyses::all`.
**CN:** 本节包含与 AMDGPUPreloadKernArgProlog::addBackCompatLoads 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPreloadKernArgProlog::addBackCompatLoads`, `AMDGPUPreloadKernArgPrologPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `LoadConfig`, `AMDGPUPreloadKernArgProlog`, `AMDGPUPreloadKernArgPrologLegacy`, `llvm::createAMDGPUPreloadKernArgPrologLegacyPass`, `AMDGPUPreloadKernArgPrologLegacy::runOnMachineFunction`, `AMDGPUPreloadKernArgProlog::AMDGPUPreloadKernArgProlog`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; feature description / 特性描述
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUPreloadKernArgProlog.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/TargetParser/TargetParser.h"`
