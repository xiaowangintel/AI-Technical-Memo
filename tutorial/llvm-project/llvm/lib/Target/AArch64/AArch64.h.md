# AArch64.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file contains the entry points for global functions defined in the LLVM AArch64 back-end. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Documented code section
```cpp
//==-- AArch64.h - Top-level interface for AArch64  --------------*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the entry points for global functions defined in the LLVM
// AArch64 back-end.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_AARCH64_H
#define LLVM_LIB_TARGET_AARCH64_AARCH64_H

#include "MCTargetDesc/AArch64MCTargetDesc.h"
#include "Utils/AArch64BaseInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionAnalysisManager.h"
#include "llvm/Pass.h"
#include "llvm/PassRegistry.h"
#include "llvm/Support/DataTypes.h"
#include "llvm/Target/TargetMachine.h"
#include <map>
#include <memory>
#include <unordered_map>

struct AArch64O0PreLegalizerCombinerImplRuleConfig;
struct AArch64PreLegalizerCombinerImplRuleConfig;
struct AArch64PostLegalizerLoweringImplRuleConfig;

namespace llvm {
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 34-69: Class AArch64RegisterBankInfo
```cpp

class AArch64RegisterBankInfo;
class AArch64Subtarget;
class AArch64TargetMachine;
enum class CodeGenOptLevel;
class FunctionPass;
class InstructionSelector;
class ModulePass;

FunctionPass *createAArch64DeadRegisterDefinitions();
FunctionPass *createAArch64RedundantCopyEliminationPass();
FunctionPass *createAArch64RedundantCondBranchPass();
FunctionPass *createAArch64CondBrTuning();
FunctionPass *createAArch64CompressJumpTablesPass();
FunctionPass *createAArch64ConditionalCompares();
FunctionPass *createAArch64AdvSIMDScalar();
FunctionPass *createAArch64ISelDag(AArch64TargetMachine &TM,
                                   CodeGenOptLevel OptLevel);
FunctionPass *createAArch64StorePairSuppressPass();
FunctionPass *createAArch64ExpandPseudoLegacyPass();
FunctionPass *createAArch64SLSHardeningLegacyPass();
FunctionPass *createAArch64SpeculationHardeningPass();
FunctionPass *createAArch64LoadStoreOptLegacyPass();
ModulePass *createAArch64LowerHomogeneousPrologEpilogPass();
FunctionPass *createAArch64SIMDInstrOptPass();
ModulePass *createAArch64PromoteConstantPass();
FunctionPass *createAArch64ConditionOptimizerLegacyPass();
FunctionPass *createAArch64A57FPLoadBalancingLegacyPass();
FunctionPass *createAArch64A53Fix835769LegacyPass();
FunctionPass *createFalkorHWPFFixPass();
FunctionPass *createFalkorMarkStridedAccessesPass();
FunctionPass *createAArch64PointerAuthPass();
FunctionPass *createAArch64BranchTargetsPass();
FunctionPass *createAArch64CodeLayoutOptPass();
FunctionPass *createAArch64MIPeepholeOptLegacyPass();
FunctionPass *createAArch64PostCoalescerPass();
```
**EN:** This block defines AArch64RegisterBankInfo, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64RegisterBankInfo，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 70-102: Core AArch64 backend logic
```cpp

FunctionPass *createAArch64CleanupLocalDynamicTLSPass();

FunctionPass *createAArch64CollectLOHPass();
FunctionPass *createSMEPeepholeOptPass();
FunctionPass *createMachineSMEABIPass(CodeGenOptLevel);
FunctionPass *createAArch64SRLTDefineSuperRegsPass();
ModulePass *createSVEIntrinsicOptsPass();
InstructionSelector *
createAArch64InstructionSelector(const AArch64TargetMachine &,
                                 const AArch64Subtarget &,
                                 const AArch64RegisterBankInfo &);
class AArch64O0PreLegalizerCombinerPass
    : public OptionalPassInfoMixin<AArch64O0PreLegalizerCombinerPass> {
  std::unique_ptr<AArch64O0PreLegalizerCombinerImplRuleConfig> RuleConfig;

public:
  AArch64O0PreLegalizerCombinerPass();
  AArch64O0PreLegalizerCombinerPass(AArch64O0PreLegalizerCombinerPass &&);
  ~AArch64O0PreLegalizerCombinerPass();

  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64PreLegalizerCombinerPass
    : public OptionalPassInfoMixin<AArch64PreLegalizerCombinerPass> {
  std::unique_ptr<AArch64PreLegalizerCombinerImplRuleConfig> RuleConfig;

public:
  AArch64PreLegalizerCombinerPass();
  AArch64PreLegalizerCombinerPass(AArch64PreLegalizerCombinerPass &&);
  ~AArch64PreLegalizerCombinerPass();
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 103-131: Class AArch64PostSelectOptimizePass
```cpp

  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64PostSelectOptimizePass
    : public OptionalPassInfoMixin<AArch64PostSelectOptimizePass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64PostLegalizerLoweringPass
    : public OptionalPassInfoMixin<AArch64PostLegalizerLoweringPass> {
  std::unique_ptr<AArch64PostLegalizerLoweringImplRuleConfig> RuleConfig;

public:
  AArch64PostLegalizerLoweringPass();
  AArch64PostLegalizerLoweringPass(AArch64PostLegalizerLoweringPass &&);
  ~AArch64PostLegalizerLoweringPass();

  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);

  MachineFunctionProperties getRequiredProperties() const {
    return MachineFunctionProperties().set(
        MachineFunctionProperties::Property::Legalized);
  }
};
```
**EN:** This block defines AArch64PostSelectOptimizePass, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64PostSelectOptimizePass，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 132-167: Core AArch64 backend logic
```cpp

FunctionPass *createAArch64O0PreLegalizerCombiner();
FunctionPass *createAArch64PreLegalizerCombiner();
FunctionPass *createAArch64PostLegalizerCombiner(bool IsOptNone);
FunctionPass *createAArch64PostLegalizerLowering();
FunctionPass *createAArch64PostSelectOptimize();
FunctionPass *createAArch64StackTaggingPass(bool IsOptNone);
FunctionPass *createAArch64StackTaggingPreRALegacyPass();
ModulePass *createAArch64Arm64ECCallLoweringPass();

void initializeAArch64A53Fix835769LegacyPass(PassRegistry &);
void initializeAArch64A57FPLoadBalancingLegacyPass(PassRegistry &);
void initializeAArch64AdvSIMDScalarLegacyPass(PassRegistry &);
void initializeAArch64AsmPrinterPass(PassRegistry &);
void initializeAArch64PointerAuthLegacyPass(PassRegistry &);
void initializeAArch64BranchTargetsLegacyPass(PassRegistry &);
void initializeAArch64CFIFixupPass(PassRegistry&);
void initializeAArch64CollectLOHLegacyPass(PassRegistry &);
void initializeAArch64CompressJumpTablesLegacyPass(PassRegistry &);
void initializeAArch64CondBrTuningPass(PassRegistry &);
void initializeAArch64ConditionOptimizerLegacyPass(PassRegistry &);
void initializeAArch64ConditionalComparesLegacyPass(PassRegistry &);
void initializeAArch64DAGToDAGISelLegacyPass(PassRegistry &);
void initializeAArch64DeadRegisterDefinitionsLegacyPass(PassRegistry &);
void initializeAArch64ExpandPseudoLegacyPass(PassRegistry &);
void initializeAArch64LoadStoreOptLegacyPass(PassRegistry &);
void initializeAArch64LowerHomogeneousPrologEpilogLegacyPass(PassRegistry &);
void initializeAArch64CodeLayoutOptPass(PassRegistry &);
void initializeAArch64MIPeepholeOptLegacyPass(PassRegistry &);
void initializeAArch64O0PreLegalizerCombinerLegacyPass(PassRegistry &);
void initializeAArch64PostCoalescerLegacyPass(PassRegistry &);
void initializeAArch64PostLegalizerCombinerPass(PassRegistry &);
void initializeAArch64PostSelectOptimizeLegacyPass(PassRegistry &);
void initializeAArch64PostLegalizerLoweringLegacyPass(PassRegistry &);
void initializeAArch64PreLegalizerCombinerLegacyPass(PassRegistry &);
void initializeAArch64PromoteConstantPass(PassRegistry&);
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 168-197: Core AArch64 backend logic
```cpp
void initializeAArch64RedundantCopyEliminationLegacyPass(PassRegistry &);
void initializeAArch64RedundantCondBranchLegacyPass(PassRegistry &);
void initializeAArch64SIMDInstrOptLegacyPass(PassRegistry &);
void initializeAArch64SLSHardeningLegacyPass(PassRegistry &);
void initializeAArch64SpeculationHardeningPass(PassRegistry &);
void initializeAArch64StackTaggingPass(PassRegistry &);
void initializeAArch64StackTaggingPreRALegacyPass(PassRegistry &);
void initializeAArch64StorePairSuppressPass(PassRegistry&);
void initializeFalkorHWPFFixPass(PassRegistry&);
void initializeFalkorMarkStridedAccessesLegacyPass(PassRegistry&);
void initializeLDTLSCleanupPass(PassRegistry &);
void initializeSMEPeepholeOptPass(PassRegistry &);
void initializeMachineSMEABIPass(PassRegistry &);
void initializeAArch64SRLTDefineSuperRegsPass(PassRegistry &);
void initializeSVEIntrinsicOptsPass(PassRegistry &);
void initializeAArch64Arm64ECCallLoweringPass(PassRegistry &);

class AArch64StackTaggingPreRAPass
    : public OptionalPassInfoMixin<AArch64StackTaggingPreRAPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64A57FPLoadBalancingPass
    : public OptionalPassInfoMixin<AArch64A57FPLoadBalancingPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 198-232: Class AArch64LoadStoreOptPass
```cpp

class AArch64LoadStoreOptPass
    : public OptionalPassInfoMixin<AArch64LoadStoreOptPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64A53Fix835769Pass
    : public OptionalPassInfoMixin<AArch64A53Fix835769Pass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64BranchTargetsPass
    : public OptionalPassInfoMixin<AArch64BranchTargetsPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64RedundantCondBranchPass
    : public OptionalPassInfoMixin<AArch64RedundantCondBranchPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64AdvSIMDScalarPass
    : public OptionalPassInfoMixin<AArch64AdvSIMDScalarPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};
```
**EN:** This block defines AArch64LoadStoreOptPass, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64LoadStoreOptPass，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 233-267: Class AArch64CollectLOHPass
```cpp

class AArch64CollectLOHPass
    : public OptionalPassInfoMixin<AArch64CollectLOHPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64CompressJumpTablesPass
    : public OptionalPassInfoMixin<AArch64CompressJumpTablesPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64DeadRegisterDefinitionsPass
    : public OptionalPassInfoMixin<AArch64DeadRegisterDefinitionsPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64ExpandPseudoPass
    : public OptionalPassInfoMixin<AArch64ExpandPseudoPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64MIPeepholeOptPass
    : public OptionalPassInfoMixin<AArch64MIPeepholeOptPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};
```
**EN:** This block defines AArch64CollectLOHPass, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64CollectLOHPass，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 268-297: Class AArch64ConditionOptimizerPass
```cpp

class AArch64ConditionOptimizerPass
    : public OptionalPassInfoMixin<AArch64ConditionOptimizerPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64SIMDInstrOptPass
    : public OptionalPassInfoMixin<AArch64SIMDInstrOptPass> {
  std::map<std::pair<unsigned, std::string>, bool> SIMDInstrTable;
  std::unordered_map<std::string, bool> InterlEarlyExit;

public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64PointerAuthPass
    : public OptionalPassInfoMixin<AArch64PointerAuthPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64SLSHardeningPass : public PassInfoMixin<AArch64SLSHardeningPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};
```
**EN:** This block defines AArch64ConditionOptimizerPass, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64ConditionOptimizerPass，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 298-328: Class AArch64PostCoalescerPass
```cpp

class AArch64PostCoalescerPass
    : public OptionalPassInfoMixin<AArch64PostCoalescerPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64RedundantCopyEliminationPass
    : public OptionalPassInfoMixin<AArch64RedundantCopyEliminationPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64ConditionalComparesPass
    : public OptionalPassInfoMixin<AArch64ConditionalComparesPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AArch64LowerHomogeneousPrologEpilogPass
    : public PassInfoMixin<AArch64LowerHomogeneousPrologEpilogPass> {
public:
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
};

} // end namespace llvm

#endif
```
**EN:** This block defines AArch64PostCoalescerPass, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64PostCoalescerPass，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: MCTargetDesc/AArch64MCTargetDesc.h, Utils/AArch64BaseInfo.h **CN:** 目标本地依赖：MCTargetDesc/AArch64MCTargetDesc.h, Utils/AArch64BaseInfo.h
- **EN:** Core LLVM interfaces: llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionAnalysisManager.h, llvm/Pass.h, llvm/PassRegistry.h, llvm/Support/DataTypes.h, llvm/Target/TargetMachine.h **CN:** 核心 LLVM 接口：llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionAnalysisManager.h, llvm/Pass.h, llvm/PassRegistry.h, llvm/Support/DataTypes.h, llvm/Target/TargetMachine.h
- **EN:** Standard-library support: map, memory, unordered_map **CN:** 标准库支持：map, memory, unordered_map
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
