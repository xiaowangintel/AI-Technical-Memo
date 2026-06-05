# CrossDSOCFI.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/Transforms/IPO/CrossDSOCFI.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This pass exports all llvm.bitset's found in the module in the form of a __cfi_check function, which can be used to verify cross-DSO call targets. / 该文件位于 `Transforms/IPO`，主要实现 `CrossDSOCFI` 相关的 LLVM 变换逻辑、辅助流程以及 pass 接线代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===-- CrossDSOCFI.cpp - Externalize this module's CFI checks ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass exports all llvm.bitset's found in the module in the form of a
// __cfi_check function, which can be used to verify cross-DSO call targets.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/IPO/CrossDSOCFI.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalObject.h"
#include "llvm/IR/IRBuilder.h"
```

- **L1**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L2**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L8**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L9**: Comment documents the nearby logic or transformation intent: `This pass exports all llvm.bitset's found in the module in the form of a`. / 注释说明了附近代码的逻辑或变换意图：`This pass exports all llvm.bitset's found in the module in the form of a`。
- **L10**: Comment documents the nearby logic or transformation intent: `__cfi_check function, which can be used to verify cross-DSO call targets.`. / 注释说明了附近代码的逻辑或变换意图：`__cfi_check function, which can be used to verify cross-DSO call targets.`。
- **L11**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L12**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L13**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L14**: Includes "llvm/Transforms/IPO/CrossDSOCFI.h" to access transform-specific declarations. / 引入 "llvm/Transforms/IPO/CrossDSOCFI.h" 以使用变换相关声明。
- **L15**: Includes "llvm/ADT/SetVector.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/SetVector.h" 以使用LLVM ADT 数据结构/工具。
- **L16**: Includes "llvm/ADT/Statistic.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/Statistic.h" 以使用LLVM ADT 数据结构/工具。
- **L17**: Includes "llvm/IR/Constants.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Constants.h" 以使用LLVM IR 核心类型与构造工具。
- **L18**: Includes "llvm/IR/Function.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Function.h" 以使用LLVM IR 核心类型与构造工具。
- **L19**: Includes "llvm/IR/GlobalObject.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/GlobalObject.h" 以使用LLVM IR 核心类型与构造工具。
- **L20**: Includes "llvm/IR/IRBuilder.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/IRBuilder.h" 以使用LLVM IR 核心类型与构造工具。

### Lines 21-40

```cpp
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/Module.h"
#include "llvm/TargetParser/Triple.h"

using namespace llvm;

#define DEBUG_TYPE "cross-dso-cfi"

STATISTIC(NumTypeIds, "Number of unique type identifiers");

namespace {

struct CrossDSOCFI {
  MDNode *VeryLikelyWeights;

  ConstantInt *extractNumericTypeId(MDNode *MD);
  void buildCFICheck(Module &M);
  bool runOnModule(Module &M);
```

- **L21**: Includes "llvm/IR/Instructions.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Instructions.h" 以使用LLVM IR 核心类型与构造工具。
- **L22**: Includes "llvm/IR/Intrinsics.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Intrinsics.h" 以使用LLVM IR 核心类型与构造工具。
- **L23**: Includes "llvm/IR/MDBuilder.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/MDBuilder.h" 以使用LLVM IR 核心类型与构造工具。
- **L24**: Includes "llvm/IR/Module.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Module.h" 以使用LLVM IR 核心类型与构造工具。
- **L25**: Includes "llvm/TargetParser/Triple.h" to access local declarations used by this file. / 引入 "llvm/TargetParser/Triple.h" 以使用本文件使用的本地声明。
- **L26**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L27**: Brings namespace `llvm` into the local scope. / 将命名空间 `llvm` 引入当前作用域。
- **L28**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L29**: Defines macro `DEBUG_TYPE` for later conditional logic, flags, or diagnostics. / 定义宏 `DEBUG_TYPE`，供后续条件逻辑、标志位或诊断使用。
- **L30**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L31**: Registers LLVM statistic counter `NumTypeIds`. / 注册 LLVM 统计计数器 `NumTypeIds`。
- **L32**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L33**: Opens namespace scope ``. / 打开命名空间作用域 ``。
- **L34**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L35**: Declares struct `CrossDSOCFI`. / 声明 struct `CrossDSOCFI`。
- **L36**: Executes a standalone statement or declaration: `MDNode *VeryLikelyWeights;`. / 执行一条独立语句或声明：`MDNode *VeryLikelyWeights;`。
- **L37**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L38**: Executes call or statement centered on `*extractNumericTypeId`. / 执行以 `*extractNumericTypeId` 为核心的调用或语句。
- **L39**: Executes call or statement centered on `buildCFICheck`. / 执行以 `buildCFICheck` 为核心的调用或语句。
- **L40**: Executes call or statement centered on `runOnModule`. / 执行以 `runOnModule` 为核心的调用或语句。

### Lines 41-60

```cpp
};

} // anonymous namespace

/// Extracts a numeric type identifier from an MDNode containing type metadata.
ConstantInt *CrossDSOCFI::extractNumericTypeId(MDNode *MD) {
  // This check excludes vtables for classes inside anonymous namespaces.
  auto TM = dyn_cast<ValueAsMetadata>(MD->getOperand(1));
  if (!TM)
    return nullptr;
  auto C = dyn_cast_or_null<ConstantInt>(TM->getValue());
  if (!C) return nullptr;
  // We are looking for i64 constants.
  if (C->getBitWidth() != 64) return nullptr;

  return C;
}

/// buildCFICheck - emits __cfi_check for the current module.
void CrossDSOCFI::buildCFICheck(Module &M) {
```

- **L41**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L42**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L43**: Continues the surrounding expression or declaration: `} // anonymous namespace`. / 继续构造周围的表达式或声明：`} // anonymous namespace`。
- **L44**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L45**: Comment documents the nearby logic or transformation intent: `Extracts a numeric type identifier from an MDNode containing type metadata.`. / 注释说明了附近代码的逻辑或变换意图：`Extracts a numeric type identifier from an MDNode containing type metadata.`。
- **L46**: Starts a function, method, or lambda body: `ConstantInt *CrossDSOCFI::extractNumericTypeId(MDNode *MD) {`. / 开始一个函数、方法或 lambda 的主体：`ConstantInt *CrossDSOCFI::extractNumericTypeId(MDNode *MD) {`。
- **L47**: Comment documents the nearby logic or transformation intent: `This check excludes vtables for classes inside anonymous namespaces.`. / 注释说明了附近代码的逻辑或变换意图：`This check excludes vtables for classes inside anonymous namespaces.`。
- **L48**: Initializes variable `TM` from the right-hand expression. / 使用右侧表达式初始化变量 `TM`。
- **L49**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L50**: Returns from the current function with `nullptr`. / 以 `nullptr` 从当前函数返回。
- **L51**: Initializes variable `C` from the right-hand expression. / 使用右侧表达式初始化变量 `C`。
- **L52**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L53**: Comment documents the nearby logic or transformation intent: `We are looking for i64 constants.`. / 注释说明了附近代码的逻辑或变换意图：`We are looking for i64 constants.`。
- **L54**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L55**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L56**: Returns from the current function with `C`. / 以 `C` 从当前函数返回。
- **L57**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L58**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L59**: Comment documents the nearby logic or transformation intent: `buildCFICheck - emits __cfi_check for the current module.`. / 注释说明了附近代码的逻辑或变换意图：`buildCFICheck - emits __cfi_check for the current module.`。
- **L60**: Starts a function, method, or lambda body: `void CrossDSOCFI::buildCFICheck(Module &M) {`. / 开始一个函数、方法或 lambda 的主体：`void CrossDSOCFI::buildCFICheck(Module &M) {`。

### Lines 61-80

```cpp
  // FIXME: verify that __cfi_check ends up near the end of the code section,
  // but before the jump slots created in LowerTypeTests.
  SetVector<uint64_t> TypeIds;
  SmallVector<MDNode *, 2> Types;
  for (GlobalObject &GO : M.global_objects()) {
    Types.clear();
    GO.getMetadata(LLVMContext::MD_type, Types);
    for (MDNode *Type : Types)
      if (ConstantInt *TypeId = extractNumericTypeId(Type))
        TypeIds.insert(TypeId->getZExtValue());
  }

  NamedMDNode *CfiFunctionsMD = M.getNamedMetadata("cfi.functions");
  if (CfiFunctionsMD) {
    for (auto *Func : CfiFunctionsMD->operands()) {
      assert(Func->getNumOperands() >= 2);
      for (unsigned I = 2; I < Func->getNumOperands(); ++I)
        if (ConstantInt *TypeId =
                extractNumericTypeId(cast<MDNode>(Func->getOperand(I).get())))
          TypeIds.insert(TypeId->getZExtValue());
```

- **L61**: Comment records a pending task or caution: `FIXME: verify that __cfi_check ends up near the end of the code section,`. / 注释记录了待办事项或注意点：`FIXME: verify that __cfi_check ends up near the end of the code section,`。
- **L62**: Comment documents the nearby logic or transformation intent: `but before the jump slots created in LowerTypeTests.`. / 注释说明了附近代码的逻辑或变换意图：`but before the jump slots created in LowerTypeTests.`。
- **L63**: Executes a standalone statement or declaration: `SetVector<uint64_t> TypeIds;`. / 执行一条独立语句或声明：`SetVector<uint64_t> TypeIds;`。
- **L64**: Executes a standalone statement or declaration: `SmallVector<MDNode *, 2> Types;`. / 执行一条独立语句或声明：`SmallVector<MDNode *, 2> Types;`。
- **L65**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L66**: Executes call or statement centered on `Types.clear`. / 执行以 `Types.clear` 为核心的调用或语句。
- **L67**: Executes call or statement centered on `GO.getMetadata`. / 执行以 `GO.getMetadata` 为核心的调用或语句。
- **L68**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L69**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L70**: Executes call or statement centered on `TypeIds.insert`. / 执行以 `TypeIds.insert` 为核心的调用或语句。
- **L71**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L72**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L73**: Executes call or statement centered on `M.getNamedMetadata`. / 执行以 `M.getNamedMetadata` 为核心的调用或语句。
- **L74**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L75**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L76**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L77**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L78**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L79**: Continues the surrounding expression or declaration: `extractNumericTypeId(cast<MDNode>(Func->getOperand(I).get())))`. / 继续构造周围的表达式或声明：`extractNumericTypeId(cast<MDNode>(Func->getOperand(I).get())))`。
- **L80**: Executes call or statement centered on `TypeIds.insert`. / 执行以 `TypeIds.insert` 为核心的调用或语句。

### Lines 81-100

```cpp
    }
  }

  LLVMContext &Ctx = M.getContext();
  FunctionCallee C = M.getOrInsertFunction(
      "__cfi_check", Type::getVoidTy(Ctx), Type::getInt64Ty(Ctx),
      PointerType::getUnqual(Ctx), PointerType::getUnqual(Ctx));
  Function *F = cast<Function>(C.getCallee());
  // Take over the existing function. The frontend emits a weak stub so that the
  // linker knows about the symbol; this pass replaces the function body.
  F->deleteBody();
  F->setAlignment(Align(4096));

  Triple T(M.getTargetTriple());
  if (T.isARM() || T.isThumb())
    F->addFnAttr("target-features", "+thumb-mode");

  auto args = F->arg_begin();
  Value &CallSiteTypeId = *(args++);
  CallSiteTypeId.setName("CallSiteTypeId");
```

- **L81**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L82**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L83**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L84**: Executes call or statement centered on `M.getContext`. / 执行以 `M.getContext` 为核心的调用或语句。
- **L85**: Continues the surrounding expression or declaration: `FunctionCallee C = M.getOrInsertFunction(`. / 继续构造周围的表达式或声明：`FunctionCallee C = M.getOrInsertFunction(`。
- **L86**: Continues a multi-line argument list or initializer: `"__cfi_check", Type::getVoidTy(Ctx), Type::getInt64Ty(Ctx),`. / 继续一个多行参数列表或初始化器：`"__cfi_check", Type::getVoidTy(Ctx), Type::getInt64Ty(Ctx),`。
- **L87**: Executes call or statement centered on `PointerType::getUnqual`. / 执行以 `PointerType::getUnqual` 为核心的调用或语句。
- **L88**: Executes call or statement centered on `cast<Function>`. / 执行以 `cast<Function>` 为核心的调用或语句。
- **L89**: Comment documents the nearby logic or transformation intent: `Take over the existing function. The frontend emits a weak stub so that the`. / 注释说明了附近代码的逻辑或变换意图：`Take over the existing function. The frontend emits a weak stub so that the`。
- **L90**: Comment documents the nearby logic or transformation intent: `linker knows about the symbol; this pass replaces the function body.`. / 注释说明了附近代码的逻辑或变换意图：`linker knows about the symbol; this pass replaces the function body.`。
- **L91**: Executes call or statement centered on `F->deleteBody`. / 执行以 `F->deleteBody` 为核心的调用或语句。
- **L92**: Executes call or statement centered on `F->setAlignment`. / 执行以 `F->setAlignment` 为核心的调用或语句。
- **L93**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L94**: Executes call or statement centered on `T`. / 执行以 `T` 为核心的调用或语句。
- **L95**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L96**: Executes call or statement centered on `F->addFnAttr`. / 执行以 `F->addFnAttr` 为核心的调用或语句。
- **L97**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L98**: Initializes variable `args` from the right-hand expression. / 使用右侧表达式初始化变量 `args`。
- **L99**: Executes call or statement centered on `*`. / 执行以 `*` 为核心的调用或语句。
- **L100**: Executes call or statement centered on `CallSiteTypeId.setName`. / 执行以 `CallSiteTypeId.setName` 为核心的调用或语句。

### Lines 101-120

```cpp
  Value &Addr = *(args++);
  Addr.setName("Addr");
  Value &CFICheckFailData = *(args++);
  CFICheckFailData.setName("CFICheckFailData");
  assert(args == F->arg_end());

  BasicBlock *BB = BasicBlock::Create(Ctx, "entry", F);
  BasicBlock *ExitBB = BasicBlock::Create(Ctx, "exit", F);

  BasicBlock *TrapBB = BasicBlock::Create(Ctx, "fail", F);
  IRBuilder<> IRBFail(TrapBB);
  FunctionCallee CFICheckFailFn = M.getOrInsertFunction(
      "__cfi_check_fail", Type::getVoidTy(Ctx), PointerType::getUnqual(Ctx),
      PointerType::getUnqual(Ctx));
  IRBFail.CreateCall(CFICheckFailFn, {&CFICheckFailData, &Addr});
  IRBFail.CreateBr(ExitBB);

  IRBuilder<> IRBExit(ExitBB);
  IRBExit.CreateRetVoid();

```

- **L101**: Executes call or statement centered on `*`. / 执行以 `*` 为核心的调用或语句。
- **L102**: Executes call or statement centered on `Addr.setName`. / 执行以 `Addr.setName` 为核心的调用或语句。
- **L103**: Executes call or statement centered on `*`. / 执行以 `*` 为核心的调用或语句。
- **L104**: Executes call or statement centered on `CFICheckFailData.setName`. / 执行以 `CFICheckFailData.setName` 为核心的调用或语句。
- **L105**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L106**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L107**: Executes call or statement centered on `BasicBlock::Create`. / 执行以 `BasicBlock::Create` 为核心的调用或语句。
- **L108**: Executes call or statement centered on `BasicBlock::Create`. / 执行以 `BasicBlock::Create` 为核心的调用或语句。
- **L109**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L110**: Executes call or statement centered on `BasicBlock::Create`. / 执行以 `BasicBlock::Create` 为核心的调用或语句。
- **L111**: Executes call or statement centered on `IRBFail`. / 执行以 `IRBFail` 为核心的调用或语句。
- **L112**: Continues the surrounding expression or declaration: `FunctionCallee CFICheckFailFn = M.getOrInsertFunction(`. / 继续构造周围的表达式或声明：`FunctionCallee CFICheckFailFn = M.getOrInsertFunction(`。
- **L113**: Continues a multi-line argument list or initializer: `"__cfi_check_fail", Type::getVoidTy(Ctx), PointerType::getUnqual(Ctx),`. / 继续一个多行参数列表或初始化器：`"__cfi_check_fail", Type::getVoidTy(Ctx), PointerType::getUnqual(Ctx),`。
- **L114**: Executes call or statement centered on `PointerType::getUnqual`. / 执行以 `PointerType::getUnqual` 为核心的调用或语句。
- **L115**: Executes call or statement centered on `IRBFail.CreateCall`. / 执行以 `IRBFail.CreateCall` 为核心的调用或语句。
- **L116**: Executes call or statement centered on `IRBFail.CreateBr`. / 执行以 `IRBFail.CreateBr` 为核心的调用或语句。
- **L117**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L118**: Executes call or statement centered on `IRBExit`. / 执行以 `IRBExit` 为核心的调用或语句。
- **L119**: Executes call or statement centered on `IRBExit.CreateRetVoid`. / 执行以 `IRBExit.CreateRetVoid` 为核心的调用或语句。
- **L120**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 121-140

```cpp
  IRBuilder<> IRB(BB);
  SwitchInst *SI = IRB.CreateSwitch(&CallSiteTypeId, TrapBB, TypeIds.size());
  for (uint64_t TypeId : TypeIds) {
    ConstantInt *CaseTypeId = ConstantInt::get(Type::getInt64Ty(Ctx), TypeId);
    BasicBlock *TestBB = BasicBlock::Create(Ctx, "test", F);
    IRBuilder<> IRBTest(TestBB);

    Value *Test = IRBTest.CreateIntrinsic(
        Intrinsic::type_test,
        {&Addr,
         MetadataAsValue::get(Ctx, ConstantAsMetadata::get(CaseTypeId))});
    CondBrInst *BI = IRBTest.CreateCondBr(Test, ExitBB, TrapBB);
    BI->setMetadata(LLVMContext::MD_prof, VeryLikelyWeights);

    SI->addCase(CaseTypeId, TestBB);
    ++NumTypeIds;
  }
}

bool CrossDSOCFI::runOnModule(Module &M) {
```

- **L121**: Executes call or statement centered on `IRB`. / 执行以 `IRB` 为核心的调用或语句。
- **L122**: Executes call or statement centered on `IRB.CreateSwitch`. / 执行以 `IRB.CreateSwitch` 为核心的调用或语句。
- **L123**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L124**: Executes call or statement centered on `ConstantInt::get`. / 执行以 `ConstantInt::get` 为核心的调用或语句。
- **L125**: Executes call or statement centered on `BasicBlock::Create`. / 执行以 `BasicBlock::Create` 为核心的调用或语句。
- **L126**: Executes call or statement centered on `IRBTest`. / 执行以 `IRBTest` 为核心的调用或语句。
- **L127**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L128**: Continues the surrounding expression or declaration: `Value *Test = IRBTest.CreateIntrinsic(`. / 继续构造周围的表达式或声明：`Value *Test = IRBTest.CreateIntrinsic(`。
- **L129**: Continues a multi-line argument list or initializer: `Intrinsic::type_test,`. / 继续一个多行参数列表或初始化器：`Intrinsic::type_test,`。
- **L130**: Continues a multi-line argument list or initializer: `{&Addr,`. / 继续一个多行参数列表或初始化器：`{&Addr,`。
- **L131**: Executes call or statement centered on `MetadataAsValue::get`. / 执行以 `MetadataAsValue::get` 为核心的调用或语句。
- **L132**: Executes call or statement centered on `IRBTest.CreateCondBr`. / 执行以 `IRBTest.CreateCondBr` 为核心的调用或语句。
- **L133**: Executes call or statement centered on `BI->setMetadata`. / 执行以 `BI->setMetadata` 为核心的调用或语句。
- **L134**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L135**: Executes call or statement centered on `SI->addCase`. / 执行以 `SI->addCase` 为核心的调用或语句。
- **L136**: Executes a standalone statement or declaration: `++NumTypeIds;`. / 执行一条独立语句或声明：`++NumTypeIds;`。
- **L137**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L138**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L139**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L140**: Starts a function, method, or lambda body: `bool CrossDSOCFI::runOnModule(Module &M) {`. / 开始一个函数、方法或 lambda 的主体：`bool CrossDSOCFI::runOnModule(Module &M) {`。

### Lines 141-154

```cpp
  VeryLikelyWeights = MDBuilder(M.getContext()).createLikelyBranchWeights();
  if (M.getModuleFlag("Cross-DSO CFI") == nullptr)
    return false;
  buildCFICheck(M);
  return true;
}

PreservedAnalyses CrossDSOCFIPass::run(Module &M, ModuleAnalysisManager &AM) {
  CrossDSOCFI Impl;
  bool Changed = Impl.runOnModule(M);
  if (!Changed)
    return PreservedAnalyses::all();
  return PreservedAnalyses::none();
}
```

- **L141**: Executes call or statement centered on `MDBuilder`. / 执行以 `MDBuilder` 为核心的调用或语句。
- **L142**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L143**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L144**: Executes call or statement centered on `buildCFICheck`. / 执行以 `buildCFICheck` 为核心的调用或语句。
- **L145**: Returns from the current function with `true`. / 以 `true` 从当前函数返回。
- **L146**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L147**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L148**: Starts a function, method, or lambda body: `PreservedAnalyses CrossDSOCFIPass::run(Module &M, ModuleAnalysisManager &AM) {`. / 开始一个函数、方法或 lambda 的主体：`PreservedAnalyses CrossDSOCFIPass::run(Module &M, ModuleAnalysisManager &AM) {`。
- **L149**: Executes a standalone statement or declaration: `CrossDSOCFI Impl;`. / 执行一条独立语句或声明：`CrossDSOCFI Impl;`。
- **L150**: Initializes variable `Changed` from the right-hand expression. / 使用右侧表达式初始化变量 `Changed`。
- **L151**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L152**: Returns from the current function with `PreservedAnalyses::all()`. / 以 `PreservedAnalyses::all()` 从当前函数返回。
- **L153**: Returns from the current function with `PreservedAnalyses::none()`. / 以 `PreservedAnalyses::none()` 从当前函数返回。
- **L154**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **IPO transform pipeline / IPO 变换流水线**
- **IR construction and rewriting / IR 构造与重写**
- **Function-level traversal and updates / 函数级遍历与更新**
- **Module-wide coordination / 模块范围的协调**
- **Pass statistics and instrumentation / Pass 统计与观测**
- **Analysis preservation reporting / 分析保持情况报告**

## Dependencies / 依赖关系

- `llvm/Transforms/IPO/CrossDSOCFI.h`: Provides transform-specific declarations. / 提供变换相关声明。
- `llvm/ADT/SetVector.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/ADT/Statistic.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/IR/Constants.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Function.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/GlobalObject.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/IRBuilder.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Instructions.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Intrinsics.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/MDBuilder.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Module.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/TargetParser/Triple.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
