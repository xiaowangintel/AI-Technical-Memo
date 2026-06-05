# Context.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/PDLL/ODS/Context.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Context.cpp --------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-15
```cpp

#include "mlir/Tools/PDLL/ODS/Context.h"
#include "mlir/Tools/PDLL/ODS/Constraint.h"
#include "mlir/Tools/PDLL/ODS/Dialect.h"
#include "mlir/Tools/PDLL/ODS/Operation.h"
#include "llvm/Support/ScopedPrinter.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/PDLL/ODS/Context.h`, `mlir/Tools/PDLL/ODS/Constraint.h`, `mlir/Tools/PDLL/ODS/Dialect.h`, `mlir/Tools/PDLL/ODS/Operation.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/PDLL/ODS/Context.h`, `mlir/Tools/PDLL/ODS/Constraint.h`, `mlir/Tools/PDLL/ODS/Dialect.h`, `mlir/Tools/PDLL/ODS/Operation.h`。

### Lines 16-22
```cpp
using namespace mlir;
using namespace mlir::pdll::ods;

//===----------------------------------------------------------------------===//
// Context
//===----------------------------------------------------------------------===//

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 23-36
```cpp
Context::Context() = default;
Context::~Context() = default;

const AttributeConstraint &
Context::insertAttributeConstraint(StringRef name, StringRef summary,
                                   StringRef cppClass) {
  std::unique_ptr<AttributeConstraint> &constraint = attributeConstraints[name];
  if (!constraint) {
    constraint.reset(new AttributeConstraint(name, summary, cppClass));
  } else {
    assert(constraint->getCppClass() == cppClass &&
           constraint->getSummary() == summary &&
           "constraint with the same name was already registered with a "
           "different class");
```
- **EN**: Implements logic around `Context`, `~Context`, `insertAttributeConstraint`, `reset`, and 2 more symbols.
- **CN**: 围绕 `Context`、`~Context`、`insertAttributeConstraint`、`reset` 等另外 2 个符号 实现具体逻辑。

### Lines 37-49
```cpp
  }
  return *constraint;
}

const TypeConstraint &Context::insertTypeConstraint(StringRef name,
                                                    StringRef summary,
                                                    StringRef cppClass) {
  std::unique_ptr<TypeConstraint> &constraint = typeConstraints[name];
  if (!constraint)
    constraint.reset(new TypeConstraint(name, summary, cppClass));
  return *constraint;
}

```
- **EN**: Implements logic around `insertTypeConstraint`, `reset`.
- **CN**: 围绕 `insertTypeConstraint`、`reset` 实现具体逻辑。

### Lines 50-56
```cpp
Dialect &Context::insertDialect(StringRef name) {
  std::unique_ptr<Dialect> &dialect = dialects[name];
  if (!dialect)
    dialect.reset(new Dialect(name));
  return *dialect;
}

```
- **EN**: Implements logic around `insertDialect`, `reset`.
- **CN**: 围绕 `insertDialect`、`reset` 实现具体逻辑。

### Lines 57-70
```cpp
const Dialect *Context::lookupDialect(StringRef name) const {
  auto it = dialects.find(name);
  return it == dialects.end() ? nullptr : &*it->second;
}

std::pair<Operation *, bool>
Context::insertOperation(StringRef name, StringRef summary, StringRef desc,
                         StringRef nativeClassName,
                         bool supportsResultTypeInferrence, SMLoc loc) {
  std::pair<StringRef, StringRef> dialectAndName = name.split('.');
  return insertDialect(dialectAndName.first)
      .insertOperation(name, summary, desc, nativeClassName,
                       supportsResultTypeInferrence, loc);
}
```
- **EN**: Implements logic around `lookupDialect`, `find`, `end`, `insertOperation`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `lookupDialect`、`find`、`end`、`insertOperation` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 71-78
```cpp

const Operation *Context::lookupOperation(StringRef name) const {
  std::pair<StringRef, StringRef> dialectAndName = name.split('.');
  if (const Dialect *dialect = lookupDialect(dialectAndName.first))
    return dialect->lookupOperation(name);
  return nullptr;
}

```
- **EN**: Implements logic around `lookupOperation`, `split`, `lookupDialect`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `lookupOperation`、`split`、`lookupDialect` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 79-90
```cpp
template <typename T>
static SmallVector<T *>
sortMapByName(const llvm::StringMap<std::unique_ptr<T>> &map) {
  SmallVector<T *> storage;
  for (auto &entry : map)
    storage.push_back(entry.second.get());
  llvm::sort(storage, [](const auto &lhs, const auto &rhs) {
    return lhs->getName() < rhs->getName();
  });
  return storage;
}

```
- **EN**: Implements logic around `sortMapByName`, `push_back`, `sort`, `getName`.
- **CN**: 围绕 `sortMapByName`、`push_back`、`sort`、`getName` 实现具体逻辑。

### Lines 91-104
```cpp
void Context::print(raw_ostream &os) const {
  auto printVariableLengthCst = [&](StringRef cst, VariableLengthKind kind) {
    switch (kind) {
    case VariableLengthKind::Optional:
      os << "Optional<" << cst << ">";
      break;
    case VariableLengthKind::Single:
      os << cst;
      break;
    case VariableLengthKind::Variadic:
      os << "Variadic<" << cst << ">";
      break;
    }
  };
```
- **EN**: Implements logic around `print`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `print` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 105-111
```cpp

  llvm::ScopedPrinter printer(os);
  llvm::DictScope odsScope(printer, "ODSContext");
  for (const Dialect *dialect : sortMapByName(dialects)) {
    printer.startLine() << "Dialect `" << dialect->getName() << "` {\n";
    printer.indent();

```
- **EN**: Implements logic around `printer`, `odsScope`, `sortMapByName`, `startLine`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printer`、`odsScope`、`sortMapByName`、`startLine` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 112-122
```cpp
    for (const Operation *op : sortMapByName(dialect->getOperations())) {
      printer.startLine() << "Operation `" << op->getName() << "` {\n";
      printer.indent();

      // Attributes.
      ArrayRef<Attribute> attributes = op->getAttributes();
      if (!attributes.empty()) {
        printer.startLine() << "Attributes { ";
        llvm::interleaveComma(attributes, os, [&](const Attribute &attr) {
          os << attr.getName() << " : ";

```
- **EN**: Implements logic around `sortMapByName`, `startLine`, `indent`, `getAttributes`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `sortMapByName`、`startLine`、`indent`、`getAttributes` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 123-129
```cpp
          auto kind = attr.isOptional() ? VariableLengthKind::Optional
                                        : VariableLengthKind::Single;
          printVariableLengthCst(attr.getConstraint().getDemangledName(), kind);
        });
        os << " }\n";
      }

```
- **EN**: Implements logic around `isOptional`, `printVariableLengthCst`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `isOptional`、`printVariableLengthCst` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 130-142
```cpp
      // Operands.
      ArrayRef<OperandOrResult> operands = op->getOperands();
      if (!operands.empty()) {
        printer.startLine() << "Operands { ";
        llvm::interleaveComma(
            operands, os, [&](const OperandOrResult &operand) {
              os << operand.getName() << " : ";
              printVariableLengthCst(operand.getConstraint().getDemangledName(),
                                     operand.getVariableLengthKind());
            });
        os << " }\n";
      }

```
- **EN**: Implements logic around `getOperands`, `empty`, `startLine`, `interleaveComma`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getOperands`、`empty`、`startLine`、`interleaveComma` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 143-154
```cpp
      // Results.
      ArrayRef<OperandOrResult> results = op->getResults();
      if (!results.empty()) {
        printer.startLine() << "Results { ";
        llvm::interleaveComma(results, os, [&](const OperandOrResult &result) {
          os << result.getName() << " : ";
          printVariableLengthCst(result.getConstraint().getDemangledName(),
                                 result.getVariableLengthKind());
        });
        os << " }\n";
      }

```
- **EN**: Implements logic around `getResults`, `empty`, `startLine`, `interleaveComma`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getResults`、`empty`、`startLine`、`interleaveComma` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 155-163
```cpp
      printer.objectEnd();
    }
    printer.objectEnd();
  }
  for (const AttributeConstraint *cst : sortMapByName(attributeConstraints)) {
    printer.startLine() << "AttributeConstraint `" << cst->getDemangledName()
                        << "` {\n";
    printer.indent();

```
- **EN**: Implements logic around `objectEnd`, `sortMapByName`, `startLine`, `indent`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `objectEnd`、`sortMapByName`、`startLine`、`indent` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 164-172
```cpp
    printer.startLine() << "Summary: " << cst->getSummary() << "\n";
    printer.startLine() << "CppClass: " << cst->getCppClass() << "\n";
    printer.objectEnd();
  }
  for (const TypeConstraint *cst : sortMapByName(typeConstraints)) {
    printer.startLine() << "TypeConstraint `" << cst->getDemangledName()
                        << "` {\n";
    printer.indent();

```
- **EN**: Implements logic around `startLine`, `objectEnd`, `sortMapByName`, `indent`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `startLine`、`objectEnd`、`sortMapByName`、`indent` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 173-178
```cpp
    printer.startLine() << "Summary: " << cst->getSummary() << "\n";
    printer.startLine() << "CppClass: " << cst->getCppClass() << "\n";
    printer.objectEnd();
  }
  printer.objectEnd();
}
```
- **EN**: Implements logic around `startLine`, `objectEnd`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `startLine`、`objectEnd` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/PDLL/ODS/Context.h`, `mlir/Tools/PDLL/ODS/Constraint.h`, `mlir/Tools/PDLL/ODS/Dialect.h`, `mlir/Tools/PDLL/ODS/Operation.h`, `llvm/Support/ScopedPrinter.h`, `llvm/Support/raw_ostream.h`
- **Subsystem categories / 子系统类别**: tooling support declarations / 工具支持声明 (4), LLVM support-library helpers / LLVM Support 库辅助工具 (2)
