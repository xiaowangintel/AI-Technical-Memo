# PerfMonitor.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/CodeGen/PerfMonitor.cpp` | `polly/lib/CodeGen/PerfMonitor.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements Polly code generation, AST lowering, and runtime emission logic. The leading comment describes it as: Generate a run-time performance monitor.. | 实现 Polly 的代码生成、AST 降级与运行时生成逻辑。 文件开头注释将其概括为：Generate a run-time performance monitor.。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===------ PerfMonitor.cpp - Generate a run-time performance monitor. -======//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//===----------------------------------------------------------------------===//

#include "polly/CodeGen/PerfMonitor.h"
#include "polly/CodeGen/RuntimeDebugBuilder.h"
#include "polly/ScopInfo.h"
#include "llvm/ADT/Twine.h"
#include "llvm/IR/IntrinsicsX86.h"
#include "llvm/IR/Module.h"
#include "llvm/TargetParser/Triple.h"

````
- **EN**: This block records the standard LLVM file banner and license metadata; imports Polly, LLVM-family headers needed by the surrounding code; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 引入周边逻辑所需的 Polly、LLVM-family 头文件; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 19-32

````cpp
using namespace llvm;
using namespace polly;

Function *PerfMonitor::getAtExit() {
  const char *Name = "atexit";
  Function *F = M->getFunction(Name);

  if (!F) {
    GlobalValue::LinkageTypes Linkage = Function::ExternalLinkage;
    FunctionType *Ty =
        FunctionType::get(Builder.getInt32Ty(), {Builder.getPtrTy()}, false);
    F = Function::Create(Ty, Linkage, Name, M);
  }

````
- **EN**: This block declares or defines routines around `getAtExit`, `getFunction`, `get`, `Create`; contains control flow with 1 conditional check(s).
- **CN**: 该代码块 声明或定义与 `getAtExit`, `getFunction`, `get`, `Create` 相关的例程; 包含控制流结构：1 处条件判断.

### Lines 33-47

````cpp
  return F;
}

void PerfMonitor::addToGlobalConstructors(Function *Fn) {
  const char *Name = "llvm.global_ctors";
  GlobalVariable *GV = M->getGlobalVariable(Name);
  std::vector<Constant *> V;

  if (GV) {
    Constant *Array = GV->getInitializer();
    for (Value *X : Array->operand_values())
      V.push_back(cast<Constant>(X));
    GV->eraseFromParent();
  }

````
- **EN**: This block declares or defines routines around `addToGlobalConstructors`, `getGlobalVariable`, `getInitializer`, `push_back` (+1 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `addToGlobalConstructors`, `getGlobalVariable`, `getInitializer`, `push_back` (+1 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 48-60

````cpp
  StructType *ST =
      StructType::get(Builder.getInt32Ty(), Fn->getType(), Builder.getPtrTy());

  V.push_back(
      ConstantStruct::get(ST, Builder.getInt32(10), Fn,
                          ConstantPointerNull::get(Builder.getPtrTy())));
  ArrayType *Ty = ArrayType::get(ST, V.size());

  GV = new GlobalVariable(*M, Ty, true, GlobalValue::AppendingLinkage,
                          ConstantArray::get(Ty, V), Name, nullptr,
                          GlobalVariable::NotThreadLocal);
}

````
- **EN**: This block declares or defines routines around `get`, `push_back`, `GlobalVariable`.
- **CN**: 该代码块 声明或定义与 `get`, `push_back`, `GlobalVariable` 相关的例程.

### Lines 61-72

````cpp
Function *PerfMonitor::getRDTSCP() {
  return Intrinsic::getOrInsertDeclaration(M, Intrinsic::x86_rdtscp);
}

PerfMonitor::PerfMonitor(const Scop &S, Module *M)
    : M(M), Builder(M->getContext()), S(S) {
  if (M->getTargetTriple().getArch() == llvm::Triple::x86_64)
    Supported = true;
  else
    Supported = false;
}

````
- **EN**: This block declares or defines routines around `getRDTSCP`, `PerfMonitor`, `M`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `getRDTSCP`, `PerfMonitor`, `M` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 73-88

````cpp
static void TryRegisterGlobal(Module *M, const char *Name,
                              Constant *InitialValue, Value **Location) {
  *Location = M->getGlobalVariable(Name);

  if (!*Location)
    *Location = new GlobalVariable(
        *M, InitialValue->getType(), true, GlobalValue::WeakAnyLinkage,
        InitialValue, Name, nullptr, GlobalVariable::InitialExecTLSModel);
}

// Generate a unique name that is usable as a LLVM name for a scop to name its
// performance counter.
static std::string GetScopUniqueVarname(const Scop &S) {
  std::string EntryString, ExitString;
  std::tie(EntryString, ExitString) = S.getEntryExitStr();

````
- **EN**: This block declares or defines routines around `TryRegisterGlobal`, `getGlobalVariable`, `GlobalVariable`, `getType` (+2 more); contains control flow with 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `TryRegisterGlobal`, `getGlobalVariable`, `GlobalVariable`, `getType` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 89-102

````cpp
  return (Twine("__polly_perf_in_") + S.getFunction().getName() + "_from__" +
          EntryString + "__to__" + ExitString)
      .str();
}

void PerfMonitor::addScopCounter() {
  const std::string varname = GetScopUniqueVarname(S);
  TryRegisterGlobal(M, (varname + "_cycles").c_str(), Builder.getInt64(0),
                    &CyclesInCurrentScopPtr);

  TryRegisterGlobal(M, (varname + "_trip_count").c_str(), Builder.getInt64(0),
                    &TripCountForCurrentScopPtr);
}

````
- **EN**: This block declares or defines routines around `str`, `addScopCounter`, `GetScopUniqueVarname`, `TryRegisterGlobal`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `str`, `addScopCounter`, `GetScopUniqueVarname`, `TryRegisterGlobal` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 103-116

````cpp
void PerfMonitor::addGlobalVariables() {
  TryRegisterGlobal(M, "__polly_perf_cycles_total_start", Builder.getInt64(0),
                    &CyclesTotalStartPtr);

  TryRegisterGlobal(M, "__polly_perf_initialized", Builder.getInt1(false),
                    &AlreadyInitializedPtr);

  TryRegisterGlobal(M, "__polly_perf_cycles_in_scops", Builder.getInt64(0),
                    &CyclesInScopsPtr);

  TryRegisterGlobal(M, "__polly_perf_cycles_in_scop_start", Builder.getInt64(0),
                    &CyclesInScopStartPtr);
}

````
- **EN**: This block declares or defines routines around `addGlobalVariables`, `TryRegisterGlobal`; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `addGlobalVariables`, `TryRegisterGlobal` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 117-131

````cpp
static const char *InitFunctionName = "__polly_perf_init";
static const char *FinalReportingFunctionName = "__polly_perf_final";

static BasicBlock *FinalStartBB = nullptr;
static ReturnInst *ReturnFromFinal = nullptr;

Function *PerfMonitor::insertFinalReporting() {
  // Create new function.
  GlobalValue::LinkageTypes Linkage = Function::WeakODRLinkage;
  FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), {}, false);
  Function *ExitFn =
      Function::Create(Ty, Linkage, FinalReportingFunctionName, M);
  FinalStartBB = BasicBlock::Create(M->getContext(), "start", ExitFn);
  Builder.SetInsertPoint(FinalStartBB);

````
- **EN**: This block declares or defines routines around `insertFinalReporting`, `get`, `Create`, `SetInsertPoint`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `insertFinalReporting`, `get`, `Create`, `SetInsertPoint` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 132-148

````cpp
  if (!Supported) {
    RuntimeDebugBuilder::createCPUPrinter(
        Builder, "Polly runtime information generation not supported\n");
    Builder.CreateRetVoid();
    return ExitFn;
  }

  // Measure current cycles and compute final timings.
  Function *RDTSCPFn = getRDTSCP();

  Type *Int64Ty = Builder.getInt64Ty();
  Value *CurrentCycles =
      Builder.CreateExtractValue(Builder.CreateCall(RDTSCPFn), {0});
  Value *CyclesStart = Builder.CreateLoad(Int64Ty, CyclesTotalStartPtr, true);
  Value *CyclesTotal = Builder.CreateSub(CurrentCycles, CyclesStart);
  Value *CyclesInScops = Builder.CreateLoad(Int64Ty, CyclesInScopsPtr, true);

````
- **EN**: This block declares or defines routines around `createCPUPrinter`, `CreateRetVoid`, `getRDTSCP`, `getInt64Ty` (+3 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `createCPUPrinter`, `CreateRetVoid`, `getRDTSCP`, `getInt64Ty` (+3 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 149-160

````cpp
  // Print the runtime information.
  RuntimeDebugBuilder::createCPUPrinter(Builder, "Polly runtime information\n");
  RuntimeDebugBuilder::createCPUPrinter(Builder, "-------------------------\n");
  RuntimeDebugBuilder::createCPUPrinter(Builder, "Total: ", CyclesTotal, "\n");
  RuntimeDebugBuilder::createCPUPrinter(Builder, "Scops: ", CyclesInScops,
                                        "\n");

  // Print the preamble for per-scop information.
  RuntimeDebugBuilder::createCPUPrinter(Builder, "\n");
  RuntimeDebugBuilder::createCPUPrinter(Builder, "Per SCoP information\n");
  RuntimeDebugBuilder::createCPUPrinter(Builder, "--------------------\n");

````
- **EN**: This block declares or defines routines around `createCPUPrinter`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createCPUPrinter` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 161-176

````cpp
  RuntimeDebugBuilder::createCPUPrinter(
      Builder, "scop function, "
               "entry block name, exit block name, total time, trip count\n");
  ReturnFromFinal = Builder.CreateRetVoid();
  return ExitFn;
}

void PerfMonitor::AppendScopReporting() {
  if (!Supported)
    return;

  assert(FinalStartBB && "Expected FinalStartBB to be initialized by "
                         "PerfMonitor::insertFinalReporting.");
  assert(ReturnFromFinal && "Expected ReturnFromFinal to be initialized by "
                            "PerfMonitor::insertFinalReporting.");

````
- **EN**: This block declares or defines routines around `createCPUPrinter`, `CreateRetVoid`, `AppendScopReporting`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `createCPUPrinter`, `CreateRetVoid`, `AppendScopReporting` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 177-189

````cpp
  Builder.SetInsertPoint(FinalStartBB);
  ReturnFromFinal->eraseFromParent();

  Type *Int64Ty = Builder.getInt64Ty();
  Value *CyclesInCurrentScop =
      Builder.CreateLoad(Int64Ty, this->CyclesInCurrentScopPtr, true);

  Value *TripCountForCurrentScop =
      Builder.CreateLoad(Int64Ty, this->TripCountForCurrentScopPtr, true);

  std::string EntryName, ExitName;
  std::tie(EntryName, ExitName) = S.getEntryExitStr();

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `eraseFromParent`, `getInt64Ty`, `CreateLoad` (+1 more); touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `eraseFromParent`, `getInt64Ty`, `CreateLoad` (+1 more) 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 190-203

````cpp
  // print in CSV for easy parsing with other tools.
  RuntimeDebugBuilder::createCPUPrinter(
      Builder, S.getFunction().getName(), ", ", EntryName, ", ", ExitName, ", ",
      CyclesInCurrentScop, ", ", TripCountForCurrentScop, "\n");

  ReturnFromFinal = Builder.CreateRetVoid();
}

static Function *FinalReporting = nullptr;

void PerfMonitor::initialize() {
  addGlobalVariables();
  addScopCounter();

````
- **EN**: This block declares or defines routines around `createCPUPrinter`, `getFunction`, `CreateRetVoid`, `initialize` (+2 more); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createCPUPrinter`, `getFunction`, `CreateRetVoid`, `initialize` (+2 more) 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 204-215

````cpp
  // Ensure that we only add the final reporting function once.
  // On later invocations, append to the reporting function.
  if (!FinalReporting) {
    FinalReporting = insertFinalReporting();

    Function *InitFn = insertInitFunction(FinalReporting);
    addToGlobalConstructors(InitFn);
  }

  AppendScopReporting();
}

````
- **EN**: This block declares or defines routines around `insertFinalReporting`, `insertInitFunction`, `addToGlobalConstructors`, `AppendScopReporting`; contains control flow with 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `insertFinalReporting`, `insertInitFunction`, `addToGlobalConstructors`, `AppendScopReporting` 相关的例程; 包含控制流结构：1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 216-227

````cpp
Function *PerfMonitor::insertInitFunction(Function *FinalReporting) {
  // Insert function definition and BBs.
  GlobalValue::LinkageTypes Linkage = Function::WeakODRLinkage;
  FunctionType *Ty = FunctionType::get(Builder.getVoidTy(), {}, false);
  Function *InitFn = Function::Create(Ty, Linkage, InitFunctionName, M);
  BasicBlock *Start = BasicBlock::Create(M->getContext(), "start", InitFn);
  BasicBlock *EarlyReturn =
      BasicBlock::Create(M->getContext(), "earlyreturn", InitFn);
  BasicBlock *InitBB = BasicBlock::Create(M->getContext(), "initbb", InitFn);

  Builder.SetInsertPoint(Start);

````
- **EN**: This block declares or defines routines around `insertInitFunction`, `get`, `Create`, `SetInsertPoint`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `insertInitFunction`, `get`, `Create`, `SetInsertPoint` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 228-242

````cpp
  // Check if this function was already run. If yes, return.
  //
  // In case profiling has been enabled in multiple translation units, the
  // initializer function will be added to the global constructors list of
  // each translation unit. When merging translation units, the global
  // constructor lists are just appended, such that the initializer will appear
  // multiple times. To avoid initializations being run multiple times (and
  // especially to avoid that atExitFn is called more than once), we bail
  // out if the initializer is run more than once.
  Value *HasRunBefore =
      Builder.CreateLoad(Builder.getInt1Ty(), AlreadyInitializedPtr);
  Builder.CreateCondBr(HasRunBefore, EarlyReturn, InitBB);
  Builder.SetInsertPoint(EarlyReturn);
  Builder.CreateRetVoid();

````
- **EN**: This block declares or defines routines around `CreateLoad`, `CreateCondBr`, `SetInsertPoint`, `CreateRetVoid`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `CreateLoad`, `CreateCondBr`, `SetInsertPoint`, `CreateRetVoid` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 243-262

````cpp
  // Keep track that this function has been run once.
  Builder.SetInsertPoint(InitBB);
  Value *True = Builder.getInt1(true);
  Builder.CreateStore(True, AlreadyInitializedPtr);

  // Register the final reporting function with atexit().
  Value *FinalReportingPtr =
      Builder.CreatePointerCast(FinalReporting, Builder.getPtrTy());
  Function *AtExitFn = getAtExit();
  Builder.CreateCall(AtExitFn, {FinalReportingPtr});

  if (Supported) {
    // Read the currently cycle counter and store the result for later.
    Function *RDTSCPFn = getRDTSCP();
    Value *CurrentCycles =
        Builder.CreateExtractValue(Builder.CreateCall(RDTSCPFn), {0});
    Builder.CreateStore(CurrentCycles, CyclesTotalStartPtr, true);
  }
  Builder.CreateRetVoid();

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `getInt1`, `CreateStore`, `CreatePointerCast` (+5 more); contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `getInt1`, `CreateStore`, `CreatePointerCast` (+5 more) 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 263-276

````cpp
  return InitFn;
}

void PerfMonitor::insertRegionStart(Instruction *InsertBefore) {
  if (!Supported)
    return;

  Builder.SetInsertPoint(InsertBefore->getIterator());
  Function *RDTSCPFn = getRDTSCP();
  Value *CurrentCycles =
      Builder.CreateExtractValue(Builder.CreateCall(RDTSCPFn), {0});
  Builder.CreateStore(CurrentCycles, CyclesInScopStartPtr, true);
}

````
- **EN**: This block declares or defines routines around `insertRegionStart`, `SetInsertPoint`, `getRDTSCP`, `CreateExtractValue` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `insertRegionStart`, `SetInsertPoint`, `getRDTSCP`, `CreateExtractValue` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 277-292

````cpp
void PerfMonitor::insertRegionEnd(Instruction *InsertBefore) {
  if (!Supported)
    return;

  Builder.SetInsertPoint(InsertBefore->getIterator());
  Function *RDTSCPFn = getRDTSCP();
  Type *Int64Ty = Builder.getInt64Ty();
  LoadInst *CyclesStart =
      Builder.CreateLoad(Int64Ty, CyclesInScopStartPtr, true);
  Value *CurrentCycles =
      Builder.CreateExtractValue(Builder.CreateCall(RDTSCPFn), {0});
  Value *CyclesInScop = Builder.CreateSub(CurrentCycles, CyclesStart);
  Value *CyclesInScops = Builder.CreateLoad(Int64Ty, CyclesInScopsPtr, true);
  CyclesInScops = Builder.CreateAdd(CyclesInScops, CyclesInScop);
  Builder.CreateStore(CyclesInScops, CyclesInScopsPtr, true);

````
- **EN**: This block declares or defines routines around `insertRegionEnd`, `SetInsertPoint`, `getRDTSCP`, `getInt64Ty` (+5 more); contains control flow with 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `insertRegionEnd`, `SetInsertPoint`, `getRDTSCP`, `getInt64Ty` (+5 more) 相关的例程; 包含控制流结构：1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 293-304

````cpp
  Value *CyclesInCurrentScop =
      Builder.CreateLoad(Int64Ty, CyclesInCurrentScopPtr, true);
  CyclesInCurrentScop = Builder.CreateAdd(CyclesInCurrentScop, CyclesInScop);
  Builder.CreateStore(CyclesInCurrentScop, CyclesInCurrentScopPtr, true);

  Value *TripCountForCurrentScop =
      Builder.CreateLoad(Int64Ty, TripCountForCurrentScopPtr, true);
  TripCountForCurrentScop =
      Builder.CreateAdd(TripCountForCurrentScop, Builder.getInt64(1));
  Builder.CreateStore(TripCountForCurrentScop, TripCountForCurrentScopPtr,
                      true);
}
````
- **EN**: This block declares or defines routines around `CreateLoad`, `CreateAdd`, `CreateStore`; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `CreateLoad`, `CreateAdd`, `CreateStore` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

## Key Concepts / 关键概念

- **Code generation**
  - **CN**: 代码生成
- **SCoP modeling**
  - **CN**: SCoP 建模
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **Loop transformation**
  - **CN**: 循环变换

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/PerfMonitor.h`, `polly/CodeGen/RuntimeDebugBuilder.h`, `polly/ScopInfo.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/PerfMonitor.h`, `polly/CodeGen/RuntimeDebugBuilder.h`, `polly/ScopInfo.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/ADT/Twine.h`, `llvm/IR/IntrinsicsX86.h`, `llvm/IR/Module.h`, `llvm/TargetParser/Triple.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/ADT/Twine.h`, `llvm/IR/IntrinsicsX86.h`, `llvm/IR/Module.h`, `llvm/TargetParser/Triple.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
