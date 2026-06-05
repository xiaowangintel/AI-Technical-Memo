# RegistryManager.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Query/Matcher/RegistryManager.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Registry map populated at static initialization time.
  - **CN**: 实现 MLIR 查询能力、匹配器基础设施或命令行查询行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- RegistryManager.cpp - Matcher registry -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-12
```cpp
//
// Registry map populated at static initialization time.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 13-18
```cpp
#include "RegistryManager.h"
#include "mlir/Query/Matcher/Registry.h"

#include <set>
#include <utility>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `RegistryManager.h`, `mlir/Query/Matcher/Registry.h`, `set`, `utility`.
- **CN**: 引入该编译单元所需的声明，其中包括 `RegistryManager.h`, `mlir/Query/Matcher/Registry.h`, `set`, `utility`。

### Lines 19-28
```cpp
namespace mlir::query::matcher {
namespace {

// Enum to string for autocomplete.
static std::string asArgString(ArgKind kind) {
  switch (kind) {
  case ArgKind::Boolean:
    return "Boolean";
  case ArgKind::Matcher:
    return "Matcher";
```
- **EN**: Introduces declarations for `mlir::query::matcher`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `mlir::query::matcher` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 29-36
```cpp
  case ArgKind::Signed:
    return "Signed";
  case ArgKind::String:
    return "String";
  }
  llvm_unreachable("Unhandled ArgKind");
}

```
- **EN**: Implements logic around `llvm_unreachable`.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑。

### Lines 37-45
```cpp
} // namespace

void Registry::registerMatcherDescriptor(
    llvm::StringRef matcherName,
    std::unique_ptr<internal::MatcherDescriptor> callback) {
  assert(!constructorMap.contains(matcherName));
  constructorMap[matcherName] = std::move(callback);
}

```
- **EN**: Implements logic around `registerMatcherDescriptor`, `assert`, `move`.
- **CN**: 围绕 `registerMatcherDescriptor`、`assert`、`move` 实现具体逻辑。

### Lines 46-54
```cpp
std::optional<MatcherCtor>
RegistryManager::lookupMatcherCtor(llvm::StringRef matcherName,
                                   const Registry &matcherRegistry) {
  auto it = matcherRegistry.constructors().find(matcherName);
  return it == matcherRegistry.constructors().end()
             ? std::optional<MatcherCtor>()
             : it->second.get();
}

```
- **EN**: Implements logic around `lookupMatcherCtor`, `constructors`, `optional`, `get`.
- **CN**: 围绕 `lookupMatcherCtor`、`constructors`、`optional`、`get` 实现具体逻辑。

### Lines 55-61
```cpp
std::vector<ArgKind> RegistryManager::getAcceptedCompletionTypes(
    llvm::ArrayRef<std::pair<MatcherCtor, unsigned>> context) {
  // Starting with the above seed of acceptable top-level matcher types, compute
  // the acceptable type set for the argument indicated by each context element.
  std::set<ArgKind> typeSet;
  typeSet.insert(ArgKind::Matcher);

```
- **EN**: Implements logic around `getAcceptedCompletionTypes`, `insert`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getAcceptedCompletionTypes`、`insert` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 62-66
```cpp
  for (const auto &ctxEntry : context) {
    MatcherCtor ctor = ctxEntry.first;
    unsigned argNumber = ctxEntry.second;
    std::vector<ArgKind> nextTypeSet;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 67-72
```cpp
    if (ctor->isVariadic() || argNumber < ctor->getNumArgs())
      ctor->getArgKinds(argNumber, nextTypeSet);

    typeSet.insert(nextTypeSet.begin(), nextTypeSet.end());
  }

```
- **EN**: Implements logic around `isVariadic`, `getArgKinds`, `insert`.
- **CN**: 围绕 `isVariadic`、`getArgKinds`、`insert` 实现具体逻辑。

### Lines 73-80
```cpp
  return std::vector<ArgKind>(typeSet.begin(), typeSet.end());
}

std::vector<MatcherCompletion>
RegistryManager::getMatcherCompletions(llvm::ArrayRef<ArgKind> acceptedTypes,
                                       const Registry &matcherRegistry) {
  std::vector<MatcherCompletion> completions;

```
- **EN**: Implements logic around `vector`, `getMatcherCompletions`.
- **CN**: 围绕 `vector`、`getMatcherCompletions` 实现具体逻辑。

### Lines 81-85
```cpp
  // Search the registry for acceptable matchers.
  for (const auto &m : matcherRegistry.constructors()) {
    const internal::MatcherDescriptor &matcher = *m.getValue();
    llvm::StringRef name = m.getKey();

```
- **EN**: Implements logic around `constructors`, `getValue`, `getKey`.
- **CN**: 围绕 `constructors`、`getValue`、`getKey` 实现具体逻辑。

### Lines 86-92
```cpp
    unsigned numArgs = matcher.isVariadic() ? 1 : matcher.getNumArgs();
    std::vector<std::vector<ArgKind>> argKinds(numArgs);

    for (const ArgKind &kind : acceptedTypes) {
      if (kind != ArgKind::Matcher)
        continue;

```
- **EN**: Implements logic around `isVariadic`, `argKinds`.
- **CN**: 围绕 `isVariadic`、`argKinds` 实现具体逻辑。

### Lines 93-99
```cpp
      for (unsigned arg = 0; arg != numArgs; ++arg)
        matcher.getArgKinds(arg, argKinds[arg]);
    }

    std::string decl;
    llvm::raw_string_ostream os(decl);

```
- **EN**: Implements logic around `getArgKinds`, `os`.
- **CN**: 围绕 `getArgKinds`、`os` 实现具体逻辑。

### Lines 100-106
```cpp
    std::string typedText = std::string(name);
    os << "Matcher: " << name << "(";

    for (const std::vector<ArgKind> &arg : argKinds) {
      if (&arg != &argKinds[0])
        os << ", ";

```
- **EN**: Implements logic around `string`.
- **CN**: 围绕 `string` 实现具体逻辑。

### Lines 107-112
```cpp
      bool firstArgKind = true;
      // Two steps. First all non-matchers, then matchers only.
      for (const ArgKind &argKind : arg) {
        if (!firstArgKind)
          os << "|";

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 113-117
```cpp
        firstArgKind = false;
        os << asArgString(argKind);
      }
    }

```
- **EN**: Implements logic around `asArgString`.
- **CN**: 围绕 `asArgString` 实现具体逻辑。

### Lines 118-123
```cpp
    if (matcher.isVariadic())
      os << ",...";

    os << ")";
    typedText += "(";

```
- **EN**: Implements logic around `isVariadic`.
- **CN**: 围绕 `isVariadic` 实现具体逻辑。

### Lines 124-128
```cpp
    if (argKinds.empty())
      typedText += ")";
    else if (argKinds[0][0] == ArgKind::String)
      typedText += "\"";

```
- **EN**: Implements logic around `empty`.
- **CN**: 围绕 `empty` 实现具体逻辑。

### Lines 129-134
```cpp
    completions.emplace_back(typedText, decl);
  }

  return completions;
}

```
- **EN**: Implements logic around `emplace_back`.
- **CN**: 围绕 `emplace_back` 实现具体逻辑。

### Lines 135-142
```cpp
VariantMatcher RegistryManager::constructMatcher(
    MatcherCtor ctor, internal::SourceRange nameRange,
    llvm::StringRef functionName, llvm::ArrayRef<ParserValue> args,
    internal::Diagnostics *error) {
  VariantMatcher out = ctor->create(nameRange, args, error);
  if (functionName.empty() || out.isNull())
    return out;

```
- **EN**: Implements logic around `constructMatcher`, `create`, `empty`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `constructMatcher`、`create`、`empty` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 143-147
```cpp
  if (std::optional<DynMatcher> result = out.getDynMatcher()) {
    result->setFunctionName(functionName);
    return VariantMatcher::SingleMatcher(*result);
  }

```
- **EN**: Implements logic around `getDynMatcher`, `setFunctionName`, `SingleMatcher`.
- **CN**: 围绕 `getDynMatcher`、`setFunctionName`、`SingleMatcher` 实现具体逻辑。

### Lines 148-152
```cpp
  error->addError(nameRange, internal::ErrorType::RegistryNotBindable);
  return {};
}

} // namespace mlir::query::matcher
```
- **EN**: Implements logic around `addError`.
- **CN**: 围绕 `addError` 实现具体逻辑。

## Key Concepts / 关键概念

- **IR querying / IR 查询**:
  - **EN**: Supports querying or filtering IR according to structural or semantic predicates.
  - **CN**: 支持依据结构或语义谓词对 IR 进行查询或过滤。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `RegistryManager.h`, `mlir/Query/Matcher/Registry.h`
- **Standard-library headers / 标准库头文件**: `<set>`, `<utility>`
- **Subsystem categories / 子系统类别**: query infrastructure / 查询基础设施 (1)
