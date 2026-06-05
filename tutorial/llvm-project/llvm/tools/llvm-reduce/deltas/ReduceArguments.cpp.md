# ReduceArguments.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/tools/llvm-reduce/deltas/ReduceArguments.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Specialized Delta Pass This file implements a function which calls the Generic Delta pass in order to reduce uninteresting Arguments from declared and defined functions.
- **Purpose (CN)**: 该文件位于 `llvm-reduce/deltas`，主要实现命令行工具 `ReduceArguments` 相关的处理流程、格式支持或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- ReduceArguments.cpp - Specialized Delta Pass -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a function which calls the Generic Delta pass in order
// to reduce uninteresting Arguments from declared and defined functions.
//
//===----------------------------------------------------------------------===//

#include "ReduceArguments.h"
#include "Utils.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/IR/FMF.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/Operator.h"
````
- **L1 EN**: Banner comment marking a file section boundary.
  **L1 CN**: 横幅注释，用于标记文件分节。
- **L2 EN**: Separator comment used to visually break up sections.
  **L2 CN**: 分隔性注释，用于在视觉上划分小节。
- **L3 EN**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used to visually break up sections.
  **L6 CN**: 分隔性注释，用于在视觉上划分小节。
- **L7 EN**: Banner comment marking a file section boundary.
  **L7 CN**: 横幅注释，用于标记文件分节。
- **L8 EN**: Separator comment used to visually break up sections.
  **L8 CN**: 分隔性注释，用于在视觉上划分小节。
- **L9 EN**: Comment documents the nearby logic or transformation intent: `This file implements a function which calls the Generic Delta pass in order`.
  **L9 CN**: 注释说明了附近代码的逻辑或变换意图：`This file implements a function which calls the Generic Delta pass in order`。
- **L10 EN**: Comment documents the nearby logic or transformation intent: `to reduce uninteresting Arguments from declared and defined functions.`.
  **L10 CN**: 注释说明了附近代码的逻辑或变换意图：`to reduce uninteresting Arguments from declared and defined functions.`。
- **L11 EN**: Separator comment used to visually break up sections.
  **L11 CN**: 分隔性注释，用于在视觉上划分小节。
- **L12 EN**: Banner comment marking a file section boundary.
  **L12 CN**: 横幅注释，用于标记文件分节。
- **L13 EN**: Blank line that separates nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L14 EN**: Includes `ReduceArguments.h` to access supporting declarations from a local or system header.
  **L14 CN**: 引入 `ReduceArguments.h` 以使用来自本地或系统头文件的辅助声明。
- **L15 EN**: Includes `Utils.h` to access supporting declarations from a local or system header.
  **L15 CN**: 引入 `Utils.h` 以使用来自本地或系统头文件的辅助声明。
- **L16 EN**: Includes `llvm/ADT/SmallVector.h` to access LLVM ADT data structures/utilities.
  **L16 CN**: 引入 `llvm/ADT/SmallVector.h` 以使用LLVM ADT 数据结构/工具。
- **L17 EN**: Includes `llvm/IR/FMF.h` to access LLVM IR core types and builders.
  **L17 CN**: 引入 `llvm/IR/FMF.h` 以使用LLVM IR 核心类型与构造工具。
- **L18 EN**: Includes `llvm/IR/Instructions.h` to access LLVM IR core types and builders.
  **L18 CN**: 引入 `llvm/IR/Instructions.h` 以使用LLVM IR 核心类型与构造工具。
- **L19 EN**: Includes `llvm/IR/Intrinsics.h` to access LLVM IR core types and builders.
  **L19 CN**: 引入 `llvm/IR/Intrinsics.h` 以使用LLVM IR 核心类型与构造工具。
- **L20 EN**: Includes `llvm/IR/Operator.h` to access LLVM IR core types and builders.
  **L20 CN**: 引入 `llvm/IR/Operator.h` 以使用LLVM IR 核心类型与构造工具。

### Lines 21-40

````cpp
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Cloning.h"
#include <set>
#include <vector>

using namespace llvm;

static bool callingConvRequiresArgument(const Function &F,
                                        const Argument &Arg) {
  switch (F.getCallingConv()) {
  case CallingConv::X86_INTR:
    // If there are any arguments, the first one must by byval.
    return Arg.getArgNo() == 0 && F.arg_size() != 1;
  default:
    return false;
  }

  llvm_unreachable("covered calling conv switch");
}

````
- **L21 EN**: Includes `llvm/Transforms/Utils/BasicBlockUtils.h` to access transform-specific declarations.
  **L21 CN**: 引入 `llvm/Transforms/Utils/BasicBlockUtils.h` 以使用变换相关声明。
- **L22 EN**: Includes `llvm/Transforms/Utils/Cloning.h` to access transform-specific declarations.
  **L22 CN**: 引入 `llvm/Transforms/Utils/Cloning.h` 以使用变换相关声明。
- **L23 EN**: Includes `set` to access supporting declarations.
  **L23 CN**: 引入 `set` 以使用所需的辅助声明。
- **L24 EN**: Includes `vector` to access supporting declarations.
  **L24 CN**: 引入 `vector` 以使用所需的辅助声明。
- **L25 EN**: Blank line that separates nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Brings namespace `llvm` into the local scope.
  **L26 CN**: 将命名空间 `llvm` 引入当前作用域。
- **L27 EN**: Blank line that separates nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Continues a multi-line argument list or initializer: `static bool callingConvRequiresArgument(const Function &F,`.
  **L28 CN**: 继续一个多行参数列表或初始化器：`static bool callingConvRequiresArgument(const Function &F,`。
- **L29 EN**: Continues the surrounding expression or declaration: `const Argument &Arg) {`.
  **L29 CN**: 继续构造周围的表达式或声明：`const Argument &Arg) {`。
- **L30 EN**: Starts a multi-way branch based on an expression: `switch (F.getCallingConv()) {`.
  **L30 CN**: 开始基于表达式的多路分支：`switch (F.getCallingConv()) {`。
- **L31 EN**: Introduces a switch dispatch label: `case CallingConv::X86_INTR:`.
  **L31 CN**: 引入一个 switch 分发标签：`case CallingConv::X86_INTR:`。
- **L32 EN**: Comment documents the nearby logic or transformation intent: `If there are any arguments, the first one must by byval.`.
  **L32 CN**: 注释说明了附近代码的逻辑或变换意图：`If there are any arguments, the first one must by byval.`。
- **L33 EN**: Returns control, optionally with a value: `return Arg.getArgNo() == 0 && F.arg_size() != 1;`.
  **L33 CN**: 返回控制流，并可附带返回值：`return Arg.getArgNo() == 0 && F.arg_size() != 1;`。
- **L34 EN**: Introduces the default switch branch: `default:`.
  **L34 CN**: 引入 switch 的默认分支：`default:`。
- **L35 EN**: Returns control, optionally with a value: `return false;`.
  **L35 CN**: 返回控制流，并可附带返回值：`return false;`。
- **L36 EN**: Closes the current lexical scope or compound statement.
  **L36 CN**: 结束当前词法作用域或复合语句块。
- **L37 EN**: Blank line that separates nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Executes call or statement centered on `llvm_unreachable`.
  **L38 CN**: 执行以 `llvm_unreachable` 为核心的调用或语句。
- **L39 EN**: Closes the current lexical scope or compound statement.
  **L39 CN**: 结束当前词法作用域或复合语句块。
- **L40 EN**: Blank line that separates nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 41-60

````cpp
/// Goes over OldF calls and replaces them with a call to NewF
static void replaceFunctionCalls(Function &OldF, Function &NewF,
                                 const std::set<int> &ArgIndexesToKeep) {
  LLVMContext &Ctx = OldF.getContext();

  const auto &Users = OldF.users();
  for (auto I = Users.begin(), E = Users.end(); I != E; )
    if (auto *CI = dyn_cast<CallInst>(*I++)) {
      // Skip uses in call instructions where OldF isn't the called function
      // (e.g. if OldF is an argument of the call).
      if (CI->getCalledFunction() != &OldF)
        continue;
      SmallVector<Value *, 8> Args;
      SmallVector<AttrBuilder, 8> ArgAttrs;

      for (auto ArgI = CI->arg_begin(), E = CI->arg_end(); ArgI != E; ++ArgI) {
        unsigned ArgIdx = ArgI - CI->arg_begin();
        if (ArgIndexesToKeep.count(ArgIdx)) {
          Args.push_back(*ArgI);
          ArgAttrs.emplace_back(Ctx, CI->getParamAttributes(ArgIdx));
````
- **L41 EN**: Comment documents the nearby logic or transformation intent: `Goes over OldF calls and replaces them with a call to NewF`.
  **L41 CN**: 注释说明了附近代码的逻辑或变换意图：`Goes over OldF calls and replaces them with a call to NewF`。
- **L42 EN**: Continues a multi-line argument list or initializer: `static void replaceFunctionCalls(Function &OldF, Function &NewF,`.
  **L42 CN**: 继续一个多行参数列表或初始化器：`static void replaceFunctionCalls(Function &OldF, Function &NewF,`。
- **L43 EN**: Continues the surrounding expression or declaration: `const std::set<int> &ArgIndexesToKeep) {`.
  **L43 CN**: 继续构造周围的表达式或声明：`const std::set<int> &ArgIndexesToKeep) {`。
- **L44 EN**: Initializes or updates `LLVMContext &Ctx` from the right-hand expression.
  **L44 CN**: 使用右侧表达式初始化或更新 `LLVMContext &Ctx`。
- **L45 EN**: Blank line that separates nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L46 EN**: Initializes or updates `const auto &Users` from the right-hand expression.
  **L46 CN**: 使用右侧表达式初始化或更新 `const auto &Users`。
- **L47 EN**: Starts a loop over a range or sequence: `for (auto I = Users.begin(), E = Users.end(); I != E; )`.
  **L47 CN**: 开始遍历某个范围或序列的循环：`for (auto I = Users.begin(), E = Users.end(); I != E; )`。
- **L48 EN**: Introduces a conditional branch: `if (auto *CI = dyn_cast<CallInst>(*I++)) {`.
  **L48 CN**: 引入条件分支：`if (auto *CI = dyn_cast<CallInst>(*I++)) {`。
- **L49 EN**: Comment documents the nearby logic or transformation intent: `Skip uses in call instructions where OldF isn't the called function`.
  **L49 CN**: 注释说明了附近代码的逻辑或变换意图：`Skip uses in call instructions where OldF isn't the called function`。
- **L50 EN**: Comment documents the nearby logic or transformation intent: `(e.g. if OldF is an argument of the call).`.
  **L50 CN**: 注释说明了附近代码的逻辑或变换意图：`(e.g. if OldF is an argument of the call).`。
- **L51 EN**: Introduces a conditional branch: `if (CI->getCalledFunction() != &OldF)`.
  **L51 CN**: 引入条件分支：`if (CI->getCalledFunction() != &OldF)`。
- **L52 EN**: Executes a standalone statement or declaration: `continue;`.
  **L52 CN**: 执行一条独立语句或声明：`continue;`。
- **L53 EN**: Executes a standalone statement or declaration: `SmallVector<Value *, 8> Args;`.
  **L53 CN**: 执行一条独立语句或声明：`SmallVector<Value *, 8> Args;`。
- **L54 EN**: Executes a standalone statement or declaration: `SmallVector<AttrBuilder, 8> ArgAttrs;`.
  **L54 CN**: 执行一条独立语句或声明：`SmallVector<AttrBuilder, 8> ArgAttrs;`。
- **L55 EN**: Blank line that separates nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L56 EN**: Starts a loop over a range or sequence: `for (auto ArgI = CI->arg_begin(), E = CI->arg_end(); ArgI != E; ++ArgI) {`.
  **L56 CN**: 开始遍历某个范围或序列的循环：`for (auto ArgI = CI->arg_begin(), E = CI->arg_end(); ArgI != E; ++ArgI) {`。
- **L57 EN**: Initializes or updates `unsigned ArgIdx` from the right-hand expression.
  **L57 CN**: 使用右侧表达式初始化或更新 `unsigned ArgIdx`。
- **L58 EN**: Introduces a conditional branch: `if (ArgIndexesToKeep.count(ArgIdx)) {`.
  **L58 CN**: 引入条件分支：`if (ArgIndexesToKeep.count(ArgIdx)) {`。
- **L59 EN**: Executes call or statement centered on `Args.push_back`.
  **L59 CN**: 执行以 `Args.push_back` 为核心的调用或语句。
- **L60 EN**: Executes call or statement centered on `ArgAttrs.emplace_back`.
  **L60 CN**: 执行以 `ArgAttrs.emplace_back` 为核心的调用或语句。

### Lines 61-80

````cpp
        }
      }

      SmallVector<OperandBundleDef, 2> OpBundles;
      CI->getOperandBundlesAsDefs(OpBundles);

      CallInst *NewCI = CallInst::Create(&NewF, Args, OpBundles);
      NewCI->setCallingConv(CI->getCallingConv());

      AttrBuilder CallSiteAttrs(Ctx, CI->getAttributes().getFnAttrs());
      NewCI->setAttributes(
          AttributeList::get(Ctx, AttributeList::FunctionIndex, CallSiteAttrs));
      NewCI->addRetAttrs(AttrBuilder(Ctx, CI->getRetAttributes()));

      unsigned AttrIdx = 0;
      for (auto ArgI = NewCI->arg_begin(), E = NewCI->arg_end(); ArgI != E;
           ++ArgI, ++AttrIdx)
        NewCI->addParamAttrs(AttrIdx, ArgAttrs[AttrIdx]);

      if (auto *FPOp = dyn_cast<FPMathOperator>(NewCI))
````
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Blank line that separates nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L64 EN**: Executes a standalone statement or declaration: `SmallVector<OperandBundleDef, 2> OpBundles;`.
  **L64 CN**: 执行一条独立语句或声明：`SmallVector<OperandBundleDef, 2> OpBundles;`。
- **L65 EN**: Executes call or statement centered on `CI->getOperandBundlesAsDefs`.
  **L65 CN**: 执行以 `CI->getOperandBundlesAsDefs` 为核心的调用或语句。
- **L66 EN**: Blank line that separates nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L67 EN**: Initializes or updates `CallInst *NewCI` from the right-hand expression.
  **L67 CN**: 使用右侧表达式初始化或更新 `CallInst *NewCI`。
- **L68 EN**: Executes call or statement centered on `NewCI->setCallingConv`.
  **L68 CN**: 执行以 `NewCI->setCallingConv` 为核心的调用或语句。
- **L69 EN**: Blank line that separates nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L70 EN**: Executes call or statement centered on `AttrBuilder CallSiteAttrs`.
  **L70 CN**: 执行以 `AttrBuilder CallSiteAttrs` 为核心的调用或语句。
- **L71 EN**: Continues a multi-line argument list or initializer: `NewCI->setAttributes(`.
  **L71 CN**: 继续一个多行参数列表或初始化器：`NewCI->setAttributes(`。
- **L72 EN**: Declares or invokes `AttributeList::get`.
  **L72 CN**: 声明或调用 `AttributeList::get`。
- **L73 EN**: Executes call or statement centered on `NewCI->addRetAttrs`.
  **L73 CN**: 执行以 `NewCI->addRetAttrs` 为核心的调用或语句。
- **L74 EN**: Blank line that separates nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Initializes or updates `unsigned AttrIdx` from the right-hand expression.
  **L75 CN**: 使用右侧表达式初始化或更新 `unsigned AttrIdx`。
- **L76 EN**: Starts a loop over a range or sequence: `for (auto ArgI = NewCI->arg_begin(), E = NewCI->arg_end(); ArgI != E;`.
  **L76 CN**: 开始遍历某个范围或序列的循环：`for (auto ArgI = NewCI->arg_begin(), E = NewCI->arg_end(); ArgI != E;`。
- **L77 EN**: Continues the surrounding expression or declaration: `++ArgI, ++AttrIdx)`.
  **L77 CN**: 继续构造周围的表达式或声明：`++ArgI, ++AttrIdx)`。
- **L78 EN**: Executes call or statement centered on `NewCI->addParamAttrs`.
  **L78 CN**: 执行以 `NewCI->addParamAttrs` 为核心的调用或语句。
- **L79 EN**: Blank line that separates nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L80 EN**: Introduces a conditional branch: `if (auto *FPOp = dyn_cast<FPMathOperator>(NewCI))`.
  **L80 CN**: 引入条件分支：`if (auto *FPOp = dyn_cast<FPMathOperator>(NewCI))`。

### Lines 81-100

````cpp
        cast<Instruction>(FPOp)->setFastMathFlags(CI->getFastMathFlags());

      NewCI->copyMetadata(*CI);

      if (!CI->use_empty())
        CI->replaceAllUsesWith(NewCI);
      ReplaceInstWithInst(CI, NewCI);
    }
}

/// Returns whether or not this function should be considered a candidate for
/// argument removal. Currently, functions with no arguments and intrinsics are
/// not considered. Intrinsics aren't considered because their signatures are
/// fixed.
static bool shouldRemoveArguments(const Function &F) {
  return !F.arg_empty() && !F.isIntrinsic();
}

static bool allFuncUsersRewritable(const Function &F) {
  for (const Use &U : F.uses()) {
````
- **L81 EN**: Executes call or statement centered on `cast<Instruction>`.
  **L81 CN**: 执行以 `cast<Instruction>` 为核心的调用或语句。
- **L82 EN**: Blank line that separates nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Executes call or statement centered on `NewCI->copyMetadata`.
  **L83 CN**: 执行以 `NewCI->copyMetadata` 为核心的调用或语句。
- **L84 EN**: Blank line that separates nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L85 EN**: Introduces a conditional branch: `if (!CI->use_empty())`.
  **L85 CN**: 引入条件分支：`if (!CI->use_empty())`。
- **L86 EN**: Executes call or statement centered on `CI->replaceAllUsesWith`.
  **L86 CN**: 执行以 `CI->replaceAllUsesWith` 为核心的调用或语句。
- **L87 EN**: Executes call or statement centered on `ReplaceInstWithInst`.
  **L87 CN**: 执行以 `ReplaceInstWithInst` 为核心的调用或语句。
- **L88 EN**: Closes the current lexical scope or compound statement.
  **L88 CN**: 结束当前词法作用域或复合语句块。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line that separates nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Comment documents the nearby logic or transformation intent: `Returns whether or not this function should be considered a candidate for`.
  **L91 CN**: 注释说明了附近代码的逻辑或变换意图：`Returns whether or not this function should be considered a candidate for`。
- **L92 EN**: Comment documents the nearby logic or transformation intent: `argument removal. Currently, functions with no arguments and intrinsics are`.
  **L92 CN**: 注释说明了附近代码的逻辑或变换意图：`argument removal. Currently, functions with no arguments and intrinsics are`。
- **L93 EN**: Comment documents the nearby logic or transformation intent: `not considered. Intrinsics aren't considered because their signatures are`.
  **L93 CN**: 注释说明了附近代码的逻辑或变换意图：`not considered. Intrinsics aren't considered because their signatures are`。
- **L94 EN**: Comment documents the nearby logic or transformation intent: `fixed.`.
  **L94 CN**: 注释说明了附近代码的逻辑或变换意图：`fixed.`。
- **L95 EN**: Starts the definition of function or method `shouldRemoveArguments`.
  **L95 CN**: 开始定义函数或方法 `shouldRemoveArguments`。
- **L96 EN**: Returns control, optionally with a value: `return !F.arg_empty() && !F.isIntrinsic();`.
  **L96 CN**: 返回控制流，并可附带返回值：`return !F.arg_empty() && !F.isIntrinsic();`。
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Blank line that separates nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L99 EN**: Starts the definition of function or method `allFuncUsersRewritable`.
  **L99 CN**: 开始定义函数或方法 `allFuncUsersRewritable`。
- **L100 EN**: Starts a loop over a range or sequence: `for (const Use &U : F.uses()) {`.
  **L100 CN**: 开始遍历某个范围或序列的循环：`for (const Use &U : F.uses()) {`。

### Lines 101-120

````cpp
    const CallBase *CB = dyn_cast<CallBase>(U.getUser());
    if (!CB || !CB->isCallee(&U))
      continue;

    // TODO: Handle all CallBase cases.
    if (!isa<CallInst>(CB))
      return false;
  }

  return true;
}

/// Removes out-of-chunk arguments from functions, and modifies their calls
/// accordingly. It also removes allocations of out-of-chunk arguments.
void llvm::reduceArgumentsDeltaPass(Oracle &O, ReducerWorkItem &WorkItem) {
  Module &Program = WorkItem.getModule();
  std::vector<Argument *> InitArgsToKeep;
  std::vector<Function *> Funcs;

  // Get inside-chunk arguments, as well as their parent function
````
- **L101 EN**: Initializes or updates `const CallBase *CB` from the right-hand expression.
  **L101 CN**: 使用右侧表达式初始化或更新 `const CallBase *CB`。
- **L102 EN**: Introduces a conditional branch: `if (!CB || !CB->isCallee(&U))`.
  **L102 CN**: 引入条件分支：`if (!CB || !CB->isCallee(&U))`。
- **L103 EN**: Executes a standalone statement or declaration: `continue;`.
  **L103 CN**: 执行一条独立语句或声明：`continue;`。
- **L104 EN**: Blank line that separates nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L105 EN**: Comment highlights an implementation note: `TODO: Handle all CallBase cases.`.
  **L105 CN**: 注释强调了一条实现说明：`TODO: Handle all CallBase cases.`。
- **L106 EN**: Introduces a conditional branch: `if (!isa<CallInst>(CB))`.
  **L106 CN**: 引入条件分支：`if (!isa<CallInst>(CB))`。
- **L107 EN**: Returns control, optionally with a value: `return false;`.
  **L107 CN**: 返回控制流，并可附带返回值：`return false;`。
- **L108 EN**: Closes the current lexical scope or compound statement.
  **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Blank line that separates nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L110 EN**: Returns control, optionally with a value: `return true;`.
  **L110 CN**: 返回控制流，并可附带返回值：`return true;`。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Blank line that separates nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L113 EN**: Comment documents the nearby logic or transformation intent: `Removes out-of-chunk arguments from functions, and modifies their calls`.
  **L113 CN**: 注释说明了附近代码的逻辑或变换意图：`Removes out-of-chunk arguments from functions, and modifies their calls`。
- **L114 EN**: Comment documents the nearby logic or transformation intent: `accordingly. It also removes allocations of out-of-chunk arguments.`.
  **L114 CN**: 注释说明了附近代码的逻辑或变换意图：`accordingly. It also removes allocations of out-of-chunk arguments.`。
- **L115 EN**: Starts the definition of function or method `llvm::reduceArgumentsDeltaPass`.
  **L115 CN**: 开始定义函数或方法 `llvm::reduceArgumentsDeltaPass`。
- **L116 EN**: Initializes or updates `Module &Program` from the right-hand expression.
  **L116 CN**: 使用右侧表达式初始化或更新 `Module &Program`。
- **L117 EN**: Executes a standalone statement or declaration: `std::vector<Argument *> InitArgsToKeep;`.
  **L117 CN**: 执行一条独立语句或声明：`std::vector<Argument *> InitArgsToKeep;`。
- **L118 EN**: Executes a standalone statement or declaration: `std::vector<Function *> Funcs;`.
  **L118 CN**: 执行一条独立语句或声明：`std::vector<Function *> Funcs;`。
- **L119 EN**: Blank line that separates nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L120 EN**: Comment documents the nearby logic or transformation intent: `Get inside-chunk arguments, as well as their parent function`.
  **L120 CN**: 注释说明了附近代码的逻辑或变换意图：`Get inside-chunk arguments, as well as their parent function`。

### Lines 121-140

````cpp
  for (auto &F : Program) {
    if (!shouldRemoveArguments(F))
      continue;
    if (!allFuncUsersRewritable(F))
      continue;
    Funcs.push_back(&F);
    for (auto &A : F.args()) {
      if (callingConvRequiresArgument(F, A) || O.shouldKeep())
        InitArgsToKeep.push_back(&A);
    }
  }

  // We create a vector first, then convert it to a set, so that we don't have
  // to pay the cost of rebalancing the set frequently if the order we insert
  // the elements doesn't match the order they should appear inside the set.
  std::set<Argument *> ArgsToKeep(InitArgsToKeep.begin(), InitArgsToKeep.end());

  for (auto *F : Funcs) {
    ValueToValueMapTy VMap;
    std::vector<WeakVH> InstToDelete;
````
- **L121 EN**: Starts a loop over a range or sequence: `for (auto &F : Program) {`.
  **L121 CN**: 开始遍历某个范围或序列的循环：`for (auto &F : Program) {`。
- **L122 EN**: Introduces a conditional branch: `if (!shouldRemoveArguments(F))`.
  **L122 CN**: 引入条件分支：`if (!shouldRemoveArguments(F))`。
- **L123 EN**: Executes a standalone statement or declaration: `continue;`.
  **L123 CN**: 执行一条独立语句或声明：`continue;`。
- **L124 EN**: Introduces a conditional branch: `if (!allFuncUsersRewritable(F))`.
  **L124 CN**: 引入条件分支：`if (!allFuncUsersRewritable(F))`。
- **L125 EN**: Executes a standalone statement or declaration: `continue;`.
  **L125 CN**: 执行一条独立语句或声明：`continue;`。
- **L126 EN**: Executes call or statement centered on `Funcs.push_back`.
  **L126 CN**: 执行以 `Funcs.push_back` 为核心的调用或语句。
- **L127 EN**: Starts a loop over a range or sequence: `for (auto &A : F.args()) {`.
  **L127 CN**: 开始遍历某个范围或序列的循环：`for (auto &A : F.args()) {`。
- **L128 EN**: Introduces a conditional branch: `if (callingConvRequiresArgument(F, A) || O.shouldKeep())`.
  **L128 CN**: 引入条件分支：`if (callingConvRequiresArgument(F, A) || O.shouldKeep())`。
- **L129 EN**: Executes call or statement centered on `InitArgsToKeep.push_back`.
  **L129 CN**: 执行以 `InitArgsToKeep.push_back` 为核心的调用或语句。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line that separates nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Comment documents the nearby logic or transformation intent: `We create a vector first, then convert it to a set, so that we don't have`.
  **L133 CN**: 注释说明了附近代码的逻辑或变换意图：`We create a vector first, then convert it to a set, so that we don't have`。
- **L134 EN**: Comment documents the nearby logic or transformation intent: `to pay the cost of rebalancing the set frequently if the order we insert`.
  **L134 CN**: 注释说明了附近代码的逻辑或变换意图：`to pay the cost of rebalancing the set frequently if the order we insert`。
- **L135 EN**: Comment documents the nearby logic or transformation intent: `the elements doesn't match the order they should appear inside the set.`.
  **L135 CN**: 注释说明了附近代码的逻辑或变换意图：`the elements doesn't match the order they should appear inside the set.`。
- **L136 EN**: Declares or invokes `ArgsToKeep`.
  **L136 CN**: 声明或调用 `ArgsToKeep`。
- **L137 EN**: Blank line that separates nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L138 EN**: Starts a loop over a range or sequence: `for (auto *F : Funcs) {`.
  **L138 CN**: 开始遍历某个范围或序列的循环：`for (auto *F : Funcs) {`。
- **L139 EN**: Executes a standalone statement or declaration: `ValueToValueMapTy VMap;`.
  **L139 CN**: 执行一条独立语句或声明：`ValueToValueMapTy VMap;`。
- **L140 EN**: Executes a standalone statement or declaration: `std::vector<WeakVH> InstToDelete;`.
  **L140 CN**: 执行一条独立语句或声明：`std::vector<WeakVH> InstToDelete;`。

### Lines 141-160

````cpp
    for (auto &A : F->args())
      if (!ArgsToKeep.count(&A)) {
        // By adding undesired arguments to the VMap, CloneFunction will remove
        // them from the resulting Function
        VMap[&A] = getDefaultValue(A.getType());
        for (auto *U : A.users())
          if (auto *I = dyn_cast<Instruction>(*&U))
            InstToDelete.push_back(I);
      }
    // Delete any (unique) instruction that uses the argument
    for (Value *V : InstToDelete) {
      if (!V)
        continue;
      auto *I = cast<Instruction>(V);
      I->replaceAllUsesWith(getDefaultValue(I->getType()));
      if (!I->isTerminator())
        I->eraseFromParent();
    }

    // No arguments to reduce
````
- **L141 EN**: Starts a loop over a range or sequence: `for (auto &A : F->args())`.
  **L141 CN**: 开始遍历某个范围或序列的循环：`for (auto &A : F->args())`。
- **L142 EN**: Introduces a conditional branch: `if (!ArgsToKeep.count(&A)) {`.
  **L142 CN**: 引入条件分支：`if (!ArgsToKeep.count(&A)) {`。
- **L143 EN**: Comment documents the nearby logic or transformation intent: `By adding undesired arguments to the VMap, CloneFunction will remove`.
  **L143 CN**: 注释说明了附近代码的逻辑或变换意图：`By adding undesired arguments to the VMap, CloneFunction will remove`。
- **L144 EN**: Comment documents the nearby logic or transformation intent: `them from the resulting Function`.
  **L144 CN**: 注释说明了附近代码的逻辑或变换意图：`them from the resulting Function`。
- **L145 EN**: Initializes or updates `VMap[&A]` from the right-hand expression.
  **L145 CN**: 使用右侧表达式初始化或更新 `VMap[&A]`。
- **L146 EN**: Starts a loop over a range or sequence: `for (auto *U : A.users())`.
  **L146 CN**: 开始遍历某个范围或序列的循环：`for (auto *U : A.users())`。
- **L147 EN**: Introduces a conditional branch: `if (auto *I = dyn_cast<Instruction>(*&U))`.
  **L147 CN**: 引入条件分支：`if (auto *I = dyn_cast<Instruction>(*&U))`。
- **L148 EN**: Executes call or statement centered on `InstToDelete.push_back`.
  **L148 CN**: 执行以 `InstToDelete.push_back` 为核心的调用或语句。
- **L149 EN**: Closes the current lexical scope or compound statement.
  **L149 CN**: 结束当前词法作用域或复合语句块。
- **L150 EN**: Comment documents the nearby logic or transformation intent: `Delete any (unique) instruction that uses the argument`.
  **L150 CN**: 注释说明了附近代码的逻辑或变换意图：`Delete any (unique) instruction that uses the argument`。
- **L151 EN**: Starts a loop over a range or sequence: `for (Value *V : InstToDelete) {`.
  **L151 CN**: 开始遍历某个范围或序列的循环：`for (Value *V : InstToDelete) {`。
- **L152 EN**: Introduces a conditional branch: `if (!V)`.
  **L152 CN**: 引入条件分支：`if (!V)`。
- **L153 EN**: Executes a standalone statement or declaration: `continue;`.
  **L153 CN**: 执行一条独立语句或声明：`continue;`。
- **L154 EN**: Initializes or updates `auto *I` from the right-hand expression.
  **L154 CN**: 使用右侧表达式初始化或更新 `auto *I`。
- **L155 EN**: Executes call or statement centered on `I->replaceAllUsesWith`.
  **L155 CN**: 执行以 `I->replaceAllUsesWith` 为核心的调用或语句。
- **L156 EN**: Introduces a conditional branch: `if (!I->isTerminator())`.
  **L156 CN**: 引入条件分支：`if (!I->isTerminator())`。
- **L157 EN**: Executes call or statement centered on `I->eraseFromParent`.
  **L157 CN**: 执行以 `I->eraseFromParent` 为核心的调用或语句。
- **L158 EN**: Closes the current lexical scope or compound statement.
  **L158 CN**: 结束当前词法作用域或复合语句块。
- **L159 EN**: Blank line that separates nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L160 EN**: Comment documents the nearby logic or transformation intent: `No arguments to reduce`.
  **L160 CN**: 注释说明了附近代码的逻辑或变换意图：`No arguments to reduce`。

### Lines 161-179

````cpp
    if (VMap.empty())
      continue;

    std::set<int> ArgIndexesToKeep;
    for (const auto &[Index, Arg] : enumerate(F->args()))
      if (ArgsToKeep.count(&Arg))
        ArgIndexesToKeep.insert(Index);

    auto *ClonedFunc = CloneFunction(F, VMap);
    // In order to preserve function order, we move Clone after old Function
    ClonedFunc->takeName(F);
    ClonedFunc->removeFromParent();
    Program.getFunctionList().insertAfter(F->getIterator(), ClonedFunc);

    replaceFunctionCalls(*F, *ClonedFunc, ArgIndexesToKeep);
    F->replaceAllUsesWith(ClonedFunc);
    F->eraseFromParent();
  }
}
````
- **L161 EN**: Introduces a conditional branch: `if (VMap.empty())`.
  **L161 CN**: 引入条件分支：`if (VMap.empty())`。
- **L162 EN**: Executes a standalone statement or declaration: `continue;`.
  **L162 CN**: 执行一条独立语句或声明：`continue;`。
- **L163 EN**: Blank line that separates nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Executes a standalone statement or declaration: `std::set<int> ArgIndexesToKeep;`.
  **L164 CN**: 执行一条独立语句或声明：`std::set<int> ArgIndexesToKeep;`。
- **L165 EN**: Starts a loop over a range or sequence: `for (const auto &[Index, Arg] : enumerate(F->args()))`.
  **L165 CN**: 开始遍历某个范围或序列的循环：`for (const auto &[Index, Arg] : enumerate(F->args()))`。
- **L166 EN**: Introduces a conditional branch: `if (ArgsToKeep.count(&Arg))`.
  **L166 CN**: 引入条件分支：`if (ArgsToKeep.count(&Arg))`。
- **L167 EN**: Executes call or statement centered on `ArgIndexesToKeep.insert`.
  **L167 CN**: 执行以 `ArgIndexesToKeep.insert` 为核心的调用或语句。
- **L168 EN**: Blank line that separates nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L169 EN**: Initializes or updates `auto *ClonedFunc` from the right-hand expression.
  **L169 CN**: 使用右侧表达式初始化或更新 `auto *ClonedFunc`。
- **L170 EN**: Comment documents the nearby logic or transformation intent: `In order to preserve function order, we move Clone after old Function`.
  **L170 CN**: 注释说明了附近代码的逻辑或变换意图：`In order to preserve function order, we move Clone after old Function`。
- **L171 EN**: Executes call or statement centered on `ClonedFunc->takeName`.
  **L171 CN**: 执行以 `ClonedFunc->takeName` 为核心的调用或语句。
- **L172 EN**: Executes call or statement centered on `ClonedFunc->removeFromParent`.
  **L172 CN**: 执行以 `ClonedFunc->removeFromParent` 为核心的调用或语句。
- **L173 EN**: Executes call or statement centered on `Program.getFunctionList`.
  **L173 CN**: 执行以 `Program.getFunctionList` 为核心的调用或语句。
- **L174 EN**: Blank line that separates nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Executes call or statement centered on `replaceFunctionCalls`.
  **L175 CN**: 执行以 `replaceFunctionCalls` 为核心的调用或语句。
- **L176 EN**: Executes call or statement centered on `F->replaceAllUsesWith`.
  **L176 CN**: 执行以 `F->replaceAllUsesWith` 为核心的调用或语句。
- **L177 EN**: Executes call or statement centered on `F->eraseFromParent`.
  **L177 CN**: 执行以 `F->eraseFromParent` 为核心的调用或语句。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Closes the current lexical scope or compound statement.
  **L179 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **LLVM utility/container usage / LLVM 工具与容器使用**
- **Core LLVM data model interaction / LLVM 核心数据模型交互**
- **Module/file-scoped coordination / 模块/文件级协调**
- **`ReduceArguments` focused implementation / 围绕 `ReduceArguments` 的实现逻辑**

## Dependencies / 依赖关系

- `ReduceArguments.h`: Provides supporting declarations from a local or system header. / 提供来自本地或系统头文件的辅助声明。
- `Utils.h`: Provides supporting declarations from a local or system header. / 提供来自本地或系统头文件的辅助声明。
- `llvm/ADT/SmallVector.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/IR/FMF.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Instructions.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Intrinsics.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Operator.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/Transforms/Utils/BasicBlockUtils.h`: Provides transform-specific declarations. / 提供变换相关声明。
- `llvm/Transforms/Utils/Cloning.h`: Provides transform-specific declarations. / 提供变换相关声明。
- `set`: Provides supporting declarations. / 提供所需的辅助声明。
- `vector`: Provides supporting declarations. / 提供所需的辅助声明。
