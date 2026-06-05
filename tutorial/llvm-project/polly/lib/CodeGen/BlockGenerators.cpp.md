# BlockGenerators.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/CodeGen/BlockGenerators.cpp` | `polly/lib/CodeGen/BlockGenerators.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements Polly code generation, AST lowering, and runtime emission logic. The leading comment describes it as: Generate code for statements. | 实现 Polly 的代码生成、AST 降级与运行时生成逻辑。 文件开头注释将其概括为：Generate code for statements。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-31

````cpp
//===--- BlockGenerators.cpp - Generate code for statements -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the BlockGenerator and VectorBlockGenerator classes,
// which generate sequential code and vectorized code for a polyhedral
// statement, respectively.
//
//===----------------------------------------------------------------------===//

#include "polly/CodeGen/BlockGenerators.h"
#include "polly/CodeGen/IslExprBuilder.h"
#include "polly/CodeGen/RuntimeDebugBuilder.h"
#include "polly/Options.h"
#include "polly/ScopInfo.h"
#include "polly/Support/ISLTools.h"
#include "polly/Support/ScopHelper.h"
#include "polly/Support/VirtualInstruction.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/RegionInfo.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Local.h"
#include "isl/ast.h"
#include <deque>

````
- **EN**: This block records the standard LLVM file banner and license metadata; imports Polly, LLVM-family, ISL, system/standard headers needed by the surrounding code; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 引入周边逻辑所需的 Polly、LLVM-family、ISL、system/standard 头文件; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 32-64

````cpp
using namespace llvm;
using namespace polly;

static cl::opt<bool> Aligned("enable-polly-aligned",
                             cl::desc("Assumed aligned memory accesses."),
                             cl::Hidden, cl::cat(PollyCategory));

bool PollyDebugPrinting;
static cl::opt<bool, true> DebugPrintingX(
    "polly-codegen-add-debug-printing",
    cl::desc("Add printf calls that show the values loaded/stored."),
    cl::location(PollyDebugPrinting), cl::Hidden, cl::cat(PollyCategory));

bool TraceStmts;
static cl::opt<bool, true> TraceStmtsX(
    "polly-codegen-trace-stmts",
    cl::desc("Add printf calls that print the statement being executed"),
    cl::location(TraceStmts), cl::Hidden, cl::cat(PollyCategory));

static cl::opt<bool> TraceScalars(
    "polly-codegen-trace-scalars",
    cl::desc("Add printf calls that print the values of all scalar values "
             "used in a statement. Requires -polly-codegen-trace-stmts."),
    cl::Hidden, cl::cat(PollyCategory));

BlockGenerator::BlockGenerator(
    PollyIRBuilder &B, LoopInfo &LI, ScalarEvolution &SE, DominatorTree &DT,
    AllocaMapTy &ScalarMap, EscapeUsersAllocaMapTy &EscapeMap,
    ValueMapT &GlobalMap, IslExprBuilder *ExprBuilder, BasicBlock *StartBlock)
    : Builder(B), LI(LI), SE(SE), ExprBuilder(ExprBuilder), DT(DT), GenDT(&DT),
      GenLI(&LI), GenSE(&SE), ScalarMap(ScalarMap), EscapeMap(EscapeMap),
      GlobalMap(GlobalMap), StartBlock(StartBlock) {}

````
- **EN**: This block registers command-line options such as `Aligned`, `DebugPrintingX`, `TraceStmtsX`, `TraceScalars`; declares or defines routines around `Aligned`, `desc`, `cat`, `DebugPrintingX` (+7 more).
- **CN**: 该代码块 注册命令行选项，例如 `Aligned`, `DebugPrintingX`, `TraceStmtsX`, `TraceScalars`; 声明或定义与 `Aligned`, `desc`, `cat`, `DebugPrintingX` (+7 more) 相关的例程.

### Lines 65-92

````cpp
Value *BlockGenerator::trySynthesizeNewValue(ScopStmt &Stmt, Value *Old,
                                             ValueMapT &BBMap,
                                             LoopToScevMapT &LTS,
                                             Loop *L) const {
  if (!SE.isSCEVable(Old->getType()))
    return nullptr;

  const SCEV *Scev = SE.getSCEVAtScope(Old, L);
  if (!Scev)
    return nullptr;

  if (isa<SCEVCouldNotCompute>(Scev))
    return nullptr;

  ValueMapT VTV;
  VTV.insert_range(BBMap);
  VTV.insert_range(GlobalMap);

  Scop &S = *Stmt.getParent();
  const DataLayout &DL = S.getFunction().getDataLayout();
  auto IP = Builder.GetInsertPoint();

  assert(IP != Builder.GetInsertBlock()->end() &&
         "Only instructions can be insert points for SCEVExpander");
  Value *Expanded = expandCodeFor(
      S, SE, Builder.GetInsertBlock()->getParent(), *GenSE, DL, "polly", Scev,
      Old->getType(), IP, &VTV, &LTS, StartBlock->getSinglePredecessor());

````
- **EN**: This block declares or defines routines around `trySynthesizeNewValue`, `getSCEVAtScope`, `insert_range`, `getParent` (+5 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `trySynthesizeNewValue`, `getSCEVAtScope`, `insert_range`, `getParent` (+5 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 93-122

````cpp
  BBMap[Old] = Expanded;
  return Expanded;
}

Value *BlockGenerator::getNewValue(ScopStmt &Stmt, Value *Old, ValueMapT &BBMap,
                                   LoopToScevMapT &LTS, Loop *L) const {

  auto lookupGlobally = [this](Value *Old) -> Value * {
    Value *New = GlobalMap.lookup(Old);
    if (!New)
      return nullptr;

    // Required by:
    // * Isl/CodeGen/OpenMP/invariant_base_pointer_preloaded.ll
    // * Isl/CodeGen/OpenMP/invariant_base_pointer_preloaded_different_bb.ll
    // * Isl/CodeGen/OpenMP/invariant_base_pointer_preloaded_pass_only_needed.ll
    // * Isl/CodeGen/OpenMP/invariant_base_pointers_preloaded.ll
    // * Isl/CodeGen/OpenMP/loop-body-references-outer-values-3.ll
    // * Isl/CodeGen/OpenMP/single_loop_with_loop_invariant_baseptr.ll
    // GlobalMap should be a mapping from (value in original SCoP) to (copied
    // value in generated SCoP), without intermediate mappings, which might
    // easily require transitiveness as well.
    if (Value *NewRemapped = GlobalMap.lookup(New))
      New = NewRemapped;

    // No test case for this code.
    if (Old->getType()->getScalarSizeInBits() <
        New->getType()->getScalarSizeInBits())
      New = Builder.CreateTruncOrBitCast(New, Old->getType());

````
- **EN**: This block declares or defines routines around `getNewValue`, `lookup`, `getType`, `CreateTruncOrBitCast`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getNewValue`, `lookup`, `getType`, `CreateTruncOrBitCast` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 123-149

````cpp
    return New;
  };

  Value *New = nullptr;
  auto VUse = VirtualUse::create(&Stmt, L, Old, true);
  switch (VUse.getKind()) {
  case VirtualUse::Block:
    // BasicBlock are constants, but the BlockGenerator copies them.
    New = BBMap.lookup(Old);
    break;

  case VirtualUse::Constant:
    // Used by:
    // * Isl/CodeGen/OpenMP/reference-argument-from-non-affine-region.ll
    // Constants should not be redefined. In this case, the GlobalMap just
    // contains a mapping to the same constant, which is unnecessary, but
    // harmless.
    if ((New = lookupGlobally(Old)))
      break;

    assert(!BBMap.count(Old));
    New = Old;
    break;

  case VirtualUse::ReadOnly:
    assert(!GlobalMap.count(Old));

````
- **EN**: This block declares or defines routines around `create`, `lookup`; contains control flow with 1 conditional check(s), 1 switch dispatch(es); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `create`, `lookup` 相关的例程; 包含控制流结构：1 处条件判断、1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 150-179

````cpp
    // Required for:
    // * Isl/CodeGen/MemAccess/create_arrays.ll
    // * Isl/CodeGen/read-only-scalars.ll
    // * ScheduleOptimizer/pattern-matching-based-opts_10.ll
    // For some reason these reload a read-only value. The reloaded value ends
    // up in BBMap, buts its value should be identical.
    //
    // Required for:
    // * Isl/CodeGen/OpenMP/single_loop_with_param.ll
    // The parallel subfunctions need to reference the read-only value from the
    // parent function, this is done by reloading them locally.
    if ((New = BBMap.lookup(Old)))
      break;

    New = Old;
    break;

  case VirtualUse::Synthesizable:
    // Used by:
    // * Isl/CodeGen/OpenMP/loop-body-references-outer-values-3.ll
    // * Isl/CodeGen/OpenMP/recomputed-srem.ll
    // * Isl/CodeGen/OpenMP/reference-other-bb.ll
    // * Isl/CodeGen/OpenMP/two-parallel-loops-reference-outer-indvar.ll
    // For some reason synthesizable values end up in GlobalMap. Their values
    // are the same as trySynthesizeNewValue would return. The legacy
    // implementation prioritized GlobalMap, so this is what we do here as well.
    // Ideally, synthesizable values should not end up in GlobalMap.
    if ((New = lookupGlobally(Old)))
      break;

````
- **EN**: This block contains control flow with 2 conditional check(s); mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 包含控制流结构：2 处条件判断; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 180-213

````cpp
    // Required for:
    // * Isl/CodeGen/RuntimeDebugBuilder/combine_different_values.ll
    // * Isl/CodeGen/getNumberOfIterations.ll
    // * Isl/CodeGen/non_affine_float_compare.ll
    // * ScheduleOptimizer/pattern-matching-based-opts_10.ll
    // Ideally, synthesizable values are synthesized by trySynthesizeNewValue,
    // not precomputed (SCEVExpander has its own caching mechanism).
    // These tests fail without this, but I think trySynthesizeNewValue would
    // just re-synthesize the same instructions.
    if ((New = BBMap.lookup(Old)))
      break;

    New = trySynthesizeNewValue(Stmt, Old, BBMap, LTS, L);
    break;

  case VirtualUse::Hoisted:
    // TODO: Hoisted invariant loads should be found in GlobalMap only, but not
    // redefined locally (which will be ignored anyway). That is, the following
    // assertion should apply: assert(!BBMap.count(Old))

    New = lookupGlobally(Old);
    break;

  case VirtualUse::Intra:
  case VirtualUse::Inter:
    assert(!GlobalMap.count(Old) &&
           "Intra and inter-stmt values are never global");
    New = BBMap.lookup(Old);
    break;
  }
  assert(New && "Unexpected scalar dependence in region!");
  return New;
}

````
- **EN**: This block declares or defines routines around `trySynthesizeNewValue`, `lookupGlobally`, `lookup`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `trySynthesizeNewValue`, `lookupGlobally`, `lookup` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 214-240

````cpp
void BlockGenerator::copyInstScalar(ScopStmt &Stmt, Instruction *Inst,
                                    ValueMapT &BBMap, LoopToScevMapT &LTS) {
  // We do not generate debug intrinsics as we did not investigate how to
  // copy them correctly. At the current state, they just crash the code
  // generation as the meta-data operands are not correctly copied.
  if (isa<DbgInfoIntrinsic>(Inst))
    return;

  Instruction *NewInst = Inst->clone();

  // Replace old operands with the new ones.
  for (Value *OldOperand : Inst->operands()) {
    Value *NewOperand =
        getNewValue(Stmt, OldOperand, BBMap, LTS, getLoopForStmt(Stmt));

    if (!NewOperand) {
      assert(!isa<StoreInst>(NewInst) &&
             "Store instructions are always needed!");
      NewInst->deleteValue();
      return;
    }

    // FIXME: We will encounter "NewOperand" again if used twice. getNewValue()
    // is meant to be called on old values only.
    NewInst->replaceUsesOfWith(OldOperand, NewOperand);
  }

````
- **EN**: This block declares or defines routines around `copyInstScalar`, `clone`, `getNewValue`, `deleteValue` (+1 more); contains control flow with 1 loop construct(s), 2 conditional check(s); adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `copyInstScalar`, `clone`, `getNewValue`, `deleteValue` (+1 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 241-267

````cpp
  Builder.Insert(NewInst);
  BBMap[Inst] = NewInst;

  assert(NewInst->getModule() == Inst->getModule() &&
         "Expecting instructions to be in the same module");

  if (!NewInst->getType()->isVoidTy())
    NewInst->setName("p_" + Inst->getName());
}

Value *
BlockGenerator::generateLocationAccessed(ScopStmt &Stmt, MemAccInst Inst,
                                         ValueMapT &BBMap, LoopToScevMapT &LTS,
                                         isl_id_to_ast_expr *NewAccesses) {
  const MemoryAccess &MA = Stmt.getArrayAccessFor(Inst);
  return generateLocationAccessed(
      Stmt, getLoopForStmt(Stmt),
      Inst.isNull() ? nullptr : Inst.getPointerOperand(), BBMap, LTS,
      NewAccesses, MA.getId().release(), MA.getAccessValue()->getType());
}

Value *BlockGenerator::generateLocationAccessed(
    ScopStmt &Stmt, Loop *L, Value *Pointer, ValueMapT &BBMap,
    LoopToScevMapT &LTS, isl_id_to_ast_expr *NewAccesses, __isl_take isl_id *Id,
    Type *ExpectedType) {
  isl_ast_expr *AccessExpr = isl_id_to_ast_expr_get(NewAccesses, Id);

````
- **EN**: This block declares or defines routines around `Insert`, `setName`, `generateLocationAccessed`, `getArrayAccessFor` (+4 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `Insert`, `setName`, `generateLocationAccessed`, `getArrayAccessFor` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 268-294

````cpp
  if (AccessExpr) {
    AccessExpr = isl_ast_expr_address_of(AccessExpr);
    return ExprBuilder->create(AccessExpr);
  }
  assert(
      Pointer &&
      "If expression was not generated, must use the original pointer value");
  return getNewValue(Stmt, Pointer, BBMap, LTS, L);
}

Value *
BlockGenerator::getImplicitAddress(MemoryAccess &Access, Loop *L,
                                   LoopToScevMapT &LTS, ValueMapT &BBMap,
                                   __isl_keep isl_id_to_ast_expr *NewAccesses) {
  if (Access.isLatestArrayKind())
    return generateLocationAccessed(*Access.getStatement(), L, nullptr, BBMap,
                                    LTS, NewAccesses, Access.getId().release(),
                                    Access.getAccessValue()->getType());

  return getOrCreateAlloca(Access);
}

Loop *BlockGenerator::getLoopForStmt(const ScopStmt &Stmt) const {
  auto *StmtBB = Stmt.getEntryBlock();
  return LI.getLoopFor(StmtBB);
}

````
- **EN**: This block declares or defines routines around `isl_ast_expr_address_of`, `getImplicitAddress`, `getId`, `getAccessValue` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_ast_expr_address_of`, `getImplicitAddress`, `getId`, `getAccessValue` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 295-320

````cpp
Value *BlockGenerator::generateArrayLoad(ScopStmt &Stmt, LoadInst *Load,
                                         ValueMapT &BBMap, LoopToScevMapT &LTS,
                                         isl_id_to_ast_expr *NewAccesses) {
  if (Value *PreloadLoad = GlobalMap.lookup(Load))
    return PreloadLoad;

  Value *NewPointer =
      generateLocationAccessed(Stmt, Load, BBMap, LTS, NewAccesses);
  Value *ScalarLoad =
      Builder.CreateAlignedLoad(Load->getType(), NewPointer, Load->getAlign(),
                                Load->getName() + "_p_scalar_");

  if (PollyDebugPrinting)
    RuntimeDebugBuilder::createCPUPrinter(Builder, "Load from ", NewPointer,
                                          ": ", ScalarLoad, "\n");

  return ScalarLoad;
}

void BlockGenerator::generateArrayStore(ScopStmt &Stmt, StoreInst *Store,
                                        ValueMapT &BBMap, LoopToScevMapT &LTS,
                                        isl_id_to_ast_expr *NewAccesses) {
  MemoryAccess &MA = Stmt.getArrayAccessFor(Store);
  isl::set AccDom = MA.getAccessRelation().domain();
  std::string Subject = MA.getId().get_name();

````
- **EN**: This block declares or defines routines around `generateArrayLoad`, `generateLocationAccessed`, `CreateAlignedLoad`, `getName` (+5 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `generateArrayLoad`, `generateLocationAccessed`, `CreateAlignedLoad`, `getName` (+5 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 321-348

````cpp
  generateConditionalExecution(Stmt, AccDom, Subject.c_str(), [&, this]() {
    Value *NewPointer =
        generateLocationAccessed(Stmt, Store, BBMap, LTS, NewAccesses);
    Value *ValueOperand = getNewValue(Stmt, Store->getValueOperand(), BBMap,
                                      LTS, getLoopForStmt(Stmt));

    if (PollyDebugPrinting)
      RuntimeDebugBuilder::createCPUPrinter(Builder, "Store to  ", NewPointer,
                                            ": ", ValueOperand, "\n");

    Builder.CreateAlignedStore(ValueOperand, NewPointer, Store->getAlign());
  });
}

bool BlockGenerator::canSyntheziseInStmt(ScopStmt &Stmt, Instruction *Inst) {
  Loop *L = getLoopForStmt(Stmt);
  return (Stmt.isBlockStmt() || !Stmt.getRegion()->contains(L)) &&
         canSynthesize(Inst, *Stmt.getParent(), &SE, L);
}

void BlockGenerator::copyInstruction(ScopStmt &Stmt, Instruction *Inst,
                                     ValueMapT &BBMap, LoopToScevMapT &LTS,
                                     isl_id_to_ast_expr *NewAccesses) {
  // Terminator instructions control the control flow. They are explicitly
  // expressed in the clast and do not need to be copied.
  if (Inst->isTerminator())
    return;

````
- **EN**: This block declares or defines routines around `generateConditionalExecution`, `generateLocationAccessed`, `getNewValue`, `getLoopForStmt` (+5 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `generateConditionalExecution`, `generateLocationAccessed`, `getNewValue`, `getLoopForStmt` (+5 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 349-374

````cpp
  // Synthesizable statements will be generated on-demand.
  if (canSyntheziseInStmt(Stmt, Inst))
    return;

  if (auto *Load = dyn_cast<LoadInst>(Inst)) {
    Value *NewLoad = generateArrayLoad(Stmt, Load, BBMap, LTS, NewAccesses);
    // Compute NewLoad before its insertion in BBMap to make the insertion
    // deterministic.
    BBMap[Load] = NewLoad;
    return;
  }

  if (auto *Store = dyn_cast<StoreInst>(Inst)) {
    // Identified as redundant by -polly-simplify.
    if (!Stmt.getArrayAccessOrNULLFor(Store))
      return;

    generateArrayStore(Stmt, Store, BBMap, LTS, NewAccesses);
    return;
  }

  if (auto *PHI = dyn_cast<PHINode>(Inst)) {
    copyPHIInstruction(Stmt, PHI, BBMap, LTS);
    return;
  }

````
- **EN**: This block declares or defines routines around `generateArrayLoad`, `generateArrayStore`, `copyPHIInstruction`; contains control flow with 5 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `generateArrayLoad`, `generateArrayStore`, `copyPHIInstruction` 相关的例程; 包含控制流结构：5 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 375-400

````cpp
  // Skip some special intrinsics for which we do not adjust the semantics to
  // the new schedule. All others are handled like every other instruction.
  if (isIgnoredIntrinsic(Inst))
    return;

  copyInstScalar(Stmt, Inst, BBMap, LTS);
}

void BlockGenerator::removeDeadInstructions(BasicBlock *BB, ValueMapT &BBMap) {
  auto NewBB = Builder.GetInsertBlock();
  for (auto I = NewBB->rbegin(); I != NewBB->rend(); I++) {
    Instruction *NewInst = &*I;

    if (!isInstructionTriviallyDead(NewInst))
      continue;

    for (auto Pair : BBMap)
      if (Pair.second == NewInst) {
        BBMap.erase(Pair.first);
      }

    NewInst->eraseFromParent();
    I = NewBB->rbegin();
  }
}

````
- **EN**: This block declares or defines routines around `copyInstScalar`, `removeDeadInstructions`, `GetInsertBlock`, `erase` (+2 more); contains control flow with 2 loop construct(s), 3 conditional check(s); mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `copyInstScalar`, `removeDeadInstructions`, `GetInsertBlock`, `erase` (+2 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 401-427

````cpp
void BlockGenerator::copyStmt(ScopStmt &Stmt, LoopToScevMapT &LTS,
                              __isl_keep isl_id_to_ast_expr *NewAccesses) {
  assert(Stmt.isBlockStmt() &&
         "Only block statements can be copied by the block generator");

  ValueMapT BBMap;

  BasicBlock *BB = Stmt.getBasicBlock();
  copyBB(Stmt, BB, BBMap, LTS, NewAccesses);
  removeDeadInstructions(BB, BBMap);
}

BasicBlock *BlockGenerator::splitBB(BasicBlock *BB) {
  BasicBlock *CopyBB = SplitBlock(Builder.GetInsertBlock(),
                                  Builder.GetInsertPoint(), GenDT, GenLI);
  CopyBB->setName("polly.stmt." + BB->getName());
  return CopyBB;
}

BasicBlock *BlockGenerator::copyBB(ScopStmt &Stmt, BasicBlock *BB,
                                   ValueMapT &BBMap, LoopToScevMapT &LTS,
                                   isl_id_to_ast_expr *NewAccesses) {
  BasicBlock *CopyBB = splitBB(BB);
  Builder.SetInsertPoint(CopyBB, CopyBB->begin());
  generateScalarLoads(Stmt, LTS, BBMap, NewAccesses);
  generateBeginStmtTrace(Stmt, LTS, BBMap);

````
- **EN**: This block declares or defines routines around `copyStmt`, `getBasicBlock`, `copyBB`, `removeDeadInstructions` (+7 more); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `copyStmt`, `getBasicBlock`, `copyBB`, `removeDeadInstructions` (+7 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 428-462

````cpp
  copyBB(Stmt, BB, CopyBB, BBMap, LTS, NewAccesses);

  // After a basic block was copied store all scalars that escape this block in
  // their alloca.
  generateScalarStores(Stmt, LTS, BBMap, NewAccesses);
  return CopyBB;
}

void BlockGenerator::switchGeneratedFunc(Function *GenFn, DominatorTree *GenDT,
                                         LoopInfo *GenLI,
                                         ScalarEvolution *GenSE) {
  assert(GenFn == GenDT->getRoot()->getParent());
  assert(GenLI->getTopLevelLoops().empty() ||
         GenFn == GenLI->getTopLevelLoops().front()->getHeader()->getParent());
  this->GenDT = GenDT;
  this->GenLI = GenLI;
  this->GenSE = GenSE;
}

void BlockGenerator::copyBB(ScopStmt &Stmt, BasicBlock *BB, BasicBlock *CopyBB,
                            ValueMapT &BBMap, LoopToScevMapT &LTS,
                            isl_id_to_ast_expr *NewAccesses) {
  // Block statements and the entry blocks of region statement are code
  // generated from instruction lists. This allow us to optimize the
  // instructions that belong to a certain scop statement. As the code
  // structure of region statements might be arbitrary complex, optimizing the
  // instruction list is not yet supported.
  if (Stmt.isBlockStmt() || (Stmt.isRegionStmt() && Stmt.getEntryBlock() == BB))
    for (Instruction *Inst : Stmt.getInstructions())
      copyInstruction(Stmt, Inst, BBMap, LTS, NewAccesses);
  else
    for (Instruction &Inst : *BB)
      copyInstruction(Stmt, &Inst, BBMap, LTS, NewAccesses);
}

````
- **EN**: This block declares or defines routines around `copyBB`, `generateScalarStores`, `switchGeneratedFunc`, `getTopLevelLoops` (+1 more); contains control flow with 2 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `copyBB`, `generateScalarStores`, `switchGeneratedFunc`, `getTopLevelLoops` (+1 more) 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 463-497

````cpp
Value *BlockGenerator::getOrCreateAlloca(const MemoryAccess &Access) {
  assert(!Access.isLatestArrayKind() && "Trying to get alloca for array kind");

  return getOrCreateAlloca(Access.getLatestScopArrayInfo());
}

Value *BlockGenerator::getOrCreateAlloca(const ScopArrayInfo *Array) {
  assert(!Array->isArrayKind() && "Trying to get alloca for array kind");

  auto &Addr = ScalarMap[Array];

  if (Addr) {
    // Allow allocas to be (temporarily) redirected once by adding a new
    // old-alloca-addr to new-addr mapping to GlobalMap. This functionality
    // is used for example by the OpenMP code generation where a first use
    // of a scalar while still in the host code allocates a normal alloca with
    // getOrCreateAlloca. When the values of this scalar are accessed during
    // the generation of the parallel subfunction, these values are copied over
    // to the parallel subfunction and each request for a scalar alloca slot
    // must be forwarded to the temporary in-subfunction slot. This mapping is
    // removed when the subfunction has been generated and again normal host
    // code is generated. Due to the following reasons it is not possible to
    // perform the GlobalMap lookup right after creating the alloca below, but
    // instead we need to check GlobalMap at each call to getOrCreateAlloca:
    //
    //   1) GlobalMap may be changed multiple times (for each parallel loop),
    //   2) The temporary mapping is commonly only known after the initial
    //      alloca has already been generated, and
    //   3) The original alloca value must be restored after leaving the
    //      sub-function.
    if (Value *NewAddr = GlobalMap.lookup(&*Addr))
      return NewAddr;
    return Addr;
  }

````
- **EN**: This block declares or defines routines around `getOrCreateAlloca`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getOrCreateAlloca` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 498-525

````cpp
  Type *Ty = Array->getElementType();
  Value *ScalarBase = Array->getBasePtr();
  std::string NameExt;
  if (Array->isPHIKind())
    NameExt = ".phiops";
  else
    NameExt = ".s2a";

  const DataLayout &DL = Builder.GetInsertBlock()->getDataLayout();

  Addr =
      new AllocaInst(Ty, DL.getAllocaAddrSpace(), nullptr,
                     DL.getPrefTypeAlign(Ty), ScalarBase->getName() + NameExt);
  BasicBlock *EntryBB = &Builder.GetInsertBlock()->getParent()->getEntryBlock();
  Addr->insertBefore(EntryBB->getFirstInsertionPt());

  return Addr;
}

void BlockGenerator::handleOutsideUsers(const Scop &S, ScopArrayInfo *Array) {
  Instruction *Inst = cast<Instruction>(Array->getBasePtr());

  // If there are escape users we get the alloca for this instruction and put it
  // in the EscapeMap for later finalization. Lastly, if the instruction was
  // copied multiple times we already did this and can exit.
  if (EscapeMap.count(Inst))
    return;

````
- **EN**: This block declares or defines routines around `getElementType`, `getBasePtr`, `GetInsertBlock`, `AllocaInst` (+4 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getElementType`, `getBasePtr`, `GetInsertBlock`, `AllocaInst` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 526-557

````cpp
  EscapeUserVectorTy EscapeUsers;
  for (User *U : Inst->users()) {

    // Non-instruction user will never escape.
    Instruction *UI = dyn_cast<Instruction>(U);
    if (!UI)
      continue;

    if (S.contains(UI))
      continue;

    EscapeUsers.push_back(UI);
  }

  // Exit if no escape uses were found.
  if (EscapeUsers.empty())
    return;

  // Get or create an escape alloca for this instruction.
  auto *ScalarAddr = getOrCreateAlloca(Array);

  // Remember that this instruction has escape uses and the escape alloca.
  EscapeMap[Inst] = std::make_pair(ScalarAddr, std::move(EscapeUsers));
}

void BlockGenerator::generateScalarLoads(
    ScopStmt &Stmt, LoopToScevMapT &LTS, ValueMapT &BBMap,
    __isl_keep isl_id_to_ast_expr *NewAccesses) {
  for (MemoryAccess *MA : Stmt) {
    if (MA->isOriginalArrayKind() || MA->isWrite())
      continue;

````
- **EN**: This block declares or defines routines around `dyn_cast<Instruction>`, `push_back`, `getOrCreateAlloca`, `make_pair` (+1 more); contains control flow with 2 loop construct(s), 4 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `dyn_cast<Instruction>`, `push_back`, `getOrCreateAlloca`, `make_pair` (+1 more) 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 558-583

````cpp
#ifndef NDEBUG
    auto StmtDom =
        Stmt.getDomain().intersect_params(Stmt.getParent()->getContext());
    auto AccDom = MA->getAccessRelation().domain();
    assert(!StmtDom.is_subset(AccDom).is_false() &&
           "Scalar must be loaded in all statement instances");
#endif

    auto *Address =
        getImplicitAddress(*MA, getLoopForStmt(Stmt), LTS, BBMap, NewAccesses);
    BBMap[MA->getAccessValue()] = Builder.CreateLoad(
        MA->getElementType(), Address, Address->getName() + ".reload");
  }
}

Value *BlockGenerator::buildContainsCondition(ScopStmt &Stmt,
                                              const isl::set &Subdomain) {
  isl::ast_build AstBuild = Stmt.getAstBuild();
  isl::set Domain = Stmt.getDomain();

  isl::union_map USchedule = AstBuild.get_schedule();
  USchedule = USchedule.intersect_domain(Domain);

  assert(!USchedule.is_empty());
  isl::map Schedule = isl::map::from_union_map(USchedule);

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `getDomain`, `getAccessRelation`, `getImplicitAddress`, `getAccessValue` (+6 more); adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `getDomain`, `getAccessRelation`, `getImplicitAddress`, `getAccessValue` (+6 more) 相关的例程; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 584-611

````cpp
  isl::set ScheduledDomain = Schedule.range();
  isl::set ScheduledSet = Subdomain.apply(Schedule);

  isl::ast_build RestrictedBuild = AstBuild.restrict(ScheduledDomain);

  isl::ast_expr IsInSet = RestrictedBuild.expr_from(ScheduledSet);
  Value *IsInSetExpr = ExprBuilder->create(IsInSet.copy());
  IsInSetExpr = Builder.CreateICmpNE(
      IsInSetExpr, ConstantInt::get(IsInSetExpr->getType(), 0));

  return IsInSetExpr;
}

void BlockGenerator::generateConditionalExecution(
    ScopStmt &Stmt, const isl::set &Subdomain, StringRef Subject,
    const std::function<void()> &GenThenFunc) {
  isl::set StmtDom = Stmt.getDomain();

  // If the condition is a tautology, don't generate a condition around the
  // code.
  bool IsPartialWrite =
      !StmtDom.intersect_params(Stmt.getParent()->getContext())
           .is_subset(Subdomain);
  if (!IsPartialWrite) {
    GenThenFunc();
    return;
  }

````
- **EN**: This block declares or defines routines around `range`, `apply`, `restrict`, `expr_from` (+9 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `range`, `apply`, `restrict`, `expr_from` (+9 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 612-637

````cpp
  // Generate the condition.
  Value *Cond = buildContainsCondition(Stmt, Subdomain);

  // Don't call GenThenFunc if it is never executed. An ast index expression
  // might not be defined in this case.
  if (auto *Const = dyn_cast<ConstantInt>(Cond))
    if (Const->isZero())
      return;

  BasicBlock *HeadBlock = Builder.GetInsertBlock();
  StringRef BlockName = HeadBlock->getName();

  // Generate the conditional block.
  DomTreeUpdater DTU(GenDT, DomTreeUpdater::UpdateStrategy::Eager);
  SplitBlockAndInsertIfThen(Cond, Builder.GetInsertPoint(), false, nullptr,
                            &DTU, GenLI);
  CondBrInst *Branch = cast<CondBrInst>(HeadBlock->getTerminator());
  BasicBlock *ThenBlock = Branch->getSuccessor(0);
  BasicBlock *TailBlock = Branch->getSuccessor(1);

  // Assign descriptive names.
  if (auto *CondInst = dyn_cast<Instruction>(Cond))
    CondInst->setName("polly." + Subject + ".cond");
  ThenBlock->setName(BlockName + "." + Subject + ".partial");
  TailBlock->setName(BlockName + ".cont");

````
- **EN**: This block declares or defines routines around `buildContainsCondition`, `GetInsertBlock`, `getName`, `DTU` (+4 more); contains control flow with 3 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `buildContainsCondition`, `GetInsertBlock`, `getName`, `DTU` (+4 more) 相关的例程; 包含控制流结构：3 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 638-667

````cpp
  // Put the client code into the conditional block and continue in the merge
  // block afterwards.
  Builder.SetInsertPoint(ThenBlock, ThenBlock->getFirstInsertionPt());
  GenThenFunc();
  Builder.SetInsertPoint(TailBlock, TailBlock->getFirstInsertionPt());
}

static std::string getInstName(Value *Val) {
  std::string Result;
  raw_string_ostream OS(Result);
  Val->printAsOperand(OS, false);
  return Result;
}

void BlockGenerator::generateBeginStmtTrace(ScopStmt &Stmt, LoopToScevMapT &LTS,
                                            ValueMapT &BBMap) {
  if (!TraceStmts)
    return;

  Scop *S = Stmt.getParent();
  const char *BaseName = Stmt.getBaseName();

  isl::ast_build AstBuild = Stmt.getAstBuild();
  isl::set Domain = Stmt.getDomain();

  isl::union_map USchedule = AstBuild.get_schedule().intersect_domain(Domain);
  isl::map Schedule = isl::map::from_union_map(USchedule);
  assert(Schedule.is_empty().is_false() &&
         "The stmt must have a valid instance");

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `GenThenFunc`, `getInstName`, `OS` (+8 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `GenThenFunc`, `getInstName`, `OS` (+8 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 668-699

````cpp
  isl::multi_pw_aff ScheduleMultiPwAff =
      isl::pw_multi_aff::from_map(Schedule.reverse());
  isl::ast_build RestrictedBuild = AstBuild.restrict(Schedule.range());

  // Sequence of strings to print.
  SmallVector<llvm::Value *, 8> Values;

  // Print the name of the statement.
  // TODO: Indent by the depth of the statement instance in the schedule tree.
  Values.push_back(RuntimeDebugBuilder::getPrintableString(Builder, BaseName));
  Values.push_back(RuntimeDebugBuilder::getPrintableString(Builder, "("));

  // Add the coordinate of the statement instance.
  for (unsigned i : rangeIslSize(0, ScheduleMultiPwAff.dim(isl::dim::out))) {
    if (i > 0)
      Values.push_back(RuntimeDebugBuilder::getPrintableString(Builder, ","));

    isl::ast_expr IsInSet = RestrictedBuild.expr_from(ScheduleMultiPwAff.at(i));
    Values.push_back(ExprBuilder->create(IsInSet.copy()));
  }

  if (TraceScalars) {
    Values.push_back(RuntimeDebugBuilder::getPrintableString(Builder, ")"));
    DenseSet<Instruction *> Encountered;

    // Add the value of each scalar (and the result of PHIs) used in the
    // statement.
    // TODO: Values used in region-statements.
    for (Instruction *Inst : Stmt.insts()) {
      if (!RuntimeDebugBuilder::isPrintable(Inst->getType()))
        continue;

````
- **EN**: This block declares or defines routines around `from_map`, `restrict`, `push_back`, `expr_from`; contains control flow with 2 loop construct(s), 3 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `from_map`, `restrict`, `push_back`, `expr_from` 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序；并延续周边实现细节。

### Lines 700-733

````cpp
      if (isa<PHINode>(Inst)) {
        Values.push_back(RuntimeDebugBuilder::getPrintableString(Builder, " "));
        Values.push_back(RuntimeDebugBuilder::getPrintableString(
            Builder, getInstName(Inst)));
        Values.push_back(RuntimeDebugBuilder::getPrintableString(Builder, "="));
        Values.push_back(getNewValue(Stmt, Inst, BBMap, LTS,
                                     LI.getLoopFor(Inst->getParent())));
      } else {
        for (Value *Op : Inst->operand_values()) {
          // Do not print values that cannot change during the execution of the
          // SCoP.
          auto *OpInst = dyn_cast<Instruction>(Op);
          if (!OpInst)
            continue;
          if (!S->contains(OpInst))
            continue;

          // Print each scalar at most once, and exclude values defined in the
          // statement itself.
          if (Encountered.count(OpInst))
            continue;

          Values.push_back(
              RuntimeDebugBuilder::getPrintableString(Builder, " "));
          Values.push_back(RuntimeDebugBuilder::getPrintableString(
              Builder, getInstName(OpInst)));
          Values.push_back(
              RuntimeDebugBuilder::getPrintableString(Builder, "="));
          Values.push_back(getNewValue(Stmt, OpInst, BBMap, LTS,
                                       LI.getLoopFor(Inst->getParent())));
          Encountered.insert(OpInst);
        }
      }

````
- **EN**: This block declares or defines routines around `push_back`, `getInstName`, `getLoopFor`, `dyn_cast<Instruction>` (+2 more); contains control flow with 1 loop construct(s), 4 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `push_back`, `getInstName`, `getLoopFor`, `dyn_cast<Instruction>` (+2 more) 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 734-760

````cpp
      Encountered.insert(Inst);
    }

    Values.push_back(RuntimeDebugBuilder::getPrintableString(Builder, "\n"));
  } else {
    Values.push_back(RuntimeDebugBuilder::getPrintableString(Builder, ")\n"));
  }

  RuntimeDebugBuilder::createCPUPrinter(Builder, ArrayRef<Value *>(Values));
}

void BlockGenerator::generateScalarStores(
    ScopStmt &Stmt, LoopToScevMapT &LTS, ValueMapT &BBMap,
    __isl_keep isl_id_to_ast_expr *NewAccesses) {
  Loop *L = LI.getLoopFor(Stmt.getBasicBlock());

  assert(Stmt.isBlockStmt() &&
         "Region statements need to use the generateScalarStores() function in "
         "the RegionGenerator");

  for (MemoryAccess *MA : Stmt) {
    if (MA->isOriginalArrayKind() || MA->isRead())
      continue;

    isl::set AccDom = MA->getAccessRelation().domain();
    std::string Subject = MA->getId().get_name();

````
- **EN**: This block declares or defines routines around `insert`, `push_back`, `createCPUPrinter`, `generateScalarStores` (+3 more); contains control flow with 1 loop construct(s), 1 conditional check(s); adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `insert`, `push_back`, `createCPUPrinter`, `generateScalarStores` (+3 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 761-789

````cpp
    generateConditionalExecution(
        Stmt, AccDom, Subject.c_str(), [&, this, MA]() {
          Value *Val = MA->getAccessValue();
          if (MA->isAnyPHIKind()) {
            assert(MA->getIncoming().size() >= 1 &&
                   "Block statements have exactly one exiting block, or "
                   "multiple but "
                   "with same incoming block and value");
            assert(std::all_of(MA->getIncoming().begin(),
                               MA->getIncoming().end(),
                               [&](std::pair<BasicBlock *, Value *> p) -> bool {
                                 return p.first == Stmt.getBasicBlock();
                               }) &&
                   "Incoming block must be statement's block");
            Val = MA->getIncoming()[0].second;
          }
          auto Address = getImplicitAddress(*MA, getLoopForStmt(Stmt), LTS,
                                            BBMap, NewAccesses);

          Val = getNewValue(Stmt, Val, BBMap, LTS, L);
          assert((!isa<Instruction>(Val) ||
                  GenDT->dominates(cast<Instruction>(Val)->getParent(),
                                   Builder.GetInsertBlock())) &&
                 "Domination violation");
          assert((!isa<Instruction>(Address) ||
                  GenDT->dominates(cast<Instruction>(Address)->getParent(),
                                   Builder.GetInsertBlock())) &&
                 "Domination violation");

````
- **EN**: This block declares or defines routines around `generateConditionalExecution`, `c_str`, `getAccessValue`, `getIncoming` (+4 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `generateConditionalExecution`, `c_str`, `getAccessValue`, `getIncoming` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查.

### Lines 790-815

````cpp
          Builder.CreateStore(Val, Address);
        });
  }
}

void BlockGenerator::createScalarInitialization(Scop &S) {
  BasicBlock *ExitBB = S.getExit();
  BasicBlock *PreEntryBB = S.getEnteringBlock();

  Builder.SetInsertPoint(StartBlock, StartBlock->begin());

  for (auto &Array : S.arrays()) {
    if (Array->getNumberOfDimensions() != 0)
      continue;
    if (Array->isPHIKind()) {
      // For PHI nodes, the only values we need to store are the ones that
      // reach the PHI node from outside the region. In general there should
      // only be one such incoming edge and this edge should enter through
      // 'PreEntryBB'.
      auto PHI = cast<PHINode>(Array->getBasePtr());

      for (auto BI = PHI->block_begin(), BE = PHI->block_end(); BI != BE; BI++)
        if (!S.contains(*BI) && *BI != PreEntryBB)
          llvm_unreachable("Incoming edges from outside the scop should always "
                           "come from PreEntryBB");

````
- **EN**: This block declares or defines routines around `CreateStore`, `createScalarInitialization`, `getExit`, `getEnteringBlock` (+2 more); contains control flow with 2 loop construct(s), 3 conditional check(s); adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `CreateStore`, `createScalarInitialization`, `getExit`, `getEnteringBlock` (+2 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 816-842

````cpp
      int Idx = PHI->getBasicBlockIndex(PreEntryBB);
      if (Idx < 0)
        continue;

      Value *ScalarValue = PHI->getIncomingValue(Idx);

      Builder.CreateStore(ScalarValue, getOrCreateAlloca(Array));
      continue;
    }

    auto *Inst = dyn_cast<Instruction>(Array->getBasePtr());

    if (Inst && S.contains(Inst))
      continue;

    // PHI nodes that are not marked as such in their SAI object are either exit
    // PHI nodes we model as common scalars but without initialization, or
    // incoming phi nodes that need to be initialized. Check if the first is the
    // case for Inst and do not create and initialize memory if so.
    if (auto *PHI = dyn_cast_or_null<PHINode>(Inst))
      if (!S.hasSingleExitEdge() && PHI->getBasicBlockIndex(ExitBB) >= 0)
        continue;

    Builder.CreateStore(Array->getBasePtr(), getOrCreateAlloca(Array));
  }
}

````
- **EN**: This block declares or defines routines around `getBasicBlockIndex`, `getIncomingValue`, `CreateStore`, `dyn_cast<Instruction>`; contains control flow with 4 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getBasicBlockIndex`, `getIncomingValue`, `CreateStore`, `dyn_cast<Instruction>` 相关的例程; 包含控制流结构：4 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 843-869

````cpp
void BlockGenerator::createScalarFinalization(Scop &S) {
  // The exit block of the __unoptimized__ region.
  BasicBlock *ExitBB = S.getExitingBlock();
  // The merge block __just after__ the region and the optimized region.
  BasicBlock *MergeBB = S.getExit();

  // The exit block of the __optimized__ region.
  BasicBlock *OptExitBB = *(pred_begin(MergeBB));
  if (OptExitBB == ExitBB)
    OptExitBB = *(++pred_begin(MergeBB));

  Builder.SetInsertPoint(OptExitBB, OptExitBB->getTerminator()->getIterator());
  for (const auto &EscapeMapping : EscapeMap) {
    // Extract the escaping instruction and the escaping users as well as the
    // alloca the instruction was demoted to.
    Instruction *EscapeInst = EscapeMapping.first;
    const auto &EscapeMappingValue = EscapeMapping.second;
    const EscapeUserVectorTy &EscapeUsers = EscapeMappingValue.second;
    auto *ScalarAddr = cast<AllocaInst>(&*EscapeMappingValue.first);

    // Reload the demoted instruction in the optimized version of the SCoP.
    Value *EscapeInstReload =
        Builder.CreateLoad(ScalarAddr->getAllocatedType(), ScalarAddr,
                           EscapeInst->getName() + ".final_reload");
    EscapeInstReload =
        Builder.CreateBitOrPointerCast(EscapeInstReload, EscapeInst->getType());

````
- **EN**: This block declares or defines routines around `createScalarFinalization`, `getExitingBlock`, `getExit`, `pred_begin` (+5 more); contains control flow with 1 loop construct(s), 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `createScalarFinalization`, `getExitingBlock`, `getExit`, `pred_begin` (+5 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 870-895

````cpp
    // Create the merge PHI that merges the optimized and unoptimized version.
    PHINode *MergePHI = PHINode::Create(EscapeInst->getType(), 2,
                                        EscapeInst->getName() + ".merge");
    MergePHI->insertBefore(MergeBB->getFirstInsertionPt());

    // Add the respective values to the merge PHI.
    MergePHI->addIncoming(EscapeInstReload, OptExitBB);
    MergePHI->addIncoming(EscapeInst, ExitBB);

    // The information of scalar evolution about the escaping instruction needs
    // to be revoked so the new merged instruction will be used.
    if (SE.isSCEVable(EscapeInst->getType()))
      SE.forgetValue(EscapeInst);

    // Replace all uses of the demoted instruction with the merge PHI.
    for (Instruction *EUser : EscapeUsers)
      EUser->replaceUsesOfWith(EscapeInst, MergePHI);
  }
}

void BlockGenerator::findOutsideUsers(Scop &S) {
  for (auto &Array : S.arrays()) {

    if (Array->getNumberOfDimensions() != 0)
      continue;

````
- **EN**: This block declares or defines routines around `Create`, `getName`, `insertBefore`, `addIncoming` (+3 more); contains control flow with 2 loop construct(s), 2 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `Create`, `getName`, `insertBefore`, `addIncoming` (+3 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 896-924

````cpp
    if (Array->isPHIKind())
      continue;

    auto *Inst = dyn_cast<Instruction>(Array->getBasePtr());

    if (!Inst)
      continue;

    // Scop invariant hoisting moves some of the base pointers out of the scop.
    // We can ignore these, as the invariant load hoisting already registers the
    // relevant outside users.
    if (!S.contains(Inst))
      continue;

    handleOutsideUsers(S, Array);
  }
}

void BlockGenerator::createExitPHINodeMerges(Scop &S) {
  if (S.hasSingleExitEdge())
    return;

  auto *ExitBB = S.getExitingBlock();
  auto *MergeBB = S.getExit();
  auto *AfterMergeBB = MergeBB->getSingleSuccessor();
  BasicBlock *OptExitBB = *(pred_begin(MergeBB));
  if (OptExitBB == ExitBB)
    OptExitBB = *(++pred_begin(MergeBB));

````
- **EN**: This block declares or defines routines around `dyn_cast<Instruction>`, `handleOutsideUsers`, `createExitPHINodeMerges`, `getExitingBlock` (+3 more); contains control flow with 5 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `dyn_cast<Instruction>`, `handleOutsideUsers`, `createExitPHINodeMerges`, `getExitingBlock` (+3 more) 相关的例程; 包含控制流结构：5 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 925-961

````cpp
  Builder.SetInsertPoint(OptExitBB, OptExitBB->getTerminator()->getIterator());

  for (auto &SAI : S.arrays()) {
    auto *Val = SAI->getBasePtr();

    // Only Value-like scalars need a merge PHI. Exit block PHIs receive either
    // the original PHI's value or the reloaded incoming values from the
    // generated code. An llvm::Value is merged between the original code's
    // value or the generated one.
    if (!SAI->isExitPHIKind())
      continue;

    PHINode *PHI = dyn_cast<PHINode>(Val);
    if (!PHI)
      continue;

    if (PHI->getParent() != AfterMergeBB)
      continue;

    std::string Name = PHI->getName().str();
    Value *ScalarAddr = getOrCreateAlloca(SAI);
    Value *Reload = Builder.CreateLoad(SAI->getElementType(), ScalarAddr,
                                       Name + ".ph.final_reload");
    Reload = Builder.CreateBitOrPointerCast(Reload, PHI->getType());
    Value *OriginalValue = PHI->getIncomingValueForBlock(MergeBB);
    assert((!isa<Instruction>(OriginalValue) ||
            cast<Instruction>(OriginalValue)->getParent() != MergeBB) &&
           "Original value must no be one we just generated.");
    auto *MergePHI = PHINode::Create(PHI->getType(), 2, Name + ".ph.merge");
    MergePHI->insertBefore(MergeBB->getFirstInsertionPt());
    MergePHI->addIncoming(Reload, OptExitBB);
    MergePHI->addIncoming(OriginalValue, ExitBB);
    int Idx = PHI->getBasicBlockIndex(MergeBB);
    PHI->setIncomingValue(Idx, MergePHI);
  }
}

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `getBasePtr`, `dyn_cast<PHINode>`, `getName` (+10 more); contains control flow with 1 loop construct(s), 3 conditional check(s); adds defensive checks for invariants or impossible states; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `getBasePtr`, `dyn_cast<PHINode>`, `getName` (+10 more) 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 保留用于解释意图、用法或算法背景的注释.

### Lines 962-988

````cpp
void BlockGenerator::invalidateScalarEvolution(Scop &S) {
  for (auto &Stmt : S)
    if (Stmt.isCopyStmt())
      continue;
    else if (Stmt.isBlockStmt())
      for (auto &Inst : *Stmt.getBasicBlock())
        SE.forgetValue(&Inst);
    else if (Stmt.isRegionStmt())
      for (auto *BB : Stmt.getRegion()->blocks())
        for (auto &Inst : *BB)
          SE.forgetValue(&Inst);
    else
      llvm_unreachable("Unexpected statement type found");

  // Invalidate SCEV of loops surrounding the EscapeUsers.
  for (const auto &EscapeMapping : EscapeMap) {
    const EscapeUserVectorTy &EscapeUsers = EscapeMapping.second.second;
    for (Instruction *EUser : EscapeUsers) {
      if (Loop *L = LI.getLoopFor(EUser->getParent()))
        while (L) {
          SE.forgetLoop(L);
          L = L->getParentLoop();
        }
    }
  }
}

````
- **EN**: This block declares or defines routines around `invalidateScalarEvolution`, `forgetValue`, `forgetLoop`, `getParentLoop`; contains control flow with 7 loop construct(s), 4 conditional check(s); adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `invalidateScalarEvolution`, `forgetValue`, `forgetLoop`, `getParentLoop` 相关的例程; 包含控制流结构：7 处循环、4 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 989-1023

````cpp
void BlockGenerator::finalizeSCoP(Scop &S) {
  findOutsideUsers(S);
  createScalarInitialization(S);
  createExitPHINodeMerges(S);
  createScalarFinalization(S);
  invalidateScalarEvolution(S);
}

BasicBlock *RegionGenerator::repairDominance(BasicBlock *BB,
                                             BasicBlock *BBCopy) {

  BasicBlock *BBIDom = DT.getNode(BB)->getIDom()->getBlock();
  BasicBlock *BBCopyIDom = EndBlockMap.lookup(BBIDom);

  if (BBCopyIDom)
    GenDT->changeImmediateDominator(BBCopy, BBCopyIDom);

  return StartBlockMap.lookup(BBIDom);
}

// This is to determine whether an llvm::Value (defined in @p BB) is usable when
// leaving a subregion. The straight-forward DT.dominates(BB, R->getExitBlock())
// does not work in cases where the exit block has edges from outside the
// region. In that case the llvm::Value would never be usable in in the exit
// block. The RegionGenerator however creates an new exit block ('ExitBBCopy')
// for the subregion's exiting edges only. We need to determine whether an
// llvm::Value is usable in there. We do this by checking whether it dominates
// all exiting blocks individually.
static bool isDominatingSubregionExit(const DominatorTree &DT, Region *R,
                                      BasicBlock *BB) {
  for (auto ExitingBB : predecessors(R->getExit())) {
    // Check for non-subregion incoming edges.
    if (!R->contains(ExitingBB))
      continue;

````
- **EN**: This block declares or defines routines around `finalizeSCoP`, `findOutsideUsers`, `createScalarInitialization`, `createExitPHINodeMerges` (+7 more); contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `finalizeSCoP`, `findOutsideUsers`, `createScalarInitialization`, `createExitPHINodeMerges` (+7 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 1024-1052

````cpp
    if (!DT.dominates(BB, ExitingBB))
      return false;
  }

  return true;
}

// Find the direct dominator of the subregion's exit block if the subregion was
// simplified.
static BasicBlock *findExitDominator(DominatorTree &DT, Region *R) {
  BasicBlock *Common = nullptr;
  for (auto ExitingBB : predecessors(R->getExit())) {
    // Check for non-subregion incoming edges.
    if (!R->contains(ExitingBB))
      continue;

    // First exiting edge.
    if (!Common) {
      Common = ExitingBB;
      continue;
    }

    Common = DT.findNearestCommonDominator(Common, ExitingBB);
  }

  assert(Common && R->contains(Common));
  return Common;
}

````
- **EN**: This block declares or defines routines around `findExitDominator`, `findNearestCommonDominator`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `findExitDominator`, `findNearestCommonDominator` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 1053-1081

````cpp
void RegionGenerator::copyStmt(ScopStmt &Stmt, LoopToScevMapT &LTS,
                               __isl_keep isl_id_to_ast_expr *IdToAstExp) {
  assert(Stmt.isRegionStmt() &&
         "Only region statements can be copied by the region generator");

  // Forget all old mappings.
  StartBlockMap.clear();
  EndBlockMap.clear();
  RegionMaps.clear();
  IncompletePHINodeMap.clear();

  // Collection of all values related to this subregion.
  ValueMapT ValueMap;

  // The region represented by the statement.
  Region *R = Stmt.getRegion();

  // Create a dedicated entry for the region where we can reload all demoted
  // inputs.
  BasicBlock *EntryBB = R->getEntry();
  BasicBlock *EntryBBCopy = SplitBlock(Builder.GetInsertBlock(),
                                       Builder.GetInsertPoint(), GenDT, GenLI);
  EntryBBCopy->setName("polly.stmt." + EntryBB->getName() + ".entry");
  Builder.SetInsertPoint(EntryBBCopy, EntryBBCopy->begin());

  ValueMapT &EntryBBMap = RegionMaps[EntryBBCopy];
  generateScalarLoads(Stmt, LTS, EntryBBMap, IdToAstExp);
  generateBeginStmtTrace(Stmt, LTS, EntryBBMap);

````
- **EN**: This block declares or defines routines around `copyStmt`, `clear`, `getRegion`, `getEntry` (+6 more); adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `copyStmt`, `clear`, `getRegion`, `getEntry` (+6 more) 相关的例程; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 1082-1114

````cpp
  for (auto PI = pred_begin(EntryBB), PE = pred_end(EntryBB); PI != PE; ++PI)
    if (!R->contains(*PI)) {
      StartBlockMap[*PI] = EntryBBCopy;
      EndBlockMap[*PI] = EntryBBCopy;
    }

  // Iterate over all blocks in the region in a breadth-first search.
  std::deque<BasicBlock *> Blocks;
  SmallSetVector<BasicBlock *, 8> SeenBlocks;
  Blocks.push_back(EntryBB);
  SeenBlocks.insert(EntryBB);

  while (!Blocks.empty()) {
    BasicBlock *BB = Blocks.front();
    Blocks.pop_front();

    // First split the block and update dominance information.
    BasicBlock *BBCopy = splitBB(BB);
    BasicBlock *BBCopyIDom = repairDominance(BB, BBCopy);

    // Get the mapping for this block and initialize it with either the scalar
    // loads from the generated entering block (which dominates all blocks of
    // this subregion) or the maps of the immediate dominator, if part of the
    // subregion. The latter necessarily includes the former.
    ValueMapT *InitBBMap;
    if (BBCopyIDom) {
      assert(RegionMaps.count(BBCopyIDom));
      InitBBMap = &RegionMaps[BBCopyIDom];
    } else
      InitBBMap = &EntryBBMap;
    auto Inserted = RegionMaps.insert(std::make_pair(BBCopy, *InitBBMap));
    ValueMapT &RegionMap = Inserted.first->second;

````
- **EN**: This block declares or defines routines around `push_back`, `insert`, `front`, `pop_front` (+2 more); contains control flow with 2 loop construct(s), 2 conditional check(s); adds defensive checks for invariants or impossible states; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `push_back`, `insert`, `front`, `pop_front` (+2 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 保留用于解释意图、用法或算法背景的注释.

### Lines 1115-1144

````cpp
    // Copy the block with the BlockGenerator.
    Builder.SetInsertPoint(BBCopy, BBCopy->begin());
    copyBB(Stmt, BB, BBCopy, RegionMap, LTS, IdToAstExp);

    // In order to remap PHI nodes we store also basic block mappings.
    StartBlockMap[BB] = BBCopy;
    EndBlockMap[BB] = Builder.GetInsertBlock();

    // Add values to incomplete PHI nodes waiting for this block to be copied.
    for (const PHINodePairTy &PHINodePair : IncompletePHINodeMap[BB])
      addOperandToPHI(Stmt, PHINodePair.first, PHINodePair.second, BB, LTS);
    IncompletePHINodeMap[BB].clear();

    // And continue with new successors inside the region.
    for (auto SI = succ_begin(BB), SE = succ_end(BB); SI != SE; SI++)
      if (R->contains(*SI) && SeenBlocks.insert(*SI))
        Blocks.push_back(*SI);

    // Remember value in case it is visible after this subregion.
    if (isDominatingSubregionExit(DT, R, BB))
      ValueMap.insert_range(RegionMap);
  }

  // Now create a new dedicated region exit block and add it to the region map.
  BasicBlock *ExitBBCopy = SplitBlock(Builder.GetInsertBlock(),
                                      Builder.GetInsertPoint(), GenDT, GenLI);
  ExitBBCopy->setName("polly.stmt." + R->getExit()->getName() + ".exit");
  StartBlockMap[R->getExit()] = ExitBBCopy;
  EndBlockMap[R->getExit()] = ExitBBCopy;

````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `copyBB`, `GetInsertBlock`, `addOperandToPHI` (+7 more); contains control flow with 2 loop construct(s), 2 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `copyBB`, `GetInsertBlock`, `addOperandToPHI` (+7 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 1145-1174

````cpp
  BasicBlock *ExitDomBBCopy = EndBlockMap.lookup(findExitDominator(DT, R));
  assert(ExitDomBBCopy &&
         "Common exit dominator must be within region; at least the entry node "
         "must match");
  GenDT->changeImmediateDominator(ExitBBCopy, ExitDomBBCopy);

  // As the block generator doesn't handle control flow we need to add the
  // region control flow by hand after all blocks have been copied.
  for (BasicBlock *BB : SeenBlocks) {

    BasicBlock *BBCopyStart = StartBlockMap[BB];
    BasicBlock *BBCopyEnd = EndBlockMap[BB];
    Instruction *TI = BB->getTerminator();
    if (isa<UnreachableInst>(TI)) {
      while (!BBCopyEnd->empty())
        BBCopyEnd->begin()->eraseFromParent();
      new UnreachableInst(BBCopyEnd->getContext(), BBCopyEnd);
      continue;
    }

    Instruction *BICopy = BBCopyEnd->getTerminator();

    ValueMapT &RegionMap = RegionMaps[BBCopyStart];
    RegionMap.insert_range(StartBlockMap);

    Builder.SetInsertPoint(BBCopyEnd, BICopy->getIterator());
    copyInstScalar(Stmt, TI, RegionMap, LTS);
    BICopy->eraseFromParent();
  }

````
- **EN**: This block declares or defines routines around `lookup`, `changeImmediateDominator`, `getTerminator`, `begin` (+5 more); contains control flow with 2 loop construct(s), 1 conditional check(s); adds defensive checks for invariants or impossible states; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `lookup`, `changeImmediateDominator`, `getTerminator`, `begin` (+5 more) 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 保留用于解释意图、用法或算法背景的注释.

### Lines 1175-1203

````cpp
  // Add counting PHI nodes to all loops in the region that can be used as
  // replacement for SCEVs referring to the old loop.
  for (BasicBlock *BB : SeenBlocks) {
    Loop *L = LI.getLoopFor(BB);
    if (L == nullptr || L->getHeader() != BB || !R->contains(L))
      continue;

    BasicBlock *BBCopy = StartBlockMap[BB];
    Value *NullVal = Builder.getInt32(0);
    PHINode *LoopPHI =
        PHINode::Create(Builder.getInt32Ty(), 2, "polly.subregion.iv");
    Instruction *LoopPHIInc = BinaryOperator::CreateAdd(
        LoopPHI, Builder.getInt32(1), "polly.subregion.iv.inc");
    LoopPHI->insertBefore(BBCopy->begin());
    LoopPHIInc->insertBefore(BBCopy->getTerminator()->getIterator());

    for (auto *PredBB : predecessors(BB)) {
      if (!R->contains(PredBB))
        continue;
      if (L->contains(PredBB))
        LoopPHI->addIncoming(LoopPHIInc, EndBlockMap[PredBB]);
      else
        LoopPHI->addIncoming(NullVal, EndBlockMap[PredBB]);
    }

    for (auto *PredBBCopy : predecessors(BBCopy))
      if (LoopPHI->getBasicBlockIndex(PredBBCopy) < 0)
        LoopPHI->addIncoming(NullVal, PredBBCopy);

````
- **EN**: This block declares or defines routines around `getLoopFor`, `getInt32`, `Create`, `CreateAdd` (+2 more); contains control flow with 3 loop construct(s), 4 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getLoopFor`, `getInt32`, `Create`, `CreateAdd` (+2 more) 相关的例程; 包含控制流结构：3 处循环、4 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 1204-1235

````cpp
    LTS[L] = SE.getUnknown(LoopPHI);
  }

  // Continue generating code in the exit block.
  Builder.SetInsertPoint(ExitBBCopy, ExitBBCopy->getFirstInsertionPt());

  // Write values visible to other statements.
  generateScalarStores(Stmt, LTS, ValueMap, IdToAstExp);
  StartBlockMap.clear();
  EndBlockMap.clear();
  RegionMaps.clear();
  IncompletePHINodeMap.clear();
}

PHINode *RegionGenerator::buildExitPHI(MemoryAccess *MA, LoopToScevMapT &LTS,
                                       ValueMapT &BBMap, Loop *L) {
  ScopStmt *Stmt = MA->getStatement();
  Region *SubR = Stmt->getRegion();
  auto Incoming = MA->getIncoming();

  PollyIRBuilder::InsertPointGuard IPGuard(Builder);
  PHINode *OrigPHI = cast<PHINode>(MA->getAccessInstruction());
  BasicBlock *NewSubregionExit = Builder.GetInsertBlock();

  // This can happen if the subregion is simplified after the ScopStmts
  // have been created; simplification happens as part of CodeGeneration.
  if (OrigPHI->getParent() != SubR->getExit()) {
    BasicBlock *FormerExit = SubR->getExitingBlock();
    if (FormerExit)
      NewSubregionExit = StartBlockMap.lookup(FormerExit);
  }

````
- **EN**: This block declares or defines routines around `getUnknown`, `SetInsertPoint`, `generateScalarStores`, `clear` (+9 more); contains control flow with 2 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getUnknown`, `SetInsertPoint`, `generateScalarStores`, `clear` (+9 more) 相关的例程; 包含控制流结构：2 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系；并延续周边实现细节。

### Lines 1236-1263

````cpp
  PHINode *NewPHI = PHINode::Create(OrigPHI->getType(), Incoming.size(),
                                    "polly." + OrigPHI->getName(),
                                    NewSubregionExit->getFirstNonPHIIt());

  // Add the incoming values to the PHI.
  for (auto &Pair : Incoming) {
    BasicBlock *OrigIncomingBlock = Pair.first;
    BasicBlock *NewIncomingBlockStart = StartBlockMap.lookup(OrigIncomingBlock);
    BasicBlock *NewIncomingBlockEnd = EndBlockMap.lookup(OrigIncomingBlock);
    Builder.SetInsertPoint(NewIncomingBlockEnd,
                           NewIncomingBlockEnd->getTerminator()->getIterator());
    assert(RegionMaps.count(NewIncomingBlockStart));
    assert(RegionMaps.count(NewIncomingBlockEnd));
    ValueMapT *LocalBBMap = &RegionMaps[NewIncomingBlockStart];

    Value *OrigIncomingValue = Pair.second;
    Value *NewIncomingValue =
        getNewValue(*Stmt, OrigIncomingValue, *LocalBBMap, LTS, L);
    NewPHI->addIncoming(NewIncomingValue, NewIncomingBlockEnd);
  }

  return NewPHI;
}

Value *RegionGenerator::getExitScalar(MemoryAccess *MA, LoopToScevMapT &LTS,
                                      ValueMapT &BBMap) {
  ScopStmt *Stmt = MA->getStatement();

````
- **EN**: This block declares or defines routines around `Create`, `getName`, `getFirstNonPHIIt`, `lookup` (+6 more); contains control flow with 1 loop construct(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `Create`, `getName`, `getFirstNonPHIIt`, `lookup` (+6 more) 相关的例程; 包含控制流结构：1 处循环; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 1264-1293

````cpp
  // TODO: Add some test cases that ensure this is really the right choice.
  Loop *L = LI.getLoopFor(Stmt->getRegion()->getExit());

  if (MA->isAnyPHIKind()) {
    auto Incoming = MA->getIncoming();
    assert(!Incoming.empty() &&
           "PHI WRITEs must have originate from at least one incoming block");

    // If there is only one incoming value, we do not need to create a PHI.
    if (Incoming.size() == 1) {
      Value *OldVal = Incoming[0].second;
      return getNewValue(*Stmt, OldVal, BBMap, LTS, L);
    }

    return buildExitPHI(MA, LTS, BBMap, L);
  }

  // MemoryKind::Value accesses leaving the subregion must dominate the exit
  // block; just pass the copied value.
  Value *OldVal = MA->getAccessValue();
  return getNewValue(*Stmt, OldVal, BBMap, LTS, L);
}

void RegionGenerator::generateScalarStores(
    ScopStmt &Stmt, LoopToScevMapT &LTS, ValueMapT &BBMap,
    __isl_keep isl_id_to_ast_expr *NewAccesses) {
  assert(Stmt.getRegion() &&
         "Block statements need to use the generateScalarStores() "
         "function in the BlockGenerator");

````
- **EN**: This block declares or defines routines around `getLoopFor`, `getIncoming`, `getAccessValue`, `generateScalarStores`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getLoopFor`, `getIncoming`, `getAccessValue`, `generateScalarStores` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 1294-1325

````cpp
  // Get the exit scalar values before generating the writes.
  // This is necessary because RegionGenerator::getExitScalar may insert
  // PHINodes that depend on the region's exiting blocks. But
  // BlockGenerator::generateConditionalExecution may insert a new basic block
  // such that the current basic block is not a direct successor of the exiting
  // blocks anymore. Hence, build the PHINodes while the current block is still
  // the direct successor.
  SmallDenseMap<MemoryAccess *, Value *> NewExitScalars;
  for (MemoryAccess *MA : Stmt) {
    if (MA->isOriginalArrayKind() || MA->isRead())
      continue;

    Value *NewVal = getExitScalar(MA, LTS, BBMap);
    NewExitScalars[MA] = NewVal;
  }

  for (MemoryAccess *MA : Stmt) {
    if (MA->isOriginalArrayKind() || MA->isRead())
      continue;

    isl::set AccDom = MA->getAccessRelation().domain();
    std::string Subject = MA->getId().get_name();
    generateConditionalExecution(
        Stmt, AccDom, Subject.c_str(), [&, this, MA]() {
          Value *NewVal = NewExitScalars.lookup(MA);
          assert(NewVal && "The exit scalar must be determined before");
          Value *Address = getImplicitAddress(*MA, getLoopForStmt(Stmt), LTS,
                                              BBMap, NewAccesses);
          assert((!isa<Instruction>(NewVal) ||
                  GenDT->dominates(cast<Instruction>(NewVal)->getParent(),
                                   Builder.GetInsertBlock())) &&
                 "Domination violation");
````
- **EN**: This block declares or defines routines around `getExitScalar`, `getAccessRelation`, `getId`, `generateConditionalExecution` (+5 more); contains control flow with 2 loop construct(s), 2 conditional check(s); adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getExitScalar`, `getAccessRelation`, `getId`, `generateConditionalExecution` (+5 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1326-1353

````cpp
          assert((!isa<Instruction>(Address) ||
                  GenDT->dominates(cast<Instruction>(Address)->getParent(),
                                   Builder.GetInsertBlock())) &&
                 "Domination violation");
          Builder.CreateStore(NewVal, Address);
        });
  }
}

void RegionGenerator::addOperandToPHI(ScopStmt &Stmt, PHINode *PHI,
                                      PHINode *PHICopy, BasicBlock *IncomingBB,
                                      LoopToScevMapT &LTS) {
  // If the incoming block was not yet copied mark this PHI as incomplete.
  // Once the block will be copied the incoming value will be added.
  BasicBlock *BBCopyStart = StartBlockMap[IncomingBB];
  BasicBlock *BBCopyEnd = EndBlockMap[IncomingBB];
  if (!BBCopyStart) {
    assert(!BBCopyEnd);
    assert(Stmt.represents(IncomingBB) &&
           "Bad incoming block for PHI in non-affine region");
    IncompletePHINodeMap[IncomingBB].push_back(std::make_pair(PHI, PHICopy));
    return;
  }

  assert(RegionMaps.count(BBCopyStart) &&
         "Incoming PHI block did not have a BBMap");
  ValueMapT &BBCopyMap = RegionMaps[BBCopyStart];

````
- **EN**: This block declares or defines routines around `dominates`, `GetInsertBlock`, `CreateStore`, `addOperandToPHI` (+1 more); contains control flow with 1 conditional check(s); adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `dominates`, `GetInsertBlock`, `CreateStore`, `addOperandToPHI` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 1354-1379

````cpp
  Value *OpCopy = nullptr;

  if (Stmt.represents(IncomingBB)) {
    Value *Op = PHI->getIncomingValueForBlock(IncomingBB);

    // If the current insert block is different from the PHIs incoming block
    // change it, otherwise do not.
    auto IP = Builder.GetInsertPoint();
    if (IP->getParent() != BBCopyEnd)
      Builder.SetInsertPoint(BBCopyEnd,
                             BBCopyEnd->getTerminator()->getIterator());
    OpCopy = getNewValue(Stmt, Op, BBCopyMap, LTS, getLoopForStmt(Stmt));
    if (IP->getParent() != BBCopyEnd)
      Builder.SetInsertPoint(IP);
  } else {
    // All edges from outside the non-affine region become a single edge
    // in the new copy of the non-affine region. Make sure to only add the
    // corresponding edge the first time we encounter a basic block from
    // outside the non-affine region.
    if (PHICopy->getBasicBlockIndex(BBCopyEnd) >= 0)
      return;

    // Get the reloaded value.
    OpCopy = getNewValue(Stmt, PHI, BBCopyMap, LTS, getLoopForStmt(Stmt));
  }

````
- **EN**: This block declares or defines routines around `getIncomingValueForBlock`, `GetInsertPoint`, `SetInsertPoint`, `getTerminator` (+1 more); contains control flow with 4 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getIncomingValueForBlock`, `GetInsertPoint`, `SetInsertPoint`, `getTerminator` (+1 more) 相关的例程; 包含控制流结构：4 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 1380-1395

````cpp
  assert(OpCopy && "Incoming PHI value was not copied properly");
  PHICopy->addIncoming(OpCopy, BBCopyEnd);
}

void RegionGenerator::copyPHIInstruction(ScopStmt &Stmt, PHINode *PHI,
                                         ValueMapT &BBMap,
                                         LoopToScevMapT &LTS) {
  unsigned NumIncoming = PHI->getNumIncomingValues();
  PHINode *PHICopy =
      Builder.CreatePHI(PHI->getType(), NumIncoming, "polly." + PHI->getName());
  PHICopy->moveBefore(PHICopy->getParent()->getFirstNonPHIIt());
  BBMap[PHI] = PHICopy;

  for (BasicBlock *IncomingBB : PHI->blocks())
    addOperandToPHI(Stmt, PHI, PHICopy, IncomingBB, LTS);
}
````
- **EN**: This block declares or defines routines around `addIncoming`, `copyPHIInstruction`, `getNumIncomingValues`, `CreatePHI` (+2 more); contains control flow with 1 loop construct(s); adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `addIncoming`, `copyPHIInstruction`, `getNumIncomingValues`, `CreatePHI` (+2 more) 相关的例程; 包含控制流结构：1 处循环; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

## Key Concepts / 关键概念

- **Code generation**
  - **CN**: 代码生成
- **Loop transformation**
  - **CN**: 循环变换
- **SCoP modeling**
  - **CN**: SCoP 建模
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/BlockGenerators.h`, `polly/CodeGen/IslExprBuilder.h`, `polly/CodeGen/RuntimeDebugBuilder.h`, `polly/Options.h`, `polly/ScopInfo.h`, `polly/Support/ISLTools.h`, `polly/Support/ScopHelper.h`, `polly/Support/VirtualInstruction.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/BlockGenerators.h`, `polly/CodeGen/IslExprBuilder.h`, `polly/CodeGen/RuntimeDebugBuilder.h`, `polly/Options.h`, `polly/ScopInfo.h`, `polly/Support/ISLTools.h`, `polly/Support/ScopHelper.h`, `polly/Support/VirtualInstruction.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/RegionInfo.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Local.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/RegionInfo.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Local.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
- **ISL headers**: `isl/ast.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/ast.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `deque` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`deque` —— 实现所需的标准库或系统声明。
