# HexagonOptimizeSZextends.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonOptimizeSZextends.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon sign/zero-extend optimization.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===- HexagonOptimizeSZextends.cpp - Remove unnecessary argument extends -===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // Pass that removes sign extends for function parameters. These parameters
    10: // are already sign extended by the caller per Hexagon's ABI
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "Hexagon.h"
    15: #include "llvm/CodeGen/StackProtector.h"
    16: #include "llvm/CodeGen/ValueTypes.h"
    17: #include "llvm/IR/Function.h"
    18: #include "llvm/IR/Instructions.h"
    19: #include "llvm/IR/IntrinsicInst.h"
    20: #include "llvm/IR/IntrinsicsHexagon.h"
    21: #include "llvm/Pass.h"
    22: #include "llvm/Transforms/Scalar.h"
    23: 
    24: using namespace llvm;
    25: 
```
- EN: It imports headers such as Hexagon.h, llvm/CodeGen/StackProtector.h, llvm/CodeGen/ValueTypes.h, llvm/IR/Function.h, ... (9 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonOptimizeSZextends, showing how the code connects to sibling backend components.
- CN: 这里引入了 Hexagon.h, llvm/CodeGen/StackProtector.h, llvm/CodeGen/ValueTypes.h, llvm/IR/Function.h, ... (9 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonOptimizeSZextends，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```cpp
    26: namespace {
    27:   struct HexagonOptimizeSZextends : public FunctionPass {
    28:   public:
    29:     static char ID;
    30:     HexagonOptimizeSZextends() : FunctionPass(ID) {}
    31:     bool runOnFunction(Function &F) override;
    32: 
    33:     StringRef getPassName() const override { return "Remove sign extends"; }
    34: 
    35:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    36:       AU.addPreserved<StackProtector>();
    37:       FunctionPass::getAnalysisUsage(AU);
    38:     }
    39: 
    40:     bool intrinsicAlreadySextended(Intrinsic::ID IntID);
    41:   };
    42: }
    43: 
    44: char HexagonOptimizeSZextends::ID = 0;
    45: 
    46: INITIALIZE_PASS(HexagonOptimizeSZextends, "reargs",
    47:                 "Remove Sign and Zero Extends for Args", false, false)
    48: 
    49: bool HexagonOptimizeSZextends::intrinsicAlreadySextended(Intrinsic::ID IntID) {
    50:   switch(IntID) {
```
- EN: It declares types such as HexagonOptimizeSZextends, which carry the state or API of this component. It declares or implements routines such as HexagonOptimizeSZextends, runOnFunction, getPassName, getAnalysisUsage, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonOptimizeSZextends, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonOptimizeSZextends 等类型，用来承载该组件的状态或接口。 这里声明或实现了 HexagonOptimizeSZextends, runOnFunction, getPassName, getAnalysisUsage, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonOptimizeSZextends，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```cpp
    51:     case llvm::Intrinsic::hexagon_A2_addh_l16_sat_ll:
    52:       return true;
    53:     default:
    54:       break;
    55:   }
    56:   return false;
    57: }
    58: 
    59: bool HexagonOptimizeSZextends::runOnFunction(Function &F) {
    60:   if (skipFunction(F))
    61:     return false;
    62: 
    63:   unsigned Idx = 0;
    64:   // Try to optimize sign extends in formal parameters. It's relying on
    65:   // callee already sign extending the values. I'm not sure if our ABI
    66:   // requires callee to sign extend though.
    67:   for (auto &Arg : F.args()) {
    68:     if (F.getAttributes().hasParamAttr(Idx, Attribute::SExt)) {
    69:       if (!isa<PointerType>(Arg.getType())) {
    70:         for (Use &U : llvm::make_early_inc_range(Arg.uses())) {
    71:           if (isa<SExtInst>(U)) {
    72:             Instruction* Use = cast<Instruction>(U);
    73:             SExtInst* SI = new SExtInst(&Arg, Use->getType());
    74:             assert (EVT::getEVT(SI->getType()) ==
    75:                     (EVT::getEVT(Use->getType())));
```
- EN: It declares or implements routines such as HexagonOptimizeSZextends::runOnFunction, cast<Instruction>, SExtInst, assert, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonOptimizeSZextends, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonOptimizeSZextends::runOnFunction, cast<Instruction>, SExtInst, assert 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonOptimizeSZextends，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76:             Use->replaceAllUsesWith(SI);
    77:             BasicBlock::iterator First = F.getEntryBlock().begin();
    78:             SI->insertBefore(First);
    79:             Use->eraseFromParent();
    80:           }
    81:         }
    82:       }
    83:     }
    84:     ++Idx;
    85:   }
    86: 
    87:   // Try to remove redundant sext operations on Hexagon. The hardware
    88:   // already sign extends many 16 bit intrinsic operations to 32 bits.
    89:   // For example:
    90:   // %34 = tail call i32 @llvm.hexagon.A2.addh.l16.sat.ll(i32 %x, i32 %y)
    91:   // %sext233 = shl i32 %34, 16
    92:   // %conv52 = ashr exact i32 %sext233, 16
    93:   for (auto &B : F) {
    94:     for (auto &I : B) {
    95:       // Look for arithmetic shift right by 16.
    96:       BinaryOperator *Ashr = dyn_cast<BinaryOperator>(&I);
    97:       if (!(Ashr && Ashr->getOpcode() == Instruction::AShr))
    98:         continue;
    99:       Value *AshrOp1 = Ashr->getOperand(1);
   100:       ConstantInt *C = dyn_cast<ConstantInt>(AshrOp1);
```
- EN: It declares or implements routines such as replaceAllUsesWith, getEntryBlock, insertBefore, eraseFromParent, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 replaceAllUsesWith, getEntryBlock, insertBefore, eraseFromParent, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-125 / 第 101-125 行

```cpp
   101:       // Right shifted by 16.
   102:       if (!(C && C->getSExtValue() == 16))
   103:         continue;
   104: 
   105:       // The first operand of Ashr comes from logical shift left.
   106:       Instruction *Shl = dyn_cast<Instruction>(Ashr->getOperand(0));
   107:       if (!(Shl && Shl->getOpcode() == Instruction::Shl))
   108:         continue;
   109:       Value *Intr = Shl->getOperand(0);
   110:       Value *ShlOp1 = Shl->getOperand(1);
   111:       C = dyn_cast<ConstantInt>(ShlOp1);
   112:       // Left shifted by 16.
   113:       if (!(C && C->getSExtValue() == 16))
   114:         continue;
   115: 
   116:       // The first operand of Shl comes from an intrinsic.
   117:       if (IntrinsicInst *I = dyn_cast<IntrinsicInst>(Intr)) {
   118:         if (!intrinsicAlreadySextended(I->getIntrinsicID()))
   119:           continue;
   120:         // All is well. Replace all uses of AShr with I.
   121:         for (auto UI = Ashr->user_begin(), UE = Ashr->user_end();
   122:              UI != UE; ++UI) {
   123:           const Use &TheUse = UI.getUse();
   124:           if (Instruction *J = dyn_cast<Instruction>(TheUse.getUser())) {
   125:             J->replaceUsesOfWith(Ashr, I);
```
- EN: It declares or implements routines such as dyn_cast<Instruction>, getOperand, dyn_cast<ConstantInt>, getUse, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 dyn_cast<Instruction>, getOperand, dyn_cast<ConstantInt>, getUse, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 126-138 / 第 126-138 行

```cpp
   126:           }
   127:         }
   128:       }
   129:     }
   130:   }
   131: 
   132:   return true;
   133: }
   134: 
   135: 
   136: FunctionPass *llvm::createHexagonOptimizeSZextends() {
   137:   return new HexagonOptimizeSZextends();
   138: }
```
- EN: It declares or implements routines such as llvm::createHexagonOptimizeSZextends, HexagonOptimizeSZextends, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonOptimizeSZextends, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::createHexagonOptimizeSZextends, HexagonOptimizeSZextends 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonOptimizeSZextends，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- intrinsic mapping / Intrinsic 映射

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, llvm/CodeGen/StackProtector.h, llvm/CodeGen/ValueTypes.h, llvm/IR/Function.h, llvm/IR/Instructions.h, llvm/IR/IntrinsicInst.h, llvm/IR/IntrinsicsHexagon.h, llvm/Pass.h, llvm/Transforms/Scalar.h`
- Hexagon symbols / Hexagon 符号: `HexagonOptimizeSZextends`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
