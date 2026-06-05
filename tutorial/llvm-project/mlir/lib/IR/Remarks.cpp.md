# Remarks.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/IR/Remarks.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLIR core IR objects, printing/parsing support, and builtin type or attribute behavior.
  - **CN**: 实现 MLIR 核心 IR 对象、打印/解析支持以及内建类型或属性行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- Remarks.cpp - MLIR Remarks -----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/IR/Remarks.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/IR/Remarks.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/IR/Remarks.h`。

### Lines 11-23
```cpp
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/Diagnostics.h"
#include "mlir/IR/Value.h"

#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringRef.h"

using namespace mlir::remark::detail;
using namespace mlir::remark;
//------------------------------------------------------------------------------
// Remark
//------------------------------------------------------------------------------

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/IR/BuiltinAttributes.h`, `mlir/IR/Diagnostics.h`, `mlir/IR/Value.h`, `llvm/ADT/StringExtras.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/IR/BuiltinAttributes.h`, `mlir/IR/Diagnostics.h`, `mlir/IR/Value.h`, `llvm/ADT/StringExtras.h`。

### Lines 24-33
```cpp
Remark::Arg::Arg(llvm::StringRef k, Value v) : key(k) {
  llvm::raw_string_ostream os(val);
  os << v;
}

Remark::Arg::Arg(llvm::StringRef k, Type t) : key(k) {
  llvm::raw_string_ostream os(val);
  os << t;
}

```
- **EN**: Implements logic around `Arg`, `os`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `Arg`、`os` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 34-46
```cpp
Remark::Arg::Arg(llvm::StringRef k, Attribute a) : key(k), attr(a) {
  llvm::raw_string_ostream os(val);
  os << a;
}

void Remark::insert(llvm::StringRef s) { args.emplace_back(s); }
void Remark::insert(Arg a) { args.push_back(std::move(a)); }

// Simple helper to print key=val list (sorted).
static void printArgs(llvm::raw_ostream &os, llvm::ArrayRef<Remark::Arg> args) {
  if (args.empty())
    return;

```
- **EN**: Implements logic around `Arg`, `os`, `insert`, `printArgs`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `Arg`、`os`、`insert`、`printArgs` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 47-63
```cpp
  llvm::SmallVector<Remark::Arg, 8> sorted(args.begin(), args.end());
  llvm::sort(sorted, [](const Remark::Arg &a, const Remark::Arg &b) {
    return a.key < b.key;
  });

  for (size_t i = 0; i < sorted.size(); ++i) {
    const auto &a = sorted[i];
    os << a.key << "=";

    llvm::StringRef val(a.val);
    bool needsQuote = val.contains(' ') || val.contains(',') ||
                      val.contains('{') || val.contains('}');
    if (needsQuote)
      os << '"' << val << '"';
    else
      os << val;

```
- **EN**: Implements logic around `sorted`, `sort`, `size`, `val`, and 1 more symbols.
- **CN**: 围绕 `sorted`、`sort`、`size`、`val` 等另外 1 个符号 实现具体逻辑。

### Lines 64-73
```cpp
    if (i + 1 < sorted.size())
      os << ", ";
  }
}

/// Print the remark to the given output stream.
/// Example output:
// clang-format off
/// [Missed] Category: Loop | Pass:Unroller |  Function=main | Reason="tripCount=4 < threshold=256"
/// [Failure] LoopOptimizer | Reason="failed due to unsupported pattern"
```
- **EN**: Implements logic around `size`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `size` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 74-87
```cpp
// clang-format on
void Remark::print(llvm::raw_ostream &os, bool printLocation) const {
  // Header: [Type] pass:remarkName
  StringRef type = getRemarkTypeString();
  StringRef categoryName = getCombinedCategoryName();
  StringRef name = remarkName;

  os << '[' << type << "] ";
  os << name << " | ";
  if (!categoryName.empty())
    os << "Category:" << categoryName << " | ";
  if (!functionName.empty())
    os << "Function=" << getFunction() << " | ";

```
- **EN**: Implements logic around `print`, `getRemarkTypeString`, `getCombinedCategoryName`, `empty`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `print`、`getRemarkTypeString`、`getCombinedCategoryName`、`empty` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 88-97
```cpp
  if (printLocation) {
    if (auto flc = mlir::dyn_cast<mlir::FileLineColLoc>(getLocation())) {
      os << " @" << flc.getFilename() << ":" << flc.getLine() << ":"
         << flc.getColumn();
    }
  }

  printArgs(os, getArgs());
}

```
- **EN**: Implements logic around `FileLineColLoc>`, `getFilename`, `getColumn`, `printArgs`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `FileLineColLoc>`、`getFilename`、`getColumn`、`printArgs` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 98-117
```cpp
std::string Remark::getMsg() const {
  std::string s;
  llvm::raw_string_ostream os(s);
  print(os);
  os.flush();
  return s;
}

llvm::StringRef Remark::getRemarkTypeString() const {
  switch (remarkKind) {
  case RemarkKind::RemarkUnknown:
    return "Unknown";
  case RemarkKind::RemarkPassed:
    return "Passed";
  case RemarkKind::RemarkMissed:
    return "Missed";
  case RemarkKind::RemarkFailure:
    return "Failure";
  case RemarkKind::RemarkAnalysis:
    return "Analysis";
```
- **EN**: Implements logic around `getMsg`, `os`, `print`, `flush`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getMsg`、`os`、`print`、`flush` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 118-137
```cpp
  }
  llvm_unreachable("Unknown remark kind");
}

llvm::remarks::Type Remark::getRemarkType() const {
  switch (remarkKind) {
  case RemarkKind::RemarkUnknown:
    return llvm::remarks::Type::Unknown;
  case RemarkKind::RemarkPassed:
    return llvm::remarks::Type::Passed;
  case RemarkKind::RemarkMissed:
    return llvm::remarks::Type::Missed;
  case RemarkKind::RemarkFailure:
    return llvm::remarks::Type::Failure;
  case RemarkKind::RemarkAnalysis:
    return llvm::remarks::Type::Analysis;
  }
  llvm_unreachable("Unknown remark kind");
}

```
- **EN**: Implements logic around `llvm_unreachable`, `getRemarkType`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `llvm_unreachable`、`getRemarkType` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 138-157
```cpp
llvm::remarks::Remark Remark::generateRemark() const {
  auto locLambda = [&]() -> llvm::remarks::RemarkLocation {
    if (auto flc = dyn_cast<FileLineColLoc>(getLocation()))
      return {flc.getFilename(), flc.getLine(), flc.getColumn()};
    return {"<unknown file>", 0, 0};
  };

  llvm::remarks::Remark r; // The result.
  r.RemarkType = getRemarkType();
  r.RemarkName = getRemarkName();
  // MLIR does not use passes; instead, it has categories and sub-categories.
  r.PassName = getCombinedCategoryName();
  r.FunctionName = getFunction();
  r.Loc = locLambda();
  // Add all args (includes RemarkId and RelatedTo if they were added).
  for (const Remark::Arg &arg : getArgs()) {
    r.Args.emplace_back();
    r.Args.back().Key = arg.key;
    r.Args.back().Val = arg.val;
  }
```
- **EN**: Implements logic around `generateRemark`, `dyn_cast`, `getFilename`, `getRemarkType`, and 7 more symbols.
- **CN**: 围绕 `generateRemark`、`dyn_cast`、`getFilename`、`getRemarkType` 等另外 7 个符号 实现具体逻辑。

### Lines 158-170
```cpp
  return r;
}

//===----------------------------------------------------------------------===//
// InFlightRemark
//===----------------------------------------------------------------------===//

InFlightRemark::~InFlightRemark() {
  if (remark && owner)
    owner->report(std::move(*remark));
  owner = nullptr;
}

```
- **EN**: Implements logic around `~InFlightRemark`, `report`.
- **CN**: 围绕 `~InFlightRemark`、`report` 实现具体逻辑。

### Lines 171-183
```cpp
//===----------------------------------------------------------------------===//
// Remark Engine
//===----------------------------------------------------------------------===//

template <typename RemarkT>
InFlightRemark RemarkEngine::makeRemark(Location loc, RemarkOpts opts) {
  static_assert(std::is_base_of_v<Remark, RemarkT>,
                "RemarkT must derive from Remark");
  auto remark = std::make_unique<RemarkT>(loc, opts);
  remark->setId(generateRemarkId());
  return InFlightRemark(*this, std::move(remark));
}

```
- **EN**: Implements logic around `makeRemark`, `static_assert`, `make_unique`, `setId`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `makeRemark`、`static_assert`、`make_unique`、`setId` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 184-195
```cpp
template <typename RemarkT>
InFlightRemark
RemarkEngine::emitIfEnabled(Location loc, RemarkOpts opts,
                            bool (RemarkEngine::*isEnabled)(StringRef) const) {
  return (this->*isEnabled)(opts.categoryName) ? makeRemark<RemarkT>(loc, opts)
                                               : InFlightRemark{};
}

bool RemarkEngine::isMissedOptRemarkEnabled(StringRef categoryName) const {
  return missFilter && missFilter->match(categoryName);
}

```
- **EN**: Implements logic around `emitIfEnabled`, `bool`, `makeRemark`, `isMissedOptRemarkEnabled`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `emitIfEnabled`、`bool`、`makeRemark`、`isMissedOptRemarkEnabled` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 196-207
```cpp
bool RemarkEngine::isPassedOptRemarkEnabled(StringRef categoryName) const {
  return passedFilter && passedFilter->match(categoryName);
}

bool RemarkEngine::isAnalysisOptRemarkEnabled(StringRef categoryName) const {
  return analysisFilter && analysisFilter->match(categoryName);
}

bool RemarkEngine::isFailedOptRemarkEnabled(StringRef categoryName) const {
  return failedFilter && failedFilter->match(categoryName);
}

```
- **EN**: Implements logic around `isPassedOptRemarkEnabled`, `match`, `isAnalysisOptRemarkEnabled`, `isFailedOptRemarkEnabled`.
- **CN**: 围绕 `isPassedOptRemarkEnabled`、`match`、`isAnalysisOptRemarkEnabled`、`isFailedOptRemarkEnabled` 实现具体逻辑。

### Lines 208-219
```cpp
InFlightRemark RemarkEngine::emitOptimizationRemark(Location loc,
                                                    RemarkOpts opts) {
  return emitIfEnabled<OptRemarkPass>(loc, opts,
                                      &RemarkEngine::isPassedOptRemarkEnabled);
}

InFlightRemark RemarkEngine::emitOptimizationRemarkMiss(Location loc,
                                                        RemarkOpts opts) {
  return emitIfEnabled<OptRemarkMissed>(
      loc, opts, &RemarkEngine::isMissedOptRemarkEnabled);
}

```
- **EN**: Implements logic around `emitOptimizationRemark`, `emitIfEnabled`, `emitOptimizationRemarkMiss`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `emitOptimizationRemark`、`emitIfEnabled`、`emitOptimizationRemarkMiss` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 220-231
```cpp
InFlightRemark RemarkEngine::emitOptimizationRemarkFailure(Location loc,
                                                           RemarkOpts opts) {
  return emitIfEnabled<OptRemarkFailure>(
      loc, opts, &RemarkEngine::isFailedOptRemarkEnabled);
}

InFlightRemark RemarkEngine::emitOptimizationRemarkAnalysis(Location loc,
                                                            RemarkOpts opts) {
  return emitIfEnabled<OptRemarkAnalysis>(
      loc, opts, &RemarkEngine::isAnalysisOptRemarkEnabled);
}

```
- **EN**: Implements logic around `emitOptimizationRemarkFailure`, `emitIfEnabled`, `emitOptimizationRemarkAnalysis`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `emitOptimizationRemarkFailure`、`emitIfEnabled`、`emitOptimizationRemarkAnalysis` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 232-241
```cpp
//===----------------------------------------------------------------------===//
// RemarkEngine
//===----------------------------------------------------------------------===//

void RemarkEngine::reportImpl(const Remark &remark) {
  // Stream the remark
  if (remarkStreamer) {
    remarkStreamer->streamOptimizationRemark(remark);
  }

```
- **EN**: Implements logic around `reportImpl`, `streamOptimizationRemark`.
- **CN**: 围绕 `reportImpl`、`streamOptimizationRemark` 实现具体逻辑。

### Lines 242-251
```cpp
  // Print using MLIR's diagnostic
  if (printAsEmitRemarks)
    emitRemark(remark.getLocation(), remark.getMsg());
}

void RemarkEngine::report(const Remark &&remark) {
  if (remarkEmittingPolicy)
    remarkEmittingPolicy->reportRemark(remark);
}

```
- **EN**: Implements logic around `emitRemark`, `report`, `reportRemark`; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `emitRemark`、`report`、`reportRemark` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 252-265
```cpp
RemarkEngine::~RemarkEngine() {
  if (remarkEmittingPolicy)
    remarkEmittingPolicy->finalize();

  if (remarkStreamer)
    remarkStreamer->finalize();
}

llvm::LogicalResult RemarkEngine::initialize(
    std::unique_ptr<MLIRRemarkStreamerBase> streamer,
    std::unique_ptr<RemarkEmittingPolicyBase> remarkEmittingPolicy,
    std::string *errMsg) {
  remarkStreamer = std::move(streamer);

```
- **EN**: Implements logic around `~RemarkEngine`, `finalize`, `initialize`, `move`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `~RemarkEngine`、`finalize`、`initialize`、`move` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 266-278
```cpp
  auto reportFunc = llvm::bind_front<&RemarkEngine::reportImpl>(this);
  remarkEmittingPolicy->initialize(ReportFn(std::move(reportFunc)));

  this->remarkEmittingPolicy = std::move(remarkEmittingPolicy);
  return success();
}

/// Returns true if filter is already anchored like ^...$
static bool isAnchored(llvm::StringRef s) {
  s = s.trim();
  return s.starts_with("^") && s.ends_with("$"); // note: startswith/endswith
}

```
- **EN**: Implements logic around `reportImpl>`, `initialize`, `move`, `success`, and 3 more symbols.
- **CN**: 围绕 `reportImpl>`、`initialize`、`move`、`success` 等另外 3 个符号 实现具体逻辑。

### Lines 279-288
```cpp
/// Anchor the entire pattern so it matches the whole string.
static std::string anchorWhole(llvm::StringRef filter) {
  if (isAnchored(filter))
    return filter.str();
  return (llvm::Twine("^(") + filter + ")$").str();
}

/// Build a combined filter from cats.all and a category-specific pattern.
/// If neither is present, return std::nullopt. Otherwise "(all|specific)"
/// and anchor once. Also validate before returning.
```
- **EN**: Implements logic around `anchorWhole`, `isAnchored`, `str`, `Twine`.
- **CN**: 围绕 `anchorWhole`、`isAnchored`、`str`、`Twine` 实现具体逻辑。

### Lines 289-300
```cpp
static std::optional<llvm::Regex>
buildFilter(const mlir::remark::RemarkCategories &cats,
            const std::optional<std::string> &specific) {
  llvm::SmallVector<llvm::StringRef, 2> parts;
  if (cats.all && !cats.all->empty())
    parts.emplace_back(*cats.all);
  if (specific && !specific->empty())
    parts.emplace_back(*specific);

  if (parts.empty())
    return std::nullopt;

```
- **EN**: Implements logic around `buildFilter`, `empty`, `emplace_back`.
- **CN**: 围绕 `buildFilter`、`empty`、`emplace_back` 实现具体逻辑。

### Lines 301-311
```cpp
  std::string joined = llvm::join(parts, "|");
  std::string anchored = anchorWhole(joined);

  llvm::Regex rx(anchored);
  std::string err;
  if (!rx.isValid(err))
    return std::nullopt;

  return std::make_optional<llvm::Regex>(std::move(rx));
}

```
- **EN**: Implements logic around `join`, `anchorWhole`, `rx`, `isValid`, and 1 more symbols.
- **CN**: 围绕 `join`、`anchorWhole`、`rx`、`isValid` 等另外 1 个符号 实现具体逻辑。

### Lines 312-324
```cpp
RemarkEngine::RemarkEngine(bool printAsEmitRemarks,
                           const RemarkCategories &cats)
    : printAsEmitRemarks(printAsEmitRemarks) {
  if (cats.passed)
    passedFilter = buildFilter(cats, cats.passed);
  if (cats.missed)
    missFilter = buildFilter(cats, cats.missed);
  if (cats.analysis)
    analysisFilter = buildFilter(cats, cats.analysis);
  if (cats.failed)
    failedFilter = buildFilter(cats, cats.failed);
}

```
- **EN**: Implements logic around `RemarkEngine`, `printAsEmitRemarks`, `buildFilter`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `RemarkEngine`、`printAsEmitRemarks`、`buildFilter` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 325-339
```cpp
llvm::LogicalResult mlir::remark::enableOptimizationRemarks(
    MLIRContext &ctx, std::unique_ptr<detail::MLIRRemarkStreamerBase> streamer,
    std::unique_ptr<detail::RemarkEmittingPolicyBase> remarkEmittingPolicy,
    const RemarkCategories &cats, bool printAsEmitRemarks) {
  auto engine =
      std::make_unique<detail::RemarkEngine>(printAsEmitRemarks, cats);

  std::string errMsg;
  if (failed(engine->initialize(std::move(streamer),
                                std::move(remarkEmittingPolicy), &errMsg))) {
    llvm::report_fatal_error(
        llvm::Twine("Failed to initialize remark engine. Error: ") + errMsg);
  }
  ctx.setRemarkEngine(std::move(engine));

```
- **EN**: Implements logic around `enableOptimizationRemarks`, `RemarkEngine>`, `failed`, `move`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `enableOptimizationRemarks`、`RemarkEngine>`、`failed`、`move` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 340-350
```cpp
  return success();
}

//===----------------------------------------------------------------------===//
// Remark emitting policies
//===----------------------------------------------------------------------===//

namespace mlir::remark {
RemarkEmittingPolicyAll::RemarkEmittingPolicyAll() = default;
RemarkEmittingPolicyFinal::RemarkEmittingPolicyFinal() = default;

```
- **EN**: Introduces declarations for `mlir::remark`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `mlir::remark` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 351-364
```cpp
void RemarkEmittingPolicyFinal::finalize() {
  assert(reportImpl && "reportImpl is not set");

  // Build ID -> Remark* lookup for resolving related remark references.
  llvm::DenseMap<uint64_t, const detail::Remark *> idMap;
  llvm::DenseSet<uint64_t> childIds; // IDs referenced as children

  for (const auto &remark : postponedRemarks) {
    if (remark.getId())
      idMap[remark.getId().getValue()] = &remark;
    for (auto relId : remark.getRelatedRemarkIds())
      childIds.insert(relId.getValue());
  }

```
- **EN**: Implements logic around `finalize`, `assert`, `getId`, `getRelatedRemarkIds`, and 1 more symbols.
- **CN**: 围绕 `finalize`、`assert`、`getId`、`getRelatedRemarkIds` 等另外 1 个符号 实现具体逻辑。

### Lines 365-374
```cpp
  // Emit remarks with related remarks grouped after their parents.
  // Parent remarks are emitted first, followed by their related (child)
  // remarks. Child-only remarks are skipped at the top level to avoid
  // duplication.
  for (const auto &remark : postponedRemarks) {
    if (remark.getId() && childIds.count(remark.getId().getValue()))
      continue; // will be printed grouped under its parent

    reportImpl(remark);

```
- **EN**: Implements logic around `getId`, `reportImpl`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getId`、`reportImpl` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 375-383
```cpp
    // Emit related remarks immediately after the parent.
    for (auto relId : remark.getRelatedRemarkIds()) {
      if (const auto *related = idMap.lookup(relId.getValue()))
        reportImpl(*related);
    }
  }
}

} // namespace mlir::remark
```
- **EN**: Implements logic around `getRelatedRemarkIds`, `lookup`, `reportImpl`.
- **CN**: 围绕 `getRelatedRemarkIds`、`lookup`、`reportImpl` 实现具体逻辑。

## Key Concepts / 关键概念

- **Core IR modeling / 核心 IR 建模**:
  - **EN**: Defines operations, regions, blocks, attributes, types, and the assembly form that ties them together.
  - **CN**: 定义操作、区域、块、属性、类型，以及将它们串联起来的汇编表示。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/IR/Remarks.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/Diagnostics.h`, `mlir/IR/Value.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`
- **Subsystem categories / 子系统类别**: core MLIR IR abstractions / 核心 MLIR IR 抽象 (4), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (2)
