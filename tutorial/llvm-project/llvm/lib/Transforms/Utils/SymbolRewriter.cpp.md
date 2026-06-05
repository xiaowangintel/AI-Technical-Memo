# SymbolRewriter.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/SymbolRewriter.cpp` | `llvm/lib/Transforms/Utils/SymbolRewriter.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements symbol Rewriter within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 SymbolRewriter 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-47

```cpp
//===- SymbolRewriter.cpp - Symbol Rewriter -------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// SymbolRewriter is a LLVM pass which can rewrite symbols transparently within
// existing code.  It is implemented as a compiler pass and is configured via a
// YAML configuration file.
//
// The YAML configuration file format is as follows:
//
// RewriteMapFile := RewriteDescriptors
// RewriteDescriptors := RewriteDescriptor | RewriteDescriptors
// RewriteDescriptor := RewriteDescriptorType ':' '{' RewriteDescriptorFields '}'
// RewriteDescriptorFields := RewriteDescriptorField | RewriteDescriptorFields
// RewriteDescriptorField := FieldIdentifier ':' FieldValue ','
// RewriteDescriptorType := Identifier
// FieldIdentifier := Identifier
// FieldValue := Identifier
// Identifier := [0-9a-zA-Z]+
//
// Currently, the following descriptor types are supported:
//
// - function:          (function rewriting)
//      + Source        (original name of the function)
//      + Target        (explicit transformation)
//      + Transform     (pattern transformation)
//      + Naked         (boolean, whether the function is undecorated)
// - global variable:   (external linkage global variable rewriting)
//      + Source        (original name of externally visible variable)
//      + Target        (explicit transformation)
//      + Transform     (pattern transformation)
// - global alias:      (global alias rewriting)
//      + Source        (original name of the aliased name)
//      + Target        (explicit transformation)
//      + Transform     (pattern transformation)
//
// Note that source and exactly one of [Target, Transform] must be provided
//
// New rewrite descriptors can be created.  Addding a new rewrite descriptor
// involves:
//
//  a) extended the rewrite descriptor kind enumeration
//     (<anonymous>::RewriteDescriptor::RewriteDescriptorType)
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 48-98

```cpp
//  b) implementing the new descriptor
//     (c.f. <anonymous>::ExplicitRewriteFunctionDescriptor)
//  c) extending the rewrite map parser
//     (<anonymous>::RewriteMapParser::parseEntry)
//
//  Specify to rewrite the symbols using the `-rewrite-symbols` option, and
//  specify the map file to use for the rewriting via the `-rewrite-map-file`
//  option.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/SymbolRewriter.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/ilist.h"
#include "llvm/ADT/iterator_range.h"
#include "llvm/IR/Comdat.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalAlias.h"
#include "llvm/IR/GlobalObject.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Value.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/ErrorOr.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Regex.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/YAMLParser.h"
#include <memory>
#include <string>
#include <vector>

using namespace llvm;
using namespace SymbolRewriter;

#define DEBUG_TYPE "symbol-rewriter"

static cl::list<std::string> RewriteMapFiles("rewrite-map-file",
                                             cl::desc("Symbol Rewrite Map"),
                                             cl::value_desc("filename"),
                                             cl::Hidden);

static void rewriteComdat(Module &M, GlobalObject *GO,
                          const std::string &Source,
                          const std::string &Target) {
  if (Comdat *CD = GO->getComdat()) {
    auto &Comdats = M.getComdatSymbolTable();

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 99-147

```cpp
    Comdat *C = M.getOrInsertComdat(Target);
    C->setSelectionKind(CD->getSelectionKind());
    GO->setComdat(C);

    Comdats.erase(Comdats.find(Source));
  }
}

namespace {

template <RewriteDescriptor::Type DT, typename ValueType,
          ValueType *(Module::*Get)(StringRef) const>
class ExplicitRewriteDescriptor : public RewriteDescriptor {
public:
  const std::string Source;
  const std::string Target;

  ExplicitRewriteDescriptor(StringRef S, StringRef T, const bool Naked)
      : RewriteDescriptor(DT),
        Source(std::string(Naked ? StringRef("\01" + S.str()) : S)),
        Target(std::string(T)) {}

  bool performOnModule(Module &M) override;

  static bool classof(const RewriteDescriptor *RD) {
    return RD->getType() == DT;
  }
};

} // end anonymous namespace

template <RewriteDescriptor::Type DT, typename ValueType,
          ValueType *(Module::*Get)(StringRef) const>
bool ExplicitRewriteDescriptor<DT, ValueType, Get>::performOnModule(Module &M) {
  bool Changed = false;
  if (ValueType *S = (M.*Get)(Source)) {
    if (GlobalObject *GO = dyn_cast<GlobalObject>(S))
      rewriteComdat(M, GO, Source, Target);

    if (Value *T = (M.*Get)(Target))
      S->setValueName(T->getValueName());
    else
      S->setName(Target);

    Changed = true;
  }
  return Changed;
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include ExplicitRewriteDescriptor, classof, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 ExplicitRewriteDescriptor, classof，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 148-197

```cpp
namespace {

template <RewriteDescriptor::Type DT, typename ValueType,
          ValueType *(Module::*Get)(StringRef) const,
          iterator_range<typename iplist<ValueType>::iterator>
          (Module::*Iterator)()>
class PatternRewriteDescriptor : public RewriteDescriptor {
public:
  const std::string Pattern;
  const std::string Transform;

  PatternRewriteDescriptor(StringRef P, StringRef T)
      : RewriteDescriptor(DT), Pattern(std::string(P)),
        Transform(std::string(T)) {}

  bool performOnModule(Module &M) override;

  static bool classof(const RewriteDescriptor *RD) {
    return RD->getType() == DT;
  }
};

} // end anonymous namespace

template <RewriteDescriptor::Type DT, typename ValueType,
          ValueType *(Module::*Get)(StringRef) const,
          iterator_range<typename iplist<ValueType>::iterator>
          (Module::*Iterator)()>
bool PatternRewriteDescriptor<DT, ValueType, Get, Iterator>::
performOnModule(Module &M) {
  bool Changed = false;
  for (auto &C : (M.*Iterator)()) {
    std::string Error;

    std::string Name = Regex(Pattern).sub(Transform, C.getName(), &Error);
    if (!Error.empty())
      report_fatal_error(Twine("unable to transforn ") + C.getName() + " in " +
                         M.getModuleIdentifier() + ": " + Error);

    if (C.getName() == Name)
      continue;

    if (GlobalObject *GO = dyn_cast<GlobalObject>(&C))
      rewriteComdat(M, GO, std::string(C.getName()), Name);

    if (Value *V = (M.*Get)(Name))
      C.setValueName(V->getValueName());
    else
      C.setName(Name);

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include PatternRewriteDescriptor, classof, performOnModule, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 PatternRewriteDescriptor, classof, performOnModule，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 198-247

```cpp
    Changed = true;
  }
  return Changed;
}

namespace {

/// Represents a rewrite for an explicitly named (function) symbol.  Both the
/// source function name and target function name of the transformation are
/// explicitly spelt out.
using ExplicitRewriteFunctionDescriptor =
    ExplicitRewriteDescriptor<RewriteDescriptor::Type::Function, Function,
                              &Module::getFunction>;

/// Represents a rewrite for an explicitly named (global variable) symbol.  Both
/// the source variable name and target variable name are spelt out.  This
/// applies only to module level variables.
using ExplicitRewriteGlobalVariableDescriptor =
    ExplicitRewriteDescriptor<RewriteDescriptor::Type::GlobalVariable,
                              GlobalVariable, &Module::getGlobalVariable>;

/// Represents a rewrite for an explicitly named global alias.  Both the source
/// and target name are explicitly spelt out.
using ExplicitRewriteNamedAliasDescriptor =
    ExplicitRewriteDescriptor<RewriteDescriptor::Type::NamedAlias, GlobalAlias,
                              &Module::getNamedAlias>;

/// Represents a rewrite for a regular expression based pattern for functions.
/// A pattern for the function name is provided and a transformation for that
/// pattern to determine the target function name create the rewrite rule.
using PatternRewriteFunctionDescriptor =
    PatternRewriteDescriptor<RewriteDescriptor::Type::Function, Function,
                             &Module::getFunction, &Module::functions>;

/// Represents a rewrite for a global variable based upon a matching pattern.
/// Each global variable matching the provided pattern will be transformed as
/// described in the transformation pattern for the target.  Applies only to
/// module level variables.
using PatternRewriteGlobalVariableDescriptor =
    PatternRewriteDescriptor<RewriteDescriptor::Type::GlobalVariable,
                             GlobalVariable, &Module::getGlobalVariable,
                             &Module::globals>;

/// PatternRewriteNamedAliasDescriptor - represents a rewrite for global
/// aliases which match a given pattern.  The provided transformation will be
/// applied to each of the matching names.
using PatternRewriteNamedAliasDescriptor =
    PatternRewriteDescriptor<RewriteDescriptor::Type::NamedAlias, GlobalAlias,
                             &Module::getNamedAlias, &Module::aliases>;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 248-297

```cpp
} // end anonymous namespace

bool RewriteMapParser::parse(const std::string &MapFile,
                             RewriteDescriptorList *DL) {
  ErrorOr<std::unique_ptr<MemoryBuffer>> Mapping =
      MemoryBuffer::getFile(MapFile);

  if (!Mapping)
    report_fatal_error(Twine("unable to read rewrite map '") + MapFile +
                       "': " + Mapping.getError().message());

  if (!parse(*Mapping, DL))
    report_fatal_error(Twine("unable to parse rewrite map '") + MapFile + "'");

  return true;
}

bool RewriteMapParser::parse(std::unique_ptr<MemoryBuffer> &MapFile,
                             RewriteDescriptorList *DL) {
  SourceMgr SM;
  yaml::Stream YS(MapFile->getBuffer(), SM);

  for (auto &Document : YS) {
    yaml::MappingNode *DescriptorList;

    // ignore empty documents
    if (isa<yaml::NullNode>(Document.getRoot()))
      continue;

    DescriptorList = dyn_cast<yaml::MappingNode>(Document.getRoot());
    if (!DescriptorList) {
      YS.printError(Document.getRoot(), "DescriptorList node must be a map");
      return false;
    }

    for (auto &Descriptor : *DescriptorList)
      if (!parseEntry(YS, Descriptor, DL))
        return false;
  }

  return true;
}

bool RewriteMapParser::parseEntry(yaml::Stream &YS, yaml::KeyValueNode &Entry,
                                  RewriteDescriptorList *DL) {
  yaml::ScalarNode *Key;
  yaml::MappingNode *Value;
  SmallString<32> KeyStorage;
  StringRef RewriteType;

```
- EN: This region continues the SymbolRewriter implementation with local helper logic centered on RewriteMapParser, MapFile, RewriteDescriptorList, ErrorOr.
- CN: 这一段延续了 SymbolRewriter 的主体实现，围绕 RewriteMapParser, MapFile, RewriteDescriptorList, ErrorOr 等局部辅助逻辑展开。

### Lines 298-349

```cpp
  Key = dyn_cast_if_present<yaml::ScalarNode>(Entry.getKey());
  if (!Key) {
    YS.printError(Entry.getKey(), "rewrite type must be a scalar");
    return false;
  }

  Value = dyn_cast_if_present<yaml::MappingNode>(Entry.getValue());
  if (!Value) {
    YS.printError(Entry.getValue(), "rewrite descriptor must be a map");
    return false;
  }

  RewriteType = Key->getValue(KeyStorage);
  if (RewriteType == "function")
    return parseRewriteFunctionDescriptor(YS, Key, Value, DL);
  else if (RewriteType == "global variable")
    return parseRewriteGlobalVariableDescriptor(YS, Key, Value, DL);
  else if (RewriteType == "global alias")
    return parseRewriteGlobalAliasDescriptor(YS, Key, Value, DL);

  YS.printError(Entry.getKey(), "unknown rewrite type");
  return false;
}

bool RewriteMapParser::
parseRewriteFunctionDescriptor(yaml::Stream &YS, yaml::ScalarNode *K,
                               yaml::MappingNode *Descriptor,
                               RewriteDescriptorList *DL) {
  bool Naked = false;
  std::string Source;
  std::string Target;
  std::string Transform;

  for (auto &Field : *Descriptor) {
    yaml::ScalarNode *Key;
    yaml::ScalarNode *Value;
    SmallString<32> KeyStorage;
    SmallString<32> ValueStorage;
    StringRef KeyValue;

    Key = dyn_cast_if_present<yaml::ScalarNode>(Field.getKey());
    if (!Key) {
      YS.printError(Field.getKey(), "descriptor key must be a scalar");
      return false;
    }

    Value = dyn_cast_if_present<yaml::ScalarNode>(Field.getValue());
    if (!Value) {
      YS.printError(Field.getValue(), "descriptor value must be a scalar");
      return false;
    }

```
- EN: This region continues the SymbolRewriter implementation with local helper logic centered on Key, ScalarNode, Entry, Value.
- CN: 这一段延续了 SymbolRewriter 的主体实现，围绕 Key, ScalarNode, Entry, Value 等局部辅助逻辑展开。

### Lines 350-403

```cpp
    KeyValue = Key->getValue(KeyStorage);
    if (KeyValue == "source") {
      Source = std::string(Value->getValue(ValueStorage));
    } else if (KeyValue == "target") {
      Target = std::string(Value->getValue(ValueStorage));
    } else if (KeyValue == "transform") {
      Transform = std::string(Value->getValue(ValueStorage));
    } else if (KeyValue == "naked") {
      std::string Undecorated;

      Undecorated = std::string(Value->getValue(ValueStorage));
      Naked = StringRef(Undecorated).lower() == "true" || Undecorated == "1";
    } else {
      YS.printError(Field.getKey(), "unknown key for function");
      return false;
    }
  }

  if (Transform.empty() == Target.empty()) {
    YS.printError(Descriptor,
                  "exactly one of transform or target must be specified");
    return false;
  }

  // TODO see if there is a more elegant solution to selecting the rewrite
  // descriptor type
  if (!Target.empty()) {
    DL->push_back(std::make_unique<ExplicitRewriteFunctionDescriptor>(
        Source, Target, Naked));
    return true;
  }

  {
    std::string Error;
    if (!Regex(Source).isValid(Error)) {
      YS.printError(Descriptor, "invalid Source regex: " + Error);
      return false;
    }
  }

  DL->push_back(
      std::make_unique<PatternRewriteFunctionDescriptor>(Source, Transform));

  return true;
}

bool RewriteMapParser::
parseRewriteGlobalVariableDescriptor(yaml::Stream &YS, yaml::ScalarNode *K,
                                     yaml::MappingNode *Descriptor,
                                     RewriteDescriptorList *DL) {
  std::string Source;
  std::string Target;
  std::string Transform;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 404-456

```cpp
  for (auto &Field : *Descriptor) {
    yaml::ScalarNode *Key;
    yaml::ScalarNode *Value;
    SmallString<32> KeyStorage;
    SmallString<32> ValueStorage;
    StringRef KeyValue;

    Key = dyn_cast_if_present<yaml::ScalarNode>(Field.getKey());
    if (!Key) {
      YS.printError(Field.getKey(), "descriptor Key must be a scalar");
      return false;
    }

    Value = dyn_cast_if_present<yaml::ScalarNode>(Field.getValue());
    if (!Value) {
      YS.printError(Field.getValue(), "descriptor value must be a scalar");
      return false;
    }

    KeyValue = Key->getValue(KeyStorage);
    if (KeyValue == "source") {
      Source = std::string(Value->getValue(ValueStorage));
    } else if (KeyValue == "target") {
      Target = std::string(Value->getValue(ValueStorage));
    } else if (KeyValue == "transform") {
      Transform = std::string(Value->getValue(ValueStorage));
    } else {
      YS.printError(Field.getKey(), "unknown Key for Global Variable");
      return false;
    }
  }

  if (Transform.empty() == Target.empty()) {
    YS.printError(Descriptor,
                  "exactly one of transform or target must be specified");
    return false;
  }

  if (!Target.empty()) {
    DL->push_back(std::make_unique<ExplicitRewriteGlobalVariableDescriptor>(
        Source, Target,
        /*Naked*/ false));
    return true;
  }

  {
    std::string Error;
    if (!Regex(Source).isValid(Error)) {
      YS.printError(Descriptor, "invalid Source regex: " + Error);
      return false;
    }
  }

```
- EN: This region continues the SymbolRewriter implementation with local helper logic centered on Field, Descriptor, ScalarNode, Key.
- CN: 这一段延续了 SymbolRewriter 的主体实现，围绕 Field, Descriptor, ScalarNode, Key 等局部辅助逻辑展开。

### Lines 457-508

```cpp
  DL->push_back(std::make_unique<PatternRewriteGlobalVariableDescriptor>(
      Source, Transform));

  return true;
}

bool RewriteMapParser::
parseRewriteGlobalAliasDescriptor(yaml::Stream &YS, yaml::ScalarNode *K,
                                  yaml::MappingNode *Descriptor,
                                  RewriteDescriptorList *DL) {
  std::string Source;
  std::string Target;
  std::string Transform;

  for (auto &Field : *Descriptor) {
    yaml::ScalarNode *Key;
    yaml::ScalarNode *Value;
    SmallString<32> KeyStorage;
    SmallString<32> ValueStorage;
    StringRef KeyValue;

    Key = dyn_cast_if_present<yaml::ScalarNode>(Field.getKey());
    if (!Key) {
      YS.printError(Field.getKey(), "descriptor key must be a scalar");
      return false;
    }

    Value = dyn_cast_if_present<yaml::ScalarNode>(Field.getValue());
    if (!Value) {
      YS.printError(Field.getValue(), "descriptor value must be a scalar");
      return false;
    }

    KeyValue = Key->getValue(KeyStorage);
    if (KeyValue == "source") {
      Source = std::string(Value->getValue(ValueStorage));
    } else if (KeyValue == "target") {
      Target = std::string(Value->getValue(ValueStorage));
    } else if (KeyValue == "transform") {
      Transform = std::string(Value->getValue(ValueStorage));
    } else {
      YS.printError(Field.getKey(), "unknown key for Global Alias");
      return false;
    }
  }

  if (Transform.empty() == Target.empty()) {
    YS.printError(Descriptor,
                  "exactly one of transform or target must be specified");
    return false;
  }

```
- EN: This region continues the SymbolRewriter implementation with local helper logic centered on PatternRewriteGlobalVariableDescriptor, Source, Transform, RewriteMapParser.
- CN: 这一段延续了 SymbolRewriter 的主体实现，围绕 PatternRewriteGlobalVariableDescriptor, Source, Transform, RewriteMapParser 等局部辅助逻辑展开。

### Lines 509-553

```cpp
  if (!Target.empty()) {
    DL->push_back(std::make_unique<ExplicitRewriteNamedAliasDescriptor>(
        Source, Target,
        /*Naked*/ false));
    return true;
  }

  {
    std::string Error;
    if (!Regex(Source).isValid(Error)) {
      YS.printError(Descriptor, "invalid Source regex: " + Error);
      return false;
    }
  }

  DL->push_back(
      std::make_unique<PatternRewriteNamedAliasDescriptor>(Source, Transform));

  return true;
}

PreservedAnalyses RewriteSymbolPass::run(Module &M, ModuleAnalysisManager &AM) {
  if (!runImpl(M))
    return PreservedAnalyses::all();

  return PreservedAnalyses::none();
}

bool RewriteSymbolPass::runImpl(Module &M) {
  bool Changed;

  Changed = false;
  for (auto &Descriptor : Descriptors)
    Changed |= Descriptor->performOnModule(M);

  return Changed;
}

void RewriteSymbolPass::loadAndParseMapFiles() {
  const std::vector<std::string> MapFiles(RewriteMapFiles);
  SymbolRewriter::RewriteMapParser Parser;

  for (const auto &MapFile : MapFiles)
    Parser.parse(MapFile, &Descriptors);
}
```
- EN: Core entities appearing here include run, runImpl, loadAndParseMapFiles, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 run, runImpl, loadAndParseMapFiles，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `ExplicitRewriteDescriptor, PatternRewriteDescriptor, classof, performOnModule, run, runImpl, loadAndParseMapFiles` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`ExplicitRewriteDescriptor, PatternRewriteDescriptor, classof, performOnModule, run, runImpl, loadAndParseMapFiles` 构成该文件对外 API 与主要实现挂钩。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Comdat.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalAlias.h`, `llvm/IR/GlobalObject.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Module.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/SymbolRewriter.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Comdat.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalAlias.h`, `llvm/IR/GlobalObject.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Module.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/SymbolRewriter.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/ErrorOr.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/Regex.h`, `llvm/Support/SourceMgr.h`, `llvm/Support/YAMLParser.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/ErrorOr.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/Regex.h`, `llvm/Support/SourceMgr.h`, `llvm/Support/YAMLParser.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `memory`, `string`, `vector` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`memory`, `string`, `vector` 为 LLVM API 之外的 C++ 语言工具提供支持。
