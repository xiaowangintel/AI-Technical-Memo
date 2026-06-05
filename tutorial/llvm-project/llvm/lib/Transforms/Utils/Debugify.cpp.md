# Debugify.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/Debugify.cpp` | `llvm/lib/Transforms/Utils/Debugify.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements check debug info preservation in optimizations within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 Debugify 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-74

```cpp
//===- Debugify.cpp - Check debug info preservation in optimizations ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file In the `synthetic` mode, the `-debugify` attaches synthetic debug info
/// to everything. It can be used to create targeted tests for debug info
/// preservation. In addition, when using the `original` mode, it can check
/// original debug info preservation. The `synthetic` mode is default one.
///
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/Debugify.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/IR/DIBuilder.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    "debugify-level", cl::desc("Kind of debug info to add"),
    cl::values(clEnumValN(Level::Locations, "locations", "Locations only"),
               clEnumValN(Level::LocationsAndVariables, "location+variables",
                          "Locations and Variables")),
    cl::init(Level::LocationsAndVariables));

raw_ostream &dbg() { return Quiet ? nulls() : errs(); }

#if LLVM_ENABLE_DEBUGLOC_TRACKING_ORIGIN
// These maps refer to addresses in the current LLVM process, so we can reuse
// them everywhere - therefore, we store them at file scope.
static SymbolizedAddressMap SymbolizedAddrs;
static AddressSet UnsymbolizedAddrs;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 75-162

```cpp
std::string symbolizeStackTrace(const Instruction *I) {
  // We flush the set of unsymbolized addresses at the latest possible moment,
  // i.e. now.
  if (!UnsymbolizedAddrs.empty()) {
    sys::symbolizeAddresses(UnsymbolizedAddrs, SymbolizedAddrs);
    UnsymbolizedAddrs.clear();
  }
  const DbgLocOrigin::StackTracesTy &OriginStackTraces =
      I->getDebugLoc().getOriginStackTraces();
  std::string Result;
  raw_string_ostream OS(Result);
  for (size_t TraceIdx = 0; TraceIdx < OriginStackTraces.size(); ++TraceIdx) {
    if (TraceIdx != 0)
      OS << "========================================\n";
    auto &[Depth, StackTrace] = OriginStackTraces[TraceIdx];
    unsigned VirtualFrameNo = 0;
    for (int Frame = 0; Frame < Depth; ++Frame) {
      assert(SymbolizedAddrs.contains(StackTrace[Frame]) &&
             "Expected each address to have been symbolized.");
      for (std::string &SymbolizedFrame : SymbolizedAddrs[StackTrace[Frame]]) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  auto *Int32Ty = Type::getInt32Ty(Ctx);

  // Get a DIType which corresponds to Ty.
  DenseMap<uint64_t, DIType *> TypeCache;
  auto getCachedDIType = [&](Type *Ty) -> DIType * {
    uint64_t Size = getAllocSizeInBits(M, Ty);
    DIType *&DTy = TypeCache[Size];
    if (!DTy) {
      std::string Name = "ty" + utostr(Size);
      DTy = DIB.createBasicType(Name, Size, dwarf::DW_ATE_unsigned);
    }
    return DTy;
  };

```
- EN: Core entities appearing here include symbolizeStackTrace, collectStackAddresses, getAllocSizeInBits, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 symbolizeStackTrace, collectStackAddresses, getAllocSizeInBits，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 163-242

```cpp
  unsigned NextLine = 1;
  unsigned NextVar = 1;
  auto File = DIB.createFile(M.getName(), "/");
  auto CU = DIB.createCompileUnit(DISourceLanguageName(dwarf::DW_LANG_C), File,
                                  "debugify", /*isOptimized=*/true, "", 0);

  // Visit each instruction.
  for (Function &F : Functions) {
    if (isFunctionSkipped(F))
      continue;

    bool InsertedDbgVal = false;
    auto SPType = DIB.createSubroutineType(DIB.getOrCreateTypeArray({}));
    DISubprogram::DISPFlags SPFlags =
        DISubprogram::SPFlagDefinition | DISubprogram::SPFlagOptimized;
    if (F.hasPrivateLinkage() || F.hasInternalLinkage())
      SPFlags |= DISubprogram::SPFlagLocalToUnit;
    auto SP = DIB.createFunction(CU, F.getName(), F.getName(), File, NextLine,
                                 SPType, NextLine, DINode::FlagZero, SPFlags,
                                 nullptr, nullptr, nullptr, nullptr, "",
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // Insert after existing debug values to preserve order.
      InsertPt.setHeadBit(false);

      // Attach debug values.
      for (Instruction *I = &*BB.begin(); I != LastInst; I = I->getNextNode()) {
        // Skip void-valued instructions.
        if (I->getType()->isVoidTy())
          continue;

        // Phis and EH pads must be grouped at the beginning of the block.
        // Only advance the insertion point when we finish visiting these.
        if (!isa<PHINode>(I) && !I->isEHPad())
          InsertPt = std::next(I->getIterator());

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 243-320

```cpp
        insertDbgVal(*I, InsertPt);
        InsertedDbgVal = true;
      }
    }
    // Make sure we emit at least one dbg.value, otherwise MachineDebugify may
    // not have anything to work with as it goes about inserting DBG_VALUEs.
    // (It's common for MIR tests to be written containing skeletal IR with
    // empty functions -- we're still interested in debugifying the MIR within
    // those tests, and this helps with that.)
    if (DebugifyLevel == Level::LocationsAndVariables && !InsertedDbgVal) {
      auto *Term = findTerminatingInstruction(F.getEntryBlock());
      insertDbgVal(*Term, Term->getIterator());
    }
    if (ApplyToMF)
      ApplyToMF(DIB, F);
  }
  DIB.finalize();

  // Track the number of distinct lines and variables.
  NamedMDNode *NMD = M.getOrInsertNamedMetadata("llvm.debugify");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  bool Changed = false;

  // Remove the llvm.debugify and llvm.mir.debugify module-level named metadata.
  NamedMDNode *DebugifyMD = M.getNamedMetadata("llvm.debugify");
  if (DebugifyMD) {
    M.eraseNamedMetadata(DebugifyMD);
    Changed = true;
  }

  if (auto *MIRDebugifyMD = M.getNamedMetadata("llvm.mir.debugify")) {
    M.eraseNamedMetadata(MIRDebugifyMD);
    Changed = true;
  }

```
- EN: Core entities appearing here include stripDebugifyMetadata, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 stripDebugifyMetadata，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 321-402

```cpp
  // Strip out all debug intrinsics and supporting metadata (subprograms, types,
  // variables, etc).
  Changed |= StripDebugInfo(M);

  // Strip out the dead dbg.value prototype.
  Function *DbgValF = M.getFunction("llvm.dbg.value");
  if (DbgValF) {
    assert(DbgValF->isDeclaration() && DbgValF->use_empty() &&
           "Not all debug info stripped?");
    DbgValF->eraseFromParent();
    Changed = true;
  }

  // Strip out the module-level Debug Info Version metadata.
  // FIXME: There must be an easier way to remove an operand from a NamedMDNode.
  NamedMDNode *NMD = M.getModuleFlagsMetadata();
  if (!NMD)
    return Changed;
  SmallVector<MDNode *, 4> Flags(NMD->operands());
  NMD->clearOperands();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (++FunctionsCnt >= DebugifyFunctionsLimit)
      break;
    // Collect the DISubprogram.
    auto *SP = F.getSubprogram();
    DebugInfoBeforePass.DIFunctions.insert({&F, SP});
    if (SP) {
      LLVM_DEBUG(dbgs() << "  Collecting subprogram: " << *SP << '\n');
      for (const DINode *DN : SP->getRetainedNodes()) {
        if (const auto *DV = dyn_cast<DILocalVariable>(DN)) {
          DebugInfoBeforePass.DIVariables[DV] = 0;
        }
      }
    }

```
- EN: Core entities appearing here include hasLoc, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 hasLoc，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 403-479

```cpp
    for (BasicBlock &BB : F) {
      // Collect debug locations (!dbg) and debug variable intrinsics.
      for (Instruction &I : BB) {
        // Skip PHIs.
        if (isa<PHINode>(I))
          continue;

        // Cllect dbg.values and dbg.declare.
        if (DebugifyLevel > Level::Locations) {
          auto HandleDbgVariable = [&](DbgVariableRecord *DbgVar) {
            if (!SP)
              return;
            // Skip inlined variables.
            if (DbgVar->getDebugLoc().getInlinedAt())
              return;
            // Skip undef values.
            if (DbgVar->isKillLocation())
              return;

            auto *Var = DbgVar->getVariable();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        continue;
      // If the function had the SP attached before the pass, consider it as
      // a debug info bug.
      if (ShouldWriteIntoJSON)
        Bugs.push_back(llvm::json::Object({{"metadata", "DISubprogram"},
                                           {"name", F.first->getName()},
                                           {"action", "drop"}}));
      else
        dbg() << "ERROR: " << NameOfWrappedPass << " dropped DISubprogram of "
              << F.first->getName() << " from " << FileNameFromCU << '\n';
      Preserved = false;
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 480-560

```cpp
  return Preserved;
}

// This checks the preservation of the original debug info attached to
// instructions.
static bool checkInstructions(const DebugInstMap &DILocsBefore,
                              const DebugInstMap &DILocsAfter,
                              const WeakInstValueMap &InstToDelete,
                              StringRef NameOfWrappedPass,
                              StringRef FileNameFromCU,
                              bool ShouldWriteIntoJSON,
                              llvm::json::Array &Bugs) {
  bool Preserved = true;
  for (const auto &L : DILocsAfter) {
    if (L.second)
      continue;
    auto Instr = L.first;

    // In order to avoid pointer reuse/recycling, skip the values that might
    // have been deleted during a pass.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return Preserved;
}

// This checks the preservation of original debug variable intrinsics.
static bool checkVars(const DebugVarMap &DIVarsBefore,
                      const DebugVarMap &DIVarsAfter,
                      StringRef NameOfWrappedPass, StringRef FileNameFromCU,
                      bool ShouldWriteIntoJSON, llvm::json::Array &Bugs) {
  bool Preserved = true;
  for (const auto &V : DIVarsBefore) {
    auto VarIt = DIVarsAfter.find(V.first);
    if (VarIt == DIVarsAfter.end())
      continue;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 561-638

```cpp
    unsigned NumOfDbgValsAfter = VarIt->second;

    if (V.second > NumOfDbgValsAfter) {
      if (ShouldWriteIntoJSON)
        Bugs.push_back(llvm::json::Object(
            {{"metadata", "dbg-var-intrinsic"},
             {"name", V.first->getName()},
             {"fn-name", V.first->getScope()->getSubprogram()->getName()},
             {"action", "drop"}}));
      else
        dbg() << "WARNING: " << NameOfWrappedPass
              << " drops dbg.value()/dbg.declare() for " << V.first->getName()
              << " from "
              << "function " << V.first->getScope()->getSubprogram()->getName()
              << " (file " << FileNameFromCU << ")\n";
      Preserved = false;
    }
  }

  return Preserved;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // Visit each instruction.
  for (Function &F : Functions) {
    if (isFunctionSkipped(F))
      continue;

    // Don't process functions without DI collected before the Pass.
    if (!DebugInfoBeforePass.DIFunctions.count(&F))
      continue;
    // TODO: Collect metadata other than DISubprograms.
    // Collect the DISubprogram.
    auto *SP = F.getSubprogram();
    DebugInfoAfterPass.DIFunctions.insert({&F, SP});

```
- EN: This region continues the Debugify implementation with local helper logic centered on NumOfDbgValsAfter, VarIt, ShouldWriteIntoJSON, Bugs.
- CN: 这一段延续了 Debugify 的主体实现，围绕 NumOfDbgValsAfter, VarIt, ShouldWriteIntoJSON, Bugs 等局部辅助逻辑展开。

### Lines 639-717

```cpp
    if (SP) {
      LLVM_DEBUG(dbgs() << "  Collecting subprogram: " << *SP << '\n');
      for (const DINode *DN : SP->getRetainedNodes()) {
        if (const auto *DV = dyn_cast<DILocalVariable>(DN)) {
          DebugInfoAfterPass.DIVariables[DV] = 0;
        }
      }
    }

    for (BasicBlock &BB : F) {
      // Collect debug locations (!dbg) and debug variable intrinsics.
      for (Instruction &I : BB) {
        // Skip PHIs.
        if (isa<PHINode>(I))
          continue;

        // Collect dbg.values and dbg.declares.
        if (DebugifyLevel > Level::Locations) {
          auto HandleDbgVariable = [&](DbgVariableRecord *DbgVar) {
            if (!SP)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                     FileNameFromCU, ShouldWriteIntoJSON, Bugs);
  bool ResultForInsts = checkInstructions(
      DILocsBefore, DILocsAfter, InstToDelete, NameOfWrappedPass,
      FileNameFromCU, ShouldWriteIntoJSON, Bugs);

#if LLVM_ENABLE_DEBUGLOC_TRACKING_COVERAGE
  // If we are tracking DebugLoc coverage, replace each empty DebugLoc with an
  // annotated location now so that it does not show up in future passes even if
  // it is propagated to other instructions.
  for (auto &L : DILocsAfter)
    if (!L.second)
      const_cast<Instruction *>(L.first)->setDebugLoc(DebugLoc::getUnknown());
#endif

```
- EN: This region continues the Debugify implementation with local helper logic centered on LLVM_DEBUG, Collecting, DINode, DILocalVariable.
- CN: 这一段延续了 Debugify 的主体实现，围绕 LLVM_DEBUG, Collecting, DINode, DILocalVariable 等局部辅助逻辑展开。

### Lines 718-795

```cpp
  bool ResultForVars = checkVars(DIVarsBefore, DIVarsAfter, NameOfWrappedPass,
                                 FileNameFromCU, ShouldWriteIntoJSON, Bugs);

  bool Result = ResultForFunc && ResultForInsts && ResultForVars;

  StringRef ResultBanner = NameOfWrappedPass != "" ? NameOfWrappedPass : Banner;
  if (ShouldWriteIntoJSON && !Bugs.empty())
    writeJSON(OrigDIVerifyBugsReportFilePath, FileNameFromCU, NameOfWrappedPass,
              Bugs);

  if (Result)
    dbg() << ResultBanner << ": PASS\n";
  else
    dbg() << ResultBanner << ": FAIL\n";

  // In the case of the `debugify-each`, no need to go over all the instructions
  // again in the collectDebugInfoMetadata(), since as an input we can use
  // the debugging information from the previous pass.
  DebugInfoBeforePass = DebugInfoAfterPass;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return HasBadSize;
}

bool checkDebugifyMetadata(Module &M,
                           iterator_range<Module::iterator> Functions,
                           StringRef NameOfWrappedPass, StringRef Banner,
                           bool Strip, DebugifyStatsMap *StatsMap) {
  // Skip modules without debugify metadata.
  NamedMDNode *NMD = M.getNamedMetadata("llvm.debugify");
  if (!NMD) {
    dbg() << Banner << ": Skipping module without debugify metadata\n";
    return false;
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include diagnoseMisSizedDbgValue, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 diagnoseMisSizedDbgValue，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 796-874

```cpp
  auto getDebugifyOperand = [&](unsigned Idx) -> unsigned {
    return mdconst::extract<ConstantInt>(NMD->getOperand(Idx)->getOperand(0))
        ->getZExtValue();
  };
  assert(NMD->getNumOperands() == 2 &&
         "llvm.debugify should have exactly 2 operands!");
  unsigned OriginalNumLines = getDebugifyOperand(0);
  unsigned OriginalNumVars = getDebugifyOperand(1);
  bool HasErrors = false;

  // Track debug info loss statistics if able.
  DebugifyStatistics *Stats = nullptr;
  if (StatsMap && !NameOfWrappedPass.empty())
    Stats = &StatsMap->operator[](NameOfWrappedPass);

  BitVector MissingLines{OriginalNumLines, true};
  BitVector MissingVars{OriginalNumVars, true};
  for (Function &F : Functions) {
    if (isFunctionSkipped(F))
      continue;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Stats->NumDbgValuesExpected += OriginalNumVars;
    Stats->NumDbgValuesMissing += MissingVars.count();
  }

  dbg() << Banner;
  if (!NameOfWrappedPass.empty())
    dbg() << " [" << NameOfWrappedPass << "]";
  dbg() << ": " << (HasErrors ? "FAIL" : "PASS") << '\n';

  // Strip debugify metadata if required.
  bool Ret = false;
  if (Strip)
    Ret = stripDebugifyMetadata(M);

```
- EN: This region continues the Debugify implementation with local helper logic centered on Idx, ConstantInt, NMD, OriginalNumLines.
- CN: 这一段延续了 Debugify 的主体实现，围绕 Idx, ConstantInt, NMD, OriginalNumLines 等局部辅助逻辑展开。

### Lines 875-960

```cpp
  return Ret;
}

/// ModulePass for attaching synthetic debug info to everything, used with the
/// legacy module pass manager.
struct DebugifyModulePass : public ModulePass {
  bool runOnModule(Module &M) override {
    bool Result =
        applyDebugify(M, Mode, DebugInfoBeforePass, NameOfWrappedPass);
    return Result;
  }

  DebugifyModulePass(enum DebugifyMode Mode = DebugifyMode::SyntheticDebugInfo,
                     StringRef NameOfWrappedPass = "",
                     DebugInfoPerPass *DebugInfoBeforePass = nullptr)
      : ModulePass(ID), NameOfWrappedPass(NameOfWrappedPass),
        DebugInfoBeforePass(DebugInfoBeforePass), Mode(Mode) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return Result;
  }

  CheckDebugifyModulePass(
      bool Strip = false, StringRef NameOfWrappedPass = "",
      DebugifyStatsMap *StatsMap = nullptr,
      enum DebugifyMode Mode = DebugifyMode::SyntheticDebugInfo,
      DebugInfoPerPass *DebugInfoBeforePass = nullptr,
      StringRef OrigDIVerifyBugsReportFilePath = "")
      : ModulePass(ID), NameOfWrappedPass(NameOfWrappedPass),
        OrigDIVerifyBugsReportFilePath(OrigDIVerifyBugsReportFilePath),
        StatsMap(StatsMap), DebugInfoBeforePass(DebugInfoBeforePass), Mode(Mode),
        Strip(Strip) {}

```
- EN: Core entities appearing here include DebugifyModulePass, DebugifyMode, DebugifyFunctionPass, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 DebugifyModulePass, DebugifyMode, DebugifyFunctionPass，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 961-1038

```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
  }

  static char ID; // Pass identification.

private:
  StringRef NameOfWrappedPass;
  StringRef OrigDIVerifyBugsReportFilePath;
  DebugifyStatsMap *StatsMap;
  DebugInfoPerPass *DebugInfoBeforePass;
  enum DebugifyMode Mode;
  bool Strip;
};

/// FunctionPass for checking debug info inserted by -debugify-function, used
/// with the legacy module pass manager.
struct CheckDebugifyFunctionPass : public FunctionPass {
  bool runOnFunction(Function &F) override {
    Module &M = *F.getParent();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  std::error_code EC;
  raw_fd_ostream OS{Path, EC};
  if (EC) {
    errs() << "Could not open file: " << EC.message() << ", " << Path << '\n';
    return;
  }

  OS << "Pass Name" << ',' << "# of missing debug values" << ','
     << "# of missing locations" << ',' << "Missing/Expected value ratio" << ','
     << "Missing/Expected location ratio" << '\n';
  for (const auto &Entry : Map) {
    StringRef Pass = Entry.first;
    DebugifyStatistics Stats = Entry.second;

```
- EN: Core entities appearing here include DebugifyMode, CheckDebugifyFunctionPass, exportDebugifyStats, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 DebugifyMode, CheckDebugifyFunctionPass, exportDebugifyStats，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1039-1122

```cpp
    OS << Pass << ',' << Stats.NumDbgValuesMissing << ','
       << Stats.NumDbgLocsMissing << ',' << Stats.getMissingValueRatio() << ','
       << Stats.getEmptyLocationRatio() << '\n';
  }
}

ModulePass *createDebugifyModulePass(enum DebugifyMode Mode,
                                     llvm::StringRef NameOfWrappedPass,
                                     DebugInfoPerPass *DebugInfoBeforePass) {
  if (Mode == DebugifyMode::SyntheticDebugInfo)
    return new DebugifyModulePass();
  assert(Mode == DebugifyMode::OriginalDebugInfo && "Must be original mode");
  return new DebugifyModulePass(Mode, NameOfWrappedPass, DebugInfoBeforePass);
}

FunctionPass *
createDebugifyFunctionPass(enum DebugifyMode Mode,
                           llvm::StringRef NameOfWrappedPass,
                           DebugInfoPerPass *DebugInfoBeforePass) {
  if (Mode == DebugifyMode::SyntheticDebugInfo)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                       OrigDIVerifyBugsReportFilePath);
}

PreservedAnalyses NewPMCheckDebugifyPass::run(Module &M,
                                              ModuleAnalysisManager &) {
  if (Mode == DebugifyMode::SyntheticDebugInfo)
    checkDebugifyMetadata(M, M.functions(), NameOfWrappedPass,
                                   "CheckModuleDebugify", Strip, StatsMap);
  else
    checkDebugInfoMetadata(
      M, M.functions(), *DebugInfoBeforePass,
      "CheckModuleDebugify (original debuginfo)", NameOfWrappedPass,
      OrigDIVerifyBugsReportFilePath);

```
- EN: Core entities appearing here include run, DebugifyMode, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 run, DebugifyMode，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1123-1199

```cpp
  return PreservedAnalyses::all();
}

static bool isIgnoredPass(StringRef PassID) {
  return isSpecialPass(PassID, {"PassManager", "PassAdaptor",
                                "AnalysisManagerProxy", "PrintFunctionPass",
                                "PrintModulePass", "BitcodeWriterPass",
                                "ThinLTOBitcodeWriterPass", "VerifierPass"});
}

void DebugifyEachInstrumentation::registerCallbacks(
    PassInstrumentationCallbacks &PIC, ModuleAnalysisManager &MAM) {
  PIC.registerBeforeNonSkippedPassCallback([this, &MAM](StringRef P, Any IR) {
    if (isIgnoredPass(P))
      return;
    PreservedAnalyses PA;
    PA.preserveSet<CFGAnalyses>();
    if (const auto **CF = llvm::any_cast<const Function *>(&IR)) {
      Function &F = *const_cast<Function *>(*CF);
      applyDebugify(F, Mode, DebugInfoBeforePass, P);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

char DebugifyModulePass::ID = 0;
static RegisterPass<DebugifyModulePass> DM("debugify",
                                           "Attach debug info to everything");

char CheckDebugifyModulePass::ID = 0;
static RegisterPass<CheckDebugifyModulePass>
    CDM("check-debugify", "Check debug info from -debugify");

char DebugifyFunctionPass::ID = 0;
static RegisterPass<DebugifyFunctionPass> DF("debugify-function",
                                             "Attach debug info to a function");

```
- EN: Core entities appearing here include isIgnoredPass, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 isIgnoredPass，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1200-1202

```cpp
char CheckDebugifyFunctionPass::ID = 0;
static RegisterPass<CheckDebugifyFunctionPass>
    CDF("check-debugify-function", "Check debug info from -debugify-function");
```
- EN: This region continues the Debugify implementation with local helper logic centered on CheckDebugifyFunctionPass, RegisterPass, CDF, Check.
- CN: 这一段延续了 Debugify 的主体实现，围绕 CheckDebugifyFunctionPass, RegisterPass, CDF, Check 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `Level, DebugifyModulePass, DebugifyMode, DebugifyFunctionPass, CheckDebugifyModulePass, CheckDebugifyFunctionPass, symbolizeStackTrace, collectStackAddresses` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`Level, DebugifyModulePass, DebugifyMode, DebugifyFunctionPass, CheckDebugifyModulePass, CheckDebugifyFunctionPass, symbolizeStackTrace, collectStackAddresses` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/DIBuilder.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DebugLoc.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/IR/PassInstrumentation.h`, `llvm/Transforms/Utils/Debugify.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DIBuilder.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DebugLoc.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/IR/PassInstrumentation.h`, `llvm/Transforms/Utils/Debugify.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/FileSystem.h`, `llvm/Support/JSON.h`, `llvm/Support/Signals.h`, `llvm/ADT/BitVector.h`, `llvm/ADT/StringExtras.h`, `llvm/Config/llvm-config.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/FileSystem.h`, `llvm/Support/JSON.h`, `llvm/Support/Signals.h`, `llvm/ADT/BitVector.h`, `llvm/ADT/StringExtras.h`, `llvm/Config/llvm-config.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
