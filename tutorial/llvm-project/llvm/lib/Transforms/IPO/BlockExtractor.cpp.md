# BlockExtractor.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/Transforms/IPO/BlockExtractor.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This pass extracts the specified basic blocks from the module into their own functions. / 该文件位于 `Transforms/IPO`，主要实现 `BlockExtractor` 相关的 LLVM 变换逻辑、辅助流程以及 pass 接线代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- BlockExtractor.cpp - Extracts blocks into their own functions ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass extracts the specified basic blocks from the module into their
// own functions.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/IPO/BlockExtractor.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PassManager.h"
#include "llvm/Support/CommandLine.h"
```

- **L1**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L2**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L8**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L9**: Comment documents the nearby logic or transformation intent: `This pass extracts the specified basic blocks from the module into their`. / 注释说明了附近代码的逻辑或变换意图：`This pass extracts the specified basic blocks from the module into their`。
- **L10**: Comment documents the nearby logic or transformation intent: `own functions.`. / 注释说明了附近代码的逻辑或变换意图：`own functions.`。
- **L11**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L12**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L13**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L14**: Includes "llvm/Transforms/IPO/BlockExtractor.h" to access transform-specific declarations. / 引入 "llvm/Transforms/IPO/BlockExtractor.h" 以使用变换相关声明。
- **L15**: Includes "llvm/ADT/STLExtras.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/STLExtras.h" 以使用LLVM ADT 数据结构/工具。
- **L16**: Includes "llvm/ADT/Statistic.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/Statistic.h" 以使用LLVM ADT 数据结构/工具。
- **L17**: Includes "llvm/IR/Instructions.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Instructions.h" 以使用LLVM IR 核心类型与构造工具。
- **L18**: Includes "llvm/IR/Module.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Module.h" 以使用LLVM IR 核心类型与构造工具。
- **L19**: Includes "llvm/IR/PassManager.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/PassManager.h" 以使用LLVM IR 核心类型与构造工具。
- **L20**: Includes "llvm/Support/CommandLine.h" to access support-library helpers. / 引入 "llvm/Support/CommandLine.h" 以使用Support 库辅助功能。

### Lines 21-40

```cpp
#include "llvm/Support/Debug.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Transforms/IPO.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/CodeExtractor.h"

using namespace llvm;

#define DEBUG_TYPE "block-extractor"

STATISTIC(NumExtracted, "Number of basic blocks extracted");

static cl::opt<std::string> BlockExtractorFile(
    "extract-blocks-file", cl::value_desc("filename"),
    cl::desc("A file containing list of basic blocks to extract"), cl::Hidden);

static cl::opt<bool>
    BlockExtractorEraseFuncs("extract-blocks-erase-funcs",
                             cl::desc("Erase the existing functions"),
                             cl::Hidden);
```

- **L21**: Includes "llvm/Support/Debug.h" to access support-library helpers. / 引入 "llvm/Support/Debug.h" 以使用Support 库辅助功能。
- **L22**: Includes "llvm/Support/MemoryBuffer.h" to access support-library helpers. / 引入 "llvm/Support/MemoryBuffer.h" 以使用Support 库辅助功能。
- **L23**: Includes "llvm/Transforms/IPO.h" to access transform-specific declarations. / 引入 "llvm/Transforms/IPO.h" 以使用变换相关声明。
- **L24**: Includes "llvm/Transforms/Utils/BasicBlockUtils.h" to access shared transform utilities. / 引入 "llvm/Transforms/Utils/BasicBlockUtils.h" 以使用共享的变换辅助工具。
- **L25**: Includes "llvm/Transforms/Utils/CodeExtractor.h" to access shared transform utilities. / 引入 "llvm/Transforms/Utils/CodeExtractor.h" 以使用共享的变换辅助工具。
- **L26**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L27**: Brings namespace `llvm` into the local scope. / 将命名空间 `llvm` 引入当前作用域。
- **L28**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L29**: Defines macro `DEBUG_TYPE` for later conditional logic, flags, or diagnostics. / 定义宏 `DEBUG_TYPE`，供后续条件逻辑、标志位或诊断使用。
- **L30**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L31**: Registers LLVM statistic counter `NumExtracted`. / 注册 LLVM 统计计数器 `NumExtracted`。
- **L32**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L33**: Declares a command-line option or tunable parameter: `static cl::opt<std::string> BlockExtractorFile(`. / 声明一个命令行选项或可调参数：`static cl::opt<std::string> BlockExtractorFile(`。
- **L34**: Continues a multi-line argument list or initializer: `"extract-blocks-file", cl::value_desc("filename"),`. / 继续一个多行参数列表或初始化器：`"extract-blocks-file", cl::value_desc("filename"),`。
- **L35**: Executes call or statement centered on `cl::desc`. / 执行以 `cl::desc` 为核心的调用或语句。
- **L36**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L37**: Declares a command-line option or tunable parameter: `static cl::opt<bool>`. / 声明一个命令行选项或可调参数：`static cl::opt<bool>`。
- **L38**: Continues a multi-line argument list or initializer: `BlockExtractorEraseFuncs("extract-blocks-erase-funcs",`. / 继续一个多行参数列表或初始化器：`BlockExtractorEraseFuncs("extract-blocks-erase-funcs",`。
- **L39**: Continues a multi-line argument list or initializer: `cl::desc("Erase the existing functions"),`. / 继续一个多行参数列表或初始化器：`cl::desc("Erase the existing functions"),`。
- **L40**: Executes a standalone statement or declaration: `cl::Hidden);`. / 执行一条独立语句或声明：`cl::Hidden);`。

### Lines 41-60

```cpp
namespace {
class BlockExtractor {
public:
  BlockExtractor(bool EraseFunctions) : EraseFunctions(EraseFunctions) {}
  bool runOnModule(Module &M);
  void
  init(const std::vector<std::vector<BasicBlock *>> &GroupsOfBlocksToExtract) {
    GroupsOfBlocks = GroupsOfBlocksToExtract;
    if (!BlockExtractorFile.empty())
      loadFile();
  }

private:
  std::vector<std::vector<BasicBlock *>> GroupsOfBlocks;
  bool EraseFunctions;
  /// Map a function name to groups of blocks.
  SmallVector<std::pair<std::string, SmallVector<std::string, 4>>, 4>
      BlocksByName;

  void loadFile();
```

- **L41**: Opens namespace scope ``. / 打开命名空间作用域 ``。
- **L42**: Declares class `BlockExtractor`. / 声明 class `BlockExtractor`。
- **L43**: Sets the following members to `public` access. / 将后续成员的访问级别设为 `public`。
- **L44**: Continues the surrounding expression or declaration: `BlockExtractor(bool EraseFunctions) : EraseFunctions(EraseFunctions) {}`. / 继续构造周围的表达式或声明：`BlockExtractor(bool EraseFunctions) : EraseFunctions(EraseFunctions) {}`。
- **L45**: Executes call or statement centered on `runOnModule`. / 执行以 `runOnModule` 为核心的调用或语句。
- **L46**: Continues the surrounding expression or declaration: `void`. / 继续构造周围的表达式或声明：`void`。
- **L47**: Starts a function, method, or lambda body: `init(const std::vector<std::vector<BasicBlock *>> &GroupsOfBlocksToExtract) {`. / 开始一个函数、方法或 lambda 的主体：`init(const std::vector<std::vector<BasicBlock *>> &GroupsOfBlocksToExtract) {`。
- **L48**: Executes a standalone statement or declaration: `GroupsOfBlocks = GroupsOfBlocksToExtract;`. / 执行一条独立语句或声明：`GroupsOfBlocks = GroupsOfBlocksToExtract;`。
- **L49**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L50**: Executes call or statement centered on `loadFile`. / 执行以 `loadFile` 为核心的调用或语句。
- **L51**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L52**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L53**: Sets the following members to `private` access. / 将后续成员的访问级别设为 `private`。
- **L54**: Executes a standalone statement or declaration: `std::vector<std::vector<BasicBlock *>> GroupsOfBlocks;`. / 执行一条独立语句或声明：`std::vector<std::vector<BasicBlock *>> GroupsOfBlocks;`。
- **L55**: Executes a standalone statement or declaration: `bool EraseFunctions;`. / 执行一条独立语句或声明：`bool EraseFunctions;`。
- **L56**: Comment documents the nearby logic or transformation intent: `Map a function name to groups of blocks.`. / 注释说明了附近代码的逻辑或变换意图：`Map a function name to groups of blocks.`。
- **L57**: Continues the surrounding expression or declaration: `SmallVector<std::pair<std::string, SmallVector<std::string, 4>>, 4>`. / 继续构造周围的表达式或声明：`SmallVector<std::pair<std::string, SmallVector<std::string, 4>>, 4>`。
- **L58**: Executes a standalone statement or declaration: `BlocksByName;`. / 执行一条独立语句或声明：`BlocksByName;`。
- **L59**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L60**: Executes call or statement centered on `loadFile`. / 执行以 `loadFile` 为核心的调用或语句。

### Lines 61-80

```cpp
  void splitLandingPadPreds(Function &F);
};

} // end anonymous namespace

/// Gets all of the blocks specified in the input file.
void BlockExtractor::loadFile() {
  auto ErrOrBuf = MemoryBuffer::getFile(BlockExtractorFile);
  if (ErrOrBuf.getError())
    report_fatal_error("BlockExtractor couldn't load the file.");
  // Read the file.
  auto &Buf = *ErrOrBuf;
  SmallVector<StringRef, 16> Lines;
  Buf->getBuffer().split(Lines, '\n', /*MaxSplit=*/-1,
                         /*KeepEmpty=*/false);
  for (const auto &Line : Lines) {
    SmallVector<StringRef, 4> LineSplit;
    Line.split(LineSplit, ' ', /*MaxSplit=*/-1,
               /*KeepEmpty=*/false);
    if (LineSplit.empty())
```

- **L61**: Executes call or statement centered on `splitLandingPadPreds`. / 执行以 `splitLandingPadPreds` 为核心的调用或语句。
- **L62**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L63**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L64**: Continues the surrounding expression or declaration: `} // end anonymous namespace`. / 继续构造周围的表达式或声明：`} // end anonymous namespace`。
- **L65**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L66**: Comment documents the nearby logic or transformation intent: `Gets all of the blocks specified in the input file.`. / 注释说明了附近代码的逻辑或变换意图：`Gets all of the blocks specified in the input file.`。
- **L67**: Starts a function, method, or lambda body: `void BlockExtractor::loadFile() {`. / 开始一个函数、方法或 lambda 的主体：`void BlockExtractor::loadFile() {`。
- **L68**: Initializes variable `ErrOrBuf` from the right-hand expression. / 使用右侧表达式初始化变量 `ErrOrBuf`。
- **L69**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L70**: Executes call or statement centered on `report_fatal_error`. / 执行以 `report_fatal_error` 为核心的调用或语句。
- **L71**: Comment documents the nearby logic or transformation intent: `Read the file.`. / 注释说明了附近代码的逻辑或变换意图：`Read the file.`。
- **L72**: Executes a standalone statement or declaration: `auto &Buf = *ErrOrBuf;`. / 执行一条独立语句或声明：`auto &Buf = *ErrOrBuf;`。
- **L73**: Executes a standalone statement or declaration: `SmallVector<StringRef, 16> Lines;`. / 执行一条独立语句或声明：`SmallVector<StringRef, 16> Lines;`。
- **L74**: Continues a multi-line argument list or initializer: `Buf->getBuffer().split(Lines, '\n', /*MaxSplit=*/-1,`. / 继续一个多行参数列表或初始化器：`Buf->getBuffer().split(Lines, '\n', /*MaxSplit=*/-1,`。
- **L75**: Comment documents the nearby logic or transformation intent: `KeepEmpty=*/false);`. / 注释说明了附近代码的逻辑或变换意图：`KeepEmpty=*/false);`。
- **L76**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L77**: Executes a standalone statement or declaration: `SmallVector<StringRef, 4> LineSplit;`. / 执行一条独立语句或声明：`SmallVector<StringRef, 4> LineSplit;`。
- **L78**: Continues a multi-line argument list or initializer: `Line.split(LineSplit, ' ', /*MaxSplit=*/-1,`. / 继续一个多行参数列表或初始化器：`Line.split(LineSplit, ' ', /*MaxSplit=*/-1,`。
- **L79**: Comment documents the nearby logic or transformation intent: `KeepEmpty=*/false);`. / 注释说明了附近代码的逻辑或变换意图：`KeepEmpty=*/false);`。
- **L80**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。

### Lines 81-100

```cpp
      continue;
    if (LineSplit.size()!=2)
      reportFatalUsageError(
          "Invalid line format, expecting lines like: 'funcname bb1[;bb2..]'");
    SmallVector<StringRef, 4> BBNames;
    LineSplit[1].split(BBNames, ';', /*MaxSplit=*/-1,
                       /*KeepEmpty=*/false);
    if (BBNames.empty())
      report_fatal_error("Missing bbs name");
    BlocksByName.push_back(
        {std::string(LineSplit[0]), {BBNames.begin(), BBNames.end()}});
  }
}

/// Extracts the landing pads to make sure all of them have only one
/// predecessor.
void BlockExtractor::splitLandingPadPreds(Function &F) {
  for (BasicBlock &BB : F) {
    for (Instruction &I : BB) {
      if (!isa<InvokeInst>(&I))
```

- **L81**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L82**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L83**: Continues the surrounding expression or declaration: `reportFatalUsageError(`. / 继续构造周围的表达式或声明：`reportFatalUsageError(`。
- **L84**: Executes a standalone statement or declaration: `"Invalid line format, expecting lines like: 'funcname bb1[;bb2..]'");`. / 执行一条独立语句或声明：`"Invalid line format, expecting lines like: 'funcname bb1[;bb2..]'");`。
- **L85**: Executes a standalone statement or declaration: `SmallVector<StringRef, 4> BBNames;`. / 执行一条独立语句或声明：`SmallVector<StringRef, 4> BBNames;`。
- **L86**: Continues a multi-line argument list or initializer: `LineSplit[1].split(BBNames, ';', /*MaxSplit=*/-1,`. / 继续一个多行参数列表或初始化器：`LineSplit[1].split(BBNames, ';', /*MaxSplit=*/-1,`。
- **L87**: Comment documents the nearby logic or transformation intent: `KeepEmpty=*/false);`. / 注释说明了附近代码的逻辑或变换意图：`KeepEmpty=*/false);`。
- **L88**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L89**: Executes call or statement centered on `report_fatal_error`. / 执行以 `report_fatal_error` 为核心的调用或语句。
- **L90**: Continues the surrounding expression or declaration: `BlocksByName.push_back(`. / 继续构造周围的表达式或声明：`BlocksByName.push_back(`。
- **L91**: Executes call or statement centered on `{std::string`. / 执行以 `{std::string` 为核心的调用或语句。
- **L92**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L93**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L94**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L95**: Comment documents the nearby logic or transformation intent: `Extracts the landing pads to make sure all of them have only one`. / 注释说明了附近代码的逻辑或变换意图：`Extracts the landing pads to make sure all of them have only one`。
- **L96**: Comment documents the nearby logic or transformation intent: `predecessor.`. / 注释说明了附近代码的逻辑或变换意图：`predecessor.`。
- **L97**: Starts a function, method, or lambda body: `void BlockExtractor::splitLandingPadPreds(Function &F) {`. / 开始一个函数、方法或 lambda 的主体：`void BlockExtractor::splitLandingPadPreds(Function &F) {`。
- **L98**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L99**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L100**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。

### Lines 101-120

```cpp
        continue;
      InvokeInst *II = cast<InvokeInst>(&I);
      BasicBlock *Parent = II->getParent();
      BasicBlock *LPad = II->getUnwindDest();

      // Look through the landing pad's predecessors. If one of them ends in an
      // 'invoke', then we want to split the landing pad.
      bool Split = false;
      for (auto *PredBB : predecessors(LPad)) {
        if (PredBB->isLandingPad() && PredBB != Parent &&
            isa<InvokeInst>(Parent->getTerminator())) {
          Split = true;
          break;
        }
      }

      if (!Split)
        continue;

      SmallVector<BasicBlock *, 2> NewBBs;
```

- **L101**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L102**: Executes call or statement centered on `cast<InvokeInst>`. / 执行以 `cast<InvokeInst>` 为核心的调用或语句。
- **L103**: Executes call or statement centered on `II->getParent`. / 执行以 `II->getParent` 为核心的调用或语句。
- **L104**: Executes call or statement centered on `II->getUnwindDest`. / 执行以 `II->getUnwindDest` 为核心的调用或语句。
- **L105**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L106**: Comment documents the nearby logic or transformation intent: `Look through the landing pad's predecessors. If one of them ends in an`. / 注释说明了附近代码的逻辑或变换意图：`Look through the landing pad's predecessors. If one of them ends in an`。
- **L107**: Comment documents the nearby logic or transformation intent: `'invoke', then we want to split the landing pad.`. / 注释说明了附近代码的逻辑或变换意图：`'invoke', then we want to split the landing pad.`。
- **L108**: Initializes variable `Split` from the right-hand expression. / 使用右侧表达式初始化变量 `Split`。
- **L109**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L110**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L111**: Starts a function, method, or lambda body: `isa<InvokeInst>(Parent->getTerminator())) {`. / 开始一个函数、方法或 lambda 的主体：`isa<InvokeInst>(Parent->getTerminator())) {`。
- **L112**: Executes a standalone statement or declaration: `Split = true;`. / 执行一条独立语句或声明：`Split = true;`。
- **L113**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L114**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L115**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L116**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L117**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L118**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L119**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L120**: Executes a standalone statement or declaration: `SmallVector<BasicBlock *, 2> NewBBs;`. / 执行一条独立语句或声明：`SmallVector<BasicBlock *, 2> NewBBs;`。

### Lines 121-140

```cpp
      SplitLandingPadPredecessors(LPad, Parent, ".1", ".2", NewBBs);
    }
  }
}

bool BlockExtractor::runOnModule(Module &M) {
  bool Changed = false;

  // Get all the functions.
  SmallVector<Function *, 4> Functions;
  for (Function &F : M) {
    splitLandingPadPreds(F);
    Functions.push_back(&F);
  }

  // Get all the blocks specified in the input file.
  unsigned NextGroupIdx = GroupsOfBlocks.size();
  GroupsOfBlocks.resize(NextGroupIdx + BlocksByName.size());
  for (const auto &BInfo : BlocksByName) {
    Function *F = M.getFunction(BInfo.first);
```

- **L121**: Executes call or statement centered on `SplitLandingPadPredecessors`. / 执行以 `SplitLandingPadPredecessors` 为核心的调用或语句。
- **L122**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L123**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L124**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L125**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L126**: Starts a function, method, or lambda body: `bool BlockExtractor::runOnModule(Module &M) {`. / 开始一个函数、方法或 lambda 的主体：`bool BlockExtractor::runOnModule(Module &M) {`。
- **L127**: Initializes variable `Changed` from the right-hand expression. / 使用右侧表达式初始化变量 `Changed`。
- **L128**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L129**: Comment documents the nearby logic or transformation intent: `Get all the functions.`. / 注释说明了附近代码的逻辑或变换意图：`Get all the functions.`。
- **L130**: Executes a standalone statement or declaration: `SmallVector<Function *, 4> Functions;`. / 执行一条独立语句或声明：`SmallVector<Function *, 4> Functions;`。
- **L131**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L132**: Executes call or statement centered on `splitLandingPadPreds`. / 执行以 `splitLandingPadPreds` 为核心的调用或语句。
- **L133**: Executes call or statement centered on `Functions.push_back`. / 执行以 `Functions.push_back` 为核心的调用或语句。
- **L134**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L135**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L136**: Comment documents the nearby logic or transformation intent: `Get all the blocks specified in the input file.`. / 注释说明了附近代码的逻辑或变换意图：`Get all the blocks specified in the input file.`。
- **L137**: Initializes variable `NextGroupIdx` from the right-hand expression. / 使用右侧表达式初始化变量 `NextGroupIdx`。
- **L138**: Executes call or statement centered on `GroupsOfBlocks.resize`. / 执行以 `GroupsOfBlocks.resize` 为核心的调用或语句。
- **L139**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L140**: Executes call or statement centered on `M.getFunction`. / 执行以 `M.getFunction` 为核心的调用或语句。

### Lines 141-160

```cpp
    if (!F)
      reportFatalUsageError(
          "Invalid function name specified in the input file");
    for (const auto &BBInfo : BInfo.second) {
      auto Res = llvm::find_if(
          *F, [&](const BasicBlock &BB) { return BB.getName() == BBInfo; });
      if (Res == F->end())
        reportFatalUsageError("Invalid block name specified in the input file");
      GroupsOfBlocks[NextGroupIdx].push_back(&*Res);
    }
    ++NextGroupIdx;
  }

  // Extract each group of basic blocks.
  for (auto &BBs : GroupsOfBlocks) {
    SmallVector<BasicBlock *, 32> BlocksToExtractVec;
    for (BasicBlock *BB : BBs) {
      // Check if the module contains BB.
      if (BB->getParent()->getParent() != &M)
        reportFatalUsageError("Invalid basic block");
```

- **L141**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L142**: Continues the surrounding expression or declaration: `reportFatalUsageError(`. / 继续构造周围的表达式或声明：`reportFatalUsageError(`。
- **L143**: Executes a standalone statement or declaration: `"Invalid function name specified in the input file");`. / 执行一条独立语句或声明：`"Invalid function name specified in the input file");`。
- **L144**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L145**: Continues the surrounding expression or declaration: `auto Res = llvm::find_if(`. / 继续构造周围的表达式或声明：`auto Res = llvm::find_if(`。
- **L146**: Comment documents the nearby logic or transformation intent: `F, [&](const BasicBlock &BB) { return BB.getName() == BBInfo; });`. / 注释说明了附近代码的逻辑或变换意图：`F, [&](const BasicBlock &BB) { return BB.getName() == BBInfo; });`。
- **L147**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L148**: Executes call or statement centered on `reportFatalUsageError`. / 执行以 `reportFatalUsageError` 为核心的调用或语句。
- **L149**: Executes call or statement centered on `GroupsOfBlocks[NextGroupIdx].push_back`. / 执行以 `GroupsOfBlocks[NextGroupIdx].push_back` 为核心的调用或语句。
- **L150**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L151**: Executes a standalone statement or declaration: `++NextGroupIdx;`. / 执行一条独立语句或声明：`++NextGroupIdx;`。
- **L152**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L153**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L154**: Comment documents the nearby logic or transformation intent: `Extract each group of basic blocks.`. / 注释说明了附近代码的逻辑或变换意图：`Extract each group of basic blocks.`。
- **L155**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L156**: Executes a standalone statement or declaration: `SmallVector<BasicBlock *, 32> BlocksToExtractVec;`. / 执行一条独立语句或声明：`SmallVector<BasicBlock *, 32> BlocksToExtractVec;`。
- **L157**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L158**: Comment documents the nearby logic or transformation intent: `Check if the module contains BB.`. / 注释说明了附近代码的逻辑或变换意图：`Check if the module contains BB.`。
- **L159**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L160**: Executes call or statement centered on `reportFatalUsageError`. / 执行以 `reportFatalUsageError` 为核心的调用或语句。

### Lines 161-180

```cpp
      LLVM_DEBUG(dbgs() << "BlockExtractor: Extracting "
                        << BB->getParent()->getName() << ":" << BB->getName()
                        << "\n");
      BlocksToExtractVec.push_back(BB);
      if (const InvokeInst *II = dyn_cast<InvokeInst>(BB->getTerminator()))
        BlocksToExtractVec.push_back(II->getUnwindDest());
      ++NumExtracted;
      Changed = true;
    }
    CodeExtractorAnalysisCache CEAC(*BBs[0]->getParent());
    Function *F = CodeExtractor(BlocksToExtractVec).extractCodeRegion(CEAC);
    if (F)
      LLVM_DEBUG(dbgs() << "Extracted group '" << (*BBs.begin())->getName()
                        << "' in: " << F->getName() << '\n');
    else
      LLVM_DEBUG(dbgs() << "Failed to extract for group '"
                        << (*BBs.begin())->getName() << "'\n");
  }

  // Erase the functions.
```

- **L161**: Continues the surrounding expression or declaration: `LLVM_DEBUG(dbgs() << "BlockExtractor: Extracting "`. / 继续构造周围的表达式或声明：`LLVM_DEBUG(dbgs() << "BlockExtractor: Extracting "`。
- **L162**: Continues the surrounding expression or declaration: `<< BB->getParent()->getName() << ":" << BB->getName()`. / 继续构造周围的表达式或声明：`<< BB->getParent()->getName() << ":" << BB->getName()`。
- **L163**: Executes a standalone statement or declaration: `<< "\n");`. / 执行一条独立语句或声明：`<< "\n");`。
- **L164**: Executes call or statement centered on `BlocksToExtractVec.push_back`. / 执行以 `BlocksToExtractVec.push_back` 为核心的调用或语句。
- **L165**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L166**: Executes call or statement centered on `BlocksToExtractVec.push_back`. / 执行以 `BlocksToExtractVec.push_back` 为核心的调用或语句。
- **L167**: Executes a standalone statement or declaration: `++NumExtracted;`. / 执行一条独立语句或声明：`++NumExtracted;`。
- **L168**: Executes a standalone statement or declaration: `Changed = true;`. / 执行一条独立语句或声明：`Changed = true;`。
- **L169**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L170**: Executes call or statement centered on `CEAC`. / 执行以 `CEAC` 为核心的调用或语句。
- **L171**: Executes call or statement centered on `CodeExtractor`. / 执行以 `CodeExtractor` 为核心的调用或语句。
- **L172**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L173**: Continues the surrounding expression or declaration: `LLVM_DEBUG(dbgs() << "Extracted group '" << (*BBs.begin())->getName()`. / 继续构造周围的表达式或声明：`LLVM_DEBUG(dbgs() << "Extracted group '" << (*BBs.begin())->getName()`。
- **L174**: Executes call or statement centered on `F->getName`. / 执行以 `F->getName` 为核心的调用或语句。
- **L175**: Starts the alternative branch of the preceding conditional. / 开始前一个条件语句的备选分支。
- **L176**: Continues the surrounding expression or declaration: `LLVM_DEBUG(dbgs() << "Failed to extract for group '"`. / 继续构造周围的表达式或声明：`LLVM_DEBUG(dbgs() << "Failed to extract for group '"`。
- **L177**: Executes call or statement centered on `<<`. / 执行以 `<<` 为核心的调用或语句。
- **L178**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L179**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L180**: Comment documents the nearby logic or transformation intent: `Erase the functions.`. / 注释说明了附近代码的逻辑或变换意图：`Erase the functions.`。

### Lines 181-200

```cpp
  if (EraseFunctions || BlockExtractorEraseFuncs) {
    for (Function *F : Functions) {
      LLVM_DEBUG(dbgs() << "BlockExtractor: Trying to delete " << F->getName()
                        << "\n");
      F->deleteBody();
    }
    // Set linkage as ExternalLinkage to avoid erasing unreachable functions.
    for (Function &F : M)
      F.setLinkage(GlobalValue::ExternalLinkage);
    Changed = true;
  }

  return Changed;
}

BlockExtractorPass::BlockExtractorPass(
    std::vector<std::vector<BasicBlock *>> &&GroupsOfBlocks,
    bool EraseFunctions)
    : GroupsOfBlocks(std::move(GroupsOfBlocks)),
      EraseFunctions(EraseFunctions) {}
```

- **L181**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L182**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L183**: Continues the surrounding expression or declaration: `LLVM_DEBUG(dbgs() << "BlockExtractor: Trying to delete " << F->getName()`. / 继续构造周围的表达式或声明：`LLVM_DEBUG(dbgs() << "BlockExtractor: Trying to delete " << F->getName()`。
- **L184**: Executes a standalone statement or declaration: `<< "\n");`. / 执行一条独立语句或声明：`<< "\n");`。
- **L185**: Executes call or statement centered on `F->deleteBody`. / 执行以 `F->deleteBody` 为核心的调用或语句。
- **L186**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L187**: Comment documents the nearby logic or transformation intent: `Set linkage as ExternalLinkage to avoid erasing unreachable functions.`. / 注释说明了附近代码的逻辑或变换意图：`Set linkage as ExternalLinkage to avoid erasing unreachable functions.`。
- **L188**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L189**: Executes call or statement centered on `F.setLinkage`. / 执行以 `F.setLinkage` 为核心的调用或语句。
- **L190**: Executes a standalone statement or declaration: `Changed = true;`. / 执行一条独立语句或声明：`Changed = true;`。
- **L191**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L192**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L193**: Returns from the current function with `Changed`. / 以 `Changed` 从当前函数返回。
- **L194**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L195**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L196**: Continues the surrounding expression or declaration: `BlockExtractorPass::BlockExtractorPass(`. / 继续构造周围的表达式或声明：`BlockExtractorPass::BlockExtractorPass(`。
- **L197**: Continues a multi-line argument list or initializer: `std::vector<std::vector<BasicBlock *>> &&GroupsOfBlocks,`. / 继续一个多行参数列表或初始化器：`std::vector<std::vector<BasicBlock *>> &&GroupsOfBlocks,`。
- **L198**: Continues the surrounding expression or declaration: `bool EraseFunctions)`. / 继续构造周围的表达式或声明：`bool EraseFunctions)`。
- **L199**: Continues a multi-line argument list or initializer: `: GroupsOfBlocks(std::move(GroupsOfBlocks)),`. / 继续一个多行参数列表或初始化器：`: GroupsOfBlocks(std::move(GroupsOfBlocks)),`。
- **L200**: Continues the surrounding expression or declaration: `EraseFunctions(EraseFunctions) {}`. / 继续构造周围的表达式或声明：`EraseFunctions(EraseFunctions) {}`。

### Lines 201-208

```cpp

PreservedAnalyses BlockExtractorPass::run(Module &M,
                                          ModuleAnalysisManager &AM) {
  BlockExtractor BE(EraseFunctions);
  BE.init(GroupsOfBlocks);
  return BE.runOnModule(M) ? PreservedAnalyses::none()
                           : PreservedAnalyses::all();
}
```

- **L201**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L202**: Continues a multi-line argument list or initializer: `PreservedAnalyses BlockExtractorPass::run(Module &M,`. / 继续一个多行参数列表或初始化器：`PreservedAnalyses BlockExtractorPass::run(Module &M,`。
- **L203**: Continues the surrounding expression or declaration: `ModuleAnalysisManager &AM) {`. / 继续构造周围的表达式或声明：`ModuleAnalysisManager &AM) {`。
- **L204**: Executes call or statement centered on `BE`. / 执行以 `BE` 为核心的调用或语句。
- **L205**: Executes call or statement centered on `BE.init`. / 执行以 `BE.init` 为核心的调用或语句。
- **L206**: Returns from the current function with `BE.runOnModule(M) ? PreservedAnalyses::none()`. / 以 `BE.runOnModule(M) ? PreservedAnalyses::none()` 从当前函数返回。
- **L207**: Executes call or statement centered on `PreservedAnalyses::all`. / 执行以 `PreservedAnalyses::all` 为核心的调用或语句。
- **L208**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **IPO transform pipeline / IPO 变换流水线**
- **Function-level traversal and updates / 函数级遍历与更新**
- **Module-wide coordination / 模块范围的协调**
- **Pass statistics and instrumentation / Pass 统计与观测**
- **Analysis preservation reporting / 分析保持情况报告**

## Dependencies / 依赖关系

- `llvm/Transforms/IPO/BlockExtractor.h`: Provides transform-specific declarations. / 提供变换相关声明。
- `llvm/ADT/STLExtras.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/ADT/Statistic.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/IR/Instructions.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Module.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/PassManager.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/Support/CommandLine.h`: Provides support-library helpers. / 提供Support 库辅助功能。
- `llvm/Support/Debug.h`: Provides support-library helpers. / 提供Support 库辅助功能。
- `llvm/Support/MemoryBuffer.h`: Provides support-library helpers. / 提供Support 库辅助功能。
- `llvm/Transforms/IPO.h`: Provides transform-specific declarations. / 提供变换相关声明。
- `llvm/Transforms/Utils/BasicBlockUtils.h`: Provides shared transform utilities. / 提供共享的变换辅助工具。
- `llvm/Transforms/Utils/CodeExtractor.h`: Provides shared transform utilities. / 提供共享的变换辅助工具。
