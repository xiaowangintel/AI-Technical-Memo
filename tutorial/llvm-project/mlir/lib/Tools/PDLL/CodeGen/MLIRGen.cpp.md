# MLIRGen.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/PDLL/CodeGen/MLIRGen.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===- MLIRGen.cpp --------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Tools/PDLL/CodeGen/MLIRGen.h"
#include "mlir/AsmParser/AsmParser.h"
#include "mlir/Dialect/PDL/IR/PDL.h"
#include "mlir/Dialect/PDL/IR/PDLOps.h"
#include "mlir/Dialect/PDL/IR/PDLTypes.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/IR/Verifier.h"
#include "mlir/Tools/PDLL/AST/Context.h"
#include "mlir/Tools/PDLL/AST/Nodes.h"
#include "mlir/Tools/PDLL/AST/Types.h"
#include "mlir/Tools/PDLL/ODS/Context.h"
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/PDLL/CodeGen/MLIRGen.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/PDL/IR/PDL.h`, `mlir/Dialect/PDL/IR/PDLOps.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/PDLL/CodeGen/MLIRGen.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/PDL/IR/PDL.h`, `mlir/Dialect/PDL/IR/PDLOps.h`。

### Lines 21-30
```cpp
#include "mlir/Tools/PDLL/ODS/Operation.h"
#include "llvm/ADT/ScopedHashTable.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/TypeSwitch.h"
#include <optional>

using namespace mlir;
using namespace mlir::pdll;

//===----------------------------------------------------------------------===//
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/PDLL/ODS/Operation.h`, `llvm/ADT/ScopedHashTable.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/TypeSwitch.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/PDLL/ODS/Operation.h`, `llvm/ADT/ScopedHashTable.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/TypeSwitch.h`。

### Lines 31-44
```cpp
// CodeGen
//===----------------------------------------------------------------------===//

namespace {
class CodeGen {
public:
  CodeGen(MLIRContext *mlirContext, const ast::Context &context,
          const llvm::SourceMgr &sourceMgr)
      : builder(mlirContext), odsContext(context.getODSContext()),
        sourceMgr(sourceMgr) {
    // Make sure that the PDL dialect is loaded.
    mlirContext->loadDialect<pdl::PDLDialect>();
  }

```
- **EN**: Introduces declarations for `CodeGen`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `CodeGen` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 45-54
```cpp
  OwningOpRef<ModuleOp> generate(const ast::Module &module);

private:
  /// Generate an MLIR location from the given source location.
  Location genLoc(llvm::SMLoc loc);
  Location genLoc(llvm::SMRange loc) { return genLoc(loc.Start); }

  /// Generate an MLIR type from the given source type.
  Type genType(ast::Type type);

```
- **EN**: Implements logic around `generate`, `genLoc`, `genType`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `generate`、`genLoc`、`genType` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 55-68
```cpp
  /// Generate MLIR for the given AST node.
  void gen(const ast::Node *node);

  //===--------------------------------------------------------------------===//
  // Statements
  //===--------------------------------------------------------------------===//

  void genImpl(const ast::CompoundStmt *stmt);
  void genImpl(const ast::EraseStmt *stmt);
  void genImpl(const ast::LetStmt *stmt);
  void genImpl(const ast::ReplaceStmt *stmt);
  void genImpl(const ast::RewriteStmt *stmt);
  void genImpl(const ast::ReturnStmt *stmt);

```
- **EN**: Implements logic around `gen`, `genImpl`.
- **CN**: 围绕 `gen`、`genImpl` 实现具体逻辑。

### Lines 69-78
```cpp
  //===--------------------------------------------------------------------===//
  // Decls
  //===--------------------------------------------------------------------===//

  void genImpl(const ast::UserConstraintDecl *decl);
  void genImpl(const ast::UserRewriteDecl *decl);
  void genImpl(const ast::PatternDecl *decl);

  /// Generate the set of MLIR values defined for the given variable decl, and
  /// apply any attached constraints.
```
- **EN**: Implements logic around `genImpl`.
- **CN**: 围绕 `genImpl` 实现具体逻辑。

### Lines 79-89
```cpp
  SmallVector<Value> genVar(const ast::VariableDecl *varDecl);

  /// Generate the value for a variable that does not have an initializer
  /// expression, i.e. create the PDL value based on the type/constraints of the
  /// variable.
  Value genNonInitializerVar(const ast::VariableDecl *varDecl, Location loc);

  /// Apply the constraints of the given variable to `values`, which correspond
  /// to the MLIR values of the variable.
  void applyVarConstraints(const ast::VariableDecl *varDecl, ValueRange values);

```
- **EN**: Implements logic around `genVar`, `genNonInitializerVar`, `applyVarConstraints`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genVar`、`genNonInitializerVar`、`applyVarConstraints` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 90-104
```cpp
  //===--------------------------------------------------------------------===//
  // Expressions
  //===--------------------------------------------------------------------===//

  Value genSingleExpr(const ast::Expr *expr);
  SmallVector<Value> genExpr(const ast::Expr *expr);
  Value genExprImpl(const ast::AttributeExpr *expr);
  SmallVector<Value> genExprImpl(const ast::CallExpr *expr);
  SmallVector<Value> genExprImpl(const ast::DeclRefExpr *expr);
  Value genExprImpl(const ast::MemberAccessExpr *expr);
  Value genExprImpl(const ast::OperationExpr *expr);
  Value genExprImpl(const ast::RangeExpr *expr);
  SmallVector<Value> genExprImpl(const ast::TupleExpr *expr);
  Value genExprImpl(const ast::TypeExpr *expr);

```
- **EN**: Implements logic around `genSingleExpr`, `genExpr`, `genExprImpl`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genSingleExpr`、`genExpr`、`genExprImpl` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 105-114
```cpp
  SmallVector<Value> genConstraintCall(const ast::UserConstraintDecl *decl,
                                       Location loc, ValueRange inputs,
                                       bool isNegated = false);
  SmallVector<Value> genRewriteCall(const ast::UserRewriteDecl *decl,
                                    Location loc, ValueRange inputs);
  template <typename PDLOpT, typename T>
  SmallVector<Value> genConstraintOrRewriteCall(const T *decl, Location loc,
                                                ValueRange inputs,
                                                bool isNegated = false);

```
- **EN**: Implements logic around `genConstraintCall`, `genRewriteCall`, `genConstraintOrRewriteCall`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genConstraintCall`、`genRewriteCall`、`genConstraintOrRewriteCall` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 115-126
```cpp
  //===--------------------------------------------------------------------===//
  // Fields
  //===--------------------------------------------------------------------===//

  /// The MLIR builder used for building the resultant IR.
  OpBuilder builder;

  /// A map from variable declarations to the MLIR equivalent.
  using VariableMapTy =
      llvm::ScopedHashTable<const ast::VariableDecl *, SmallVector<Value>>;
  VariableMapTy variables;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 127-139
```cpp
  /// A reference to the ODS context.
  const ods::Context &odsContext;

  /// The source manager of the PDLL ast.
  const llvm::SourceMgr &sourceMgr;
};
} // namespace

OwningOpRef<ModuleOp> CodeGen::generate(const ast::Module &module) {
  OwningOpRef<ModuleOp> mlirModule =
      ModuleOp::create(builder, genLoc(module.getLoc()));
  builder.setInsertionPointToStart(mlirModule->getBody());

```
- **EN**: Implements logic around `generate`, `create`, `setInsertionPointToStart`.
- **CN**: 围绕 `generate`、`create`、`setInsertionPointToStart` 实现具体逻辑。

### Lines 140-149
```cpp
  // Generate code for each of the decls within the module.
  for (const ast::Decl *decl : module.getChildren())
    gen(decl);

  return mlirModule;
}

Location CodeGen::genLoc(llvm::SMLoc loc) {
  unsigned fileID = sourceMgr.FindBufferContainingLoc(loc);

```
- **EN**: Implements logic around `getChildren`, `gen`, `genLoc`, `FindBufferContainingLoc`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getChildren`、`gen`、`genLoc`、`FindBufferContainingLoc` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 150-169
```cpp
  auto [lineNo, column] = sourceMgr.getLineAndColumn(loc);
  auto *buffer = sourceMgr.getMemoryBuffer(fileID);

  return FileLineColLoc::get(builder.getContext(),
                             buffer->getBufferIdentifier(), lineNo, column);
}

Type CodeGen::genType(ast::Type type) {
  return TypeSwitch<ast::Type, Type>(type)
      .Case([&](ast::AttributeType astType) -> Type {
        return builder.getType<pdl::AttributeType>();
      })
      .Case([&](ast::OperationType astType) -> Type {
        return builder.getType<pdl::OperationType>();
      })
      .Case([&](ast::TypeType astType) -> Type {
        return builder.getType<pdl::TypeType>();
      })
      .Case([&](ast::ValueType astType) -> Type {
        return builder.getType<pdl::ValueType>();
```
- **EN**: Implements logic around `getLineAndColumn`, `getMemoryBuffer`, `get`, `getBufferIdentifier`, and 7 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getLineAndColumn`、`getMemoryBuffer`、`get`、`getBufferIdentifier` 等另外 7 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 170-185
```cpp
      })
      .Case([&](ast::RangeType astType) -> Type {
        return pdl::RangeType::get(genType(astType.getElementType()));
      });
}

void CodeGen::gen(const ast::Node *node) {
  TypeSwitch<const ast::Node *>(node)
      .Case<const ast::CompoundStmt, const ast::EraseStmt, const ast::LetStmt,
            const ast::ReplaceStmt, const ast::RewriteStmt,
            const ast::ReturnStmt, const ast::UserConstraintDecl,
            const ast::UserRewriteDecl, const ast::PatternDecl>(
          [&](auto derivedNode) { this->genImpl(derivedNode); })
      .Case([&](const ast::Expr *expr) { genExpr(expr); });
}

```
- **EN**: Implements logic around `Case`, `get`, `gen`, `PatternDecl>`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `Case`、`get`、`gen`、`PatternDecl>` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 186-195
```cpp
//===----------------------------------------------------------------------===//
// CodeGen: Statements
//===----------------------------------------------------------------------===//

void CodeGen::genImpl(const ast::CompoundStmt *stmt) {
  VariableMapTy::ScopeTy varScope(variables);
  for (const ast::Stmt *childStmt : stmt->getChildren())
    gen(childStmt);
}

```
- **EN**: Implements logic around `genImpl`, `varScope`, `getChildren`, `gen`.
- **CN**: 围绕 `genImpl`、`varScope`、`getChildren`、`gen` 实现具体逻辑。

### Lines 196-208
```cpp
/// If the given builder is nested under a PDL PatternOp, build a rewrite
/// operation and update the builder to nest under it. This is necessary for
/// PDLL operation rewrite statements that are directly nested within a Pattern.
static void checkAndNestUnderRewriteOp(OpBuilder &builder, Value rootExpr,
                                       Location loc) {
  if (isa<pdl::PatternOp>(builder.getInsertionBlock()->getParentOp())) {
    pdl::RewriteOp rewrite =
        pdl::RewriteOp::create(builder, loc, rootExpr, /*name=*/StringAttr(),
                               /*externalArgs=*/ValueRange());
    builder.createBlock(&rewrite.getBodyRegion());
  }
}

```
- **EN**: Implements logic around `checkAndNestUnderRewriteOp`, `PatternOp>`, `create`, `ValueRange`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `checkAndNestUnderRewriteOp`、`PatternOp>`、`create`、`ValueRange` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 209-219
```cpp
void CodeGen::genImpl(const ast::EraseStmt *stmt) {
  OpBuilder::InsertionGuard insertGuard(builder);
  Value rootExpr = genSingleExpr(stmt->getRootOpExpr());
  Location loc = genLoc(stmt->getLoc());

  // Make sure we are nested in a RewriteOp.
  OpBuilder::InsertionGuard guard(builder);
  checkAndNestUnderRewriteOp(builder, rootExpr, loc);
  pdl::EraseOp::create(builder, loc, rootExpr);
}

```
- **EN**: Implements logic around `genImpl`, `insertGuard`, `genSingleExpr`, `genLoc`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genImpl`、`insertGuard`、`genSingleExpr`、`genLoc` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 220-230
```cpp
void CodeGen::genImpl(const ast::LetStmt *stmt) { genVar(stmt->getVarDecl()); }

void CodeGen::genImpl(const ast::ReplaceStmt *stmt) {
  OpBuilder::InsertionGuard insertGuard(builder);
  Value rootExpr = genSingleExpr(stmt->getRootOpExpr());
  Location loc = genLoc(stmt->getLoc());

  // Make sure we are nested in a RewriteOp.
  OpBuilder::InsertionGuard guard(builder);
  checkAndNestUnderRewriteOp(builder, rootExpr, loc);

```
- **EN**: Implements logic around `genImpl`, `insertGuard`, `genSingleExpr`, `genLoc`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genImpl`、`insertGuard`、`genSingleExpr`、`genLoc` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 231-244
```cpp
  SmallVector<Value> replValues;
  for (ast::Expr *replExpr : stmt->getReplExprs())
    replValues.push_back(genSingleExpr(replExpr));

  // Check to see if the statement has a replacement operation, or a range of
  // replacement values.
  bool usesReplOperation =
      replValues.size() == 1 &&
      isa<pdl::OperationType>(replValues.front().getType());
  pdl::ReplaceOp::create(
      builder, loc, rootExpr, usesReplOperation ? replValues[0] : Value(),
      usesReplOperation ? ValueRange() : ValueRange(replValues));
}

```
- **EN**: Implements logic around `getReplExprs`, `push_back`, `size`, `OperationType>`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getReplExprs`、`push_back`、`size`、`OperationType>` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 245-254
```cpp
void CodeGen::genImpl(const ast::RewriteStmt *stmt) {
  OpBuilder::InsertionGuard insertGuard(builder);
  Value rootExpr = genSingleExpr(stmt->getRootOpExpr());

  // Make sure we are nested in a RewriteOp.
  OpBuilder::InsertionGuard guard(builder);
  checkAndNestUnderRewriteOp(builder, rootExpr, genLoc(stmt->getLoc()));
  gen(stmt->getRewriteBody());
}

```
- **EN**: Implements logic around `genImpl`, `insertGuard`, `genSingleExpr`, `guard`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genImpl`、`insertGuard`、`genSingleExpr`、`guard` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 255-269
```cpp
void CodeGen::genImpl(const ast::ReturnStmt *stmt) {
  // ReturnStmt generation is handled by the respective constraint or rewrite
  // parent node.
}

//===----------------------------------------------------------------------===//
// CodeGen: Decls
//===----------------------------------------------------------------------===//

void CodeGen::genImpl(const ast::UserConstraintDecl *decl) {
  // All PDLL constraints get inlined when called, and the main native
  // constraint declarations doesn't require any MLIR to be generated, only uses
  // of it do.
}

```
- **EN**: Implements logic around `genImpl`.
- **CN**: 围绕 `genImpl` 实现具体逻辑。

### Lines 270-285
```cpp
void CodeGen::genImpl(const ast::UserRewriteDecl *decl) {
  // All PDLL rewrites get inlined when called, and the main native
  // rewrite declarations doesn't require any MLIR to be generated, only uses
  // of it do.
}

void CodeGen::genImpl(const ast::PatternDecl *decl) {
  const ast::Name *name = decl->getName();

  // FIXME: Properly model HasBoundedRecursion in PDL so that we don't drop it
  // here.
  pdl::PatternOp pattern = pdl::PatternOp::create(
      builder, genLoc(decl->getLoc()), decl->getBenefit(),
      name ? std::optional<StringRef>(name->getName())
           : std::optional<StringRef>());

```
- **EN**: Implements logic around `genImpl`, `getName`, `create`, `genLoc`, and 1 more symbols.
- **CN**: 围绕 `genImpl`、`getName`、`create`、`genLoc` 等另外 1 个符号 实现具体逻辑。

### Lines 286-295
```cpp
  OpBuilder::InsertionGuard savedInsertPoint(builder);
  builder.setInsertionPointToStart(pattern.getBody());
  gen(decl->getBody());
}

SmallVector<Value> CodeGen::genVar(const ast::VariableDecl *varDecl) {
  auto it = variables.begin(varDecl);
  if (it != variables.end())
    return *it;

```
- **EN**: Implements logic around `savedInsertPoint`, `setInsertionPointToStart`, `gen`, `genVar`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `savedInsertPoint`、`setInsertionPointToStart`、`gen`、`genVar` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 296-306
```cpp
  // If the variable has an initial value, use that as the base value.
  // Otherwise, generate a value using the constraint list.
  SmallVector<Value> values;
  if (const ast::Expr *initExpr = varDecl->getInitExpr())
    values = genExpr(initExpr);
  else
    values.push_back(genNonInitializerVar(varDecl, genLoc(varDecl->getLoc())));

  // Apply the constraints of the values of the variable.
  applyVarConstraints(varDecl, values);

```
- **EN**: Implements logic around `getInitExpr`, `genExpr`, `push_back`, `applyVarConstraints`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getInitExpr`、`genExpr`、`push_back`、`applyVarConstraints` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 307-326
```cpp
  variables.insert(varDecl, values);
  return values;
}

Value CodeGen::genNonInitializerVar(const ast::VariableDecl *varDecl,
                                    Location loc) {
  // A functor used to generate expressions nested
  auto getTypeConstraint = [&]() -> Value {
    for (const ast::ConstraintRef &constraint : varDecl->getConstraints()) {
      Value typeValue =
          TypeSwitch<const ast::Node *, Value>(constraint.constraint)
              .Case<ast::AttrConstraintDecl, ast::ValueConstraintDecl,
                    ast::ValueRangeConstraintDecl>(
                  [&, this](auto *cst) -> Value {
                    if (auto *typeConstraintExpr = cst->getTypeExpr())
                      return this->genSingleExpr(typeConstraintExpr);
                    return Value();
                  })
              .Default(Value());
      if (typeValue)
```
- **EN**: Implements logic around `insert`, `genNonInitializerVar`, `getConstraints`, `Value>`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `insert`、`genNonInitializerVar`、`getConstraints`、`Value>` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 327-346
```cpp
        return typeValue;
    }
    return Value();
  };

  // Generate a value based on the type of the variable.
  ast::Type type = varDecl->getType();
  Type mlirType = genType(type);
  if (isa<ast::ValueType>(type))
    return pdl::OperandOp::create(builder, loc, mlirType, getTypeConstraint());
  if (isa<ast::TypeType>(type))
    return pdl::TypeOp::create(builder, loc, mlirType, /*type=*/TypeAttr());
  if (isa<ast::AttributeType>(type))
    return pdl::AttributeOp::create(builder, loc, getTypeConstraint());
  if (ast::OperationType opType = dyn_cast<ast::OperationType>(type)) {
    Value operands = pdl::OperandsOp::create(
        builder, loc, pdl::RangeType::get(builder.getType<pdl::ValueType>()),
        /*type=*/Value());
    Value results = pdl::TypesOp::create(
        builder, loc, pdl::RangeType::get(builder.getType<pdl::TypeType>()),
```
- **EN**: Implements logic around `Value`, `getType`, `genType`, `ValueType>`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `Value`、`getType`、`genType`、`ValueType>` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 347-362
```cpp
        /*types=*/ArrayAttr());
    return pdl::OperationOp::create(builder, loc, opType.getName(), operands,
                                    ArrayRef<StringRef>(), ValueRange(),
                                    results);
  }

  if (ast::RangeType rangeTy = dyn_cast<ast::RangeType>(type)) {
    ast::Type eleTy = rangeTy.getElementType();
    if (isa<ast::ValueType>(eleTy))
      return pdl::OperandsOp::create(builder, loc, mlirType,
                                     getTypeConstraint());
    if (isa<ast::TypeType>(eleTy))
      return pdl::TypesOp::create(builder, loc, mlirType,
                                  /*types=*/ArrayAttr());
  }

```
- **EN**: Implements logic around `ArrayAttr`, `create`, `ArrayRef`, `RangeType>`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `ArrayAttr`、`create`、`ArrayRef`、`RangeType>` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 363-374
```cpp
  llvm_unreachable("invalid non-initialized variable type");
}

void CodeGen::applyVarConstraints(const ast::VariableDecl *varDecl,
                                  ValueRange values) {
  // Generate calls to any user constraints that were attached via the
  // constraint list.
  for (const ast::ConstraintRef &ref : varDecl->getConstraints())
    if (const auto *userCst = dyn_cast<ast::UserConstraintDecl>(ref.constraint))
      genConstraintCall(userCst, genLoc(ref.referenceLoc), values);
}

```
- **EN**: Implements logic around `llvm_unreachable`, `applyVarConstraints`, `getConstraints`, `UserConstraintDecl>`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `llvm_unreachable`、`applyVarConstraints`、`getConstraints`、`UserConstraintDecl>` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 375-390
```cpp
//===----------------------------------------------------------------------===//
// CodeGen: Expressions
//===----------------------------------------------------------------------===//

Value CodeGen::genSingleExpr(const ast::Expr *expr) {
  return TypeSwitch<const ast::Expr *, Value>(expr)
      .Case<const ast::AttributeExpr, const ast::MemberAccessExpr,
            const ast::OperationExpr, const ast::RangeExpr,
            const ast::TypeExpr>(
          [&](auto derivedNode) { return this->genExprImpl(derivedNode); })
      .Case<const ast::CallExpr, const ast::DeclRefExpr, const ast::TupleExpr>(
          [&](auto derivedNode) {
            return llvm::getSingleElement(this->genExprImpl(derivedNode));
          });
}

```
- **EN**: Implements logic around `genSingleExpr`, `Value>`, `TypeExpr>`, `genExprImpl`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genSingleExpr`、`Value>`、`TypeExpr>`、`genExprImpl` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 391-405
```cpp
SmallVector<Value> CodeGen::genExpr(const ast::Expr *expr) {
  return TypeSwitch<const ast::Expr *, SmallVector<Value>>(expr)
      .Case<const ast::CallExpr, const ast::DeclRefExpr, const ast::TupleExpr>(
          [&](auto derivedNode) { return this->genExprImpl(derivedNode); })
      .Default([&](const ast::Expr *expr) -> SmallVector<Value> {
        return {genSingleExpr(expr)};
      });
}

Value CodeGen::genExprImpl(const ast::AttributeExpr *expr) {
  Attribute attr = parseAttribute(expr->getValue(), builder.getContext());
  assert(attr && "invalid MLIR attribute data");
  return pdl::AttributeOp::create(builder, genLoc(expr->getLoc()), attr);
}

```
- **EN**: Implements logic around `genExpr`, `SmallVector`, `TupleExpr>`, `genExprImpl`, and 5 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `genExpr`、`SmallVector`、`TupleExpr>`、`genExprImpl` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 406-415
```cpp
SmallVector<Value> CodeGen::genExprImpl(const ast::CallExpr *expr) {
  Location loc = genLoc(expr->getLoc());
  SmallVector<Value> arguments;
  for (const ast::Expr *arg : expr->getArguments())
    arguments.push_back(genSingleExpr(arg));

  // Resolve the callable expression of this call.
  auto *callableExpr = dyn_cast<ast::DeclRefExpr>(expr->getCallableExpr());
  assert(callableExpr && "unhandled CallExpr callable");

```
- **EN**: Implements logic around `genExprImpl`, `genLoc`, `getArguments`, `push_back`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genExprImpl`、`genLoc`、`getArguments`、`push_back` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 416-430
```cpp
  // Generate the PDL based on the type of callable.
  const ast::Decl *callable = callableExpr->getDecl();
  if (const auto *decl = dyn_cast<ast::UserConstraintDecl>(callable))
    return genConstraintCall(decl, loc, arguments, expr->getIsNegated());
  if (const auto *decl = dyn_cast<ast::UserRewriteDecl>(callable))
    return genRewriteCall(decl, loc, arguments);
  llvm_unreachable("unhandled CallExpr callable");
}

SmallVector<Value> CodeGen::genExprImpl(const ast::DeclRefExpr *expr) {
  if (const auto *varDecl = dyn_cast<ast::VariableDecl>(expr->getDecl()))
    return genVar(varDecl);
  llvm_unreachable("unknown decl reference expression");
}

```
- **EN**: Implements logic around `getDecl`, `UserConstraintDecl>`, `genConstraintCall`, `UserRewriteDecl>`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getDecl`、`UserConstraintDecl>`、`genConstraintCall`、`UserRewriteDecl>` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 431-446
```cpp
Value CodeGen::genExprImpl(const ast::MemberAccessExpr *expr) {
  Location loc = genLoc(expr->getLoc());
  StringRef name = expr->getMemberName();
  SmallVector<Value> parentExprs = genExpr(expr->getParentExpr());
  ast::Type parentType = expr->getParentExpr()->getType();

  // Handle operation based member access.
  if (ast::OperationType opType = dyn_cast<ast::OperationType>(parentType)) {
    if (isa<ast::AllResultsMemberAccessExpr>(expr)) {
      Type mlirType = genType(expr->getType());
      if (isa<pdl::ValueType>(mlirType))
        return pdl::ResultOp::create(builder, loc, mlirType, parentExprs[0],
                                     builder.getI32IntegerAttr(0));
      return pdl::ResultsOp::create(builder, loc, mlirType, parentExprs[0]);
    }

```
- **EN**: Implements logic around `genExprImpl`, `genLoc`, `getMemberName`, `genExpr`, and 7 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genExprImpl`、`genLoc`、`getMemberName`、`genExpr` 等另外 7 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 447-457
```cpp
    const ods::Operation *odsOp = opType.getODSOperation();
    if (!odsOp) {
      assert(llvm::isDigit(name[0]) &&
             "unregistered op only allows numeric indexing");
      unsigned resultIndex;
      name.getAsInteger(/*Radix=*/10, resultIndex);
      IntegerAttr index = builder.getI32IntegerAttr(resultIndex);
      return pdl::ResultOp::create(builder, loc, genType(expr->getType()),
                                   parentExprs[0], index);
    }

```
- **EN**: Implements logic around `getODSOperation`, `assert`, `getAsInteger`, `getI32IntegerAttr`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getODSOperation`、`assert`、`getAsInteger`、`getI32IntegerAttr` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 458-470
```cpp
    // Find the result with the member name or by index.
    ArrayRef<ods::OperandOrResult> results = odsOp->getResults();
    unsigned resultIndex = results.size();
    if (llvm::isDigit(name[0])) {
      name.getAsInteger(/*Radix=*/10, resultIndex);
    } else {
      auto findFn = [&](const ods::OperandOrResult &result) {
        return result.getName() == name;
      };
      resultIndex = llvm::find_if(results, findFn) - results.begin();
    }
    assert(resultIndex < results.size() && "invalid result index");

```
- **EN**: Implements logic around `getResults`, `size`, `isDigit`, `getAsInteger`, and 3 more symbols.
- **CN**: 围绕 `getResults`、`size`、`isDigit`、`getAsInteger` 等另外 3 个符号 实现具体逻辑。

### Lines 471-480
```cpp
    // Generate the result access.
    IntegerAttr index = builder.getI32IntegerAttr(resultIndex);
    return pdl::ResultsOp::create(builder, loc, genType(expr->getType()),
                                  parentExprs[0], index);
  }

  // Handle tuple based member access.
  if (auto tupleType = dyn_cast<ast::TupleType>(parentType)) {
    auto elementNames = tupleType.getElementNames();

```
- **EN**: Implements logic around `getI32IntegerAttr`, `create`, `TupleType>`, `getElementNames`.
- **CN**: 围绕 `getI32IntegerAttr`、`create`、`TupleType>`、`getElementNames` 实现具体逻辑。

### Lines 481-491
```cpp
    // The index is either a numeric index, or a name.
    unsigned index = 0;
    if (llvm::isDigit(name[0]))
      name.getAsInteger(/*Radix=*/10, index);
    else
      index = llvm::find(elementNames, name) - elementNames.begin();

    assert(index < parentExprs.size() && "invalid result index");
    return parentExprs[index];
  }

```
- **EN**: Implements logic around `isDigit`, `getAsInteger`, `find`, `assert`.
- **CN**: 围绕 `isDigit`、`getAsInteger`、`find`、`assert` 实现具体逻辑。

### Lines 492-503
```cpp
  llvm_unreachable("unhandled member access expression");
}

Value CodeGen::genExprImpl(const ast::OperationExpr *expr) {
  Location loc = genLoc(expr->getLoc());
  std::optional<StringRef> opName = expr->getName();

  // Operands.
  SmallVector<Value> operands;
  for (const ast::Expr *operand : expr->getOperands())
    operands.push_back(genSingleExpr(operand));

```
- **EN**: Implements logic around `llvm_unreachable`, `genExprImpl`, `genLoc`, `getName`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `llvm_unreachable`、`genExprImpl`、`genLoc`、`getName` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 504-516
```cpp
  // Attributes.
  SmallVector<StringRef> attrNames;
  SmallVector<Value> attrValues;
  for (const ast::NamedAttributeDecl *attr : expr->getAttributes()) {
    attrNames.push_back(attr->getName().getName());
    attrValues.push_back(genSingleExpr(attr->getValue()));
  }

  // Results.
  SmallVector<Value> results;
  for (const ast::Expr *result : expr->getResultTypes())
    results.push_back(genSingleExpr(result));

```
- **EN**: Implements logic around `getAttributes`, `push_back`, `getResultTypes`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getAttributes`、`push_back`、`getResultTypes` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 517-529
```cpp
  return pdl::OperationOp::create(builder, loc, opName, operands, attrNames,
                                  attrValues, results);
}

Value CodeGen::genExprImpl(const ast::RangeExpr *expr) {
  SmallVector<Value> elements;
  for (const ast::Expr *element : expr->getElements())
    llvm::append_range(elements, genExpr(element));

  return pdl::RangeOp::create(builder, genLoc(expr->getLoc()),
                              genType(expr->getType()), elements);
}

```
- **EN**: Implements logic around `create`, `genExprImpl`, `getElements`, `append_range`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `create`、`genExprImpl`、`getElements`、`append_range` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 530-544
```cpp
SmallVector<Value> CodeGen::genExprImpl(const ast::TupleExpr *expr) {
  SmallVector<Value> elements;
  for (const ast::Expr *element : expr->getElements())
    elements.push_back(genSingleExpr(element));
  return elements;
}

Value CodeGen::genExprImpl(const ast::TypeExpr *expr) {
  Type type = parseType(expr->getValue(), builder.getContext());
  assert(type && "invalid MLIR type data");
  return pdl::TypeOp::create(builder, genLoc(expr->getLoc()),
                             builder.getType<pdl::TypeType>(),
                             TypeAttr::get(type));
}

```
- **EN**: Implements logic around `genExprImpl`, `getElements`, `push_back`, `parseType`, and 4 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `genExprImpl`、`getElements`、`push_back`、`parseType` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 545-556
```cpp
SmallVector<Value>
CodeGen::genConstraintCall(const ast::UserConstraintDecl *decl, Location loc,
                           ValueRange inputs, bool isNegated) {
  // Apply any constraints defined on the arguments to the input values.
  for (auto it : llvm::zip(decl->getInputs(), inputs))
    applyVarConstraints(std::get<0>(it), std::get<1>(it));

  // Generate the constraint call.
  SmallVector<Value> results =
      genConstraintOrRewriteCall<pdl::ApplyNativeConstraintOp>(
          decl, loc, inputs, isNegated);

```
- **EN**: Implements logic around `genConstraintCall`, `zip`, `applyVarConstraints`, `ApplyNativeConstraintOp>`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genConstraintCall`、`zip`、`applyVarConstraints`、`ApplyNativeConstraintOp>` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 557-568
```cpp
  // Apply any constraints defined on the results of the constraint.
  for (auto it : llvm::zip(decl->getResults(), results))
    applyVarConstraints(std::get<0>(it), std::get<1>(it));
  return results;
}

SmallVector<Value> CodeGen::genRewriteCall(const ast::UserRewriteDecl *decl,
                                           Location loc, ValueRange inputs) {
  return genConstraintOrRewriteCall<pdl::ApplyNativeRewriteOp>(decl, loc,
                                                               inputs);
}

```
- **EN**: Implements logic around `zip`, `applyVarConstraints`, `genRewriteCall`, `ApplyNativeRewriteOp>`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `zip`、`applyVarConstraints`、`genRewriteCall`、`ApplyNativeRewriteOp>` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 569-588
```cpp
template <typename PDLOpT, typename T>
SmallVector<Value>
CodeGen::genConstraintOrRewriteCall(const T *decl, Location loc,
                                    ValueRange inputs, bool isNegated) {
  const ast::CompoundStmt *cstBody = decl->getBody();

  // If the decl doesn't have a statement body, it is a native decl.
  if (!cstBody) {
    ast::Type declResultType = decl->getResultType();
    SmallVector<Type> resultTypes;
    if (ast::TupleType tupleType = dyn_cast<ast::TupleType>(declResultType)) {
      for (ast::Type type : tupleType.getElementTypes())
        resultTypes.push_back(genType(type));
    } else {
      resultTypes.push_back(genType(declResultType));
    }
    PDLOpT pdlOp = PDLOpT::create(builder, loc, resultTypes,
                                  decl->getName().getName(), inputs);
    if (isNegated && std::is_same_v<PDLOpT, pdl::ApplyNativeConstraintOp>)
      cast<pdl::ApplyNativeConstraintOp>(pdlOp).setIsNegated(true);
```
- **EN**: Implements logic around `genConstraintOrRewriteCall`, `getBody`, `getResultType`, `TupleType>`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `genConstraintOrRewriteCall`、`getBody`、`getResultType`、`TupleType>` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 589-600
```cpp
    return pdlOp->getResults();
  }

  // Otherwise, this is a PDLL decl.
  VariableMapTy::ScopeTy varScope(variables);

  // Map the inputs of the call to the decl arguments.
  // Note: This is only valid because we do not support recursion, meaning
  // we don't need to worry about conflicting mappings here.
  for (auto it : llvm::zip(inputs, decl->getInputs()))
    variables.insert(std::get<1>(it), {std::get<0>(it)});

```
- **EN**: Implements logic around `getResults`, `varScope`, `zip`, `insert`.
- **CN**: 围绕 `getResults`、`varScope`、`zip`、`insert` 实现具体逻辑。

### Lines 601-610
```cpp
  // Visit the body of the call as normal.
  gen(cstBody);

  // If the decl has no results, there is nothing to do.
  if (cstBody->getChildren().empty())
    return SmallVector<Value>();
  auto *returnStmt = dyn_cast<ast::ReturnStmt>(cstBody->getChildren().back());
  if (!returnStmt)
    return SmallVector<Value>();

```
- **EN**: Implements logic around `gen`, `getChildren`, `SmallVector`, `ReturnStmt>`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `gen`、`getChildren`、`SmallVector`、`ReturnStmt>` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 611-627
```cpp
  // Otherwise, grab the results from the return statement.
  return genExpr(returnStmt->getResultExpr());
}

//===----------------------------------------------------------------------===//
// MLIRGen
//===----------------------------------------------------------------------===//

OwningOpRef<ModuleOp> mlir::pdll::codegenPDLLToMLIR(
    MLIRContext *mlirContext, const ast::Context &context,
    const llvm::SourceMgr &sourceMgr, const ast::Module &module) {
  CodeGen codegen(mlirContext, context, sourceMgr);
  OwningOpRef<ModuleOp> mlirModule = codegen.generate(module);
  if (failed(verify(*mlirModule)))
    return nullptr;
  return mlirModule;
}
```
- **EN**: Implements logic around `genExpr`, `codegenPDLLToMLIR`, `codegen`, `generate`, and 1 more symbols.
- **CN**: 围绕 `genExpr`、`codegenPDLLToMLIR`、`codegen`、`generate` 等另外 1 个符号 实现具体逻辑。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/PDLL/CodeGen/MLIRGen.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/PDL/IR/PDL.h`, `mlir/Dialect/PDL/IR/PDLOps.h`, `mlir/Dialect/PDL/IR/PDLTypes.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinOps.h`, `mlir/IR/Verifier.h`, `mlir/Tools/PDLL/AST/Context.h`, `mlir/Tools/PDLL/AST/Nodes.h` ... (+6 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: tooling support declarations / 工具支持声明 (6), core MLIR IR abstractions / 核心 MLIR IR 抽象 (3), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (3), MLIR assembly parsing interfaces / MLIR 汇编解析接口 (1)
