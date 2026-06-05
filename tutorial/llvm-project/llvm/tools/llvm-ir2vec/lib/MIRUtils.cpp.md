# MIRUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/tools/llvm-ir2vec/lib/MIRUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: MIR2Vec Embedding Generation This file implements the MIR2VecTool class for MIR2Vec embedding generation from LLVM Machine IR. It has no dependency on the IR2Vec embedding API. / 该文件位于 `llvm-ir2vec/lib`，主要实现与 `MIRUtils` 相关的逻辑、数据处理或辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- MIRUtils.cpp - MIR2Vec Embedding Generation --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the MIR2VecTool class for MIR2Vec embedding generation
/// from LLVM Machine IR. It has no dependency on the IR2Vec embedding API.
///
//===----------------------------------------------------------------------===//

#include "MIRUtils.h"
#include "llvm/CodeGen/MIR2Vec.h"
#include "llvm/CodeGen/MIRParser/MIRParser.h"
#include "llvm/CodeGen/MachineFunction.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used to visually divide sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment explains nearby logic or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic or intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually divide sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Separator comment used to visually divide sections. / 分隔性注释，用于在视觉上划分小节。
- **L9**: Comment explains nearby logic or intent: `\file`. / 注释说明了附近代码的逻辑或设计意图：`\file`。
- **L10**: Comment explains nearby logic or intent: `This file implements the MIR2VecTool class for MIR2Vec embedding generation`. / 注释说明了附近代码的逻辑或设计意图：`This file implements the MIR2VecTool class for MIR2Vec embedding generation`。
- **L11**: Comment explains nearby logic or intent: `from LLVM Machine IR. It has no dependency on the IR2Vec embedding API.`. / 注释说明了附近代码的逻辑或设计意图：`from LLVM Machine IR. It has no dependency on the IR2Vec embedding API.`。
- **L12**: Separator comment used to visually divide sections. / 分隔性注释，用于在视觉上划分小节。
- **L13**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L14**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L15**: Includes `MIRUtils.h` to access local declarations paired with this implementation file. / 引入 `MIRUtils.h` 以使用与该实现文件配套的本地声明。
- **L16**: Includes `llvm/CodeGen/MIR2Vec.h` to access code-generation infrastructure. / 引入 `llvm/CodeGen/MIR2Vec.h` 以使用代码生成基础设施。
- **L17**: Includes `llvm/CodeGen/MIRParser/MIRParser.h` to access code-generation infrastructure. / 引入 `llvm/CodeGen/MIRParser/MIRParser.h` 以使用代码生成基础设施。
- **L18**: Includes `llvm/CodeGen/MachineFunction.h` to access code-generation infrastructure. / 引入 `llvm/CodeGen/MachineFunction.h` 以使用代码生成基础设施。

### Lines 19-36

```cpp
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/WithColor.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Target/TargetMachine.h"

#define DEBUG_TYPE "ir2vec"

namespace llvm {
namespace mir2vec {

bool MIR2VecTool::initializeVocabulary(const Module &M) {
  MIR2VecVocabProvider Provider(MMI);
  auto VocabOrErr = Provider.getVocabulary(M);
  if (!VocabOrErr) {
    WithColor::error(errs(), ToolName)
```

- **L19**: Includes `llvm/CodeGen/MachineModuleInfo.h` to access code-generation infrastructure. / 引入 `llvm/CodeGen/MachineModuleInfo.h` 以使用代码生成基础设施。
- **L20**: Includes `llvm/CodeGen/TargetInstrInfo.h` to access code-generation infrastructure. / 引入 `llvm/CodeGen/TargetInstrInfo.h` 以使用代码生成基础设施。
- **L21**: Includes `llvm/CodeGen/TargetRegisterInfo.h` to access code-generation infrastructure. / 引入 `llvm/CodeGen/TargetRegisterInfo.h` 以使用代码生成基础设施。
- **L22**: Includes `llvm/Support/Debug.h` to access LLVM support-library facilities. / 引入 `llvm/Support/Debug.h` 以使用LLVM 支持库设施。
- **L23**: Includes `llvm/Support/WithColor.h` to access LLVM support-library facilities. / 引入 `llvm/Support/WithColor.h` 以使用LLVM 支持库设施。
- **L24**: Includes `llvm/Support/raw_ostream.h` to access LLVM support-library facilities. / 引入 `llvm/Support/raw_ostream.h` 以使用LLVM 支持库设施。
- **L25**: Includes `llvm/Target/TargetMachine.h` to access target backend interfaces. / 引入 `llvm/Target/TargetMachine.h` 以使用目标后端接口。
- **L26**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L27**: Defines macro `DEBUG_TYPE` for later conditional logic or annotations. / 定义宏 `DEBUG_TYPE`，供后续条件逻辑或注解使用。
- **L28**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L29**: Opens namespace scope `llvm`. / 打开命名空间作用域 `llvm`。
- **L30**: Opens namespace scope `mir2vec`. / 打开命名空间作用域 `mir2vec`。
- **L31**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L32**: Starts the definition of function or method `MIR2VecTool::initializeVocabulary`. / 开始定义函数或方法 `MIR2VecTool::initializeVocabulary`。
- **L33**: Declares or invokes `Provider`. / 声明或调用 `Provider`。
- **L34**: Declares or invokes `Provider.getVocabulary`. / 声明或调用 `Provider.getVocabulary`。
- **L35**: Introduces a conditional branch: `if (!VocabOrErr) {`. / 引入条件分支：`if (!VocabOrErr) {`。
- **L36**: Continues the surrounding expression or declaration: `WithColor::error(errs(), ToolName)`. / 继续构造周围的表达式或声明：`WithColor::error(errs(), ToolName)`。

### Lines 37-54

```cpp
        << "Failed to load MIR2Vec vocabulary - "
        << toString(VocabOrErr.takeError()) << "\n";
    return false;
  }
  Vocab = std::make_unique<MIRVocabulary>(std::move(*VocabOrErr));
  return true;
}

bool MIR2VecTool::initializeVocabularyForLayout(const Module &M) {
  for (const Function &F : M.getFunctionDefs()) {
    MachineFunction *MF = MMI.getMachineFunction(F);
    if (!MF)
      continue;

    const TargetInstrInfo &TII = *MF->getSubtarget().getInstrInfo();
    const TargetRegisterInfo &TRI = *MF->getSubtarget().getRegisterInfo();
    const MachineRegisterInfo &MRI = MF->getRegInfo();

```

- **L37**: Continues the surrounding expression or declaration: `<< "Failed to load MIR2Vec vocabulary - "`. / 继续构造周围的表达式或声明：`<< "Failed to load MIR2Vec vocabulary - "`。
- **L38**: Declares or invokes `toString`. / 声明或调用 `toString`。
- **L39**: Returns control, optionally with a value: `return false;`. / 返回控制流，并可附带返回值：`return false;`。
- **L40**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L41**: Declares or invokes `std::make_unique<MIRVocabulary>`. / 声明或调用 `std::make_unique<MIRVocabulary>`。
- **L42**: Returns control, optionally with a value: `return true;`. / 返回控制流，并可附带返回值：`return true;`。
- **L43**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L44**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L45**: Starts the definition of function or method `MIR2VecTool::initializeVocabularyForLayout`. / 开始定义函数或方法 `MIR2VecTool::initializeVocabularyForLayout`。
- **L46**: Starts a loop over a range or sequence: `for (const Function &F : M.getFunctionDefs()) {`. / 开始遍历范围或序列的循环：`for (const Function &F : M.getFunctionDefs()) {`。
- **L47**: Declares or invokes `MMI.getMachineFunction`. / 声明或调用 `MMI.getMachineFunction`。
- **L48**: Introduces a conditional branch: `if (!MF)`. / 引入条件分支：`if (!MF)`。
- **L49**: Executes a standalone statement or declaration: `continue;`. / 执行一条独立语句或声明：`continue;`。
- **L50**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L51**: Declares or invokes `MF->getSubtarget`. / 声明或调用 `MF->getSubtarget`。
- **L52**: Declares or invokes `MF->getSubtarget`. / 声明或调用 `MF->getSubtarget`。
- **L53**: Declares or invokes `MF->getRegInfo`. / 声明或调用 `MF->getRegInfo`。
- **L54**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 55-72

```cpp
    auto VocabOrErr = MIRVocabulary::createDummyVocabForTest(TII, TRI, MRI, 1);
    if (!VocabOrErr) {
      WithColor::error(errs(), ToolName)
          << "Failed to create dummy vocabulary - "
          << toString(VocabOrErr.takeError()) << "\n";
      return false;
    }
    Vocab = std::make_unique<MIRVocabulary>(std::move(*VocabOrErr));
    return true;
  }

  WithColor::error(errs(), ToolName)
      << "No machine functions found to initialize vocabulary\n";
  return false;
}

TripletResult MIR2VecTool::generateTriplets(const MachineFunction &MF) const {
  TripletResult Result;
```

- **L55**: Declares or invokes `MIRVocabulary::createDummyVocabForTest`. / 声明或调用 `MIRVocabulary::createDummyVocabForTest`。
- **L56**: Introduces a conditional branch: `if (!VocabOrErr) {`. / 引入条件分支：`if (!VocabOrErr) {`。
- **L57**: Continues the surrounding expression or declaration: `WithColor::error(errs(), ToolName)`. / 继续构造周围的表达式或声明：`WithColor::error(errs(), ToolName)`。
- **L58**: Continues the surrounding expression or declaration: `<< "Failed to create dummy vocabulary - "`. / 继续构造周围的表达式或声明：`<< "Failed to create dummy vocabulary - "`。
- **L59**: Declares or invokes `toString`. / 声明或调用 `toString`。
- **L60**: Returns control, optionally with a value: `return false;`. / 返回控制流，并可附带返回值：`return false;`。
- **L61**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L62**: Declares or invokes `std::make_unique<MIRVocabulary>`. / 声明或调用 `std::make_unique<MIRVocabulary>`。
- **L63**: Returns control, optionally with a value: `return true;`. / 返回控制流，并可附带返回值：`return true;`。
- **L64**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L65**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L66**: Continues the surrounding expression or declaration: `WithColor::error(errs(), ToolName)`. / 继续构造周围的表达式或声明：`WithColor::error(errs(), ToolName)`。
- **L67**: Executes a standalone statement or declaration: `<< "No machine functions found to initialize vocabulary\n";`. / 执行一条独立语句或声明：`<< "No machine functions found to initialize vocabulary\n";`。
- **L68**: Returns control, optionally with a value: `return false;`. / 返回控制流，并可附带返回值：`return false;`。
- **L69**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L70**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L71**: Starts the definition of function or method `MIR2VecTool::generateTriplets`. / 开始定义函数或方法 `MIR2VecTool::generateTriplets`。
- **L72**: Executes a standalone statement or declaration: `TripletResult Result;`. / 执行一条独立语句或声明：`TripletResult Result;`。

### Lines 73-90

```cpp
  Result.MaxRelation = MIRNextRelation;

  if (!Vocab) {
    WithColor::error(errs(), ToolName)
        << "MIR Vocabulary must be initialized for triplet generation.\n";
    return Result;
  }

  unsigned PrevOpcode = 0;
  bool HasPrevOpcode = false;
  for (const MachineBasicBlock &MBB : MF) {
    for (const MachineInstr &MI : MBB) {
      if (MI.isDebugInstr())
        continue;

      unsigned OpcodeID = Vocab->getEntityIDForOpcode(MI.getOpcode());

      if (HasPrevOpcode) {
```

- **L73**: Initializes or updates `Result.MaxRelation` from the right-hand expression. / 使用右侧表达式初始化或更新 `Result.MaxRelation`。
- **L74**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L75**: Introduces a conditional branch: `if (!Vocab) {`. / 引入条件分支：`if (!Vocab) {`。
- **L76**: Continues the surrounding expression or declaration: `WithColor::error(errs(), ToolName)`. / 继续构造周围的表达式或声明：`WithColor::error(errs(), ToolName)`。
- **L77**: Executes a standalone statement or declaration: `<< "MIR Vocabulary must be initialized for triplet generation.\n";`. / 执行一条独立语句或声明：`<< "MIR Vocabulary must be initialized for triplet generation.\n";`。
- **L78**: Returns control, optionally with a value: `return Result;`. / 返回控制流，并可附带返回值：`return Result;`。
- **L79**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L80**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L81**: Initializes or updates `unsigned PrevOpcode` from the right-hand expression. / 使用右侧表达式初始化或更新 `unsigned PrevOpcode`。
- **L82**: Initializes or updates `bool HasPrevOpcode` from the right-hand expression. / 使用右侧表达式初始化或更新 `bool HasPrevOpcode`。
- **L83**: Starts a loop over a range or sequence: `for (const MachineBasicBlock &MBB : MF) {`. / 开始遍历范围或序列的循环：`for (const MachineBasicBlock &MBB : MF) {`。
- **L84**: Starts a loop over a range or sequence: `for (const MachineInstr &MI : MBB) {`. / 开始遍历范围或序列的循环：`for (const MachineInstr &MI : MBB) {`。
- **L85**: Introduces a conditional branch: `if (MI.isDebugInstr())`. / 引入条件分支：`if (MI.isDebugInstr())`。
- **L86**: Executes a standalone statement or declaration: `continue;`. / 执行一条独立语句或声明：`continue;`。
- **L87**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L88**: Declares or invokes `Vocab->getEntityIDForOpcode`. / 声明或调用 `Vocab->getEntityIDForOpcode`。
- **L89**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L90**: Introduces a conditional branch: `if (HasPrevOpcode) {`. / 引入条件分支：`if (HasPrevOpcode) {`。

### Lines 91-108

```cpp
        Result.Triplets.push_back({PrevOpcode, OpcodeID, MIRNextRelation});
        LLVM_DEBUG(dbgs() << Vocab->getStringKey(PrevOpcode) << '\t'
                          << Vocab->getStringKey(OpcodeID) << '\t' << "Next\n");
      }

      unsigned ArgIndex = 0;
      for (const MachineOperand &MO : MI.operands()) {
        auto OperandID = Vocab->getEntityIDForMachineOperand(MO);
        unsigned RelationID = MIRArgRelation + ArgIndex;
        Result.Triplets.push_back({OpcodeID, OperandID, RelationID});
        LLVM_DEBUG({
          std::string OperandStr = Vocab->getStringKey(OperandID);
          dbgs() << Vocab->getStringKey(OpcodeID) << '\t' << OperandStr << '\t'
                 << "Arg" << ArgIndex << '\n';
        });

        ++ArgIndex;
      }
```

- **L91**: Declares or invokes `Result.Triplets.push_back`. / 声明或调用 `Result.Triplets.push_back`。
- **L92**: Continues the surrounding expression or declaration: `LLVM_DEBUG(dbgs() << Vocab->getStringKey(PrevOpcode) << '\t'`. / 继续构造周围的表达式或声明：`LLVM_DEBUG(dbgs() << Vocab->getStringKey(PrevOpcode) << '\t'`。
- **L93**: Declares or invokes `Vocab->getStringKey`. / 声明或调用 `Vocab->getStringKey`。
- **L94**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L95**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L96**: Initializes or updates `unsigned ArgIndex` from the right-hand expression. / 使用右侧表达式初始化或更新 `unsigned ArgIndex`。
- **L97**: Starts a loop over a range or sequence: `for (const MachineOperand &MO : MI.operands()) {`. / 开始遍历范围或序列的循环：`for (const MachineOperand &MO : MI.operands()) {`。
- **L98**: Declares or invokes `Vocab->getEntityIDForMachineOperand`. / 声明或调用 `Vocab->getEntityIDForMachineOperand`。
- **L99**: Initializes or updates `unsigned RelationID` from the right-hand expression. / 使用右侧表达式初始化或更新 `unsigned RelationID`。
- **L100**: Declares or invokes `Result.Triplets.push_back`. / 声明或调用 `Result.Triplets.push_back`。
- **L101**: Starts the definition of function or method `LLVM_DEBUG`. / 开始定义函数或方法 `LLVM_DEBUG`。
- **L102**: Declares or invokes `Vocab->getStringKey`. / 声明或调用 `Vocab->getStringKey`。
- **L103**: Continues the surrounding expression or declaration: `dbgs() << Vocab->getStringKey(OpcodeID) << '\t' << OperandStr << '\t'`. / 继续构造周围的表达式或声明：`dbgs() << Vocab->getStringKey(OpcodeID) << '\t' << OperandStr << '\t'`。
- **L104**: Executes a standalone statement or declaration: `<< "Arg" << ArgIndex << '\n';`. / 执行一条独立语句或声明：`<< "Arg" << ArgIndex << '\n';`。
- **L105**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L106**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L107**: Executes a standalone statement or declaration: `++ArgIndex;`. / 执行一条独立语句或声明：`++ArgIndex;`。
- **L108**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 109-126

```cpp

      if (ArgIndex > 0)
        Result.MaxRelation =
            std::max(Result.MaxRelation, MIRArgRelation + ArgIndex - 1);

      PrevOpcode = OpcodeID;
      HasPrevOpcode = true;
    }
  }

  return Result;
}

TripletResult MIR2VecTool::generateTriplets(const Module &M) const {
  TripletResult Result;
  Result.MaxRelation = MIRNextRelation;

  for (const Function &F : M.getFunctionDefs()) {
```

- **L109**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L110**: Introduces a conditional branch: `if (ArgIndex > 0)`. / 引入条件分支：`if (ArgIndex > 0)`。
- **L111**: Continues the surrounding expression or declaration: `Result.MaxRelation =`. / 继续构造周围的表达式或声明：`Result.MaxRelation =`。
- **L112**: Declares or invokes `std::max`. / 声明或调用 `std::max`。
- **L113**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L114**: Initializes or updates `PrevOpcode` from the right-hand expression. / 使用右侧表达式初始化或更新 `PrevOpcode`。
- **L115**: Initializes or updates `HasPrevOpcode` from the right-hand expression. / 使用右侧表达式初始化或更新 `HasPrevOpcode`。
- **L116**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L117**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L118**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L119**: Returns control, optionally with a value: `return Result;`. / 返回控制流，并可附带返回值：`return Result;`。
- **L120**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L121**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L122**: Starts the definition of function or method `MIR2VecTool::generateTriplets`. / 开始定义函数或方法 `MIR2VecTool::generateTriplets`。
- **L123**: Executes a standalone statement or declaration: `TripletResult Result;`. / 执行一条独立语句或声明：`TripletResult Result;`。
- **L124**: Initializes or updates `Result.MaxRelation` from the right-hand expression. / 使用右侧表达式初始化或更新 `Result.MaxRelation`。
- **L125**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L126**: Starts a loop over a range or sequence: `for (const Function &F : M.getFunctionDefs()) {`. / 开始遍历范围或序列的循环：`for (const Function &F : M.getFunctionDefs()) {`。

### Lines 127-144

```cpp
    MachineFunction *MF = MMI.getMachineFunction(F);
    if (!MF) {
      WithColor::warning(errs(), ToolName)
          << "No MachineFunction for " << F.getName() << "\n";
      continue;
    }

    TripletResult FuncResult = generateTriplets(*MF);
    Result.MaxRelation = std::max(Result.MaxRelation, FuncResult.MaxRelation);
    Result.Triplets.insert(Result.Triplets.end(), FuncResult.Triplets.begin(),
                           FuncResult.Triplets.end());
  }

  return Result;
}

void MIR2VecTool::writeTripletsToStream(const Module &M,
                                        raw_ostream &OS) const {
```

- **L127**: Declares or invokes `MMI.getMachineFunction`. / 声明或调用 `MMI.getMachineFunction`。
- **L128**: Introduces a conditional branch: `if (!MF) {`. / 引入条件分支：`if (!MF) {`。
- **L129**: Continues the surrounding expression or declaration: `WithColor::warning(errs(), ToolName)`. / 继续构造周围的表达式或声明：`WithColor::warning(errs(), ToolName)`。
- **L130**: Declares or invokes `F.getName`. / 声明或调用 `F.getName`。
- **L131**: Executes a standalone statement or declaration: `continue;`. / 执行一条独立语句或声明：`continue;`。
- **L132**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L133**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L134**: Declares or invokes `generateTriplets`. / 声明或调用 `generateTriplets`。
- **L135**: Declares or invokes `std::max`. / 声明或调用 `std::max`。
- **L136**: Continues a multi-line argument list or initializer: `Result.Triplets.insert(Result.Triplets.end(), FuncResult.Triplets.begin(),`. / 继续一个多行参数列表或初始化器：`Result.Triplets.insert(Result.Triplets.end(), FuncResult.Triplets.begin(),`。
- **L137**: Declares or invokes `FuncResult.Triplets.end`. / 声明或调用 `FuncResult.Triplets.end`。
- **L138**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L139**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L140**: Returns control, optionally with a value: `return Result;`. / 返回控制流，并可附带返回值：`return Result;`。
- **L141**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L142**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L143**: Continues a multi-line argument list or initializer: `void MIR2VecTool::writeTripletsToStream(const Module &M,`. / 继续一个多行参数列表或初始化器：`void MIR2VecTool::writeTripletsToStream(const Module &M,`。
- **L144**: Continues the surrounding expression or declaration: `raw_ostream &OS) const {`. / 继续构造周围的表达式或声明：`raw_ostream &OS) const {`。

### Lines 145-162

```cpp
  auto Result = generateTriplets(M);
  OS << "MAX_RELATION=" << Result.MaxRelation << '\n';
  for (const auto &T : Result.Triplets)
    OS << T.Head << '\t' << T.Tail << '\t' << T.Relation << '\n';
}

EntityList MIR2VecTool::collectEntityMappings() const {
  if (!Vocab) {
    WithColor::error(errs(), ToolName)
        << "Vocabulary must be initialized for entity mappings.\n";
    return {};
  }

  const unsigned EntityCount = Vocab->getCanonicalSize();
  EntityList Result;
  for (unsigned EntityID = 0; EntityID < EntityCount; ++EntityID)
    Result.push_back(Vocab->getStringKey(EntityID));

```

- **L145**: Declares or invokes `generateTriplets`. / 声明或调用 `generateTriplets`。
- **L146**: Initializes or updates `OS << "MAX_RELATION` from the right-hand expression. / 使用右侧表达式初始化或更新 `OS << "MAX_RELATION`。
- **L147**: Starts a loop over a range or sequence: `for (const auto &T : Result.Triplets)`. / 开始遍历范围或序列的循环：`for (const auto &T : Result.Triplets)`。
- **L148**: Executes a standalone statement or declaration: `OS << T.Head << '\t' << T.Tail << '\t' << T.Relation << '\n';`. / 执行一条独立语句或声明：`OS << T.Head << '\t' << T.Tail << '\t' << T.Relation << '\n';`。
- **L149**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L150**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L151**: Starts the definition of function or method `MIR2VecTool::collectEntityMappings`. / 开始定义函数或方法 `MIR2VecTool::collectEntityMappings`。
- **L152**: Introduces a conditional branch: `if (!Vocab) {`. / 引入条件分支：`if (!Vocab) {`。
- **L153**: Continues the surrounding expression or declaration: `WithColor::error(errs(), ToolName)`. / 继续构造周围的表达式或声明：`WithColor::error(errs(), ToolName)`。
- **L154**: Executes a standalone statement or declaration: `<< "Vocabulary must be initialized for entity mappings.\n";`. / 执行一条独立语句或声明：`<< "Vocabulary must be initialized for entity mappings.\n";`。
- **L155**: Returns control, optionally with a value: `return {};`. / 返回控制流，并可附带返回值：`return {};`。
- **L156**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L157**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L158**: Declares or invokes `Vocab->getCanonicalSize`. / 声明或调用 `Vocab->getCanonicalSize`。
- **L159**: Executes a standalone statement or declaration: `EntityList Result;`. / 执行一条独立语句或声明：`EntityList Result;`。
- **L160**: Starts a loop over a range or sequence: `for (unsigned EntityID = 0; EntityID < EntityCount; ++EntityID)`. / 开始遍历范围或序列的循环：`for (unsigned EntityID = 0; EntityID < EntityCount; ++EntityID)`。
- **L161**: Declares or invokes `Result.push_back`. / 声明或调用 `Result.push_back`。
- **L162**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 163-180

```cpp
  return Result;
}

void MIR2VecTool::writeEntitiesToStream(raw_ostream &OS) const {
  auto Entities = collectEntityMappings();
  if (Entities.empty())
    return;

  OS << Entities.size() << "\n";
  for (unsigned EntityID = 0; EntityID < Entities.size(); ++EntityID)
    OS << Entities[EntityID] << '\t' << EntityID << '\n';
}

void MIR2VecTool::writeEmbeddingsToStream(const Module &M, raw_ostream &OS,
                                          EmbeddingLevel Level) const {
  if (!Vocab) {
    WithColor::error(errs(), ToolName) << "Vocabulary not initialized.\n";
    return;
```

- **L163**: Returns control, optionally with a value: `return Result;`. / 返回控制流，并可附带返回值：`return Result;`。
- **L164**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L165**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L166**: Starts the definition of function or method `MIR2VecTool::writeEntitiesToStream`. / 开始定义函数或方法 `MIR2VecTool::writeEntitiesToStream`。
- **L167**: Declares or invokes `collectEntityMappings`. / 声明或调用 `collectEntityMappings`。
- **L168**: Introduces a conditional branch: `if (Entities.empty())`. / 引入条件分支：`if (Entities.empty())`。
- **L169**: Executes a standalone statement or declaration: `return;`. / 执行一条独立语句或声明：`return;`。
- **L170**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L171**: Declares or invokes `Entities.size`. / 声明或调用 `Entities.size`。
- **L172**: Starts a loop over a range or sequence: `for (unsigned EntityID = 0; EntityID < Entities.size(); ++EntityID)`. / 开始遍历范围或序列的循环：`for (unsigned EntityID = 0; EntityID < Entities.size(); ++EntityID)`。
- **L173**: Executes a standalone statement or declaration: `OS << Entities[EntityID] << '\t' << EntityID << '\n';`. / 执行一条独立语句或声明：`OS << Entities[EntityID] << '\t' << EntityID << '\n';`。
- **L174**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L175**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L176**: Continues a multi-line argument list or initializer: `void MIR2VecTool::writeEmbeddingsToStream(const Module &M, raw_ostream &OS,`. / 继续一个多行参数列表或初始化器：`void MIR2VecTool::writeEmbeddingsToStream(const Module &M, raw_ostream &OS,`。
- **L177**: Continues the surrounding expression or declaration: `EmbeddingLevel Level) const {`. / 继续构造周围的表达式或声明：`EmbeddingLevel Level) const {`。
- **L178**: Introduces a conditional branch: `if (!Vocab) {`. / 引入条件分支：`if (!Vocab) {`。
- **L179**: Declares or invokes `WithColor::error`. / 声明或调用 `WithColor::error`。
- **L180**: Executes a standalone statement or declaration: `return;`. / 执行一条独立语句或声明：`return;`。

### Lines 181-198

```cpp
  }

  for (const Function &F : M.getFunctionDefs()) {
    MachineFunction *MF = MMI.getMachineFunction(F);
    if (!MF) {
      WithColor::warning(errs(), ToolName)
          << "No MachineFunction for " << F.getName() << "\n";
      continue;
    }

    writeEmbeddingsToStream(*MF, OS, Level);
  }
}

void MIR2VecTool::writeEmbeddingsToStream(MachineFunction &MF, raw_ostream &OS,
                                          EmbeddingLevel Level) const {
  if (!Vocab) {
    WithColor::error(errs(), ToolName) << "Vocabulary not initialized.\n";
```

- **L181**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L182**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L183**: Starts a loop over a range or sequence: `for (const Function &F : M.getFunctionDefs()) {`. / 开始遍历范围或序列的循环：`for (const Function &F : M.getFunctionDefs()) {`。
- **L184**: Declares or invokes `MMI.getMachineFunction`. / 声明或调用 `MMI.getMachineFunction`。
- **L185**: Introduces a conditional branch: `if (!MF) {`. / 引入条件分支：`if (!MF) {`。
- **L186**: Continues the surrounding expression or declaration: `WithColor::warning(errs(), ToolName)`. / 继续构造周围的表达式或声明：`WithColor::warning(errs(), ToolName)`。
- **L187**: Declares or invokes `F.getName`. / 声明或调用 `F.getName`。
- **L188**: Executes a standalone statement or declaration: `continue;`. / 执行一条独立语句或声明：`continue;`。
- **L189**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L190**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L191**: Declares or invokes `writeEmbeddingsToStream`. / 声明或调用 `writeEmbeddingsToStream`。
- **L192**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L193**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L194**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L195**: Continues a multi-line argument list or initializer: `void MIR2VecTool::writeEmbeddingsToStream(MachineFunction &MF, raw_ostream &OS,`. / 继续一个多行参数列表或初始化器：`void MIR2VecTool::writeEmbeddingsToStream(MachineFunction &MF, raw_ostream &OS,`。
- **L196**: Continues the surrounding expression or declaration: `EmbeddingLevel Level) const {`. / 继续构造周围的表达式或声明：`EmbeddingLevel Level) const {`。
- **L197**: Introduces a conditional branch: `if (!Vocab) {`. / 引入条件分支：`if (!Vocab) {`。
- **L198**: Declares or invokes `WithColor::error`. / 声明或调用 `WithColor::error`。

### Lines 199-216

```cpp
    return;
  }

  auto Emb = MIREmbedder::create(MIR2VecKind::Symbolic, MF, *Vocab);
  if (!Emb) {
    WithColor::error(errs(), ToolName)
        << "Failed to create embedder for " << MF.getName() << "\n";
    return;
  }

  OS << "MIR2Vec embeddings for machine function " << MF.getName() << ":\n";

  switch (Level) {
  case FunctionLevel:
    OS << "Function vector: ";
    Emb->getMFunctionVector().print(OS);
    break;
  case BasicBlockLevel:
```

- **L199**: Executes a standalone statement or declaration: `return;`. / 执行一条独立语句或声明：`return;`。
- **L200**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L201**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L202**: Declares or invokes `MIREmbedder::create`. / 声明或调用 `MIREmbedder::create`。
- **L203**: Introduces a conditional branch: `if (!Emb) {`. / 引入条件分支：`if (!Emb) {`。
- **L204**: Continues the surrounding expression or declaration: `WithColor::error(errs(), ToolName)`. / 继续构造周围的表达式或声明：`WithColor::error(errs(), ToolName)`。
- **L205**: Declares or invokes `MF.getName`. / 声明或调用 `MF.getName`。
- **L206**: Executes a standalone statement or declaration: `return;`. / 执行一条独立语句或声明：`return;`。
- **L207**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L208**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L209**: Declares or invokes `MF.getName`. / 声明或调用 `MF.getName`。
- **L210**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L211**: Starts a multi-way branch based on an expression: `switch (Level) {`. / 开始基于表达式的多路分支：`switch (Level) {`。
- **L212**: Introduces a switch dispatch label: `case FunctionLevel:`. / 引入一个 switch 分发标签：`case FunctionLevel:`。
- **L213**: Executes a standalone statement or declaration: `OS << "Function vector: ";`. / 执行一条独立语句或声明：`OS << "Function vector: ";`。
- **L214**: Declares or invokes `Emb->getMFunctionVector`. / 声明或调用 `Emb->getMFunctionVector`。
- **L215**: Executes a standalone statement or declaration: `break;`. / 执行一条独立语句或声明：`break;`。
- **L216**: Introduces a switch dispatch label: `case BasicBlockLevel:`. / 引入一个 switch 分发标签：`case BasicBlockLevel:`。

### Lines 217-234

```cpp
    OS << "Basic block vectors:\n";
    for (const MachineBasicBlock &MBB : MF) {
      OS << "MBB " << MBB.getName() << ": ";
      Emb->getMBBVector(MBB).print(OS);
    }
    break;
  case InstructionLevel:
    OS << "Instruction vectors:\n";
    for (const MachineBasicBlock &MBB : MF) {
      for (const MachineInstr &MI : MBB) {
        OS << MI << " -> ";
        Emb->getMInstVector(MI).print(OS);
      }
    }
    break;
  }
}

```

- **L217**: Executes a standalone statement or declaration: `OS << "Basic block vectors:\n";`. / 执行一条独立语句或声明：`OS << "Basic block vectors:\n";`。
- **L218**: Starts a loop over a range or sequence: `for (const MachineBasicBlock &MBB : MF) {`. / 开始遍历范围或序列的循环：`for (const MachineBasicBlock &MBB : MF) {`。
- **L219**: Declares or invokes `MBB.getName`. / 声明或调用 `MBB.getName`。
- **L220**: Declares or invokes `Emb->getMBBVector`. / 声明或调用 `Emb->getMBBVector`。
- **L221**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L222**: Executes a standalone statement or declaration: `break;`. / 执行一条独立语句或声明：`break;`。
- **L223**: Introduces a switch dispatch label: `case InstructionLevel:`. / 引入一个 switch 分发标签：`case InstructionLevel:`。
- **L224**: Executes a standalone statement or declaration: `OS << "Instruction vectors:\n";`. / 执行一条独立语句或声明：`OS << "Instruction vectors:\n";`。
- **L225**: Starts a loop over a range or sequence: `for (const MachineBasicBlock &MBB : MF) {`. / 开始遍历范围或序列的循环：`for (const MachineBasicBlock &MBB : MF) {`。
- **L226**: Starts a loop over a range or sequence: `for (const MachineInstr &MI : MBB) {`. / 开始遍历范围或序列的循环：`for (const MachineInstr &MI : MBB) {`。
- **L227**: Executes a standalone statement or declaration: `OS << MI << " -> ";`. / 执行一条独立语句或声明：`OS << MI << " -> ";`。
- **L228**: Declares or invokes `Emb->getMInstVector`. / 声明或调用 `Emb->getMInstVector`。
- **L229**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L230**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L231**: Executes a standalone statement or declaration: `break;`. / 执行一条独立语句或声明：`break;`。
- **L232**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L233**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L234**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 235-236

```cpp
} // namespace mir2vec
} // namespace llvm
```

- **L235**: Closes a namespace scope with a trailing comment: `} // namespace mir2vec`. / 结束一个带尾注释的命名空间作用域：`} // namespace mir2vec`。
- **L236**: Closes a namespace scope with a trailing comment: `} // namespace llvm`. / 结束一个带尾注释的命名空间作用域：`} // namespace llvm`。

## Key Concepts / 关键概念

- **LLVM utility/container usage / LLVM 工具与容器使用**
- **Module/file-scoped coordination / 模块/文件级协调**
- **`MIRUtils` focused implementation / 围绕 `MIRUtils` 的实现逻辑**

## Dependencies / 依赖关系

- **Include / 包含** `MIRUtils.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `llvm/CodeGen/MIR2Vec.h`: Provides code-generation infrastructure. / 提供代码生成基础设施。
- **Include / 包含** `llvm/CodeGen/MIRParser/MIRParser.h`: Provides code-generation infrastructure. / 提供代码生成基础设施。
- **Include / 包含** `llvm/CodeGen/MachineFunction.h`: Provides code-generation infrastructure. / 提供代码生成基础设施。
- **Include / 包含** `llvm/CodeGen/MachineModuleInfo.h`: Provides code-generation infrastructure. / 提供代码生成基础设施。
- **Include / 包含** `llvm/CodeGen/TargetInstrInfo.h`: Provides code-generation infrastructure. / 提供代码生成基础设施。
- **Include / 包含** `llvm/CodeGen/TargetRegisterInfo.h`: Provides code-generation infrastructure. / 提供代码生成基础设施。
- **Include / 包含** `llvm/Support/Debug.h`: Provides LLVM support-library facilities. / 提供LLVM 支持库设施。
- **Include / 包含** `llvm/Support/WithColor.h`: Provides LLVM support-library facilities. / 提供LLVM 支持库设施。
- **Include / 包含** `llvm/Support/raw_ostream.h`: Provides LLVM support-library facilities. / 提供LLVM 支持库设施。
- **Include / 包含** `llvm/Target/TargetMachine.h`: Provides target backend interfaces. / 提供目标后端接口。
