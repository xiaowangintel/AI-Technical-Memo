# JSONExporter.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/Exchange/JSONExporter.cpp` | `polly/lib/Exchange/JSONExporter.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Provides source code related to `JSONExporter` inside the Polly subtree. The leading comment describes it as: Export Scops as JSON. | 提供 Polly 子目录中与 `JSONExporter` 相关的源码。 文件开头注释将其概括为：Export Scops as JSON。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- JSONExporter.cpp  - Export Scops as JSON  -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Export the Scops build by ScopInfo pass as a JSON file.
//
//===----------------------------------------------------------------------===//

#include "polly/JSONExporter.h"
#include "polly/DependenceInfo.h"
#include "polly/Options.h"
#include "polly/ScopInfo.h"
#include "polly/Support/ISLTools.h"
#include "polly/Support/ScopLocation.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/JSON.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/ToolOutputFile.h"
````
- **EN**: This block records the standard LLVM file banner and license metadata; imports Polly, LLVM-family headers needed by the surrounding code; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 引入周边逻辑所需的 Polly、LLVM-family 头文件; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 25-50

````cpp
#include "llvm/Support/raw_ostream.h"
#include "isl/map.h"
#include "isl/set.h"
#include <memory>
#include <string>
#include <system_error>

using namespace llvm;
using namespace polly;

#define DEBUG_TYPE "polly-import-jscop"

static cl::opt<bool>
    PollyPrintImportJscop("polly-print-import-jscop",
                          cl::desc("Polly - Print Scop import result"),
                          cl::cat(PollyCategory));

STATISTIC(NewAccessMapFound, "Number of updated access functions");

namespace {
static cl::opt<std::string>
    ImportDir("polly-import-jscop-dir",
              cl::desc("The directory to import the .jscop files from."),
              cl::Hidden, cl::value_desc("Directory path"), cl::ValueRequired,
              cl::init("."), cl::cat(PollyCategory));

````
- **EN**: This block imports LLVM-family, ISL, system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; opens or organizes declarations inside a C++ namespace; defines macros like `DEBUG_TYPE`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 LLVM-family、ISL、system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 在 C++ 命名空间中组织声明或实现; 定义宏，例如 `DEBUG_TYPE`；并延续周边实现细节。

### Lines 51-77

````cpp
static cl::opt<std::string>
    ImportPostfix("polly-import-jscop-postfix",
                  cl::desc("Postfix to append to the import .jsop files."),
                  cl::Hidden, cl::value_desc("File postfix"), cl::ValueRequired,
                  cl::init(""), cl::cat(PollyCategory));
} // namespace

static std::string getFileName(Scop &S, StringRef Suffix = "") {
  std::string FunctionName = S.getFunction().getName().str();
  std::string FileName = FunctionName + "___" + S.getNameStr() + ".jscop";

  if (Suffix != "")
    FileName += "." + Suffix.str();

  return FileName;
}

/// Export all arrays from the Scop.
///
/// @param S The Scop containing the arrays.
///
/// @returns Json::Value containing the arrays.
static json::Array exportArrays(const Scop &S) {
  json::Array Arrays;
  std::string Buffer;
  llvm::raw_string_ostream RawStringOstream(Buffer);

````
- **EN**: This block declares or defines routines around `ImportPostfix`, `desc`, `value_desc`, `init` (+6 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `ImportPostfix`, `desc`, `value_desc`, `init` (+6 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 78-103

````cpp
  for (auto &SAI : S.arrays()) {
    if (!SAI->isArrayKind())
      continue;

    json::Object Array;
    json::Array Sizes;
    Array["name"] = SAI->getName();
    unsigned i = 0;
    if (!SAI->getDimensionSize(i)) {
      Sizes.push_back("*");
      i++;
    }
    for (; i < SAI->getNumberOfDimensions(); i++) {
      SAI->getDimensionSize(i)->print(RawStringOstream);
      Sizes.push_back(Buffer);
      Buffer.clear();
    }
    Array["sizes"] = std::move(Sizes);
    SAI->getElementType()->print(RawStringOstream);
    Array["type"] = Buffer;
    Buffer.clear();
    Arrays.push_back(std::move(Array));
  }
  return Arrays;
}

````
- **EN**: This block declares or defines routines around `getName`, `push_back`, `getDimensionSize`, `clear` (+2 more); contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `getName`, `push_back`, `getDimensionSize`, `clear` (+2 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 104-123

````cpp
static json::Value getJSON(Scop &S) {
  json::Object root;
  unsigned LineBegin, LineEnd;
  std::string FileName;

  getDebugLocation(&S.getRegion(), LineBegin, LineEnd, FileName);
  std::string Location;
  if (LineBegin != (unsigned)-1)
    Location = FileName + ":" + std::to_string(LineBegin) + "-" +
               std::to_string(LineEnd);

  root["name"] = S.getNameStr();
  root["context"] = S.getContextStr();
  if (LineBegin != (unsigned)-1)
    root["location"] = Location;

  root["arrays"] = exportArrays(S);

  root["statements"];

````
- **EN**: This block declares or defines routines around `getJSON`, `getDebugLocation`, `to_string`, `getNameStr` (+2 more); contains control flow with 2 conditional check(s); touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `getJSON`, `getDebugLocation`, `to_string`, `getNameStr` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 124-145

````cpp
  json::Array Statements;
  for (ScopStmt &Stmt : S) {
    json::Object statement;

    statement["name"] = Stmt.getBaseName();
    statement["domain"] = Stmt.getDomainStr();
    statement["schedule"] = Stmt.getScheduleStr();

    json::Array Accesses;
    for (MemoryAccess *MA : Stmt) {
      json::Object access;

      access["kind"] = MA->isRead() ? "read" : "write";
      access["relation"] = MA->getAccessRelationStr();

      Accesses.push_back(std::move(access));
    }
    statement["accesses"] = std::move(Accesses);

    Statements.push_back(std::move(statement));
  }

````
- **EN**: This block declares or defines routines around `getBaseName`, `getDomainStr`, `getScheduleStr`, `isRead` (+3 more); contains control flow with 2 loop construct(s); touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getBaseName`, `getDomainStr`, `getScheduleStr`, `isRead` (+3 more) 相关的例程; 包含控制流结构：2 处循环; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系；并延续周边实现细节。

### Lines 146-172

````cpp
  root["statements"] = std::move(Statements);
  return json::Value(std::move(root));
}

static void exportScop(Scop &S) {
  std::string FileName = ImportDir + "/" + getFileName(S);

  json::Value jscop = getJSON(S);

  // Write to file.
  std::error_code EC;
  ToolOutputFile F(FileName, EC, llvm::sys::fs::OF_TextWithCRLF);

  std::string FunctionName = S.getFunction().getName().str();
  errs() << "Writing JScop '" << S.getNameStr() << "' in function '"
         << FunctionName << "' to '" << FileName << "'.\n";

  if (!EC) {
    F.os() << formatv("{0:3}", jscop);
    F.os().close();
    if (!F.os().has_error()) {
      errs() << "\n";
      F.keep();
      return;
    }
  }

````
- **EN**: This block declares or defines routines around `move`, `exportScop`, `getFileName`, `getJSON` (+5 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `move`, `exportScop`, `getFileName`, `getJSON` (+5 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 173-193

````cpp
  errs() << "  error opening file for writing!\n";
  F.os().clear_error();
}

typedef Dependences::StatementToIslMapTy StatementToIslMapTy;

/// Import a new context from JScop.
///
/// @param S The scop to update.
/// @param JScop The JScop file describing the new schedule.
///
/// @returns True if the import succeeded, otherwise False.
static bool importContext(Scop &S, const json::Object &JScop) {
  isl::set OldContext = S.getContext();

  // Check if key 'context' is present.
  if (!JScop.get("context")) {
    errs() << "JScop file has no key named 'context'.\n";
    return false;
  }

````
- **EN**: This block declares or defines routines around `errs`, `os`, `importContext`, `getContext`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `errs`, `os`, `importContext`, `getContext` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 194-219

````cpp
  isl::set NewContext =
      isl::set{S.getIslCtx().get(), JScop.getString("context").value().str()};

  // Check whether the context was parsed successfully.
  if (NewContext.is_null()) {
    errs() << "The context was not parsed successfully by ISL.\n";
    return false;
  }

  // Check if the isl_set is a parameter set.
  if (!NewContext.is_params()) {
    errs() << "The isl_set is not a parameter set.\n";
    return false;
  }

  unsigned OldContextDim = unsignedFromIslSize(OldContext.dim(isl::dim::param));
  unsigned NewContextDim = unsignedFromIslSize(NewContext.dim(isl::dim::param));

  // Check if the imported context has the right number of parameters.
  if (OldContextDim != NewContextDim) {
    errs() << "Imported context has the wrong number of parameters : "
           << "Found " << NewContextDim << " Expected " << OldContextDim
           << "\n";
    return false;
  }

````
- **EN**: This block declares or defines routines around `getIslCtx`, `errs`, `unsignedFromIslSize`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getIslCtx`, `errs`, `unsignedFromIslSize` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 220-242

````cpp
  for (unsigned i = 0; i < OldContextDim; i++) {
    isl::id Id = OldContext.get_dim_id(isl::dim::param, i);
    NewContext = NewContext.set_dim_id(isl::dim::param, i, Id);
  }

  S.setContext(NewContext);
  return true;
}

/// Import a new schedule from JScop.
///
/// ... and verify that the new schedule does preserve existing data
/// dependences.
///
/// @param S The scop to update.
/// @param JScop The JScop file describing the new schedule.
/// @param D The data dependences of the @p S.
///
/// @returns True if the import succeeded, otherwise False.
static bool importSchedule(Scop &S, const json::Object &JScop,
                           const Dependences &D) {
  StatementToIslMapTy NewSchedule;

````
- **EN**: This block declares or defines routines around `get_dim_id`, `set_dim_id`, `setContext`, `importSchedule`; contains control flow with 1 loop construct(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `get_dim_id`, `set_dim_id`, `setContext`, `importSchedule` 相关的例程; 包含控制流结构：1 处循环; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 243-270

````cpp
  // Check if key 'statements' is present.
  if (!JScop.get("statements")) {
    errs() << "JScop file has no key name 'statements'.\n";
    return false;
  }

  const json::Array &statements = *JScop.getArray("statements");

  // Check whether the number of indices equals the number of statements
  if (statements.size() != S.getSize()) {
    errs() << "The number of indices and the number of statements differ.\n";
    return false;
  }

  int Index = 0;
  for (ScopStmt &Stmt : S) {
    // Check if key 'schedule' is present.
    if (!statements[Index].getAsObject()->get("schedule")) {
      errs() << "Statement " << Index << " has no 'schedule' key.\n";
      return false;
    }
    std::optional<StringRef> Schedule =
        statements[Index].getAsObject()->getString("schedule");
    assert(Schedule.has_value() &&
           "Schedules that contain extension nodes require special handling.");
    isl_map *Map = isl_map_read_from_str(S.getIslCtx().get(),
                                         Schedule.value().str().c_str());

````
- **EN**: This block declares or defines routines around `errs`, `getArray`, `getAsObject`, `isl_map_read_from_str` (+1 more); contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `errs`, `getArray`, `getAsObject`, `isl_map_read_from_str` (+1 more) 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 271-292

````cpp
    // Check whether the schedule was parsed successfully
    if (!Map) {
      errs() << "The schedule was not parsed successfully (index = " << Index
             << ").\n";
      return false;
    }

    isl_space *Space = Stmt.getDomainSpace().release();

    // Copy the old tuple id. This is necessary to retain the user pointer,
    // that stores the reference to the ScopStmt this schedule belongs to.
    Map = isl_map_set_tuple_id(Map, isl_dim_in,
                               isl_space_get_tuple_id(Space, isl_dim_set));
    for (isl_size i = 0; i < isl_space_dim(Space, isl_dim_param); i++) {
      isl_id *Id = isl_space_get_dim_id(Space, isl_dim_param, i);
      Map = isl_map_set_dim_id(Map, isl_dim_param, i, Id);
    }
    isl_space_free(Space);
    NewSchedule[&Stmt] = isl::manage(Map);
    Index++;
  }

````
- **EN**: This block declares or defines routines around `errs`, `getDomainSpace`, `isl_map_set_tuple_id`, `isl_space_get_tuple_id` (+4 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `errs`, `getDomainSpace`, `isl_map_set_tuple_id`, `isl_space_get_tuple_id` (+4 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 293-312

````cpp
  // Check whether the new schedule is valid or not.
  if (!D.isValidSchedule(S, NewSchedule)) {
    errs() << "JScop file contains a schedule that changes the "
           << "dependences. Use -disable-polly-legality to continue anyways\n";
    return false;
  }

  auto ScheduleMap = isl::union_map::empty(S.getIslCtx());
  for (ScopStmt &Stmt : S) {
    if (NewSchedule.contains(&Stmt))
      ScheduleMap = ScheduleMap.unite(NewSchedule[&Stmt]);
    else
      ScheduleMap = ScheduleMap.unite(Stmt.getSchedule());
  }

  S.setSchedule(ScheduleMap);

  return true;
}

````
- **EN**: This block declares or defines routines around `errs`, `empty`, `unite`, `setSchedule`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `errs`, `empty`, `unite`, `setSchedule` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 313-332

````cpp
/// Import new memory accesses from JScop.
///
/// @param S The scop to update.
/// @param JScop The JScop file describing the new schedule.
/// @param DL The data layout to assume.
/// @param NewAccessStrings optionally record the imported access strings
///
/// @returns True if the import succeeded, otherwise False.
static bool
importAccesses(Scop &S, const json::Object &JScop, const DataLayout &DL,
               std::vector<std::string> *NewAccessStrings = nullptr) {
  int StatementIdx = 0;

  // Check if key 'statements' is present.
  if (!JScop.get("statements")) {
    errs() << "JScop file has no key name 'statements'.\n";
    return false;
  }
  const json::Array &statements = *JScop.getArray("statements");

````
- **EN**: This block declares or defines routines around `importAccesses`, `errs`, `getArray`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `importAccesses`, `errs`, `getArray` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 333-352

````cpp
  // Check whether the number of indices equals the number of statements
  if (statements.size() != S.getSize()) {
    errs() << "The number of indices and the number of statements differ.\n";
    return false;
  }

  for (ScopStmt &Stmt : S) {
    int MemoryAccessIdx = 0;
    const json::Object *Statement = statements[StatementIdx].getAsObject();
    assert(Statement);

    // Check if key 'accesses' is present.
    if (!Statement->get("accesses")) {
      errs()
          << "Statement from JScop file has no key name 'accesses' for index "
          << StatementIdx << ".\n";
      return false;
    }
    const json::Array &JsonAccesses = *Statement->getArray("accesses");

````
- **EN**: This block declares or defines routines around `errs`, `getAsObject`, `getArray`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `errs`, `getAsObject`, `getArray` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 353-376

````cpp
    // Check whether the number of indices equals the number of memory
    // accesses
    if (Stmt.size() != JsonAccesses.size()) {
      errs() << "The number of memory accesses in the JSop file and the number "
                "of memory accesses differ for index "
             << StatementIdx << ".\n";
      return false;
    }

    for (MemoryAccess *MA : Stmt) {
      // Check if key 'relation' is present.
      const json::Object *JsonMemoryAccess =
          JsonAccesses[MemoryAccessIdx].getAsObject();
      assert(JsonMemoryAccess);
      if (!JsonMemoryAccess->get("relation")) {
        errs() << "Memory access number " << MemoryAccessIdx
               << " has no key name 'relation' for statement number "
               << StatementIdx << ".\n";
        return false;
      }
      StringRef Accesses = *JsonMemoryAccess->getString("relation");
      isl_map *NewAccessMap =
          isl_map_read_from_str(S.getIslCtx().get(), Accesses.str().c_str());

````
- **EN**: This block declares or defines routines around `errs`, `getAsObject`, `getString`, `isl_map_read_from_str`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `errs`, `getAsObject`, `getString`, `isl_map_read_from_str` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 377-400

````cpp
      // Check whether the access was parsed successfully
      if (!NewAccessMap) {
        errs() << "The access was not parsed successfully by ISL.\n";
        return false;
      }
      isl_map *CurrentAccessMap = MA->getAccessRelation().release();

      // Check if the number of parameter change
      if (isl_map_dim(NewAccessMap, isl_dim_param) !=
          isl_map_dim(CurrentAccessMap, isl_dim_param)) {
        errs() << "JScop file changes the number of parameter dimensions.\n";
        isl_map_free(CurrentAccessMap);
        isl_map_free(NewAccessMap);
        return false;
      }

      isl_id *NewOutId;

      // If the NewAccessMap has zero dimensions, it is the scalar access; it
      // must be the same as before.
      // If it has at least one dimension, it's an array access; search for
      // its ScopArrayInfo.
      if (isl_map_dim(NewAccessMap, isl_dim_out) >= 1) {
        NewOutId = isl_map_get_tuple_id(NewAccessMap, isl_dim_out);
````
- **EN**: This block declares or defines routines around `errs`, `getAccessRelation`, `isl_map_dim`, `isl_map_free` (+1 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `errs`, `getAccessRelation`, `isl_map_dim`, `isl_map_free` (+1 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 401-424

````cpp
        auto *SAI = S.getArrayInfoByName(isl_id_get_name(NewOutId));
        isl_id *OutId = isl_map_get_tuple_id(CurrentAccessMap, isl_dim_out);
        auto *OutSAI = ScopArrayInfo::getFromId(isl::manage(OutId));
        if (!SAI || SAI->getElementType() != OutSAI->getElementType()) {
          errs() << "JScop file contains access function with undeclared "
                    "ScopArrayInfo\n";
          isl_map_free(CurrentAccessMap);
          isl_map_free(NewAccessMap);
          isl_id_free(NewOutId);
          return false;
        }
        isl_id_free(NewOutId);
        NewOutId = SAI->getBasePtrId().release();
      } else {
        NewOutId = isl_map_get_tuple_id(CurrentAccessMap, isl_dim_out);
      }

      NewAccessMap = isl_map_set_tuple_id(NewAccessMap, isl_dim_out, NewOutId);

      if (MA->isArrayKind()) {
        // We keep the old alignment, thus we cannot allow accesses to memory
        // locations that were not accessed before if the alignment of the
        // access is not the default alignment.
        bool SpecialAlignment = true;
````
- **EN**: This block declares or defines routines around `getArrayInfoByName`, `isl_map_get_tuple_id`, `getFromId`, `errs` (+4 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getArrayInfoByName`, `isl_map_get_tuple_id`, `getFromId`, `errs` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 425-452

````cpp
        if (LoadInst *LoadI = dyn_cast<LoadInst>(MA->getAccessInstruction())) {
          SpecialAlignment =
              DL.getABITypeAlign(LoadI->getType()) != LoadI->getAlign();
        } else if (StoreInst *StoreI =
                       dyn_cast<StoreInst>(MA->getAccessInstruction())) {
          SpecialAlignment =
              DL.getABITypeAlign(StoreI->getValueOperand()->getType()) !=
              StoreI->getAlign();
        }

        if (SpecialAlignment) {
          isl_set *NewAccessSet = isl_map_range(isl_map_copy(NewAccessMap));
          isl_set *CurrentAccessSet =
              isl_map_range(isl_map_copy(CurrentAccessMap));
          bool IsSubset = isl_set_is_subset(NewAccessSet, CurrentAccessSet);
          isl_set_free(NewAccessSet);
          isl_set_free(CurrentAccessSet);

          // Check if the JScop file changes the accessed memory.
          if (!IsSubset) {
            errs() << "JScop file changes the accessed memory\n";
            isl_map_free(CurrentAccessMap);
            isl_map_free(NewAccessMap);
            return false;
          }
        }
      }

````
- **EN**: This block declares or defines routines around `getABITypeAlign`, `dyn_cast<StoreInst>`, `getAlign`, `isl_map_range` (+4 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getABITypeAlign`, `dyn_cast<StoreInst>`, `getAlign`, `isl_map_range` (+4 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 453-479

````cpp
      // We need to copy the isl_ids for the parameter dimensions to the new
      // map. Without doing this the current map would have different
      // ids then the new one, even though both are named identically.
      for (isl_size i = 0; i < isl_map_dim(CurrentAccessMap, isl_dim_param);
           i++) {
        isl_id *Id = isl_map_get_dim_id(CurrentAccessMap, isl_dim_param, i);
        NewAccessMap = isl_map_set_dim_id(NewAccessMap, isl_dim_param, i, Id);
      }

      // Copy the old tuple id. This is necessary to retain the user pointer,
      // that stores the reference to the ScopStmt this access belongs to.
      isl_id *Id = isl_map_get_tuple_id(CurrentAccessMap, isl_dim_in);
      NewAccessMap = isl_map_set_tuple_id(NewAccessMap, isl_dim_in, Id);

      auto NewAccessDomain = isl_map_domain(isl_map_copy(NewAccessMap));
      auto CurrentAccessDomain = isl_map_domain(isl_map_copy(CurrentAccessMap));

      if (!isl_set_has_equal_space(NewAccessDomain, CurrentAccessDomain)) {
        errs() << "JScop file contains access function with incompatible "
               << "dimensions\n";
        isl_map_free(CurrentAccessMap);
        isl_map_free(NewAccessMap);
        isl_set_free(NewAccessDomain);
        isl_set_free(CurrentAccessDomain);
        return false;
      }

````
- **EN**: This block declares or defines routines around `isl_map_get_dim_id`, `isl_map_set_dim_id`, `isl_map_get_tuple_id`, `isl_map_set_tuple_id` (+4 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_map_get_dim_id`, `isl_map_set_dim_id`, `isl_map_get_tuple_id`, `isl_map_set_tuple_id` (+4 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 480-500

````cpp
      NewAccessDomain =
          isl_set_intersect_params(NewAccessDomain, S.getContext().release());
      CurrentAccessDomain = isl_set_intersect_params(CurrentAccessDomain,
                                                     S.getContext().release());
      CurrentAccessDomain =
          isl_set_intersect(CurrentAccessDomain, Stmt.getDomain().release());

      if (MA->isRead() &&
          isl_set_is_subset(CurrentAccessDomain, NewAccessDomain) ==
              isl_bool_false) {
        errs() << "Mapping not defined for all iteration domain elements\n";
        isl_set_free(CurrentAccessDomain);
        isl_set_free(NewAccessDomain);
        isl_map_free(CurrentAccessMap);
        isl_map_free(NewAccessMap);
        return false;
      }

      isl_set_free(CurrentAccessDomain);
      isl_set_free(NewAccessDomain);

````
- **EN**: This block declares or defines routines around `isl_set_intersect_params`, `getContext`, `isl_set_intersect`, `isl_set_is_subset` (+3 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_set_intersect_params`, `getContext`, `isl_set_intersect`, `isl_set_is_subset` (+3 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 501-523

````cpp
      if (!isl_map_is_equal(NewAccessMap, CurrentAccessMap)) {
        // Statistics.
        ++NewAccessMapFound;
        if (NewAccessStrings)
          NewAccessStrings->push_back(Accesses.str());
        MA->setNewAccessRelation(isl::manage(NewAccessMap));
      } else {
        isl_map_free(NewAccessMap);
      }
      isl_map_free(CurrentAccessMap);
      MemoryAccessIdx++;
    }
    StatementIdx++;
  }

  return true;
}

/// Check whether @p SAI and @p Array represent the same array.
static bool areArraysEqual(ScopArrayInfo *SAI, const json::Object &Array) {
  std::string Buffer;
  llvm::raw_string_ostream RawStringOstream(Buffer);

````
- **EN**: This block declares or defines routines around `push_back`, `setNewAccessRelation`, `isl_map_free`, `areArraysEqual` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `push_back`, `setNewAccessRelation`, `isl_map_free`, `areArraysEqual` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 524-544

````cpp
  // Check if key 'type' is present.
  if (!Array.get("type")) {
    errs() << "Array has no key 'type'.\n";
    return false;
  }

  // Check if key 'sizes' is present.
  if (!Array.get("sizes")) {
    errs() << "Array has no key 'sizes'.\n";
    return false;
  }

  // Check if key 'name' is present.
  if (!Array.get("name")) {
    errs() << "Array has no key 'name'.\n";
    return false;
  }

  if (SAI->getName() != *Array.getString("name"))
    return false;

````
- **EN**: This block declares or defines routines around `errs`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `errs` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 545-568

````cpp
  // Check if key 'type' differs from the current one or is not valid.
  SAI->getElementType()->print(RawStringOstream);
  if (Buffer != Array.getString("type").value()) {
    errs() << "Array has not a valid type.\n";
    return false;
  }

  return true;
}

/// Get the accepted primitive type from its textual representation
///        @p TypeTextRepresentation.
///
/// @param TypeTextRepresentation The textual representation of the type.
/// @return The pointer to the primitive type, if this type is accepted
///         or nullptr otherwise.
static Type *parseTextType(const std::string &TypeTextRepresentation,
                           LLVMContext &LLVMContext) {
  std::map<std::string, Type *> MapStrToType = {
      {"void", Type::getVoidTy(LLVMContext)},
      {"half", Type::getHalfTy(LLVMContext)},
      {"float", Type::getFloatTy(LLVMContext)},
      {"double", Type::getDoubleTy(LLVMContext)},
      {"x86_fp80", Type::getX86_FP80Ty(LLVMContext)},
````
- **EN**: This block declares or defines routines around `getElementType`, `errs`, `parseTextType`, `getVoidTy` (+4 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getElementType`, `errs`, `parseTextType`, `getVoidTy` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 569-592

````cpp
      {"fp128", Type::getFP128Ty(LLVMContext)},
      {"ppc_fp128", Type::getPPC_FP128Ty(LLVMContext)},
      {"i1", Type::getInt1Ty(LLVMContext)},
      {"i8", Type::getInt8Ty(LLVMContext)},
      {"i16", Type::getInt16Ty(LLVMContext)},
      {"i32", Type::getInt32Ty(LLVMContext)},
      {"i64", Type::getInt64Ty(LLVMContext)},
      {"i128", Type::getInt128Ty(LLVMContext)}};

  auto It = MapStrToType.find(TypeTextRepresentation);
  if (It != MapStrToType.end())
    return It->second;

  errs() << "Textual representation can not be parsed: "
         << TypeTextRepresentation << "\n";
  return nullptr;
}

/// Import new arrays from JScop.
///
/// @param S The scop to update.
/// @param JScop The JScop file describing new arrays.
///
/// @returns True if the import succeeded, otherwise False.
````
- **EN**: This block declares or defines routines around `getFP128Ty`, `getPPC_FP128Ty`, `getInt1Ty`, `getInt8Ty` (+6 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getFP128Ty`, `getPPC_FP128Ty`, `getInt1Ty`, `getInt8Ty` (+6 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 593-613

````cpp
static bool importArrays(Scop &S, const json::Object &JScop) {
  if (!JScop.get("arrays"))
    return true;
  const json::Array &Arrays = *JScop.getArray("arrays");
  if (Arrays.size() == 0)
    return true;

  unsigned ArrayIdx = 0;
  for (auto &SAI : S.arrays()) {
    if (!SAI->isArrayKind())
      continue;
    if (ArrayIdx + 1 > Arrays.size()) {
      errs() << "Not enough array entries in JScop file.\n";
      return false;
    }
    const json::Object &Array = *Arrays[ArrayIdx].getAsObject();
    if (!areArraysEqual(SAI, Array)) {
      errs() << "No match for array '" << SAI->getName() << "' in JScop.\n";
      return false;
    }

````
- **EN**: This block declares or defines routines around `importArrays`, `getArray`, `errs`, `getAsObject`; contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `importArrays`, `getArray`, `errs`, `getAsObject` 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 614-636

````cpp
    auto *DimSizeType = Type::getInt64Ty(S.getSE()->getContext());
    const json::Array &SizesArray = *Array.getArray("sizes");

    SmallVector<const SCEV *> SCEVSizes;
    for (unsigned i = 0; i < SizesArray.size(); i++) {
      std::string StrSize = SizesArray[i].getAsString()->str();
      if (StrSize == "*") {
        if (i != 0) {
          errs() << "undefined size only allowed for outermost dimension\n";
          return false;
        }
        SCEVSizes.push_back(nullptr);
        continue;
      }

      int Size = std::stoi(StrSize);

      // Check if the size if positive.
      if (Size <= 0) {
        errs() << "The size at index " << i << " is =< 0.\n";
        return false;
      }

````
- **EN**: This block declares or defines routines around `getInt64Ty`, `getArray`, `getAsString`, `errs` (+2 more); contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getInt64Ty`, `getArray`, `getAsString`, `errs` (+2 more) 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 637-663

````cpp
      const SCEV *ScevSize = S.getSE()->getConstant(DimSizeType, Size);
      SCEVSizes.push_back(ScevSize);
    }

    // TODO: If changing the dimensionality of an array, all its accesses must
    //       be updated to that dimensionality. Currently there is no check
    //       whether all accesses are really updated.
    if (!SAI->updateSizes(SCEVSizes, /*CheckConsistency=*/false))
      return false;

    ArrayIdx++;
  }

  for (; ArrayIdx < Arrays.size(); ArrayIdx++) {
    const json::Object &Array = *Arrays[ArrayIdx].getAsObject();
    auto *ElementType =
        parseTextType(Array.get("type")->getAsString().value().str(),
                      S.getSE()->getContext());
    if (!ElementType) {
      errs() << "Error while parsing element type for new array.\n";
      return false;
    }
    const json::Array &SizesArray = *Array.getArray("sizes");
    std::vector<unsigned> DimSizes;
    for (unsigned i = 0; i < SizesArray.size(); i++) {
      auto Size = std::stoi(SizesArray[i].getAsString()->str());

````
- **EN**: This block declares or defines routines around `getSE`, `push_back`, `getAsObject`, `parseTextType` (+3 more); contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getSE`, `push_back`, `getAsObject`, `parseTextType` (+3 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 664-683

````cpp
      // Check if the size if positive.
      if (Size <= 0) {
        errs() << "The size at index " << i << " is =< 0.\n";
        return false;
      }

      DimSizes.push_back(Size);
    }

    auto NewSAI = S.createScopArrayInfo(
        ElementType, Array.getString("name").value().str(), DimSizes);

    if (Array.get("allocation")) {
      NewSAI->setIsOnHeap(Array.getString("allocation").value() == "heap");
    }
  }

  return true;
}

````
- **EN**: This block declares or defines routines around `errs`, `push_back`, `createScopArrayInfo`, `getString` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `errs`, `push_back`, `createScopArrayInfo`, `getString` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 684-703

````cpp
/// Import a Scop from a JSCOP file
/// @param S The scop to be modified
/// @param D Dependence Info
/// @param DL The DataLayout of the function
/// @param NewAccessStrings Optionally record the imported access strings
///
/// @returns true on success, false otherwise. Beware that if this returns
/// false, the Scop may still have been modified. In this case the Scop contains
/// invalid information.
static bool importScop(Scop &S, const Dependences &D, const DataLayout &DL,
                       std::vector<std::string> *NewAccessStrings = nullptr) {
  std::string FileName = ImportDir + "/" + getFileName(S, ImportPostfix);

  std::string FunctionName = S.getFunction().getName().str();
  errs() << "Reading JScop '" << S.getNameStr() << "' in function '"
         << FunctionName << "' from '" << FileName << "'.\n";
  ErrorOr<std::unique_ptr<MemoryBuffer>> result =
      MemoryBuffer::getFile(FileName);
  std::error_code ec = result.getError();

````
- **EN**: This block declares or defines routines around `importScop`, `getFileName`, `getFunction`, `errs` (+2 more); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `importScop`, `getFileName`, `getFunction`, `errs` (+2 more) 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 704-724

````cpp
  if (ec) {
    errs() << "File could not be read: " << ec.message() << "\n";
    return false;
  }

  Expected<json::Value> ParseResult =
      json::parse(result.get().get()->getBuffer());

  if (Error E = ParseResult.takeError()) {
    errs() << "JSCoP file could not be parsed\n";
    errs() << E << "\n";
    consumeError(std::move(E));
    return false;
  }
  json::Object &jscop = *ParseResult.get().getAsObject();

  bool Success = importContext(S, jscop);

  if (!Success)
    return false;

````
- **EN**: This block declares or defines routines around `errs`, `parse`, `consumeError`, `get` (+1 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `errs`, `parse`, `consumeError`, `get` (+1 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 725-748

````cpp
  Success = importSchedule(S, jscop, D);

  if (!Success)
    return false;

  Success = importArrays(S, jscop);

  if (!Success)
    return false;

  Success = importAccesses(S, jscop, DL, NewAccessStrings);

  if (!Success)
    return false;
  return true;
}

void polly::runImportJSON(Scop &S, DependenceAnalysis::Result &DA) {
  const Dependences &D = DA.getDependences(Dependences::AL_Statement);
  const DataLayout &DL = S.getFunction().getParent()->getDataLayout();
  std::vector<std::string> NewAccessStrings;
  if (!importScop(S, D, DL, &NewAccessStrings))
    report_fatal_error("Tried to import a malformed jscop file.");

````
- **EN**: This block declares or defines routines around `importSchedule`, `importArrays`, `importAccesses`, `runImportJSON` (+3 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `importSchedule`, `importArrays`, `importAccesses`, `runImportJSON` (+3 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 749-765

````cpp
  // Dependences must be recomputed for the new access functions
  DA.abandonDependences();

  if (PollyPrintImportJscop) {
    outs()
        << "Printing analysis 'Polly - Print Scop import result' for region: '"
        << S.getRegion().getNameStr() << "' in function '"
        << S.getFunction().getName() << "':\n";
    outs() << S;
    for (std::vector<std::string>::const_iterator I = NewAccessStrings.begin(),
                                                  E = NewAccessStrings.end();
         I != E; I++)
      outs() << "New access function '" << *I << "' detected in JSCOP file\n";
  }
}

void polly::runExportJSON(Scop &S) { exportScop(S); }
````
- **EN**: This block declares or defines routines around `abandonDependences`, `outs`, `getRegion`, `getFunction` (+2 more); contains control flow with 1 loop construct(s), 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `abandonDependences`, `outs`, `getRegion`, `getFunction` (+2 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

## Key Concepts / 关键概念

- **SCoP modeling**
  - **CN**: SCoP 建模
- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Schedule construction**
  - **CN**: 调度构建
- **Memory access tracking**
  - **CN**: 内存访问跟踪
- **Dependence analysis**
  - **CN**: 依赖分析
- **Loop transformation**
  - **CN**: 循环变换

## Dependencies / 依赖关系

- **Polly headers**: `polly/JSONExporter.h`, `polly/DependenceInfo.h`, `polly/Options.h`, `polly/ScopInfo.h`, `polly/Support/ISLTools.h`, `polly/Support/ScopLocation.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/JSONExporter.h`, `polly/DependenceInfo.h`, `polly/Options.h`, `polly/ScopInfo.h`, `polly/Support/ISLTools.h`, `polly/Support/ScopLocation.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/ADT/Statistic.h`, `llvm/IR/Module.h`, `llvm/Support/FileSystem.h`, `llvm/Support/JSON.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/ToolOutputFile.h`, `llvm/Support/raw_ostream.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/ADT/Statistic.h`, `llvm/IR/Module.h`, `llvm/Support/FileSystem.h`, `llvm/Support/JSON.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/ToolOutputFile.h`, `llvm/Support/raw_ostream.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
- **ISL headers**: `isl/map.h`, `isl/set.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/map.h`, `isl/set.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `memory`, `string`, `system_error` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`memory`, `string`, `system_error` —— 实现所需的标准库或系统声明。
