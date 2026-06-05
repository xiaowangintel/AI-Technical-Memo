# CPPGen.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/PDLL/CodeGen/CPPGen.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This files contains a PDLL generator that outputs C++ code that defines PDLL patterns as individual C++ PDLPatternModules for direct use in native code, and also defines any native constraints whose bodies were defined in PDLL.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- CPPGen.cpp ---------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-14
```cpp
//
// This files contains a PDLL generator that outputs C++ code that defines PDLL
// patterns as individual C++ PDLPatternModules for direct use in native code,
// and also defines any native constraints whose bodies were defined in PDLL.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-26
```cpp
#include "mlir/Tools/PDLL/CodeGen/CPPGen.h"
#include "mlir/Dialect/PDL/IR/PDLOps.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/Tools/PDLL/AST/Nodes.h"
#include "mlir/Tools/PDLL/ODS/Operation.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringSet.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/FormatVariadic.h"
#include <optional>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/PDLL/CodeGen/CPPGen.h`, `mlir/Dialect/PDL/IR/PDLOps.h`, `mlir/IR/BuiltinOps.h`, `mlir/Tools/PDLL/AST/Nodes.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/PDLL/CodeGen/CPPGen.h`, `mlir/Dialect/PDL/IR/PDLOps.h`, `mlir/IR/BuiltinOps.h`, `mlir/Tools/PDLL/AST/Nodes.h`。

### Lines 27-33
```cpp
using namespace mlir;
using namespace mlir::pdll;

//===----------------------------------------------------------------------===//
// CodeGen
//===----------------------------------------------------------------------===//

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 34-41
```cpp
namespace {
class CodeGen {
public:
  CodeGen(raw_ostream &os) : os(os) {}

  /// Generate C++ code for the given PDL pattern module.
  void generate(const ast::Module &astModule, ModuleOp module);

```
- **EN**: Introduces declarations for `CodeGen`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `CodeGen` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 42-55
```cpp
private:
  void generate(pdl::PatternOp pattern, StringRef patternName,
                StringSet<> &nativeFunctions);

  /// Generate C++ code for all user defined constraints and rewrites with
  /// native code.
  void generateConstraintAndRewrites(const ast::Module &astModule,
                                     ModuleOp module,
                                     StringSet<> &nativeFunctions);
  void generate(const ast::UserConstraintDecl *decl,
                StringSet<> &nativeFunctions);
  void generate(const ast::UserRewriteDecl *decl, StringSet<> &nativeFunctions);
  void generateConstraintOrRewrite(const ast::CallableDecl *decl,
                                   bool isConstraint,
```
- **EN**: Implements logic around `generate`, `generateConstraintAndRewrites`, `generateConstraintOrRewrite`.
- **CN**: 围绕 `generate`、`generateConstraintAndRewrites`、`generateConstraintOrRewrite` 实现具体逻辑。

### Lines 56-63
```cpp
                                   StringSet<> &nativeFunctions);

  /// Return the native name for the type of the given type.
  StringRef getNativeTypeName(ast::Type type);

  /// Return the native name for the type of the given variable decl.
  StringRef getNativeTypeName(ast::VariableDecl *decl);

```
- **EN**: Implements logic around `getNativeTypeName`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getNativeTypeName` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 64-72
```cpp
  /// The stream to output to.
  raw_ostream &os;
};
} // namespace

void CodeGen::generate(const ast::Module &astModule, ModuleOp module) {
  SetVector<std::string, SmallVector<std::string>, StringSet<>> patternNames;
  StringSet<> nativeFunctions;

```
- **EN**: Implements logic around `generate`.
- **CN**: 围绕 `generate` 实现具体逻辑。

### Lines 73-86
```cpp
  // Generate code for any native functions within the module.
  generateConstraintAndRewrites(astModule, module, nativeFunctions);

  os << "namespace {\n";
  std::string basePatternName = "GeneratedPDLLPattern";
  int patternIndex = 0;
  for (pdl::PatternOp pattern : module.getOps<pdl::PatternOp>()) {
    // If the pattern has a name, use that. Otherwise, generate a unique name.
    if (std::optional<StringRef> patternName = pattern.getSymName()) {
      patternNames.insert(patternName->str());
    } else {
      std::string name;
      do {
        name = (basePatternName + Twine(patternIndex++)).str();
```
- **EN**: Implements logic around `generateConstraintAndRewrites`, `PatternOp>`, `getSymName`, `insert`, and 1 more symbols.
- **CN**: 围绕 `generateConstraintAndRewrites`、`PatternOp>`、`getSymName`、`insert` 等另外 1 个符号 实现具体逻辑。

### Lines 87-93
```cpp
      } while (!patternNames.insert(name));
    }

    generate(pattern, patternNames.back(), nativeFunctions);
  }
  os << "} // end namespace\n\n";

```
- **EN**: Implements logic around `insert`, `generate`.
- **CN**: 围绕 `insert`、`generate` 实现具体逻辑。

### Lines 94-103
```cpp
  // Emit function to add the generated matchers to the pattern list.
  os << "template <typename... ConfigsT>\n"
        "[[maybe_unused]] static void populateGeneratedPDLLPatterns("
        "::mlir::RewritePatternSet &patterns, ConfigsT &&...configs) {\n";
  for (const auto &name : patternNames)
    os << "  patterns.add<" << name
       << ">(patterns.getContext(), configs...);\n";
  os << "}\n";
}

```
- **EN**: Implements logic around `populateGeneratedPDLLPatterns`, `getContext`.
- **CN**: 围绕 `populateGeneratedPDLLPatterns`、`getContext` 实现具体逻辑。

### Lines 104-113
```cpp
void CodeGen::generate(pdl::PatternOp pattern, StringRef patternName,
                       StringSet<> &nativeFunctions) {
  const char *patternClassStartStr = R"(
struct {0} : ::mlir::PDLPatternModule {{
  template <typename... ConfigsT>
  {0}(::mlir::MLIRContext *context, ConfigsT &&...configs)
    : ::mlir::PDLPatternModule(::mlir::parseSourceString<::mlir::ModuleOp>(
)";
  os << llvm::formatv(patternClassStartStr, patternName);

```
- **EN**: Implements logic around `generate`, `PDLPatternModule`, `formatv`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `generate`、`PDLPatternModule`、`formatv` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 114-127
```cpp
  os << "R\"mlir(";
  pattern->print(os, OpPrintingFlags().enableDebugInfo());
  os << "\n    )mlir\", context), std::forward<ConfigsT>(configs)...) {\n";

  // Register any native functions used within the pattern.
  StringSet<> registeredNativeFunctions;
  auto checkRegisterNativeFn = [&](StringRef fnName, StringRef fnType) {
    if (!nativeFunctions.count(fnName) ||
        !registeredNativeFunctions.insert(fnName).second)
      return;
    os << "    register" << fnType << "Function(\"" << fnName << "\", "
       << fnName << "PDLFn);\n";
  };
  pattern.walk([&](Operation *op) {
```
- **EN**: Implements logic around `mlir`, `print`, `forward`, `count`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `mlir`、`print`、`forward`、`count` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 128-135
```cpp
    if (auto constraintOp = dyn_cast<pdl::ApplyNativeConstraintOp>(op))
      checkRegisterNativeFn(constraintOp.getName(), "Constraint");
    else if (auto rewriteOp = dyn_cast<pdl::ApplyNativeRewriteOp>(op))
      checkRegisterNativeFn(rewriteOp.getName(), "Rewrite");
  });
  os << "  }\n};\n\n";
}

```
- **EN**: Implements logic around `ApplyNativeConstraintOp>`, `checkRegisterNativeFn`, `ApplyNativeRewriteOp>`.
- **CN**: 围绕 `ApplyNativeConstraintOp>`、`checkRegisterNativeFn`、`ApplyNativeRewriteOp>` 实现具体逻辑。

### Lines 136-147
```cpp
void CodeGen::generateConstraintAndRewrites(const ast::Module &astModule,
                                            ModuleOp module,
                                            StringSet<> &nativeFunctions) {
  // First check to see which constraints and rewrites are actually referenced
  // in the module.
  StringSet<> usedFns;
  module.walk([&](Operation *op) {
    TypeSwitch<Operation *>(op)
        .Case<pdl::ApplyNativeConstraintOp, pdl::ApplyNativeRewriteOp>(
            [&](auto op) { usedFns.insert(op.getName()); });
  });

```
- **EN**: Implements logic around `generateConstraintAndRewrites`, `walk`, `ApplyNativeRewriteOp>`, `insert`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `generateConstraintAndRewrites`、`walk`、`ApplyNativeRewriteOp>`、`insert` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 148-160
```cpp
  for (const ast::Decl *decl : astModule.getChildren()) {
    TypeSwitch<const ast::Decl *>(decl)
        .Case<ast::UserConstraintDecl, ast::UserRewriteDecl>(
            [&](const auto *decl) {
              // We only generate code for inline native decls that have been
              // referenced.
              if (decl->getCodeBlock() &&
                  usedFns.contains(decl->getName().getName()))
                this->generate(decl, nativeFunctions);
            });
  }
}

```
- **EN**: Implements logic around `getChildren`, `UserRewriteDecl>`, `getCodeBlock`, `contains`, and 1 more symbols.
- **CN**: 围绕 `getChildren`、`UserRewriteDecl>`、`getCodeBlock`、`contains` 等另外 1 个符号 实现具体逻辑。

### Lines 161-172
```cpp
void CodeGen::generate(const ast::UserConstraintDecl *decl,
                       StringSet<> &nativeFunctions) {
  return generateConstraintOrRewrite(cast<ast::CallableDecl>(decl),
                                     /*isConstraint=*/true, nativeFunctions);
}

void CodeGen::generate(const ast::UserRewriteDecl *decl,
                       StringSet<> &nativeFunctions) {
  return generateConstraintOrRewrite(cast<ast::CallableDecl>(decl),
                                     /*isConstraint=*/false, nativeFunctions);
}

```
- **EN**: Implements logic around `generate`, `generateConstraintOrRewrite`.
- **CN**: 围绕 `generate`、`generateConstraintOrRewrite` 实现具体逻辑。

### Lines 173-186
```cpp
StringRef CodeGen::getNativeTypeName(ast::Type type) {
  return llvm::TypeSwitch<ast::Type, StringRef>(type)
      .Case([&](ast::AttributeType) { return "::mlir::Attribute"; })
      .Case([&](ast::OperationType opType) -> StringRef {
        // Use the derived Op class when available.
        if (const auto *odsOp = opType.getODSOperation())
          return odsOp->getNativeClassName();
        return "::mlir::Operation *";
      })
      .Case([&](ast::TypeType) { return "::mlir::Type"; })
      .Case([&](ast::ValueType) { return "::mlir::Value"; })
      .Case([&](ast::TypeRangeType) { return "::mlir::TypeRange"; })
      .Case([&](ast::ValueRangeType) { return "::mlir::ValueRange"; });
}
```
- **EN**: Implements logic around `getNativeTypeName`, `StringRef>`, `Case`, `getODSOperation`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getNativeTypeName`、`StringRef>`、`Case`、`getODSOperation` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 187-197
```cpp

StringRef CodeGen::getNativeTypeName(ast::VariableDecl *decl) {
  // Try to extract a type name from the variable's constraints.
  for (ast::ConstraintRef &cst : decl->getConstraints()) {
    if (auto *userCst = dyn_cast<ast::UserConstraintDecl>(cst.constraint)) {
      if (std::optional<StringRef> name = userCst->getNativeInputType(0))
        return *name;
      return getNativeTypeName(userCst->getInputs()[0]);
    }
  }

```
- **EN**: Implements logic around `getNativeTypeName`, `getConstraints`, `UserConstraintDecl>`, `getNativeInputType`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getNativeTypeName`、`getConstraints`、`UserConstraintDecl>`、`getNativeInputType` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 198-207
```cpp
  // Otherwise, use the type of the variable.
  return getNativeTypeName(decl->getType());
}

void CodeGen::generateConstraintOrRewrite(const ast::CallableDecl *decl,
                                          bool isConstraint,
                                          StringSet<> &nativeFunctions) {
  StringRef name = decl->getName()->getName();
  nativeFunctions.insert(name);

```
- **EN**: Implements logic around `getNativeTypeName`, `generateConstraintOrRewrite`, `getName`, `insert`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getNativeTypeName`、`generateConstraintOrRewrite`、`getName`、`insert` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 208-221
```cpp
  os << "static ";

  // TODO: Work out a proper modeling for "optionality".

  // Emit the result type.
  // If this is a constraint, we always return a LogicalResult.
  // TODO: This will need to change if we allow Constraints to return values as
  // well.
  if (isConstraint) {
    os << "::llvm::LogicalResult";
  } else {
    // Otherwise, generate a type based on the results of the callable.
    // If the callable has explicit results, use those to build the result.
    // Otherwise, use the type of the callable.
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 222-235
```cpp
    ArrayRef<ast::VariableDecl *> results = decl->getResults();
    if (results.empty()) {
      os << "void";
    } else if (results.size() == 1) {
      os << getNativeTypeName(results[0]);
    } else {
      os << "std::tuple<";
      llvm::interleaveComma(results, os, [&](ast::VariableDecl *result) {
        os << getNativeTypeName(result);
      });
      os << ">";
    }
  }

```
- **EN**: Implements logic around `getResults`, `empty`, `size`, `getNativeTypeName`, and 1 more symbols.
- **CN**: 围绕 `getResults`、`empty`、`size`、`getNativeTypeName` 等另外 1 个符号 实现具体逻辑。

### Lines 236-246
```cpp
  os << " " << name << "PDLFn(::mlir::PatternRewriter &rewriter";
  if (!decl->getInputs().empty()) {
    os << ", ";
    llvm::interleaveComma(decl->getInputs(), os, [&](ast::VariableDecl *input) {
      os << getNativeTypeName(input) << " " << input->getName().getName();
    });
  }
  os << ") {\n";
  os << "  " << decl->getCodeBlock()->trim() << "\n}\n\n";
}

```
- **EN**: Implements logic around `PDLFn`, `getInputs`, `interleaveComma`, `getNativeTypeName`, and 1 more symbols; this block relies on MLIR pattern rewriting infrastructure.
- **CN**: 围绕 `PDLFn`、`getInputs`、`interleaveComma`、`getNativeTypeName` 等另外 1 个符号 实现具体逻辑；该代码块依赖 MLIR 模式重写基础设施。

### Lines 247-255
```cpp
//===----------------------------------------------------------------------===//
// CPPGen
//===----------------------------------------------------------------------===//

void mlir::pdll::codegenPDLLToCPP(const ast::Module &astModule, ModuleOp module,
                                  raw_ostream &os) {
  CodeGen codegen(os);
  codegen.generate(astModule, module);
}
```
- **EN**: Implements logic around `codegenPDLLToCPP`, `codegen`, `generate`.
- **CN**: 围绕 `codegenPDLLToCPP`、`codegen`、`generate` 实现具体逻辑。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Pattern rewriting / 模式重写**:
  - **EN**: The implementation uses rewrite patterns to match and transform IR.
  - **CN**: 该实现使用重写模式来匹配并变换 IR。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/PDLL/CodeGen/CPPGen.h`, `mlir/Dialect/PDL/IR/PDLOps.h`, `mlir/IR/BuiltinOps.h`, `mlir/Tools/PDLL/AST/Nodes.h`, `mlir/Tools/PDLL/ODS/Operation.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringSet.h`, `llvm/ADT/TypeSwitch.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/FormatVariadic.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: tooling support declarations / 工具支持声明 (3), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (3), LLVM support-library helpers / LLVM Support 库辅助工具 (2), core MLIR IR abstractions / 核心 MLIR IR 抽象 (1)
