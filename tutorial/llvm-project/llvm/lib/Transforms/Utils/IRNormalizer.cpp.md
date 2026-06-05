# IRNormalizer.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/IRNormalizer.cpp` | `llvm/lib/Transforms/Utils/IRNormalizer.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements iR Normalizer within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 IRNormalizer 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===--------------- IRNormalizer.cpp - IR Normalizer ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// This file implements the IRNormalizer class which aims to transform LLVM
/// Modules into a normal form by reordering and renaming instructions while
/// preserving the same semantics. The normalizer makes it easier to spot
/// semantic differences while diffing two modules which have undergone
/// different passes.
///
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/IRNormalizer.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallString.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// IRNormalizer aims to transform LLVM IR into normal form.
class IRNormalizer {
public:
  bool runOnFunction(Function &F);

  IRNormalizer(IRNormalizerOptions Options) : Options(Options) {}

private:
  const IRNormalizerOptions Options;

  // Random constant for hashing, so the state isn't zero.
  const uint64_t MagicHashConstant = 0x6acaa36bef8325c5ULL;
  DenseSet<const Instruction *> NamedInstructions;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 61-119

```cpp
  SmallVector<Instruction *, 16> Outputs;

  /// \name Naming.
  /// @{
  void nameFunctionArguments(Function &F) const;
  void nameBasicBlocks(Function &F) const;
  void nameInstruction(Instruction *I);
  void nameAsInitialInstruction(Instruction *I) const;
  void nameAsRegularInstruction(Instruction *I);
  void foldInstructionName(Instruction *I) const;
  /// @}

  /// \name Reordering.
  /// @{
  void reorderInstructions(Function &F) const;
  void reorderDefinition(Instruction *Definition,
                         std::stack<Instruction *> &TopologicalSort,
                         SmallPtrSet<const Instruction *, 32> &Visited) const;
  void reorderInstructionOperandsByNames(Instruction *I) const;
  void reorderPHIIncomingValues(PHINode *Phi) const;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  if (!Options.PreserveOrder)
    reorderInstructions(F);

  // TODO: Reorder basic blocks via a topological sort.

  for (auto &I : Outputs)
    nameInstruction(I);

  for (auto &I : instructions(F)) {
    if (!Options.PreserveOrder) {
      if (Options.ReorderOperands)
        reorderInstructionOperandsByNames(&I);

```
- EN: Core entities appearing here include runOnFunction, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 120-182

```cpp
      if (auto *Phi = dyn_cast<PHINode>(&I))
        reorderPHIIncomingValues(Phi);
    }
    foldInstructionName(&I);
  }

  return true;
}

/// Numbers arguments.
///
/// \param F Function whose arguments will be renamed.
void IRNormalizer::nameFunctionArguments(Function &F) const {
  int ArgumentCounter = 0;
  for (auto &A : F.args()) {
    if (Options.RenameAll || A.getName().empty()) {
      A.setName("a" + Twine(ArgumentCounter));
      ArgumentCounter += 1;
    }
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Ensure instructions are not renamed. This is done
  // to prevent situation where instructions are used
  // before their definition (in phi nodes)
  if (NamedInstructions.contains(I))
    return;
  NamedInstructions.insert(I);
  if (isInitialInstruction(I)) {
    nameAsInitialInstruction(I);
  } else {
    // This must be a regular instruction.
    nameAsRegularInstruction(I);
  }
}

```
- EN: Core entities appearing here include nameFunctionArguments, nameBasicBlocks, nameInstruction, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 nameFunctionArguments, nameBasicBlocks, nameInstruction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 183-240

```cpp
template <typename T>
void IRNormalizer::sortCommutativeOperands(Instruction *I, T &Operands) const {
  if (!(I->isCommutative() && Operands.size() >= 2))
    return;
  auto CommutativeEnd = Operands.begin();
  std::advance(CommutativeEnd, 2);
  llvm::sort(Operands.begin(), CommutativeEnd);
}

/// Names instruction following the scheme:
/// vl00000Callee(Operands)
///
/// Where 00000 is a hash calculated considering instruction's opcode and output
/// footprint. Callee's name is only included when instruction's type is
/// CallInst. In cases where instruction is commutative, operands list is also
/// sorted.
///
/// Renames instruction only when RenameAll flag is raised or instruction is
/// unnamed.
///
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Initialize to a magic constant, so the state isn't zero.
  uint64_t Hash = MagicHashConstant;

  // Consider instruction's opcode in the hash.
  Hash = hash_16_bytes(Hash, I->getOpcode());

  SmallPtrSet<const Instruction *, 32> Visited;
  // Get output footprint for I.
  SetVector<int> OutputFootprint = getOutputFootprint(I, Visited);

  // Consider output footprint in the hash.
  for (const int &Output : OutputFootprint)
    Hash = hash_16_bytes(Hash, Output);

```
- EN: Core entities appearing here include sortCommutativeOperands, nameAsInitialInstruction, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 sortCommutativeOperands, nameAsInitialInstruction，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 241-292

```cpp
  // Base instruction name.
  SmallString<256> Name;
  Name.append("vl" + std::to_string(Hash).substr(0, 5));

  // In case of CallInst, consider callee in the instruction name.
  if (const auto *CI = dyn_cast<CallInst>(I)) {
    Function *F = CI->getCalledFunction();

    if (F != nullptr)
      Name.append(F->getName());
  }

  Name.append("(");
  for (size_t i = 0; i < Operands.size(); ++i) {
    Name.append(Operands[i]);

    if (i < Operands.size() - 1)
      Name.append(", ");
  }
  Name.append(")");

  I->setName(Name);
}

/// Names instruction following the scheme:
/// op00000Callee(Operands)
///
/// Where 00000 is a hash calculated considering instruction's opcode, its
/// operands' opcodes and order. Callee's name is only included when
/// instruction's type is CallInst. In cases where instruction is commutative,
/// operand list is also sorted.
///
/// Names instructions recursively in accordance with the def-use tree,
/// starting from the initial instructions (defs), finishing at
/// the output (top-most user) instructions (depth-first).
///
/// Renames instruction only when RenameAll flag is raised or instruction is
/// unnamed.
///
/// \see getOutputFootprint()
/// \param I Instruction to be renamed.
void IRNormalizer::nameAsRegularInstruction(Instruction *I) {
  LLVM_DEBUG(dbgs() << "Naming regular instruction: " << *I << "\n");

  // Instruction operands for further sorting.
  SmallVector<SmallString<128>, 4> Operands;

  // The name of a regular instruction depends
  // on the names of its operands. Hence, all
  // operands must be named first in the use-def
  // walk.

```
- EN: Core entities appearing here include nameAsRegularInstruction, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 nameAsRegularInstruction，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 293-351

```cpp
  // Collect operands.
  for (auto &Op : I->operands()) {
    if (auto *I = dyn_cast<Instruction>(Op)) {
      // Walk down the use-def chain.
      nameInstruction(I);
      Operands.push_back(I->getName());
    } else if (!isa<Function>(Op)) {
      // This must be an immediate value.
      std::string TextRepresentation;
      raw_string_ostream Stream(TextRepresentation);
      Op->printAsOperand(Stream, false);
      Operands.push_back(StringRef(Stream.str()));
    }
  }

  sortCommutativeOperands(I, Operands);

  // Initialize to a magic constant, so the state isn't zero.
  uint64_t Hash = MagicHashConstant;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  Name.append("(");
  for (size_t i = 0; i < Operands.size(); ++i) {
    Name.append(Operands[i]);

    if (i < Operands.size() - 1)
      Name.append(", ");
  }
  Name.append(")");

  if ((I->getName().empty() || Options.RenameAll) && !I->getType()->isVoidTy())
    I->setName(Name);
}

```
- EN: This region continues the IRNormalizer implementation with local helper logic centered on Collect, Instruction, Walk, Operands.
- CN: 这一段延续了 IRNormalizer 的主体实现，围绕 Collect, Instruction, Walk, Operands 等局部辅助逻辑展开。

### Lines 352-409

```cpp
/// Shortens instruction's name. This method removes called function name from
/// the instruction name and substitutes the call chain with a corresponding
/// list of operands.
///
/// Examples:
/// op00000Callee(op00001Callee(...), vl00000Callee(1, 2), ...)  ->
/// op00000(op00001, vl00000, ...) vl00000Callee(1, 2)  ->  vl00000(1, 2)
///
/// This method omits output instructions and pre-output (instructions directly
/// used by an output instruction) instructions (by default). By default it also
/// does not affect user named instructions.
///
/// \param I Instruction whose name will be folded.
void IRNormalizer::foldInstructionName(Instruction *I) const {
  // If this flag is raised, fold all regular
  // instructions (including pre-outputs).
  if (!Options.FoldPreOutputs) {
    // Don't fold if one of the users is an output instruction.
    for (auto *U : I->users())
      if (auto *IU = dyn_cast<Instruction>(U))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Name.append(I->getName().substr(0, 7));

  Name.append("(");
  for (size_t i = 0; i < Operands.size(); ++i) {
    Name.append(Operands[i]);

    if (i < Operands.size() - 1)
      Name.append(", ");
  }
  Name.append(")");

  I->setName(Name);
}

```
- EN: Core entities appearing here include foldInstructionName, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 foldInstructionName，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 410-469

```cpp
/// Reorders instructions by walking up the tree from each operand of an output
/// instruction and reducing the def-use distance.
/// This method assumes that output instructions were collected top-down,
/// otherwise the def-use chain may be broken.
/// This method is a wrapper for recursive reorderInstruction().
///
/// \see reorderInstruction()
void IRNormalizer::reorderInstructions(Function &F) const {
  for (auto &BB : F) {
    LLVM_DEBUG(dbgs() << "Reordering instructions in basic block: "
                      << BB.getName() << "\n");
    // Find the source nodes of the DAG of instructions in this basic block.
    // Source nodes are instructions that have side effects, are terminators, or
    // don't have a parent in the DAG of instructions.
    //
    // We must iterate from the first to the last instruction otherwise side
    // effecting instructions could be reordered.

    std::stack<Instruction *> TopologicalSort;
    SmallPtrSet<const Instruction *, 32> Visited;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      auto *Instruction = TopologicalSort.top();
      auto FirstNonPHIOrDbgOrAlloca = BB.getFirstNonPHIOrDbgOrAlloca();
      if (auto *Call = dyn_cast<CallInst>(&*FirstNonPHIOrDbgOrAlloca)) {
        if (Call->getIntrinsicID() ==
                Intrinsic::experimental_convergence_entry ||
            Call->getIntrinsicID() == Intrinsic::experimental_convergence_loop)
          FirstNonPHIOrDbgOrAlloca++;
      }
      Instruction->moveBefore(FirstNonPHIOrDbgOrAlloca);
      TopologicalSort.pop();
    }
  }
}

```
- EN: Core entities appearing here include reorderInstructions, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 reorderInstructions，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 470-527

```cpp
void IRNormalizer::reorderDefinition(
    Instruction *Definition, std::stack<Instruction *> &TopologicalSort,
    SmallPtrSet<const Instruction *, 32> &Visited) const {
  if (Visited.contains(Definition))
    return;
  Visited.insert(Definition);

  {
    const auto *BasicBlock = Definition->getParent();
    const auto FirstNonPHIOrDbgOrAlloca =
        BasicBlock->getFirstNonPHIOrDbgOrAlloca();
    if (FirstNonPHIOrDbgOrAlloca == BasicBlock->end())
      return; // TODO: Is this necessary?
    if (Definition->comesBefore(&*FirstNonPHIOrDbgOrAlloca))
      return; // TODO: Do some kind of ordering for these instructions.
  }

  for (auto &Operand : Definition->operands()) {
    if (auto *Op = dyn_cast<Instruction>(Operand)) {
      if (Op->getParent() != Definition->getParent())
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  TopologicalSort.emplace(Definition);
}

/// Reorders instruction's operands alphabetically. This method assumes
/// that passed instruction is commutative. Changing the operand order
/// in other instructions may change the semantics.
///
/// \param I Instruction whose operands will be reordered.
void IRNormalizer::reorderInstructionOperandsByNames(Instruction *I) const {
  // This method assumes that passed I is commutative,
  // changing the order of operands in other instructions
  // may change the semantics.

```
- EN: Core entities appearing here include reorderInstructionOperandsByNames, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 reorderInstructionOperandsByNames，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 528-583

```cpp
  // Instruction operands for further sorting.
  SmallVector<std::pair<std::string, Value *>, 4> Operands;

  // Collect operands.
  for (auto &Op : I->operands()) {
    if (auto *V = dyn_cast<Value>(Op)) {
      if (isa<Instruction>(V)) {
        // This is an an instruction.
        Operands.push_back(std::pair<std::string, Value *>(V->getName(), V));
      } else {
        std::string TextRepresentation;
        raw_string_ostream Stream(TextRepresentation);
        Op->printAsOperand(Stream, false);
        Operands.push_back(std::pair<std::string, Value *>(Stream.str(), V));
      }
    }
  }

  // Sort operands.
  sortCommutativeOperands(I, Operands);

  // Reorder operands.
  unsigned Position = 0;
  for (auto &Op : I->operands()) {
    Op.set(Operands[Position].second);
    Position += 1;
  }
}

/// Reorders PHI node's values according to the names of corresponding basic
/// blocks.
///
/// \param Phi PHI node to normalize.
void IRNormalizer::reorderPHIIncomingValues(PHINode *Phi) const {
  // Values for further sorting.
  SmallVector<std::pair<Value *, BasicBlock *>, 2> Values;

  // Collect blocks and corresponding values.
  for (auto &BB : Phi->blocks()) {
    Value *V = Phi->getIncomingValueForBlock(BB);
    Values.push_back(std::pair<Value *, BasicBlock *>(V, BB));
  }

  // Sort values according to the name of a basic block.
  llvm::sort(Values, [](const std::pair<Value *, BasicBlock *> &LHS,
                        const std::pair<Value *, BasicBlock *> &RHS) {
    return LHS.second->getName() < RHS.second->getName();
  });

  // Swap.
  for (unsigned i = 0; i < Values.size(); ++i) {
    Phi->setIncomingBlock(i, Values[i].second);
    Phi->setIncomingValue(i, Values[i].first);
  }
}

```
- EN: Core entities appearing here include reorderPHIIncomingValues, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 reorderPHIIncomingValues，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 584-641

```cpp
/// Returns a vector of output instructions. An output is an instruction which
/// has side-effects or is ReturnInst. Uses isOutput().
///
/// \see isOutput()
/// \param F Function to collect outputs from.
SmallVector<Instruction *, 16>
IRNormalizer::collectOutputInstructions(Function &F) const {
  // Output instructions are collected top-down in each function,
  // any change may break the def-use chain in reordering methods.
  SmallVector<Instruction *, 16> Outputs;
  for (auto &I : instructions(F))
    if (isOutput(&I))
      Outputs.push_back(&I);
  return Outputs;
}

/// Helper method checking whether the instruction may have side effects or is
/// ReturnInst.
///
/// \param I Considered instruction.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

/// Helper method returning indices (distance from the beginning of the basic
/// block) of outputs using the \p I (eliminates repetitions). Walks down the
/// def-use tree recursively.
///
/// \param I Considered instruction.
/// \param Visited Set of visited instructions.
SetVector<int> IRNormalizer::getOutputFootprint(
    Instruction *I, SmallPtrSet<const Instruction *, 32> &Visited) const {

  // Vector containing indexes of outputs (no repetitions),
  // which use I in the order of walking down the def-use tree.
  SetVector<int> Outputs;

```
- EN: Core entities appearing here include collectOutputInstructions, isOutput, isInitialInstruction, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 collectOutputInstructions, isOutput, isInitialInstruction，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 642-683

```cpp
  if (!Visited.count(I)) {
    Visited.insert(I);

    if (isOutput(I)) {
      // Gets output instruction's parent function.
      Function *Func = I->getParent()->getParent();

      // Finds and inserts the index of the output to the vector.
      unsigned Count = 0;
      for (const auto &B : *Func) {
        for (const auto &E : B) {
          if (&E == I)
            Outputs.insert(Count);
          Count += 1;
        }
      }

      // Returns to the used instruction.
      return Outputs;
    }

    for (auto *U : I->users()) {
      if (auto *UI = dyn_cast<Instruction>(U)) {
        // Vector for outputs which use UI.
        SetVector<int> OutputsUsingUI = getOutputFootprint(UI, Visited);
        // Insert the indexes of outputs using UI.
        Outputs.insert_range(OutputsUsingUI);
      }
    }
  }

  // Return to the used instruction.
  return Outputs;
}

PreservedAnalyses IRNormalizerPass::run(Function &F,
                                        FunctionAnalysisManager &AM) const {
  IRNormalizer(Options).runOnFunction(F);
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
- EN: This region continues the IRNormalizer implementation with local helper logic centered on Visited, Gets, Function, Func.
- CN: 这一段延续了 IRNormalizer 的主体实现，围绕 Visited, Gets, Function, Func 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `IRNormalizer, hash_16_bytes, runOnFunction, nameFunctionArguments, nameBasicBlocks, nameInstruction, sortCommutativeOperands, nameAsInitialInstruction` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`IRNormalizer, hash_16_bytes, runOnFunction, nameFunctionArguments, nameBasicBlocks, nameInstruction, sortCommutativeOperands, nameAsInitialInstruction` 构成该文件对外 API 与主要实现挂钩。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstIterator.h`, `llvm/Transforms/Utils/IRNormalizer.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstIterator.h`, `llvm/Transforms/Utils/IRNormalizer.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `stack` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`stack` 为 LLVM API 之外的 C++ 语言工具提供支持。
