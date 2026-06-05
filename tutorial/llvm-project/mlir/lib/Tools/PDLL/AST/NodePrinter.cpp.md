# NodePrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/PDLL/AST/NodePrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- NodePrinter.cpp ----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Tools/PDLL/AST/Nodes.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/SaveAndRestore.h"
#include "llvm/Support/ScopedPrinter.h"
#include <optional>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/PDLL/AST/Nodes.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/TypeSwitch.h`, `llvm/Support/SaveAndRestore.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/PDLL/AST/Nodes.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/TypeSwitch.h`, `llvm/Support/SaveAndRestore.h`。

### Lines 16-25
```cpp
using namespace mlir;
using namespace mlir::pdll::ast;

//===----------------------------------------------------------------------===//
// NodePrinter
//===----------------------------------------------------------------------===//

namespace {
class NodePrinter {
public:
```
- **EN**: Introduces declarations for `NodePrinter`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `NodePrinter` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 26-35
```cpp
  NodePrinter(raw_ostream &os) : os(os) {}

  /// Print the given type to the stream.
  void print(Type type);

  /// Print the given node to the stream.
  void print(const Node *node);

private:
  /// Print a range containing children of a node.
```
- **EN**: Implements logic around `NodePrinter`, `print`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `NodePrinter`、`print` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 36-47
```cpp
  template <typename RangeT,
            std::enable_if_t<!std::is_convertible<RangeT, const Node *>::value>
                * = nullptr>
  void printChildren(RangeT &&range) {
    if (range.empty())
      return;

    // Print the first N-1 elements with a prefix of "|-".
    auto it = std::begin(range);
    for (unsigned i = 0, e = llvm::size(range) - 1; i < e; ++i, ++it)
      print(*it);

```
- **EN**: Implements logic around `printChildren`, `empty`, `begin`, `size`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printChildren`、`empty`、`begin`、`size` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 48-58
```cpp
    // Print the last element.
    elementIndentStack.back() = true;
    print(*it);
  }
  template <typename RangeT, typename... OthersT,
            std::enable_if_t<std::is_convertible<RangeT, const Node *>::value>
                * = nullptr>
  void printChildren(RangeT &&range, OthersT &&...others) {
    printChildren(ArrayRef<const Node *>({range, others...}));
  }
  /// Print a range containing children of a node, nesting the children under
```
- **EN**: Implements logic around `back`, `print`, `printChildren`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `back`、`print`、`printChildren` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 59-73
```cpp
  /// the given label.
  template <typename RangeT>
  void printChildren(StringRef label, RangeT &&range) {
    if (range.empty())
      return;
    elementIndentStack.reserve(elementIndentStack.size() + 1);
    llvm::SaveAndRestore lastElement(elementIndentStack.back(), true);

    printIndent();
    os << label << "`\n";
    elementIndentStack.push_back(/*isLastElt*/ false);
    printChildren(std::forward<RangeT>(range));
    elementIndentStack.pop_back();
  }

```
- **EN**: Implements logic around `printChildren`, `empty`, `reserve`, `lastElement`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printChildren`、`empty`、`reserve`、`lastElement` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 74-90
```cpp
  /// Print the given derived node to the stream.
  void printImpl(const CompoundStmt *stmt);
  void printImpl(const EraseStmt *stmt);
  void printImpl(const LetStmt *stmt);
  void printImpl(const ReplaceStmt *stmt);
  void printImpl(const ReturnStmt *stmt);
  void printImpl(const RewriteStmt *stmt);

  void printImpl(const AttributeExpr *expr);
  void printImpl(const CallExpr *expr);
  void printImpl(const DeclRefExpr *expr);
  void printImpl(const MemberAccessExpr *expr);
  void printImpl(const OperationExpr *expr);
  void printImpl(const RangeExpr *expr);
  void printImpl(const TupleExpr *expr);
  void printImpl(const TypeExpr *expr);

```
- **EN**: Implements logic around `printImpl`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 91-104
```cpp
  void printImpl(const AttrConstraintDecl *decl);
  void printImpl(const OpConstraintDecl *decl);
  void printImpl(const TypeConstraintDecl *decl);
  void printImpl(const TypeRangeConstraintDecl *decl);
  void printImpl(const UserConstraintDecl *decl);
  void printImpl(const ValueConstraintDecl *decl);
  void printImpl(const ValueRangeConstraintDecl *decl);
  void printImpl(const NamedAttributeDecl *decl);
  void printImpl(const OpNameDecl *decl);
  void printImpl(const PatternDecl *decl);
  void printImpl(const UserRewriteDecl *decl);
  void printImpl(const VariableDecl *decl);
  void printImpl(const Module *module);

```
- **EN**: Implements logic around `printImpl`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 105-114
```cpp
  /// Print the current indent stack.
  void printIndent() {
    if (elementIndentStack.empty())
      return;

    for (bool isLastElt : llvm::ArrayRef(elementIndentStack).drop_back())
      os << (isLastElt ? "  " : " |");
    os << (elementIndentStack.back() ? " `" : " |");
  }

```
- **EN**: Implements logic around `printIndent`, `empty`, `ArrayRef`, `back`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printIndent`、`empty`、`ArrayRef`、`back` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 115-130
```cpp
  /// The raw output stream.
  raw_ostream &os;

  /// A stack of indents and a flag indicating if the current element being
  /// printed at that indent is the last element.
  SmallVector<bool> elementIndentStack;
};
} // namespace

void NodePrinter::print(Type type) {
  // Protect against invalid inputs.
  if (!type) {
    os << "Type<NULL>";
    return;
  }

```
- **EN**: Implements logic around `print`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `print` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 131-150
```cpp
  TypeSwitch<Type>(type)
      .Case([&](AttributeType) { os << "Attr"; })
      .Case([&](ConstraintType) { os << "Constraint"; })
      .Case([&](OperationType type) {
        os << "Op";
        if (std::optional<StringRef> name = type.getName())
          os << "<" << *name << ">";
      })
      .Case([&](RangeType type) {
        print(type.getElementType());
        os << "Range";
      })
      .Case([&](RewriteType) { os << "Rewrite"; })
      .Case([&](TupleType type) {
        os << "Tuple<";
        llvm::interleaveComma(
            llvm::zip(type.getElementNames(), type.getElementTypes()), os,
            [&](auto it) {
              if (!std::get<0>(it).empty())
                os << std::get<0>(it) << ": ";
```
- **EN**: Implements logic around `TypeSwitch`, `Case`, `getName`, `print`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `TypeSwitch`、`Case`、`getName`、`print` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 151-163
```cpp
              this->print(std::get<1>(it));
            });
        os << ">";
      })
      .Case([&](TypeType) { os << "Type"; })
      .Case([&](ValueType) { os << "Value"; })
      .DefaultUnreachable("unknown AST type");
}

void NodePrinter::print(const Node *node) {
  printIndent();
  os << "-";

```
- **EN**: Implements logic around `print`, `Case`, `DefaultUnreachable`, `printIndent`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `print`、`Case`、`DefaultUnreachable`、`printIndent` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 164-175
```cpp
  elementIndentStack.push_back(/*isLastElt*/ false);
  TypeSwitch<const Node *>(node)
      .Case<
          // Statements.
          const CompoundStmt, const EraseStmt, const LetStmt, const ReplaceStmt,
          const ReturnStmt, const RewriteStmt,

          // Expressions.
          const AttributeExpr, const CallExpr, const DeclRefExpr,
          const MemberAccessExpr, const OperationExpr, const RangeExpr,
          const TupleExpr, const TypeExpr,

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 176-188
```cpp
          // Decls.
          const AttrConstraintDecl, const OpConstraintDecl,
          const TypeConstraintDecl, const TypeRangeConstraintDecl,
          const UserConstraintDecl, const ValueConstraintDecl,
          const ValueRangeConstraintDecl, const NamedAttributeDecl,
          const OpNameDecl, const PatternDecl, const UserRewriteDecl,
          const VariableDecl,

          const Module>([&](auto derivedNode) { this->printImpl(derivedNode); })
      .DefaultUnreachable("unknown AST node");
  elementIndentStack.pop_back();
}

```
- **EN**: Implements logic around `Module>`, `DefaultUnreachable`, `pop_back`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `Module>`、`DefaultUnreachable`、`pop_back` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 189-198
```cpp
void NodePrinter::printImpl(const CompoundStmt *stmt) {
  os << "CompoundStmt " << stmt << "\n";
  printChildren(stmt->getChildren());
}

void NodePrinter::printImpl(const EraseStmt *stmt) {
  os << "EraseStmt " << stmt << "\n";
  printChildren(stmt->getRootOpExpr());
}

```
- **EN**: Implements logic around `printImpl`, `printChildren`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`printChildren` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 199-209
```cpp
void NodePrinter::printImpl(const LetStmt *stmt) {
  os << "LetStmt " << stmt << "\n";
  printChildren(stmt->getVarDecl());
}

void NodePrinter::printImpl(const ReplaceStmt *stmt) {
  os << "ReplaceStmt " << stmt << "\n";
  printChildren(stmt->getRootOpExpr());
  printChildren("ReplValues", stmt->getReplExprs());
}

```
- **EN**: Implements logic around `printImpl`, `printChildren`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`printChildren` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 210-219
```cpp
void NodePrinter::printImpl(const ReturnStmt *stmt) {
  os << "ReturnStmt " << stmt << "\n";
  printChildren(stmt->getResultExpr());
}

void NodePrinter::printImpl(const RewriteStmt *stmt) {
  os << "RewriteStmt " << stmt << "\n";
  printChildren(stmt->getRootOpExpr(), stmt->getRewriteBody());
}

```
- **EN**: Implements logic around `printImpl`, `printChildren`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`printChildren` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 220-234
```cpp
void NodePrinter::printImpl(const AttributeExpr *expr) {
  os << "AttributeExpr " << expr << " Value<\"" << expr->getValue() << "\">\n";
}

void NodePrinter::printImpl(const CallExpr *expr) {
  os << "CallExpr " << expr << " Type<";
  print(expr->getType());
  os << ">";
  if (expr->getIsNegated())
    os << " Negated";
  os << "\n";
  printChildren(expr->getCallableExpr());
  printChildren("Arguments", expr->getArguments());
}

```
- **EN**: Implements logic around `printImpl`, `getValue`, `print`, `getIsNegated`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`getValue`、`print`、`getIsNegated` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 235-249
```cpp
void NodePrinter::printImpl(const DeclRefExpr *expr) {
  os << "DeclRefExpr " << expr << " Type<";
  print(expr->getType());
  os << ">\n";
  printChildren(expr->getDecl());
}

void NodePrinter::printImpl(const MemberAccessExpr *expr) {
  os << "MemberAccessExpr " << expr << " Member<" << expr->getMemberName()
     << "> Type<";
  print(expr->getType());
  os << ">\n";
  printChildren(expr->getParentExpr());
}

```
- **EN**: Implements logic around `printImpl`, `print`, `printChildren`, `getMemberName`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`print`、`printChildren`、`getMemberName` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 250-260
```cpp
void NodePrinter::printImpl(const OperationExpr *expr) {
  os << "OperationExpr " << expr << " Type<";
  print(expr->getType());
  os << ">\n";

  printChildren(expr->getNameDecl());
  printChildren("Operands", expr->getOperands());
  printChildren("Result Types", expr->getResultTypes());
  printChildren("Attributes", expr->getAttributes());
}

```
- **EN**: Implements logic around `printImpl`, `print`, `printChildren`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`print`、`printChildren` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 261-273
```cpp
void NodePrinter::printImpl(const RangeExpr *expr) {
  os << "RangeExpr " << expr << " Type<";
  print(expr->getType());
  os << ">\n";

  printChildren(expr->getElements());
}

void NodePrinter::printImpl(const TupleExpr *expr) {
  os << "TupleExpr " << expr << " Type<";
  print(expr->getType());
  os << ">\n";

```
- **EN**: Implements logic around `printImpl`, `print`, `printChildren`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`print`、`printChildren` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 274-286
```cpp
  printChildren(expr->getElements());
}

void NodePrinter::printImpl(const TypeExpr *expr) {
  os << "TypeExpr " << expr << " Value<\"" << expr->getValue() << "\">\n";
}

void NodePrinter::printImpl(const AttrConstraintDecl *decl) {
  os << "AttrConstraintDecl " << decl << "\n";
  if (const auto *typeExpr = decl->getTypeExpr())
    printChildren(typeExpr);
}

```
- **EN**: Implements logic around `printChildren`, `printImpl`, `getValue`, `getTypeExpr`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printChildren`、`printImpl`、`getValue`、`getTypeExpr` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 287-299
```cpp
void NodePrinter::printImpl(const OpConstraintDecl *decl) {
  os << "OpConstraintDecl " << decl << "\n";
  printChildren(decl->getNameDecl());
}

void NodePrinter::printImpl(const TypeConstraintDecl *decl) {
  os << "TypeConstraintDecl " << decl << "\n";
}

void NodePrinter::printImpl(const TypeRangeConstraintDecl *decl) {
  os << "TypeRangeConstraintDecl " << decl << "\n";
}

```
- **EN**: Implements logic around `printImpl`, `printChildren`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`printChildren` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 300-314
```cpp
void NodePrinter::printImpl(const UserConstraintDecl *decl) {
  os << "UserConstraintDecl " << decl << " Name<" << decl->getName().getName()
     << "> ResultType<" << decl->getResultType() << ">";
  if (std::optional<StringRef> codeBlock = decl->getCodeBlock()) {
    os << " Code<";
    llvm::printEscapedString(*codeBlock, os);
    os << ">";
  }
  os << "\n";
  printChildren("Inputs", decl->getInputs());
  printChildren("Results", decl->getResults());
  if (const CompoundStmt *body = decl->getBody())
    printChildren(body);
}

```
- **EN**: Implements logic around `printImpl`, `getName`, `getResultType`, `getCodeBlock`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`getName`、`getResultType`、`getCodeBlock` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 315-326
```cpp
void NodePrinter::printImpl(const ValueConstraintDecl *decl) {
  os << "ValueConstraintDecl " << decl << "\n";
  if (const auto *typeExpr = decl->getTypeExpr())
    printChildren(typeExpr);
}

void NodePrinter::printImpl(const ValueRangeConstraintDecl *decl) {
  os << "ValueRangeConstraintDecl " << decl << "\n";
  if (const auto *typeExpr = decl->getTypeExpr())
    printChildren(typeExpr);
}

```
- **EN**: Implements logic around `printImpl`, `getTypeExpr`, `printChildren`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`getTypeExpr`、`printChildren` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 327-339
```cpp
void NodePrinter::printImpl(const NamedAttributeDecl *decl) {
  os << "NamedAttributeDecl " << decl << " Name<" << decl->getName().getName()
     << ">\n";
  printChildren(decl->getValue());
}

void NodePrinter::printImpl(const OpNameDecl *decl) {
  os << "OpNameDecl " << decl;
  if (std::optional<StringRef> name = decl->getName())
    os << " Name<" << *name << ">";
  os << "\n";
}

```
- **EN**: Implements logic around `printImpl`, `getName`, `printChildren`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`getName`、`printChildren` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 340-352
```cpp
void NodePrinter::printImpl(const PatternDecl *decl) {
  os << "PatternDecl " << decl;
  if (const Name *name = decl->getName())
    os << " Name<" << name->getName() << ">";
  if (std::optional<uint16_t> benefit = decl->getBenefit())
    os << " Benefit<" << *benefit << ">";
  if (decl->hasBoundedRewriteRecursion())
    os << " Recursion";

  os << "\n";
  printChildren(decl->getBody());
}

```
- **EN**: Implements logic around `printImpl`, `getName`, `getBenefit`, `hasBoundedRewriteRecursion`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`getName`、`getBenefit`、`hasBoundedRewriteRecursion` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 353-367
```cpp
void NodePrinter::printImpl(const UserRewriteDecl *decl) {
  os << "UserRewriteDecl " << decl << " Name<" << decl->getName().getName()
     << "> ResultType<" << decl->getResultType() << ">";
  if (std::optional<StringRef> codeBlock = decl->getCodeBlock()) {
    os << " Code<";
    llvm::printEscapedString(*codeBlock, os);
    os << ">";
  }
  os << "\n";
  printChildren("Inputs", decl->getInputs());
  printChildren("Results", decl->getResults());
  if (const CompoundStmt *body = decl->getBody())
    printChildren(body);
}

```
- **EN**: Implements logic around `printImpl`, `getName`, `getResultType`, `getCodeBlock`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`getName`、`getResultType`、`getCodeBlock` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 368-381
```cpp
void NodePrinter::printImpl(const VariableDecl *decl) {
  os << "VariableDecl " << decl << " Name<" << decl->getName().getName()
     << "> Type<";
  print(decl->getType());
  os << ">\n";
  if (Expr *initExpr = decl->getInitExpr())
    printChildren(initExpr);

  auto constraints =
      llvm::map_range(decl->getConstraints(),
                      [](const ConstraintRef &ref) { return ref.constraint; });
  printChildren("Constraints", constraints);
}

```
- **EN**: Implements logic around `printImpl`, `getName`, `print`, `getInitExpr`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`getName`、`print`、`getInitExpr` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 382-392
```cpp
void NodePrinter::printImpl(const Module *module) {
  os << "Module " << module << "\n";
  printChildren(module->getChildren());
}

//===----------------------------------------------------------------------===//
// Entry point
//===----------------------------------------------------------------------===//

void Node::print(raw_ostream &os) const { NodePrinter(os).print(this); }

```
- **EN**: Implements logic around `printImpl`, `printChildren`, `print`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printImpl`、`printChildren`、`print` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 393-393
```cpp
void Type::print(raw_ostream &os) const { NodePrinter(os).print(*this); }
```
- **EN**: Implements logic around `print`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `print` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

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

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/PDLL/AST/Nodes.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/TypeSwitch.h`, `llvm/Support/SaveAndRestore.h`, `llvm/Support/ScopedPrinter.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utilities / LLVM ADT 容器与工具 (2), LLVM support-library helpers / LLVM Support 库辅助工具 (2), tooling support declarations / 工具支持声明 (1)
