# Merger.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SparseTensor/Utils/Merger.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements sparse tensor helper utilities and analysis support.
- **Purpose (CN)**: 实现稀疏张量辅助工具与分析支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===- Merger.cpp - Implementation of iteration lattices ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/SparseTensor/Utils/Merger.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Complex/IR/Complex.h"
#include "mlir/Dialect/Math/IR/Math.h"
#include "mlir/Dialect/SparseTensor/IR/SparseTensor.h"

#include "mlir/IR/Operation.h"
#include "llvm/Support/Debug.h"
#include <optional>

namespace mlir {
namespace sparse_tensor {

enum class ExpArity {
  kNullary,
  kUnary,
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes "mlir/Dialect/SparseTensor/Utils/Merger.h" to access dialect-specific IR, transforms, or shared utilities.
  **L9 CN**: 引入 "mlir/Dialect/SparseTensor/Utils/Merger.h" 以使用方言专用 IR、变换或共享工具。
- **L10 EN**: Includes "mlir/Dialect/Arith/IR/Arith.h" to access dialect-specific IR, transforms, or shared utilities.
  **L10 CN**: 引入 "mlir/Dialect/Arith/IR/Arith.h" 以使用方言专用 IR、变换或共享工具。
- **L11 EN**: Includes "mlir/Dialect/Complex/IR/Complex.h" to access dialect-specific IR, transforms, or shared utilities.
  **L11 CN**: 引入 "mlir/Dialect/Complex/IR/Complex.h" 以使用方言专用 IR、变换或共享工具。
- **L12 EN**: Includes "mlir/Dialect/Math/IR/Math.h" to access dialect-specific IR, transforms, or shared utilities.
  **L12 CN**: 引入 "mlir/Dialect/Math/IR/Math.h" 以使用方言专用 IR、变换或共享工具。
- **L13 EN**: Includes "mlir/Dialect/SparseTensor/IR/SparseTensor.h" to access dialect-specific IR, transforms, or shared utilities.
  **L13 CN**: 引入 "mlir/Dialect/SparseTensor/IR/SparseTensor.h" 以使用方言专用 IR、变换或共享工具。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Includes "mlir/IR/Operation.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L15 CN**: 引入 "mlir/IR/Operation.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L16 EN**: Includes "llvm/Support/Debug.h" to access LLVM support-library facilities such as diagnostics and casting.
  **L16 CN**: 引入 "llvm/Support/Debug.h" 以使用LLVM 支持库设施，如诊断与类型转换。
- **L17 EN**: Includes <optional> to access supporting declarations used by the current translation unit.
  **L17 CN**: 引入 <optional> 以使用当前编译单元使用的辅助声明。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Opens namespace scope `mlir`.
  **L19 CN**: 打开命名空间作用域 `mlir`。
- **L20 EN**: Opens namespace scope `sparse_tensor`.
  **L20 CN**: 打开命名空间作用域 `sparse_tensor`。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Declares enum `class`.
  **L22 CN**: 声明 enum `class`。
- **L23 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `kNullary,`.
  **L23 CN**: 继续一个多行参数列表、初始化器或聚合项：`kNullary,`。
- **L24 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `kUnary,`.
  **L24 CN**: 继续一个多行参数列表、初始化器或聚合项：`kUnary,`。

### Lines 25-48

````cpp
  kBinary,
};

static ExpArity getExpArity(TensorExp::Kind k) {
  switch (k) {
  // Leaf.
  case TensorExp::Kind::kTensor:
  case TensorExp::Kind::kInvariant:
  case TensorExp::Kind::kLoopVar:
  case TensorExp::Kind::kSynZero:
    return ExpArity::kNullary;
  case TensorExp::Kind::kAbsF:
  case TensorExp::Kind::kAbsC:
  case TensorExp::Kind::kAbsI:
  case TensorExp::Kind::kCeilF:
  case TensorExp::Kind::kFloorF:
  case TensorExp::Kind::kSqrtF:
  case TensorExp::Kind::kSqrtC:
  case TensorExp::Kind::kExpm1F:
  case TensorExp::Kind::kExpm1C:
  case TensorExp::Kind::kLog1pF:
  case TensorExp::Kind::kLog1pC:
  case TensorExp::Kind::kRelu:
  case TensorExp::Kind::kSinF:
````
- **L25 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `kBinary,`.
  **L25 CN**: 继续一个多行参数列表、初始化器或聚合项：`kBinary,`。
- **L26 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L26 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Starts a function, method, lambda, or structured scope: `static ExpArity getExpArity(TensorExp::Kind k) {`.
  **L28 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static ExpArity getExpArity(TensorExp::Kind k) {`。
- **L29 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L29 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L30 EN**: Comment explains nearby logic, invariants, or intent: `Leaf.`.
  **L30 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Leaf.`。
- **L31 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTensor:`.
  **L31 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTensor:`。
- **L32 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kInvariant:`.
  **L32 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kInvariant:`。
- **L33 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLoopVar:`.
  **L33 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLoopVar:`。
- **L34 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSynZero:`.
  **L34 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSynZero:`。
- **L35 EN**: Returns from the current function with `ExpArity::kNullary`.
  **L35 CN**: 以 `ExpArity::kNullary` 从当前函数返回。
- **L36 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsF:`.
  **L36 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsF:`。
- **L37 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsC:`.
  **L37 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsC:`。
- **L38 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsI:`.
  **L38 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsI:`。
- **L39 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCeilF:`.
  **L39 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCeilF:`。
- **L40 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kFloorF:`.
  **L40 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kFloorF:`。
- **L41 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtF:`.
  **L41 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtF:`。
- **L42 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtC:`.
  **L42 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtC:`。
- **L43 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1F:`.
  **L43 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1F:`。
- **L44 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1C:`.
  **L44 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1C:`。
- **L45 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pF:`.
  **L45 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pF:`。
- **L46 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pC:`.
  **L46 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pC:`。
- **L47 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kRelu:`.
  **L47 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kRelu:`。
- **L48 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinF:`.
  **L48 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinF:`。

### Lines 49-72

````cpp
  case TensorExp::Kind::kSinC:
  case TensorExp::Kind::kTanhF:
  case TensorExp::Kind::kTanhC:
  case TensorExp::Kind::kTruncF:
  case TensorExp::Kind::kExtF:
  case TensorExp::Kind::kCastFS:
  case TensorExp::Kind::kCastFU:
  case TensorExp::Kind::kCastSF:
  case TensorExp::Kind::kCastUF:
  case TensorExp::Kind::kCastS:
  case TensorExp::Kind::kCastU:
  case TensorExp::Kind::kCastIdx:
  case TensorExp::Kind::kTruncI:
  case TensorExp::Kind::kCIm:
  case TensorExp::Kind::kCRe:
  case TensorExp::Kind::kBitCast:
  case TensorExp::Kind::kBinaryBranch:
  case TensorExp::Kind::kUnary:
  case TensorExp::Kind::kSelect:
  case TensorExp::Kind::kNegF:
  case TensorExp::Kind::kNegC:
  case TensorExp::Kind::kNegI:
    return ExpArity::kUnary;
  // Binary operations.
````
- **L49 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinC:`.
  **L49 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinC:`。
- **L50 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhF:`.
  **L50 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhF:`。
- **L51 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhC:`.
  **L51 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhC:`。
- **L52 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncF:`.
  **L52 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncF:`。
- **L53 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExtF:`.
  **L53 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExtF:`。
- **L54 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFS:`.
  **L54 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFS:`。
- **L55 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFU:`.
  **L55 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFU:`。
- **L56 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastSF:`.
  **L56 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastSF:`。
- **L57 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastUF:`.
  **L57 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastUF:`。
- **L58 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastS:`.
  **L58 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastS:`。
- **L59 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastU:`.
  **L59 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastU:`。
- **L60 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastIdx:`.
  **L60 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastIdx:`。
- **L61 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncI:`.
  **L61 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncI:`。
- **L62 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCIm:`.
  **L62 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCIm:`。
- **L63 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCRe:`.
  **L63 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCRe:`。
- **L64 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBitCast:`.
  **L64 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBitCast:`。
- **L65 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinaryBranch:`.
  **L65 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinaryBranch:`。
- **L66 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kUnary:`.
  **L66 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kUnary:`。
- **L67 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSelect:`.
  **L67 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSelect:`。
- **L68 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegF:`.
  **L68 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegF:`。
- **L69 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegC:`.
  **L69 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegC:`。
- **L70 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegI:`.
  **L70 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegI:`。
- **L71 EN**: Returns from the current function with `ExpArity::kUnary`.
  **L71 CN**: 以 `ExpArity::kUnary` 从当前函数返回。
- **L72 EN**: Comment explains nearby logic, invariants, or intent: `Binary operations.`.
  **L72 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Binary operations.`。

### Lines 73-96

````cpp
  case TensorExp::Kind::kDivF:
  case TensorExp::Kind::kDivC:
  case TensorExp::Kind::kDivS:
  case TensorExp::Kind::kDivU:
  case TensorExp::Kind::kShrS:
  case TensorExp::Kind::kShrU:
  case TensorExp::Kind::kShlI:
  case TensorExp::Kind::kMulF:
  case TensorExp::Kind::kMulC:
  case TensorExp::Kind::kMulI:
  case TensorExp::Kind::kAndI:
  case TensorExp::Kind::kAddF:
  case TensorExp::Kind::kAddC:
  case TensorExp::Kind::kAddI:
  case TensorExp::Kind::kOrI:
  case TensorExp::Kind::kXorI:
  case TensorExp::Kind::kBinary:
  case TensorExp::Kind::kReduce:
  case TensorExp::Kind::kSubF:
  case TensorExp::Kind::kSubC:
  case TensorExp::Kind::kSubI:
  case TensorExp::Kind::kCmpF:
  case TensorExp::Kind::kCmpI:
  case TensorExp::Kind::kDenseOp: // kDenseOp can *at most* have two operands
````
- **L73 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivF:`.
  **L73 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivF:`。
- **L74 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivC:`.
  **L74 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivC:`。
- **L75 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivS:`.
  **L75 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivS:`。
- **L76 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivU:`.
  **L76 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivU:`。
- **L77 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrS:`.
  **L77 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrS:`。
- **L78 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrU:`.
  **L78 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrU:`。
- **L79 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShlI:`.
  **L79 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShlI:`。
- **L80 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulF:`.
  **L80 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulF:`。
- **L81 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulC:`.
  **L81 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulC:`。
- **L82 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulI:`.
  **L82 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulI:`。
- **L83 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAndI:`.
  **L83 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAndI:`。
- **L84 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddF:`.
  **L84 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddF:`。
- **L85 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddC:`.
  **L85 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddC:`。
- **L86 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddI:`.
  **L86 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddI:`。
- **L87 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kOrI:`.
  **L87 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kOrI:`。
- **L88 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kXorI:`.
  **L88 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kXorI:`。
- **L89 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinary:`.
  **L89 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinary:`。
- **L90 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kReduce:`.
  **L90 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kReduce:`。
- **L91 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubF:`.
  **L91 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubF:`。
- **L92 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubC:`.
  **L92 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubC:`。
- **L93 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubI:`.
  **L93 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubI:`。
- **L94 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpF:`.
  **L94 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpF:`。
- **L95 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpI:`.
  **L95 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpI:`。
- **L96 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDenseOp: // kDenseOp can *at most* have two operands`.
  **L96 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDenseOp: // kDenseOp can *at most* have two operands`。

### Lines 97-120

````cpp
    return ExpArity::kBinary;
  }
  llvm_unreachable("unexpected kind");
}

//===----------------------------------------------------------------------===//
// Constructors.
//===----------------------------------------------------------------------===//

TensorExp::TensorExp(TensorExp::Kind k, unsigned x, ExprId y, Value v,
                     Operation *o, Attribute a)
    : kind(k), val(v), op(o), attr(a) {
  switch (kind) {
  // Leaf.
  case TensorExp::Kind::kTensor:
    assert(x != detail::kInvalidId && y == detail::kInvalidId && !v && !o);
    tensor = x;
    return;
  case TensorExp::Kind::kSynZero:
    assert(x == detail::kInvalidId && y == detail::kInvalidId && !v && !o);
    return;
  case TensorExp::Kind::kInvariant:
    assert(x == detail::kInvalidId && y == detail::kInvalidId && v && !o);
    return;
````
- **L97 EN**: Returns from the current function with `ExpArity::kBinary`.
  **L97 CN**: 以 `ExpArity::kBinary` 从当前函数返回。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Marks this control path as unreachable.
  **L99 CN**: 将该控制路径标记为不可达。
- **L100 EN**: Closes the current lexical scope or compound statement.
  **L100 CN**: 结束当前词法作用域或复合语句块。
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L102 EN**: Banner comment marking a file or section boundary.
  **L102 CN**: 横幅注释，用于标记文件或章节边界。
- **L103 EN**: Comment explains nearby logic, invariants, or intent: `Constructors.`.
  **L103 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Constructors.`。
- **L104 EN**: Banner comment marking a file or section boundary.
  **L104 CN**: 横幅注释，用于标记文件或章节边界。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TensorExp::TensorExp(TensorExp::Kind k, unsigned x, ExprId y, Value v,`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`TensorExp::TensorExp(TensorExp::Kind k, unsigned x, ExprId y, Value v,`。
- **L107 EN**: Continues the surrounding expression or declaration: `Operation *o, Attribute a)`.
  **L107 CN**: 继续构造周围的表达式或声明：`Operation *o, Attribute a)`。
- **L108 EN**: Starts a function, method, lambda, or structured scope: `: kind(k), val(v), op(o), attr(a) {`.
  **L108 CN**: 开始一个函数、方法、lambda 或结构化作用域：`: kind(k), val(v), op(o), attr(a) {`。
- **L109 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L109 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L110 EN**: Comment explains nearby logic, invariants, or intent: `Leaf.`.
  **L110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Leaf.`。
- **L111 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTensor:`.
  **L111 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTensor:`。
- **L112 EN**: Checks an internal invariant in debug builds.
  **L112 CN**: 在调试构建中检查内部不变式。
- **L113 EN**: Executes a standalone statement or declaration: `tensor = x;`.
  **L113 CN**: 执行一条独立语句或声明：`tensor = x;`。
- **L114 EN**: Returns from the current function with `void`.
  **L114 CN**: 以 `void` 从当前函数返回。
- **L115 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSynZero:`.
  **L115 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSynZero:`。
- **L116 EN**: Checks an internal invariant in debug builds.
  **L116 CN**: 在调试构建中检查内部不变式。
- **L117 EN**: Returns from the current function with `void`.
  **L117 CN**: 以 `void` 从当前函数返回。
- **L118 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kInvariant:`.
  **L118 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kInvariant:`。
- **L119 EN**: Checks an internal invariant in debug builds.
  **L119 CN**: 在调试构建中检查内部不变式。
- **L120 EN**: Returns from the current function with `void`.
  **L120 CN**: 以 `void` 从当前函数返回。

### Lines 121-144

````cpp
  case TensorExp::Kind::kLoopVar:
    assert(x != detail::kInvalidId && y == detail::kInvalidId && !v && !o);
    loop = x;
    return;
  // Unary operations.
  case TensorExp::Kind::kAbsF:
  case TensorExp::Kind::kAbsC:
  case TensorExp::Kind::kAbsI:
  case TensorExp::Kind::kCeilF:
  case TensorExp::Kind::kFloorF:
  case TensorExp::Kind::kSqrtF:
  case TensorExp::Kind::kSqrtC:
  case TensorExp::Kind::kExpm1F:
  case TensorExp::Kind::kExpm1C:
  case TensorExp::Kind::kLog1pF:
  case TensorExp::Kind::kLog1pC:
  case TensorExp::Kind::kRelu:
  case TensorExp::Kind::kSinF:
  case TensorExp::Kind::kSinC:
  case TensorExp::Kind::kTanhF:
  case TensorExp::Kind::kTanhC:
  case TensorExp::Kind::kNegF:
  case TensorExp::Kind::kNegC:
  case TensorExp::Kind::kNegI:
````
- **L121 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLoopVar:`.
  **L121 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLoopVar:`。
- **L122 EN**: Checks an internal invariant in debug builds.
  **L122 CN**: 在调试构建中检查内部不变式。
- **L123 EN**: Executes a standalone statement or declaration: `loop = x;`.
  **L123 CN**: 执行一条独立语句或声明：`loop = x;`。
- **L124 EN**: Returns from the current function with `void`.
  **L124 CN**: 以 `void` 从当前函数返回。
- **L125 EN**: Comment explains nearby logic, invariants, or intent: `Unary operations.`.
  **L125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unary operations.`。
- **L126 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsF:`.
  **L126 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsF:`。
- **L127 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsC:`.
  **L127 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsC:`。
- **L128 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsI:`.
  **L128 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsI:`。
- **L129 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCeilF:`.
  **L129 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCeilF:`。
- **L130 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kFloorF:`.
  **L130 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kFloorF:`。
- **L131 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtF:`.
  **L131 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtF:`。
- **L132 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtC:`.
  **L132 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtC:`。
- **L133 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1F:`.
  **L133 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1F:`。
- **L134 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1C:`.
  **L134 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1C:`。
- **L135 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pF:`.
  **L135 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pF:`。
- **L136 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pC:`.
  **L136 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pC:`。
- **L137 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kRelu:`.
  **L137 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kRelu:`。
- **L138 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinF:`.
  **L138 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinF:`。
- **L139 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinC:`.
  **L139 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinC:`。
- **L140 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhF:`.
  **L140 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhF:`。
- **L141 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhC:`.
  **L141 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhC:`。
- **L142 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegF:`.
  **L142 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegF:`。
- **L143 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegC:`.
  **L143 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegC:`。
- **L144 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegI:`.
  **L144 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegI:`。

### Lines 145-168

````cpp
  case TensorExp::Kind::kCIm:
  case TensorExp::Kind::kCRe:
    assert(x != detail::kInvalidId && y == detail::kInvalidId && !v && !o);
    children.e0 = x;
    children.e1 = y;
    return;
  case TensorExp::Kind::kTruncF:
  case TensorExp::Kind::kExtF:
  case TensorExp::Kind::kCastFS:
  case TensorExp::Kind::kCastFU:
  case TensorExp::Kind::kCastSF:
  case TensorExp::Kind::kCastUF:
  case TensorExp::Kind::kCastS:
  case TensorExp::Kind::kCastU:
  case TensorExp::Kind::kCastIdx:
  case TensorExp::Kind::kTruncI:
  case TensorExp::Kind::kBitCast:
    assert(x != detail::kInvalidId && y == detail::kInvalidId && v && !o);
    children.e0 = x;
    children.e1 = y;
    return;
  case TensorExp::Kind::kBinaryBranch:
  case TensorExp::Kind::kSelect:
    assert(x != detail::kInvalidId && y == detail::kInvalidId && !v && o);
````
- **L145 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCIm:`.
  **L145 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCIm:`。
- **L146 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCRe:`.
  **L146 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCRe:`。
- **L147 EN**: Checks an internal invariant in debug builds.
  **L147 CN**: 在调试构建中检查内部不变式。
- **L148 EN**: Executes a standalone statement or declaration: `children.e0 = x;`.
  **L148 CN**: 执行一条独立语句或声明：`children.e0 = x;`。
- **L149 EN**: Executes a standalone statement or declaration: `children.e1 = y;`.
  **L149 CN**: 执行一条独立语句或声明：`children.e1 = y;`。
- **L150 EN**: Returns from the current function with `void`.
  **L150 CN**: 以 `void` 从当前函数返回。
- **L151 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncF:`.
  **L151 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncF:`。
- **L152 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExtF:`.
  **L152 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExtF:`。
- **L153 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFS:`.
  **L153 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFS:`。
- **L154 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFU:`.
  **L154 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFU:`。
- **L155 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastSF:`.
  **L155 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastSF:`。
- **L156 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastUF:`.
  **L156 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastUF:`。
- **L157 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastS:`.
  **L157 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastS:`。
- **L158 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastU:`.
  **L158 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastU:`。
- **L159 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastIdx:`.
  **L159 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastIdx:`。
- **L160 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncI:`.
  **L160 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncI:`。
- **L161 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBitCast:`.
  **L161 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBitCast:`。
- **L162 EN**: Checks an internal invariant in debug builds.
  **L162 CN**: 在调试构建中检查内部不变式。
- **L163 EN**: Executes a standalone statement or declaration: `children.e0 = x;`.
  **L163 CN**: 执行一条独立语句或声明：`children.e0 = x;`。
- **L164 EN**: Executes a standalone statement or declaration: `children.e1 = y;`.
  **L164 CN**: 执行一条独立语句或声明：`children.e1 = y;`。
- **L165 EN**: Returns from the current function with `void`.
  **L165 CN**: 以 `void` 从当前函数返回。
- **L166 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinaryBranch:`.
  **L166 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinaryBranch:`。
- **L167 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSelect:`.
  **L167 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSelect:`。
- **L168 EN**: Checks an internal invariant in debug builds.
  **L168 CN**: 在调试构建中检查内部不变式。

### Lines 169-192

````cpp
    children.e0 = x;
    children.e1 = y;
    return;
  case TensorExp::Kind::kUnary:
    // No assertion on y can be made, as the branching paths involve both
    // a unary (`mapSet`) and binary (`disjSet`) pathway.
    assert(x != detail::kInvalidId && !v && o);
    children.e0 = x;
    children.e1 = y;
    return;
  // Binary operations.
  case TensorExp::Kind::kMulF:
  case TensorExp::Kind::kMulC:
  case TensorExp::Kind::kMulI:
  case TensorExp::Kind::kDivF:
  case TensorExp::Kind::kDivC:
  case TensorExp::Kind::kDivS:
  case TensorExp::Kind::kDivU:
  case TensorExp::Kind::kAddF:
  case TensorExp::Kind::kAddC:
  case TensorExp::Kind::kAddI:
  case TensorExp::Kind::kSubF:
  case TensorExp::Kind::kSubC:
  case TensorExp::Kind::kSubI:
````
- **L169 EN**: Executes a standalone statement or declaration: `children.e0 = x;`.
  **L169 CN**: 执行一条独立语句或声明：`children.e0 = x;`。
- **L170 EN**: Executes a standalone statement or declaration: `children.e1 = y;`.
  **L170 CN**: 执行一条独立语句或声明：`children.e1 = y;`。
- **L171 EN**: Returns from the current function with `void`.
  **L171 CN**: 以 `void` 从当前函数返回。
- **L172 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kUnary:`.
  **L172 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kUnary:`。
- **L173 EN**: Comment explains nearby logic, invariants, or intent: `No assertion on y can be made, as the branching paths involve both`.
  **L173 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`No assertion on y can be made, as the branching paths involve both`。
- **L174 EN**: Comment explains nearby logic, invariants, or intent: `a unary (`mapSet`) and binary (`disjSet`) pathway.`.
  **L174 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a unary (`mapSet`) and binary (`disjSet`) pathway.`。
- **L175 EN**: Checks an internal invariant in debug builds.
  **L175 CN**: 在调试构建中检查内部不变式。
- **L176 EN**: Executes a standalone statement or declaration: `children.e0 = x;`.
  **L176 CN**: 执行一条独立语句或声明：`children.e0 = x;`。
- **L177 EN**: Executes a standalone statement or declaration: `children.e1 = y;`.
  **L177 CN**: 执行一条独立语句或声明：`children.e1 = y;`。
- **L178 EN**: Returns from the current function with `void`.
  **L178 CN**: 以 `void` 从当前函数返回。
- **L179 EN**: Comment explains nearby logic, invariants, or intent: `Binary operations.`.
  **L179 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Binary operations.`。
- **L180 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulF:`.
  **L180 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulF:`。
- **L181 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulC:`.
  **L181 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulC:`。
- **L182 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulI:`.
  **L182 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulI:`。
- **L183 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivF:`.
  **L183 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivF:`。
- **L184 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivC:`.
  **L184 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivC:`。
- **L185 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivS:`.
  **L185 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivS:`。
- **L186 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivU:`.
  **L186 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivU:`。
- **L187 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddF:`.
  **L187 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddF:`。
- **L188 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddC:`.
  **L188 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddC:`。
- **L189 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddI:`.
  **L189 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddI:`。
- **L190 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubF:`.
  **L190 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubF:`。
- **L191 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubC:`.
  **L191 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubC:`。
- **L192 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubI:`.
  **L192 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubI:`。

### Lines 193-216

````cpp
  case TensorExp::Kind::kAndI:
  case TensorExp::Kind::kOrI:
  case TensorExp::Kind::kXorI:
  case TensorExp::Kind::kShrS:
  case TensorExp::Kind::kShrU:
  case TensorExp::Kind::kShlI:
    assert(x != detail::kInvalidId && y != detail::kInvalidId && !v && !o);
    children.e0 = x;
    children.e1 = y;
    return;
  case TensorExp::Kind::kCmpF:
  case TensorExp::Kind::kCmpI:
    assert(x != detail::kInvalidId && y != detail::kInvalidId && !v && !o);
    children.e0 = x;
    children.e1 = y;
    return;
  case TensorExp::Kind::kBinary:
  case TensorExp::Kind::kReduce:
    assert(x != detail::kInvalidId && y != detail::kInvalidId && !v && o);
    children.e0 = x;
    children.e1 = y;
    return;
  case TensorExp::Kind::kDenseOp:
    assert(x != detail::kInvalidId && !v && o);
````
- **L193 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAndI:`.
  **L193 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAndI:`。
- **L194 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kOrI:`.
  **L194 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kOrI:`。
- **L195 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kXorI:`.
  **L195 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kXorI:`。
- **L196 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrS:`.
  **L196 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrS:`。
- **L197 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrU:`.
  **L197 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrU:`。
- **L198 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShlI:`.
  **L198 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShlI:`。
- **L199 EN**: Checks an internal invariant in debug builds.
  **L199 CN**: 在调试构建中检查内部不变式。
- **L200 EN**: Executes a standalone statement or declaration: `children.e0 = x;`.
  **L200 CN**: 执行一条独立语句或声明：`children.e0 = x;`。
- **L201 EN**: Executes a standalone statement or declaration: `children.e1 = y;`.
  **L201 CN**: 执行一条独立语句或声明：`children.e1 = y;`。
- **L202 EN**: Returns from the current function with `void`.
  **L202 CN**: 以 `void` 从当前函数返回。
- **L203 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpF:`.
  **L203 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpF:`。
- **L204 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpI:`.
  **L204 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpI:`。
- **L205 EN**: Checks an internal invariant in debug builds.
  **L205 CN**: 在调试构建中检查内部不变式。
- **L206 EN**: Executes a standalone statement or declaration: `children.e0 = x;`.
  **L206 CN**: 执行一条独立语句或声明：`children.e0 = x;`。
- **L207 EN**: Executes a standalone statement or declaration: `children.e1 = y;`.
  **L207 CN**: 执行一条独立语句或声明：`children.e1 = y;`。
- **L208 EN**: Returns from the current function with `void`.
  **L208 CN**: 以 `void` 从当前函数返回。
- **L209 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinary:`.
  **L209 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinary:`。
- **L210 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kReduce:`.
  **L210 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kReduce:`。
- **L211 EN**: Checks an internal invariant in debug builds.
  **L211 CN**: 在调试构建中检查内部不变式。
- **L212 EN**: Executes a standalone statement or declaration: `children.e0 = x;`.
  **L212 CN**: 执行一条独立语句或声明：`children.e0 = x;`。
- **L213 EN**: Executes a standalone statement or declaration: `children.e1 = y;`.
  **L213 CN**: 执行一条独立语句或声明：`children.e1 = y;`。
- **L214 EN**: Returns from the current function with `void`.
  **L214 CN**: 以 `void` 从当前函数返回。
- **L215 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDenseOp:`.
  **L215 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDenseOp:`。
- **L216 EN**: Checks an internal invariant in debug builds.
  **L216 CN**: 在调试构建中检查内部不变式。

### Lines 217-240

````cpp
    children.e0 = x;
    children.e1 = y;
    return;
  }
  llvm_unreachable("unexpected kind");
}

Merger::Merger(unsigned numInputOutputTensors, unsigned numLoops,
               unsigned maxLvlRank)
    : outTensor(numInputOutputTensors - 1),
      syntheticTensor(numInputOutputTensors),
      numTensors(numInputOutputTensors + 1), numLoops(numLoops),
      hasSparseOut(false),
      lvlTypes(numTensors,
               std::vector<LevelType>(numLoops, LevelFormat::Undef)),
      loopToLvl(numTensors,
                std::vector<std::optional<Level>>(numLoops, std::nullopt)),
      lvlToLoop(numTensors,
                std::vector<std::optional<LoopId>>(maxLvlRank, std::nullopt)),
      loopToUnresolvedLvls(numLoops, std::vector<std::optional<LvlLTPair>>(
                                         numTensors, std::nullopt)),
      levelToDependentLoop(numTensors,
                           std::vector<std::vector<LoopCoeffPair>>(
                               maxLvlRank, std::vector<LoopCoeffPair>())),
````
- **L217 EN**: Executes a standalone statement or declaration: `children.e0 = x;`.
  **L217 CN**: 执行一条独立语句或声明：`children.e0 = x;`。
- **L218 EN**: Executes a standalone statement or declaration: `children.e1 = y;`.
  **L218 CN**: 执行一条独立语句或声明：`children.e1 = y;`。
- **L219 EN**: Returns from the current function with `void`.
  **L219 CN**: 以 `void` 从当前函数返回。
- **L220 EN**: Closes the current lexical scope or compound statement.
  **L220 CN**: 结束当前词法作用域或复合语句块。
- **L221 EN**: Marks this control path as unreachable.
  **L221 CN**: 将该控制路径标记为不可达。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Merger::Merger(unsigned numInputOutputTensors, unsigned numLoops,`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`Merger::Merger(unsigned numInputOutputTensors, unsigned numLoops,`。
- **L225 EN**: Continues the surrounding expression or declaration: `unsigned maxLvlRank)`.
  **L225 CN**: 继续构造周围的表达式或声明：`unsigned maxLvlRank)`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: outTensor(numInputOutputTensors - 1),`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`: outTensor(numInputOutputTensors - 1),`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `syntheticTensor(numInputOutputTensors),`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`syntheticTensor(numInputOutputTensors),`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `numTensors(numInputOutputTensors + 1), numLoops(numLoops),`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`numTensors(numInputOutputTensors + 1), numLoops(numLoops),`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `hasSparseOut(false),`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`hasSparseOut(false),`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lvlTypes(numTensors,`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`lvlTypes(numTensors,`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::vector<LevelType>(numLoops, LevelFormat::Undef)),`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::vector<LevelType>(numLoops, LevelFormat::Undef)),`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loopToLvl(numTensors,`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`loopToLvl(numTensors,`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::vector<std::optional<Level>>(numLoops, std::nullopt)),`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::vector<std::optional<Level>>(numLoops, std::nullopt)),`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lvlToLoop(numTensors,`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`lvlToLoop(numTensors,`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::vector<std::optional<LoopId>>(maxLvlRank, std::nullopt)),`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::vector<std::optional<LoopId>>(maxLvlRank, std::nullopt)),`。
- **L236 EN**: Continues logic associated with callable symbol `loopToUnresolvedLvls`.
  **L236 CN**: 继续与可调用符号 `loopToUnresolvedLvls` 相关的逻辑。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `numTensors, std::nullopt)),`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`numTensors, std::nullopt)),`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `levelToDependentLoop(numTensors,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`levelToDependentLoop(numTensors,`。
- **L239 EN**: Continues logic associated with callable symbol `vector<LoopCoeffPair>>`.
  **L239 CN**: 继续与可调用符号 `vector<LoopCoeffPair>>` 相关的逻辑。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `maxLvlRank, std::vector<LoopCoeffPair>())),`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`maxLvlRank, std::vector<LoopCoeffPair>())),`。

### Lines 241-264

````cpp
      loopBounds(numLoops, std::make_pair(numTensors, numLoops)) {}

//===----------------------------------------------------------------------===//
// Lattice methods.
//===----------------------------------------------------------------------===//

ExprId Merger::addTensorExp(TensorId t) {
  assert(isValidTensorId(t));
  const ExprId eNew(tensorExps.size());
  tensorExps.emplace_back(TensorExp::Kind::kTensor, t, detail::kInvalidId,
                          Value(), nullptr, nullptr);
  return eNew;
}

ExprId Merger::addLoopVarExp(LoopId i) {
  assert(isValidLoopId(i));
  const ExprId eNew(tensorExps.size());
  tensorExps.emplace_back(TensorExp::Kind::kLoopVar, i, detail::kInvalidId,
                          Value(), nullptr, nullptr);
  return eNew;
}

ExprId Merger::addInvariantExp(Value v) {
  const ExprId eNew(tensorExps.size());
````
- **L241 EN**: Continues logic associated with callable symbol `loopBounds`.
  **L241 CN**: 继续与可调用符号 `loopBounds` 相关的逻辑。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L243 EN**: Banner comment marking a file or section boundary.
  **L243 CN**: 横幅注释，用于标记文件或章节边界。
- **L244 EN**: Comment explains nearby logic, invariants, or intent: `Lattice methods.`.
  **L244 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lattice methods.`。
- **L245 EN**: Banner comment marking a file or section boundary.
  **L245 CN**: 横幅注释，用于标记文件或章节边界。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L247 EN**: Starts a function, method, lambda, or structured scope: `ExprId Merger::addTensorExp(TensorId t) {`.
  **L247 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ExprId Merger::addTensorExp(TensorId t) {`。
- **L248 EN**: Checks an internal invariant in debug builds.
  **L248 CN**: 在调试构建中检查内部不变式。
- **L249 EN**: Executes a call or declaration centered on `eNew`.
  **L249 CN**: 执行以 `eNew` 为核心的调用或声明。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tensorExps.emplace_back(TensorExp::Kind::kTensor, t, detail::kInvalidId,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`tensorExps.emplace_back(TensorExp::Kind::kTensor, t, detail::kInvalidId,`。
- **L251 EN**: Executes a call or declaration centered on `Value`.
  **L251 CN**: 执行以 `Value` 为核心的调用或声明。
- **L252 EN**: Returns from the current function with `eNew`.
  **L252 CN**: 以 `eNew` 从当前函数返回。
- **L253 EN**: Closes the current lexical scope or compound statement.
  **L253 CN**: 结束当前词法作用域或复合语句块。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L255 EN**: Starts a function, method, lambda, or structured scope: `ExprId Merger::addLoopVarExp(LoopId i) {`.
  **L255 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ExprId Merger::addLoopVarExp(LoopId i) {`。
- **L256 EN**: Checks an internal invariant in debug builds.
  **L256 CN**: 在调试构建中检查内部不变式。
- **L257 EN**: Executes a call or declaration centered on `eNew`.
  **L257 CN**: 执行以 `eNew` 为核心的调用或声明。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tensorExps.emplace_back(TensorExp::Kind::kLoopVar, i, detail::kInvalidId,`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`tensorExps.emplace_back(TensorExp::Kind::kLoopVar, i, detail::kInvalidId,`。
- **L259 EN**: Executes a call or declaration centered on `Value`.
  **L259 CN**: 执行以 `Value` 为核心的调用或声明。
- **L260 EN**: Returns from the current function with `eNew`.
  **L260 CN**: 以 `eNew` 从当前函数返回。
- **L261 EN**: Closes the current lexical scope or compound statement.
  **L261 CN**: 结束当前词法作用域或复合语句块。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L263 EN**: Starts a function, method, lambda, or structured scope: `ExprId Merger::addInvariantExp(Value v) {`.
  **L263 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ExprId Merger::addInvariantExp(Value v) {`。
- **L264 EN**: Executes a call or declaration centered on `eNew`.
  **L264 CN**: 执行以 `eNew` 为核心的调用或声明。

### Lines 265-288

````cpp
  tensorExps.emplace_back(TensorExp::Kind::kInvariant, detail::kInvalidId,
                          detail::kInvalidId, v, nullptr, nullptr);
  return eNew;
}

ExprId Merger::addSynZeroExp() {
  const ExprId eNew(tensorExps.size());
  tensorExps.emplace_back(TensorExp::Kind::kSynZero, detail::kInvalidId,
                          detail::kInvalidId, Value(), nullptr, nullptr);
  return eNew;
}

ExprId Merger::addExp(TensorExp::Kind k, ExprId e0, ExprId e1, Operation *op,
                      Attribute attr) {
  assert(k > TensorExp::Kind::kLoopVar);
  const ExprId eNew(tensorExps.size());
  tensorExps.emplace_back(k, e0, e1, Value(), op, attr);
  return eNew;
}

ExprId Merger::addExp(TensorExp::Kind k, ExprId e, Value v, Operation *op,
                      Attribute attr) {
  assert(k > TensorExp::Kind::kLoopVar);
  const ExprId eNew(tensorExps.size());
````
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tensorExps.emplace_back(TensorExp::Kind::kInvariant, detail::kInvalidId,`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`tensorExps.emplace_back(TensorExp::Kind::kInvariant, detail::kInvalidId,`。
- **L266 EN**: Executes a standalone statement or declaration: `detail::kInvalidId, v, nullptr, nullptr);`.
  **L266 CN**: 执行一条独立语句或声明：`detail::kInvalidId, v, nullptr, nullptr);`。
- **L267 EN**: Returns from the current function with `eNew`.
  **L267 CN**: 以 `eNew` 从当前函数返回。
- **L268 EN**: Closes the current lexical scope or compound statement.
  **L268 CN**: 结束当前词法作用域或复合语句块。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Starts a function, method, lambda, or structured scope: `ExprId Merger::addSynZeroExp() {`.
  **L270 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ExprId Merger::addSynZeroExp() {`。
- **L271 EN**: Executes a call or declaration centered on `eNew`.
  **L271 CN**: 执行以 `eNew` 为核心的调用或声明。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tensorExps.emplace_back(TensorExp::Kind::kSynZero, detail::kInvalidId,`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`tensorExps.emplace_back(TensorExp::Kind::kSynZero, detail::kInvalidId,`。
- **L273 EN**: Executes a call or declaration centered on `Value`.
  **L273 CN**: 执行以 `Value` 为核心的调用或声明。
- **L274 EN**: Returns from the current function with `eNew`.
  **L274 CN**: 以 `eNew` 从当前函数返回。
- **L275 EN**: Closes the current lexical scope or compound statement.
  **L275 CN**: 结束当前词法作用域或复合语句块。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExprId Merger::addExp(TensorExp::Kind k, ExprId e0, ExprId e1, Operation *op,`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExprId Merger::addExp(TensorExp::Kind k, ExprId e0, ExprId e1, Operation *op,`。
- **L278 EN**: Continues the surrounding expression or declaration: `Attribute attr) {`.
  **L278 CN**: 继续构造周围的表达式或声明：`Attribute attr) {`。
- **L279 EN**: Checks an internal invariant in debug builds.
  **L279 CN**: 在调试构建中检查内部不变式。
- **L280 EN**: Executes a call or declaration centered on `eNew`.
  **L280 CN**: 执行以 `eNew` 为核心的调用或声明。
- **L281 EN**: Executes a call or declaration centered on `tensorExps.emplace_back`.
  **L281 CN**: 执行以 `tensorExps.emplace_back` 为核心的调用或声明。
- **L282 EN**: Returns from the current function with `eNew`.
  **L282 CN**: 以 `eNew` 从当前函数返回。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExprId Merger::addExp(TensorExp::Kind k, ExprId e, Value v, Operation *op,`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExprId Merger::addExp(TensorExp::Kind k, ExprId e, Value v, Operation *op,`。
- **L286 EN**: Continues the surrounding expression or declaration: `Attribute attr) {`.
  **L286 CN**: 继续构造周围的表达式或声明：`Attribute attr) {`。
- **L287 EN**: Checks an internal invariant in debug builds.
  **L287 CN**: 在调试构建中检查内部不变式。
- **L288 EN**: Executes a call or declaration centered on `eNew`.
  **L288 CN**: 执行以 `eNew` 为核心的调用或声明。

### Lines 289-312

````cpp
  tensorExps.emplace_back(k, e, detail::kInvalidId, v, op, attr);
  return eNew;
}

LatPointId Merger::addLat(TensorId t, LoopId i, ExprId e) {
  const LatPointId pNew(latPoints.size());
  const unsigned size = numLoops * numTensors;
  const TensorLoopId b = makeTensorLoopId(t, i);
  latPoints.emplace_back(size, e);
  latPoints[pNew].bits.set(b);
  return pNew;
}

LatPointId Merger::addLat(const BitVector &bits, ExprId e) {
  assert(bits.size() == numLoops * numTensors);
  const LatPointId pNew(latPoints.size());
  latPoints.emplace_back(bits, e);
  return pNew;
}

LatSetId Merger::addSet() {
  const LatSetId sNew(latSets.size());
  latSets.emplace_back();
  return sNew;
````
- **L289 EN**: Executes a call or declaration centered on `tensorExps.emplace_back`.
  **L289 CN**: 执行以 `tensorExps.emplace_back` 为核心的调用或声明。
- **L290 EN**: Returns from the current function with `eNew`.
  **L290 CN**: 以 `eNew` 从当前函数返回。
- **L291 EN**: Closes the current lexical scope or compound statement.
  **L291 CN**: 结束当前词法作用域或复合语句块。
- **L292 EN**: Blank line separating nearby declarations or logic blocks.
  **L292 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L293 EN**: Starts a function, method, lambda, or structured scope: `LatPointId Merger::addLat(TensorId t, LoopId i, ExprId e) {`.
  **L293 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatPointId Merger::addLat(TensorId t, LoopId i, ExprId e) {`。
- **L294 EN**: Executes a call or declaration centered on `pNew`.
  **L294 CN**: 执行以 `pNew` 为核心的调用或声明。
- **L295 EN**: Initializes variable `size` from the right-hand expression.
  **L295 CN**: 使用右侧表达式初始化变量 `size`。
- **L296 EN**: Initializes variable `b` from the right-hand expression.
  **L296 CN**: 使用右侧表达式初始化变量 `b`。
- **L297 EN**: Executes a call or declaration centered on `latPoints.emplace_back`.
  **L297 CN**: 执行以 `latPoints.emplace_back` 为核心的调用或声明。
- **L298 EN**: Executes a call or declaration centered on `latPoints[pNew].bits.set`.
  **L298 CN**: 执行以 `latPoints[pNew].bits.set` 为核心的调用或声明。
- **L299 EN**: Returns from the current function with `pNew`.
  **L299 CN**: 以 `pNew` 从当前函数返回。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L302 EN**: Starts a function, method, lambda, or structured scope: `LatPointId Merger::addLat(const BitVector &bits, ExprId e) {`.
  **L302 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatPointId Merger::addLat(const BitVector &bits, ExprId e) {`。
- **L303 EN**: Checks an internal invariant in debug builds.
  **L303 CN**: 在调试构建中检查内部不变式。
- **L304 EN**: Executes a call or declaration centered on `pNew`.
  **L304 CN**: 执行以 `pNew` 为核心的调用或声明。
- **L305 EN**: Executes a call or declaration centered on `latPoints.emplace_back`.
  **L305 CN**: 执行以 `latPoints.emplace_back` 为核心的调用或声明。
- **L306 EN**: Returns from the current function with `pNew`.
  **L306 CN**: 以 `pNew` 从当前函数返回。
- **L307 EN**: Closes the current lexical scope or compound statement.
  **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L309 EN**: Starts a function, method, lambda, or structured scope: `LatSetId Merger::addSet() {`.
  **L309 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatSetId Merger::addSet() {`。
- **L310 EN**: Executes a call or declaration centered on `sNew`.
  **L310 CN**: 执行以 `sNew` 为核心的调用或声明。
- **L311 EN**: Executes a call or declaration centered on `latSets.emplace_back`.
  **L311 CN**: 执行以 `latSets.emplace_back` 为核心的调用或声明。
- **L312 EN**: Returns from the current function with `sNew`.
  **L312 CN**: 以 `sNew` 从当前函数返回。

### Lines 313-336

````cpp
}

LatPointId Merger::conjLat(ExprId e, LatPointId p0, LatPointId p1,
                           Operation *op) {
  TensorExp::Kind kind = exp(e).kind;
  Attribute attr = exp(e).attr;
  const LatPointId pNew(latPoints.size());
  const auto &point0 = lat(p0);
  const auto &point1 = lat(p1);
  BitVector bits(point0.bits);
  bits |= point1.bits;
  const ExprId ne = addExp(kind, point0.exp, point1.exp, op, attr);
  latPoints.emplace_back(bits, ne);
  return pNew;
}

LatSetId Merger::conjSet(ExprId e, LatSetId s0, LatSetId s1, Operation *op) {
  const LatSetId sNew = addSet();
  auto &setNew = latSets[sNew];
  for (const LatPointId p0 : set(s0))
    for (const LatPointId p1 : set(s1))
      setNew.push_back(conjLat(e, p0, p1, op));
  return sNew;
}
````
- **L313 EN**: Closes the current lexical scope or compound statement.
  **L313 CN**: 结束当前词法作用域或复合语句块。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LatPointId Merger::conjLat(ExprId e, LatPointId p0, LatPointId p1,`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`LatPointId Merger::conjLat(ExprId e, LatPointId p0, LatPointId p1,`。
- **L316 EN**: Continues the surrounding expression or declaration: `Operation *op) {`.
  **L316 CN**: 继续构造周围的表达式或声明：`Operation *op) {`。
- **L317 EN**: Initializes variable `kind` from the right-hand expression.
  **L317 CN**: 使用右侧表达式初始化变量 `kind`。
- **L318 EN**: Initializes variable `attr` from the right-hand expression.
  **L318 CN**: 使用右侧表达式初始化变量 `attr`。
- **L319 EN**: Executes a call or declaration centered on `pNew`.
  **L319 CN**: 执行以 `pNew` 为核心的调用或声明。
- **L320 EN**: Executes a call or declaration centered on `lat`.
  **L320 CN**: 执行以 `lat` 为核心的调用或声明。
- **L321 EN**: Executes a call or declaration centered on `lat`.
  **L321 CN**: 执行以 `lat` 为核心的调用或声明。
- **L322 EN**: Executes a call or declaration centered on `bits`.
  **L322 CN**: 执行以 `bits` 为核心的调用或声明。
- **L323 EN**: Executes a standalone statement or declaration: `bits |= point1.bits;`.
  **L323 CN**: 执行一条独立语句或声明：`bits |= point1.bits;`。
- **L324 EN**: Initializes variable `ne` from the right-hand expression.
  **L324 CN**: 使用右侧表达式初始化变量 `ne`。
- **L325 EN**: Executes a call or declaration centered on `latPoints.emplace_back`.
  **L325 CN**: 执行以 `latPoints.emplace_back` 为核心的调用或声明。
- **L326 EN**: Returns from the current function with `pNew`.
  **L326 CN**: 以 `pNew` 从当前函数返回。
- **L327 EN**: Closes the current lexical scope or compound statement.
  **L327 CN**: 结束当前词法作用域或复合语句块。
- **L328 EN**: Blank line separating nearby declarations or logic blocks.
  **L328 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L329 EN**: Starts a function, method, lambda, or structured scope: `LatSetId Merger::conjSet(ExprId e, LatSetId s0, LatSetId s1, Operation *op) {`.
  **L329 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatSetId Merger::conjSet(ExprId e, LatSetId s0, LatSetId s1, Operation *op) {`。
- **L330 EN**: Initializes variable `sNew` from the right-hand expression.
  **L330 CN**: 使用右侧表达式初始化变量 `sNew`。
- **L331 EN**: Executes a standalone statement or declaration: `auto &setNew = latSets[sNew];`.
  **L331 CN**: 执行一条独立语句或声明：`auto &setNew = latSets[sNew];`。
- **L332 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L332 CN**: 开始 `for` 控制流语句并计算其条件。
- **L333 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L333 CN**: 开始 `for` 控制流语句并计算其条件。
- **L334 EN**: Executes a call or declaration centered on `setNew.push_back`.
  **L334 CN**: 执行以 `setNew.push_back` 为核心的调用或声明。
- **L335 EN**: Returns from the current function with `sNew`.
  **L335 CN**: 以 `sNew` 从当前函数返回。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。

### Lines 337-360

````cpp

LatSetId Merger::disjSet(ExprId e, LatSetId s0, LatSetId s1, Operation *op) {
  const LatSetId sNew = conjSet(e, s0, s1, op);
  TensorExp::Kind kind = exp(e).kind;
  // Followed by all in s0.
  latSets[sNew].append(latSets[s0]);
  // Map binary 0-y to unary -y.
  // TODO: move this if-else logic into buildLattices
  if (kind == TensorExp::Kind::kSubF)
    s1 = mapSet(TensorExp::Kind::kNegF, s1);
  else if (kind == TensorExp::Kind::kSubC)
    s1 = mapSet(TensorExp::Kind::kNegC, s1);
  else if (kind == TensorExp::Kind::kSubI)
    s1 = mapSet(TensorExp::Kind::kNegI, s1);
  // Followed by all in s1.
  latSets[sNew].append(latSets[s1]);
  return sNew;
}

LatSetId Merger::disjSetWithZero(ExprId e, LatSetId s0, LatSetId s1) {
  assert(exp(e).kind == TensorExp::Kind::kCmpI ||
         exp(e).kind == TensorExp::Kind::kCmpF);
  const LatSetId sNew = conjSet(e, s0, s1, nullptr);

````
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L338 EN**: Starts a function, method, lambda, or structured scope: `LatSetId Merger::disjSet(ExprId e, LatSetId s0, LatSetId s1, Operation *op) {`.
  **L338 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatSetId Merger::disjSet(ExprId e, LatSetId s0, LatSetId s1, Operation *op) {`。
- **L339 EN**: Initializes variable `sNew` from the right-hand expression.
  **L339 CN**: 使用右侧表达式初始化变量 `sNew`。
- **L340 EN**: Initializes variable `kind` from the right-hand expression.
  **L340 CN**: 使用右侧表达式初始化变量 `kind`。
- **L341 EN**: Comment explains nearby logic, invariants, or intent: `Followed by all in s0.`.
  **L341 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Followed by all in s0.`。
- **L342 EN**: Executes a call or declaration centered on `latSets[sNew].append`.
  **L342 CN**: 执行以 `latSets[sNew].append` 为核心的调用或声明。
- **L343 EN**: Comment explains nearby logic, invariants, or intent: `Map binary 0-y to unary -y.`.
  **L343 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Map binary 0-y to unary -y.`。
- **L344 EN**: Comment records a pending task or caution: `TODO: move this if-else logic into buildLattices`.
  **L344 CN**: 注释记录了待办事项或注意点：`TODO: move this if-else logic into buildLattices`。
- **L345 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L345 CN**: 开始 `if` 控制流语句并计算其条件。
- **L346 EN**: Executes a call or declaration centered on `mapSet`.
  **L346 CN**: 执行以 `mapSet` 为核心的调用或声明。
- **L347 EN**: Starts the alternative branch of the preceding conditional.
  **L347 CN**: 开始前一个条件语句的备选分支。
- **L348 EN**: Executes a call or declaration centered on `mapSet`.
  **L348 CN**: 执行以 `mapSet` 为核心的调用或声明。
- **L349 EN**: Starts the alternative branch of the preceding conditional.
  **L349 CN**: 开始前一个条件语句的备选分支。
- **L350 EN**: Executes a call or declaration centered on `mapSet`.
  **L350 CN**: 执行以 `mapSet` 为核心的调用或声明。
- **L351 EN**: Comment explains nearby logic, invariants, or intent: `Followed by all in s1.`.
  **L351 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Followed by all in s1.`。
- **L352 EN**: Executes a call or declaration centered on `latSets[sNew].append`.
  **L352 CN**: 执行以 `latSets[sNew].append` 为核心的调用或声明。
- **L353 EN**: Returns from the current function with `sNew`.
  **L353 CN**: 以 `sNew` 从当前函数返回。
- **L354 EN**: Closes the current lexical scope or compound statement.
  **L354 CN**: 结束当前词法作用域或复合语句块。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L356 EN**: Starts a function, method, lambda, or structured scope: `LatSetId Merger::disjSetWithZero(ExprId e, LatSetId s0, LatSetId s1) {`.
  **L356 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatSetId Merger::disjSetWithZero(ExprId e, LatSetId s0, LatSetId s1) {`。
- **L357 EN**: Checks an internal invariant in debug builds.
  **L357 CN**: 在调试构建中检查内部不变式。
- **L358 EN**: Executes a call or declaration centered on `exp`.
  **L358 CN**: 执行以 `exp` 为核心的调用或声明。
- **L359 EN**: Initializes variable `sNew` from the right-hand expression.
  **L359 CN**: 使用右侧表达式初始化变量 `sNew`。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 361-384

````cpp
  ExprId e0 = exp(e).children.e0;
  ExprId e1 = exp(e).children.e1;
  if (exp(e0).kind == TensorExp::Kind::kSynZero ||
      exp(e1).kind == TensorExp::Kind::kSynZero) {
    // lhs and rhs can't be synthetic zero at the same time.
    assert(exp(e0).kind != exp(e1).kind);
    // If one of the operands has already been assigned to zero (the
    // element is absent in the corresponding operand), then we do not
    // need to build disjunctive set for it.
    return sNew;
  }

  auto lhsSet = mapBinWithSynZeroSet(e, s0, false);
  auto rhsSet = mapBinWithSynZeroSet(e, s1, true);
  latSets[sNew].append(latSets[lhsSet]);
  latSets[sNew].append(latSets[rhsSet]);
  return sNew;
}

LatSetId Merger::combiSet(ExprId e, LatSetId s0, LatSetId s1, Operation *orig,
                          bool includeLeft, TensorExp::Kind ltrans,
                          Operation *opleft, bool includeRight,
                          TensorExp::Kind rtrans, Operation *opright) {
  Attribute a = exp(e).attr;
````
- **L361 EN**: Initializes variable `e0` from the right-hand expression.
  **L361 CN**: 使用右侧表达式初始化变量 `e0`。
- **L362 EN**: Initializes variable `e1` from the right-hand expression.
  **L362 CN**: 使用右侧表达式初始化变量 `e1`。
- **L363 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L363 CN**: 开始 `if` 控制流语句并计算其条件。
- **L364 EN**: Starts a function, method, lambda, or structured scope: `exp(e1).kind == TensorExp::Kind::kSynZero) {`.
  **L364 CN**: 开始一个函数、方法、lambda 或结构化作用域：`exp(e1).kind == TensorExp::Kind::kSynZero) {`。
- **L365 EN**: Comment explains nearby logic, invariants, or intent: `lhs and rhs can't be synthetic zero at the same time.`.
  **L365 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lhs and rhs can't be synthetic zero at the same time.`。
- **L366 EN**: Checks an internal invariant in debug builds.
  **L366 CN**: 在调试构建中检查内部不变式。
- **L367 EN**: Comment explains nearby logic, invariants, or intent: `If one of the operands has already been assigned to zero (the`.
  **L367 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If one of the operands has already been assigned to zero (the`。
- **L368 EN**: Comment explains nearby logic, invariants, or intent: `element is absent in the corresponding operand), then we do not`.
  **L368 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`element is absent in the corresponding operand), then we do not`。
- **L369 EN**: Comment explains nearby logic, invariants, or intent: `need to build disjunctive set for it.`.
  **L369 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`need to build disjunctive set for it.`。
- **L370 EN**: Returns from the current function with `sNew`.
  **L370 CN**: 以 `sNew` 从当前函数返回。
- **L371 EN**: Closes the current lexical scope or compound statement.
  **L371 CN**: 结束当前词法作用域或复合语句块。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L373 EN**: Initializes variable `lhsSet` from the right-hand expression.
  **L373 CN**: 使用右侧表达式初始化变量 `lhsSet`。
- **L374 EN**: Initializes variable `rhsSet` from the right-hand expression.
  **L374 CN**: 使用右侧表达式初始化变量 `rhsSet`。
- **L375 EN**: Executes a call or declaration centered on `latSets[sNew].append`.
  **L375 CN**: 执行以 `latSets[sNew].append` 为核心的调用或声明。
- **L376 EN**: Executes a call or declaration centered on `latSets[sNew].append`.
  **L376 CN**: 执行以 `latSets[sNew].append` 为核心的调用或声明。
- **L377 EN**: Returns from the current function with `sNew`.
  **L377 CN**: 以 `sNew` 从当前函数返回。
- **L378 EN**: Closes the current lexical scope or compound statement.
  **L378 CN**: 结束当前词法作用域或复合语句块。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LatSetId Merger::combiSet(ExprId e, LatSetId s0, LatSetId s1, Operation *orig,`.
  **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`LatSetId Merger::combiSet(ExprId e, LatSetId s0, LatSetId s1, Operation *orig,`。
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool includeLeft, TensorExp::Kind ltrans,`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool includeLeft, TensorExp::Kind ltrans,`。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation *opleft, bool includeRight,`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation *opleft, bool includeRight,`。
- **L383 EN**: Continues the surrounding expression or declaration: `TensorExp::Kind rtrans, Operation *opright) {`.
  **L383 CN**: 继续构造周围的表达式或声明：`TensorExp::Kind rtrans, Operation *opright) {`。
- **L384 EN**: Initializes variable `a` from the right-hand expression.
  **L384 CN**: 使用右侧表达式初始化变量 `a`。

### Lines 385-408

````cpp
  const LatSetId sNew = conjSet(e, s0, s1, orig);
  // Left Region.
  if (includeLeft) {
    if (opleft)
      s0 = mapSet(ltrans, s0, Value(), opleft, a);
    latSets[sNew].append(latSets[s0]);
  }
  // Right Region.
  if (includeRight) {
    if (opright)
      s1 = mapSet(rtrans, s1, Value(), opright, a);
    latSets[sNew].append(latSets[s1]);
  }
  return sNew;
}

LatSetId Merger::mapSet(TensorExp::Kind kind, LatSetId s0, Value v,
                        Operation *op, Attribute a) {
  assert((TensorExp::Kind::kAbsF <= kind && kind <= TensorExp::Kind::kSelect) ||
         TensorExp::Kind::kDenseOp == kind);
  const LatSetId sNew = addSet();
  auto &setNew = latSets[sNew];
  for (const LatPointId p : set(s0)) {
    const auto &point = latPoints[p];
````
- **L385 EN**: Initializes variable `sNew` from the right-hand expression.
  **L385 CN**: 使用右侧表达式初始化变量 `sNew`。
- **L386 EN**: Comment explains nearby logic, invariants, or intent: `Left Region.`.
  **L386 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Left Region.`。
- **L387 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L387 CN**: 开始 `if` 控制流语句并计算其条件。
- **L388 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L388 CN**: 开始 `if` 控制流语句并计算其条件。
- **L389 EN**: Executes a call or declaration centered on `mapSet`.
  **L389 CN**: 执行以 `mapSet` 为核心的调用或声明。
- **L390 EN**: Executes a call or declaration centered on `latSets[sNew].append`.
  **L390 CN**: 执行以 `latSets[sNew].append` 为核心的调用或声明。
- **L391 EN**: Closes the current lexical scope or compound statement.
  **L391 CN**: 结束当前词法作用域或复合语句块。
- **L392 EN**: Comment explains nearby logic, invariants, or intent: `Right Region.`.
  **L392 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Right Region.`。
- **L393 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L393 CN**: 开始 `if` 控制流语句并计算其条件。
- **L394 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L394 CN**: 开始 `if` 控制流语句并计算其条件。
- **L395 EN**: Executes a call or declaration centered on `mapSet`.
  **L395 CN**: 执行以 `mapSet` 为核心的调用或声明。
- **L396 EN**: Executes a call or declaration centered on `latSets[sNew].append`.
  **L396 CN**: 执行以 `latSets[sNew].append` 为核心的调用或声明。
- **L397 EN**: Closes the current lexical scope or compound statement.
  **L397 CN**: 结束当前词法作用域或复合语句块。
- **L398 EN**: Returns from the current function with `sNew`.
  **L398 CN**: 以 `sNew` 从当前函数返回。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LatSetId Merger::mapSet(TensorExp::Kind kind, LatSetId s0, Value v,`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`LatSetId Merger::mapSet(TensorExp::Kind kind, LatSetId s0, Value v,`。
- **L402 EN**: Continues the surrounding expression or declaration: `Operation *op, Attribute a) {`.
  **L402 CN**: 继续构造周围的表达式或声明：`Operation *op, Attribute a) {`。
- **L403 EN**: Checks an internal invariant in debug builds.
  **L403 CN**: 在调试构建中检查内部不变式。
- **L404 EN**: Executes a standalone statement or declaration: `TensorExp::Kind::kDenseOp == kind);`.
  **L404 CN**: 执行一条独立语句或声明：`TensorExp::Kind::kDenseOp == kind);`。
- **L405 EN**: Initializes variable `sNew` from the right-hand expression.
  **L405 CN**: 使用右侧表达式初始化变量 `sNew`。
- **L406 EN**: Executes a standalone statement or declaration: `auto &setNew = latSets[sNew];`.
  **L406 CN**: 执行一条独立语句或声明：`auto &setNew = latSets[sNew];`。
- **L407 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L407 CN**: 开始 `for` 控制流语句并计算其条件。
- **L408 EN**: Executes a standalone statement or declaration: `const auto &point = latPoints[p];`.
  **L408 CN**: 执行一条独立语句或声明：`const auto &point = latPoints[p];`。

### Lines 409-432

````cpp
    setNew.push_back(addLat(point.bits, addExp(kind, point.exp, v, op, a)));
  }
  return sNew;
}

LatSetId Merger::mapBinWithSynZeroSet(ExprId e, LatSetId s0, bool lhsZero) {
  TensorExp::Kind kind = exp(e).kind;
  Attribute a = exp(e).attr;
  assert(TensorExp::Kind::kMulF <= kind && kind <= TensorExp::Kind::kShlI);
  // Must be a binary operation.
  const LatSetId sNew = addSet();
  auto &setNew = latSets[sNew];
  const ExprId zeroExp = addSynZeroExp();
  for (const LatPointId p : set(s0)) {
    const auto &point = latPoints[p];
    ExprId newExp = lhsZero ? addExp(kind, zeroExp, point.exp, nullptr, a)
                            : addExp(kind, point.exp, zeroExp, nullptr, a);
    setNew.push_back(addLat(point.bits, newExp));
  }
  return sNew;
}

LatSetId Merger::optimizeSet(LatSetId s0) {
  const LatSetId sNew = addSet();
````
- **L409 EN**: Executes a call or declaration centered on `setNew.push_back`.
  **L409 CN**: 执行以 `setNew.push_back` 为核心的调用或声明。
- **L410 EN**: Closes the current lexical scope or compound statement.
  **L410 CN**: 结束当前词法作用域或复合语句块。
- **L411 EN**: Returns from the current function with `sNew`.
  **L411 CN**: 以 `sNew` 从当前函数返回。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L414 EN**: Starts a function, method, lambda, or structured scope: `LatSetId Merger::mapBinWithSynZeroSet(ExprId e, LatSetId s0, bool lhsZero) {`.
  **L414 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatSetId Merger::mapBinWithSynZeroSet(ExprId e, LatSetId s0, bool lhsZero) {`。
- **L415 EN**: Initializes variable `kind` from the right-hand expression.
  **L415 CN**: 使用右侧表达式初始化变量 `kind`。
- **L416 EN**: Initializes variable `a` from the right-hand expression.
  **L416 CN**: 使用右侧表达式初始化变量 `a`。
- **L417 EN**: Checks an internal invariant in debug builds.
  **L417 CN**: 在调试构建中检查内部不变式。
- **L418 EN**: Comment explains nearby logic, invariants, or intent: `Must be a binary operation.`.
  **L418 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Must be a binary operation.`。
- **L419 EN**: Initializes variable `sNew` from the right-hand expression.
  **L419 CN**: 使用右侧表达式初始化变量 `sNew`。
- **L420 EN**: Executes a standalone statement or declaration: `auto &setNew = latSets[sNew];`.
  **L420 CN**: 执行一条独立语句或声明：`auto &setNew = latSets[sNew];`。
- **L421 EN**: Initializes variable `zeroExp` from the right-hand expression.
  **L421 CN**: 使用右侧表达式初始化变量 `zeroExp`。
- **L422 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L422 CN**: 开始 `for` 控制流语句并计算其条件。
- **L423 EN**: Executes a standalone statement or declaration: `const auto &point = latPoints[p];`.
  **L423 CN**: 执行一条独立语句或声明：`const auto &point = latPoints[p];`。
- **L424 EN**: Continues logic associated with callable symbol `addExp`.
  **L424 CN**: 继续与可调用符号 `addExp` 相关的逻辑。
- **L425 EN**: Executes a call or declaration centered on `addExp`.
  **L425 CN**: 执行以 `addExp` 为核心的调用或声明。
- **L426 EN**: Executes a call or declaration centered on `setNew.push_back`.
  **L426 CN**: 执行以 `setNew.push_back` 为核心的调用或声明。
- **L427 EN**: Closes the current lexical scope or compound statement.
  **L427 CN**: 结束当前词法作用域或复合语句块。
- **L428 EN**: Returns from the current function with `sNew`.
  **L428 CN**: 以 `sNew` 从当前函数返回。
- **L429 EN**: Closes the current lexical scope or compound statement.
  **L429 CN**: 结束当前词法作用域或复合语句块。
- **L430 EN**: Blank line separating nearby declarations or logic blocks.
  **L430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L431 EN**: Starts a function, method, lambda, or structured scope: `LatSetId Merger::optimizeSet(LatSetId s0) {`.
  **L431 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatSetId Merger::optimizeSet(LatSetId s0) {`。
- **L432 EN**: Initializes variable `sNew` from the right-hand expression.
  **L432 CN**: 使用右侧表达式初始化变量 `sNew`。

### Lines 433-456

````cpp
  auto &setNew = latSets[sNew];
  const auto &set0 = set(s0);
  assert(!set0.empty());
  const LatPointId p0 = set0[0];
  for (const LatPointId p1 : set0) {
    bool add = true;
    if (p0 != p1) {
      // Check whether this is a straightforward copy.
      if (expIsTensor(latPoints[p1].exp, outTensor))
        continue;
      // Check whether this conjunction is already covered.
      for (const LatPointId p2 : setNew) {
        assert(!latGT(p1, p2)); // Lj => Li would be bad
        if (onlyDenseDiff(p2, p1)) {
          add = false;
          break;
        }
      }
      assert(!add || latGT(p0, p1));
    }
    if (add)
      setNew.push_back(p1);
  }
  for (const LatPointId p : setNew)
````
- **L433 EN**: Executes a standalone statement or declaration: `auto &setNew = latSets[sNew];`.
  **L433 CN**: 执行一条独立语句或声明：`auto &setNew = latSets[sNew];`。
- **L434 EN**: Executes a call or declaration centered on `set`.
  **L434 CN**: 执行以 `set` 为核心的调用或声明。
- **L435 EN**: Checks an internal invariant in debug builds.
  **L435 CN**: 在调试构建中检查内部不变式。
- **L436 EN**: Initializes variable `p0` from the right-hand expression.
  **L436 CN**: 使用右侧表达式初始化变量 `p0`。
- **L437 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L437 CN**: 开始 `for` 控制流语句并计算其条件。
- **L438 EN**: Initializes variable `add` from the right-hand expression.
  **L438 CN**: 使用右侧表达式初始化变量 `add`。
- **L439 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L439 CN**: 开始 `if` 控制流语句并计算其条件。
- **L440 EN**: Comment explains nearby logic, invariants, or intent: `Check whether this is a straightforward copy.`.
  **L440 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check whether this is a straightforward copy.`。
- **L441 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L441 CN**: 开始 `if` 控制流语句并计算其条件。
- **L442 EN**: Skips to the next loop iteration.
  **L442 CN**: 跳到下一次循环迭代。
- **L443 EN**: Comment explains nearby logic, invariants, or intent: `Check whether this conjunction is already covered.`.
  **L443 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check whether this conjunction is already covered.`。
- **L444 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L444 CN**: 开始 `for` 控制流语句并计算其条件。
- **L445 EN**: Checks an internal invariant in debug builds.
  **L445 CN**: 在调试构建中检查内部不变式。
- **L446 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L446 CN**: 开始 `if` 控制流语句并计算其条件。
- **L447 EN**: Executes a standalone statement or declaration: `add = false;`.
  **L447 CN**: 执行一条独立语句或声明：`add = false;`。
- **L448 EN**: Exits the nearest loop or switch statement.
  **L448 CN**: 退出最近的循环或 switch 语句。
- **L449 EN**: Closes the current lexical scope or compound statement.
  **L449 CN**: 结束当前词法作用域或复合语句块。
- **L450 EN**: Closes the current lexical scope or compound statement.
  **L450 CN**: 结束当前词法作用域或复合语句块。
- **L451 EN**: Checks an internal invariant in debug builds.
  **L451 CN**: 在调试构建中检查内部不变式。
- **L452 EN**: Closes the current lexical scope or compound statement.
  **L452 CN**: 结束当前词法作用域或复合语句块。
- **L453 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L453 CN**: 开始 `if` 控制流语句并计算其条件。
- **L454 EN**: Executes a call or declaration centered on `setNew.push_back`.
  **L454 CN**: 执行以 `setNew.push_back` 为核心的调用或声明。
- **L455 EN**: Closes the current lexical scope or compound statement.
  **L455 CN**: 结束当前词法作用域或复合语句块。
- **L456 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L456 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 457-480

````cpp
    latPoints[p].simple = simplifyCond(sNew, p);
  return sNew;
}

BitVector Merger::simplifyCond(LatSetId s0, LatPointId p0) {
  // First determine if this lattice point is a *singleton*, i.e.,
  // the last point in a lattice, no other is less than this one.
  bool isSingleton = true;
  for (const LatPointId p1 : set(s0)) {
    if (p0 != p1 && latGT(p0, p1)) {
      isSingleton = false;
      break;
    }
  }

  BitVector simple(latPoints[p0].bits);
  bool reset = isSingleton && hasAnySparse(simple);
  const TensorLoopId be = simple.size();
  TensorLoopId offset = 0; // relative to the end
  if (!reset)
    // Starts resetting from a dense level, so that the first bit (if kept)
    // is not undefined level-type.
    for (unsigned b = 0; b < be; b++) {
      if (simple[b] && getLvlType(TensorLoopId{b}).hasDenseSemantic()) {
````
- **L457 EN**: Executes a call or declaration centered on `simplifyCond`.
  **L457 CN**: 执行以 `simplifyCond` 为核心的调用或声明。
- **L458 EN**: Returns from the current function with `sNew`.
  **L458 CN**: 以 `sNew` 从当前函数返回。
- **L459 EN**: Closes the current lexical scope or compound statement.
  **L459 CN**: 结束当前词法作用域或复合语句块。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L461 EN**: Starts a function, method, lambda, or structured scope: `BitVector Merger::simplifyCond(LatSetId s0, LatPointId p0) {`.
  **L461 CN**: 开始一个函数、方法、lambda 或结构化作用域：`BitVector Merger::simplifyCond(LatSetId s0, LatPointId p0) {`。
- **L462 EN**: Comment explains nearby logic, invariants, or intent: `First determine if this lattice point is a *singleton*, i.e.,`.
  **L462 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`First determine if this lattice point is a *singleton*, i.e.,`。
- **L463 EN**: Comment explains nearby logic, invariants, or intent: `the last point in a lattice, no other is less than this one.`.
  **L463 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the last point in a lattice, no other is less than this one.`。
- **L464 EN**: Initializes variable `isSingleton` from the right-hand expression.
  **L464 CN**: 使用右侧表达式初始化变量 `isSingleton`。
- **L465 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L465 CN**: 开始 `for` 控制流语句并计算其条件。
- **L466 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L466 CN**: 开始 `if` 控制流语句并计算其条件。
- **L467 EN**: Executes a standalone statement or declaration: `isSingleton = false;`.
  **L467 CN**: 执行一条独立语句或声明：`isSingleton = false;`。
- **L468 EN**: Exits the nearest loop or switch statement.
  **L468 CN**: 退出最近的循环或 switch 语句。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Closes the current lexical scope or compound statement.
  **L470 CN**: 结束当前词法作用域或复合语句块。
- **L471 EN**: Blank line separating nearby declarations or logic blocks.
  **L471 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L472 EN**: Executes a call or declaration centered on `simple`.
  **L472 CN**: 执行以 `simple` 为核心的调用或声明。
- **L473 EN**: Initializes variable `reset` from the right-hand expression.
  **L473 CN**: 使用右侧表达式初始化变量 `reset`。
- **L474 EN**: Initializes variable `be` from the right-hand expression.
  **L474 CN**: 使用右侧表达式初始化变量 `be`。
- **L475 EN**: Continues the surrounding expression or declaration: `TensorLoopId offset = 0; // relative to the end`.
  **L475 CN**: 继续构造周围的表达式或声明：`TensorLoopId offset = 0; // relative to the end`。
- **L476 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L476 CN**: 开始 `if` 控制流语句并计算其条件。
- **L477 EN**: Comment explains nearby logic, invariants, or intent: `Starts resetting from a dense level, so that the first bit (if kept)`.
  **L477 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Starts resetting from a dense level, so that the first bit (if kept)`。
- **L478 EN**: Comment explains nearby logic, invariants, or intent: `is not undefined level-type.`.
  **L478 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is not undefined level-type.`。
- **L479 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L479 CN**: 开始 `for` 控制流语句并计算其条件。
- **L480 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L480 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 481-504

````cpp
        offset = be - b - 1; // relative to the end
        break;
      }
    }

  // Now apply the two basic rules. We also iterate the bits reversely to always
  // keep the rightmost bit (which could possibly be a synthetic tensor).
  for (unsigned b = be - 1 - offset, i = 0; i < be;
       b = b == 0 ? be - 1 : b - 1, i++) {
    // Slice on dense level has `locate` property as well, and can be optimized.
    if (simple[b] && !isSparseLvlWithNonTrivialIdxExp(b)) {
      const auto lt = getLvlType(b);
      if (!lt.hasSparseSemantic()) {
        if (reset)
          simple.reset(b);
        reset = true;
      }
    }
  }
  return simple;
}

bool Merger::latGT(LatPointId i, LatPointId j) const {
  const BitVector &bitsi = lat(i).bits;
````
- **L481 EN**: Continues the surrounding expression or declaration: `offset = be - b - 1; // relative to the end`.
  **L481 CN**: 继续构造周围的表达式或声明：`offset = be - b - 1; // relative to the end`。
- **L482 EN**: Exits the nearest loop or switch statement.
  **L482 CN**: 退出最近的循环或 switch 语句。
- **L483 EN**: Closes the current lexical scope or compound statement.
  **L483 CN**: 结束当前词法作用域或复合语句块。
- **L484 EN**: Closes the current lexical scope or compound statement.
  **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Blank line separating nearby declarations or logic blocks.
  **L485 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L486 EN**: Comment explains nearby logic, invariants, or intent: `Now apply the two basic rules. We also iterate the bits reversely to always`.
  **L486 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Now apply the two basic rules. We also iterate the bits reversely to always`。
- **L487 EN**: Comment explains nearby logic, invariants, or intent: `keep the rightmost bit (which could possibly be a synthetic tensor).`.
  **L487 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`keep the rightmost bit (which could possibly be a synthetic tensor).`。
- **L488 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L488 CN**: 开始 `for` 控制流语句并计算其条件。
- **L489 EN**: Continues the surrounding expression or declaration: `b = b == 0 ? be - 1 : b - 1, i++) {`.
  **L489 CN**: 继续构造周围的表达式或声明：`b = b == 0 ? be - 1 : b - 1, i++) {`。
- **L490 EN**: Comment explains nearby logic, invariants, or intent: `Slice on dense level has `locate` property as well, and can be optimized.`.
  **L490 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Slice on dense level has `locate` property as well, and can be optimized.`。
- **L491 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L491 CN**: 开始 `if` 控制流语句并计算其条件。
- **L492 EN**: Initializes variable `lt` from the right-hand expression.
  **L492 CN**: 使用右侧表达式初始化变量 `lt`。
- **L493 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L493 CN**: 开始 `if` 控制流语句并计算其条件。
- **L494 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L494 CN**: 开始 `if` 控制流语句并计算其条件。
- **L495 EN**: Executes a call or declaration centered on `simple.reset`.
  **L495 CN**: 执行以 `simple.reset` 为核心的调用或声明。
- **L496 EN**: Executes a standalone statement or declaration: `reset = true;`.
  **L496 CN**: 执行一条独立语句或声明：`reset = true;`。
- **L497 EN**: Closes the current lexical scope or compound statement.
  **L497 CN**: 结束当前词法作用域或复合语句块。
- **L498 EN**: Closes the current lexical scope or compound statement.
  **L498 CN**: 结束当前词法作用域或复合语句块。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Returns from the current function with `simple`.
  **L500 CN**: 以 `simple` 从当前函数返回。
- **L501 EN**: Closes the current lexical scope or compound statement.
  **L501 CN**: 结束当前词法作用域或复合语句块。
- **L502 EN**: Blank line separating nearby declarations or logic blocks.
  **L502 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L503 EN**: Starts a function, method, lambda, or structured scope: `bool Merger::latGT(LatPointId i, LatPointId j) const {`.
  **L503 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Merger::latGT(LatPointId i, LatPointId j) const {`。
- **L504 EN**: Executes a call or declaration centered on `lat`.
  **L504 CN**: 执行以 `lat` 为核心的调用或声明。

### Lines 505-528

````cpp
  const BitVector &bitsj = lat(j).bits;
  assert(bitsi.size() == bitsj.size());
  if (bitsi.count() > bitsj.count()) {
    for (TensorLoopId b = 0, be = bitsj.size(); b < be; b++)
      if (bitsj[b] && !bitsi[b])
        return false;
    return true;
  }
  return false;
}

bool Merger::onlyDenseDiff(LatPointId i, LatPointId j) const {
  BitVector tmp(latPoints[j].bits);
  tmp ^= latPoints[i].bits;
  return !hasAnySparse(tmp);
}

bool Merger::expContainsTensor(ExprId e, TensorId t) const {
  const auto &expr = exp(e);
  // First we check `expIsTensor`.
  if (expr.kind == TensorExp::Kind::kTensor)
    return expr.tensor == t;

  switch (getExpArity(expr.kind)) {
````
- **L505 EN**: Executes a call or declaration centered on `lat`.
  **L505 CN**: 执行以 `lat` 为核心的调用或声明。
- **L506 EN**: Checks an internal invariant in debug builds.
  **L506 CN**: 在调试构建中检查内部不变式。
- **L507 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L507 CN**: 开始 `if` 控制流语句并计算其条件。
- **L508 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L508 CN**: 开始 `for` 控制流语句并计算其条件。
- **L509 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L509 CN**: 开始 `if` 控制流语句并计算其条件。
- **L510 EN**: Returns from the current function with `false`.
  **L510 CN**: 以 `false` 从当前函数返回。
- **L511 EN**: Returns from the current function with `true`.
  **L511 CN**: 以 `true` 从当前函数返回。
- **L512 EN**: Closes the current lexical scope or compound statement.
  **L512 CN**: 结束当前词法作用域或复合语句块。
- **L513 EN**: Returns from the current function with `false`.
  **L513 CN**: 以 `false` 从当前函数返回。
- **L514 EN**: Closes the current lexical scope or compound statement.
  **L514 CN**: 结束当前词法作用域或复合语句块。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L516 EN**: Starts a function, method, lambda, or structured scope: `bool Merger::onlyDenseDiff(LatPointId i, LatPointId j) const {`.
  **L516 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Merger::onlyDenseDiff(LatPointId i, LatPointId j) const {`。
- **L517 EN**: Executes a call or declaration centered on `tmp`.
  **L517 CN**: 执行以 `tmp` 为核心的调用或声明。
- **L518 EN**: Executes a standalone statement or declaration: `tmp ^= latPoints[i].bits;`.
  **L518 CN**: 执行一条独立语句或声明：`tmp ^= latPoints[i].bits;`。
- **L519 EN**: Returns from the current function with `!hasAnySparse(tmp)`.
  **L519 CN**: 以 `!hasAnySparse(tmp)` 从当前函数返回。
- **L520 EN**: Closes the current lexical scope or compound statement.
  **L520 CN**: 结束当前词法作用域或复合语句块。
- **L521 EN**: Blank line separating nearby declarations or logic blocks.
  **L521 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L522 EN**: Starts a function, method, lambda, or structured scope: `bool Merger::expContainsTensor(ExprId e, TensorId t) const {`.
  **L522 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Merger::expContainsTensor(ExprId e, TensorId t) const {`。
- **L523 EN**: Executes a call or declaration centered on `exp`.
  **L523 CN**: 执行以 `exp` 为核心的调用或声明。
- **L524 EN**: Comment explains nearby logic, invariants, or intent: `First we check `expIsTensor`.`.
  **L524 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`First we check `expIsTensor`.`。
- **L525 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L525 CN**: 开始 `if` 控制流语句并计算其条件。
- **L526 EN**: Returns from the current function with `expr.tensor == t`.
  **L526 CN**: 以 `expr.tensor == t` 从当前函数返回。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L528 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L528 CN**: 开始 `switch` 控制流语句并计算其条件。

### Lines 529-552

````cpp
  case ExpArity::kNullary:
    return false;
  case ExpArity::kUnary: {
    const ExprId e0 = expr.children.e0;
    return expContainsTensor(e0, t);
  }
  case ExpArity::kBinary: {
    const ExprId e0 = expr.children.e0;
    const ExprId e1 = expr.children.e1;
    return expContainsTensor(e0, t) || expContainsTensor(e1, t);
  }
  }
  llvm_unreachable("unexpected arity");
}

bool Merger::hasNegateOnOut(ExprId e) const {
  const auto &expr = exp(e);
  switch (expr.kind) {
  case TensorExp::Kind::kNegF:
  case TensorExp::Kind::kNegC:
  case TensorExp::Kind::kNegI:
    return expContainsTensor(expr.children.e0, outTensor);
  case TensorExp::Kind::kSubF:
  case TensorExp::Kind::kSubC:
````
- **L529 EN**: Introduces a switch dispatch label: `case ExpArity::kNullary:`.
  **L529 CN**: 引入一个 switch 分发标签：`case ExpArity::kNullary:`。
- **L530 EN**: Returns from the current function with `false`.
  **L530 CN**: 以 `false` 从当前函数返回。
- **L531 EN**: Introduces a switch dispatch label: `case ExpArity::kUnary: {`.
  **L531 CN**: 引入一个 switch 分发标签：`case ExpArity::kUnary: {`。
- **L532 EN**: Initializes variable `e0` from the right-hand expression.
  **L532 CN**: 使用右侧表达式初始化变量 `e0`。
- **L533 EN**: Returns from the current function with `expContainsTensor(e0, t)`.
  **L533 CN**: 以 `expContainsTensor(e0, t)` 从当前函数返回。
- **L534 EN**: Closes the current lexical scope or compound statement.
  **L534 CN**: 结束当前词法作用域或复合语句块。
- **L535 EN**: Introduces a switch dispatch label: `case ExpArity::kBinary: {`.
  **L535 CN**: 引入一个 switch 分发标签：`case ExpArity::kBinary: {`。
- **L536 EN**: Initializes variable `e0` from the right-hand expression.
  **L536 CN**: 使用右侧表达式初始化变量 `e0`。
- **L537 EN**: Initializes variable `e1` from the right-hand expression.
  **L537 CN**: 使用右侧表达式初始化变量 `e1`。
- **L538 EN**: Returns from the current function with `expContainsTensor(e0, t) || expContainsTensor(e1, t)`.
  **L538 CN**: 以 `expContainsTensor(e0, t) || expContainsTensor(e1, t)` 从当前函数返回。
- **L539 EN**: Closes the current lexical scope or compound statement.
  **L539 CN**: 结束当前词法作用域或复合语句块。
- **L540 EN**: Closes the current lexical scope or compound statement.
  **L540 CN**: 结束当前词法作用域或复合语句块。
- **L541 EN**: Marks this control path as unreachable.
  **L541 CN**: 将该控制路径标记为不可达。
- **L542 EN**: Closes the current lexical scope or compound statement.
  **L542 CN**: 结束当前词法作用域或复合语句块。
- **L543 EN**: Blank line separating nearby declarations or logic blocks.
  **L543 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L544 EN**: Starts a function, method, lambda, or structured scope: `bool Merger::hasNegateOnOut(ExprId e) const {`.
  **L544 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Merger::hasNegateOnOut(ExprId e) const {`。
- **L545 EN**: Executes a call or declaration centered on `exp`.
  **L545 CN**: 执行以 `exp` 为核心的调用或声明。
- **L546 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L546 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L547 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegF:`.
  **L547 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegF:`。
- **L548 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegC:`.
  **L548 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegC:`。
- **L549 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegI:`.
  **L549 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegI:`。
- **L550 EN**: Returns from the current function with `expContainsTensor(expr.children.e0, outTensor)`.
  **L550 CN**: 以 `expContainsTensor(expr.children.e0, outTensor)` 从当前函数返回。
- **L551 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubF:`.
  **L551 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubF:`。
- **L552 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubC:`.
  **L552 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubC:`。

### Lines 553-576

````cpp
  case TensorExp::Kind::kSubI:
    return expContainsTensor(expr.children.e1, outTensor) ||
           hasNegateOnOut(expr.children.e0);
  case TensorExp::Kind::kDenseOp: {
    bool lhsNeg = hasNegateOnOut(expr.children.e0);
    if (!lhsNeg && expr.children.e1 != detail::kInvalidId)
      return hasNegateOnOut(expr.children.e1);
    return lhsNeg;
  }
  default: {
    switch (getExpArity(expr.kind)) {
    case ExpArity::kNullary:
      return false;
    case ExpArity::kUnary:
      return hasNegateOnOut(expr.children.e0);
    case ExpArity::kBinary:
      return hasNegateOnOut(expr.children.e0) ||
             hasNegateOnOut(expr.children.e1);
    }
  }
  }
  llvm_unreachable("unexpected kind");
}

````
- **L553 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubI:`.
  **L553 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubI:`。
- **L554 EN**: Returns from the current function with `expContainsTensor(expr.children.e1, outTensor) ||`.
  **L554 CN**: 以 `expContainsTensor(expr.children.e1, outTensor) ||` 从当前函数返回。
- **L555 EN**: Executes a call or declaration centered on `hasNegateOnOut`.
  **L555 CN**: 执行以 `hasNegateOnOut` 为核心的调用或声明。
- **L556 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDenseOp: {`.
  **L556 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDenseOp: {`。
- **L557 EN**: Initializes variable `lhsNeg` from the right-hand expression.
  **L557 CN**: 使用右侧表达式初始化变量 `lhsNeg`。
- **L558 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L558 CN**: 开始 `if` 控制流语句并计算其条件。
- **L559 EN**: Returns from the current function with `hasNegateOnOut(expr.children.e1)`.
  **L559 CN**: 以 `hasNegateOnOut(expr.children.e1)` 从当前函数返回。
- **L560 EN**: Returns from the current function with `lhsNeg`.
  **L560 CN**: 以 `lhsNeg` 从当前函数返回。
- **L561 EN**: Closes the current lexical scope or compound statement.
  **L561 CN**: 结束当前词法作用域或复合语句块。
- **L562 EN**: Introduces a switch dispatch label: `default: {`.
  **L562 CN**: 引入一个 switch 分发标签：`default: {`。
- **L563 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L563 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L564 EN**: Introduces a switch dispatch label: `case ExpArity::kNullary:`.
  **L564 CN**: 引入一个 switch 分发标签：`case ExpArity::kNullary:`。
- **L565 EN**: Returns from the current function with `false`.
  **L565 CN**: 以 `false` 从当前函数返回。
- **L566 EN**: Introduces a switch dispatch label: `case ExpArity::kUnary:`.
  **L566 CN**: 引入一个 switch 分发标签：`case ExpArity::kUnary:`。
- **L567 EN**: Returns from the current function with `hasNegateOnOut(expr.children.e0)`.
  **L567 CN**: 以 `hasNegateOnOut(expr.children.e0)` 从当前函数返回。
- **L568 EN**: Introduces a switch dispatch label: `case ExpArity::kBinary:`.
  **L568 CN**: 引入一个 switch 分发标签：`case ExpArity::kBinary:`。
- **L569 EN**: Returns from the current function with `hasNegateOnOut(expr.children.e0) ||`.
  **L569 CN**: 以 `hasNegateOnOut(expr.children.e0) ||` 从当前函数返回。
- **L570 EN**: Executes a call or declaration centered on `hasNegateOnOut`.
  **L570 CN**: 执行以 `hasNegateOnOut` 为核心的调用或声明。
- **L571 EN**: Closes the current lexical scope or compound statement.
  **L571 CN**: 结束当前词法作用域或复合语句块。
- **L572 EN**: Closes the current lexical scope or compound statement.
  **L572 CN**: 结束当前词法作用域或复合语句块。
- **L573 EN**: Closes the current lexical scope or compound statement.
  **L573 CN**: 结束当前词法作用域或复合语句块。
- **L574 EN**: Marks this control path as unreachable.
  **L574 CN**: 将该控制路径标记为不可达。
- **L575 EN**: Closes the current lexical scope or compound statement.
  **L575 CN**: 结束当前词法作用域或复合语句块。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 577-600

````cpp
bool Merger::isSingleCondition(TensorId t, ExprId e) const {
  assert(isValidTensorId(t));
  const auto &expr = exp(e);
  switch (expr.kind) {
  // Leaf.
  case TensorExp::Kind::kTensor:
    return expr.tensor == t;
  case TensorExp::Kind::kInvariant:
  case TensorExp::Kind::kLoopVar:
  case TensorExp::Kind::kSynZero:
    return false;
  // Unary operations.
  case TensorExp::Kind::kAbsF:
  case TensorExp::Kind::kAbsC:
  case TensorExp::Kind::kAbsI:
  case TensorExp::Kind::kCeilF:
  case TensorExp::Kind::kFloorF:
  case TensorExp::Kind::kSqrtF:
  case TensorExp::Kind::kSqrtC:
  case TensorExp::Kind::kExpm1F:
  case TensorExp::Kind::kExpm1C:
  case TensorExp::Kind::kLog1pF:
  case TensorExp::Kind::kLog1pC:
  case TensorExp::Kind::kRelu:
````
- **L577 EN**: Starts a function, method, lambda, or structured scope: `bool Merger::isSingleCondition(TensorId t, ExprId e) const {`.
  **L577 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Merger::isSingleCondition(TensorId t, ExprId e) const {`。
- **L578 EN**: Checks an internal invariant in debug builds.
  **L578 CN**: 在调试构建中检查内部不变式。
- **L579 EN**: Executes a call or declaration centered on `exp`.
  **L579 CN**: 执行以 `exp` 为核心的调用或声明。
- **L580 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L580 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L581 EN**: Comment explains nearby logic, invariants, or intent: `Leaf.`.
  **L581 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Leaf.`。
- **L582 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTensor:`.
  **L582 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTensor:`。
- **L583 EN**: Returns from the current function with `expr.tensor == t`.
  **L583 CN**: 以 `expr.tensor == t` 从当前函数返回。
- **L584 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kInvariant:`.
  **L584 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kInvariant:`。
- **L585 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLoopVar:`.
  **L585 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLoopVar:`。
- **L586 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSynZero:`.
  **L586 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSynZero:`。
- **L587 EN**: Returns from the current function with `false`.
  **L587 CN**: 以 `false` 从当前函数返回。
- **L588 EN**: Comment explains nearby logic, invariants, or intent: `Unary operations.`.
  **L588 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unary operations.`。
- **L589 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsF:`.
  **L589 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsF:`。
- **L590 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsC:`.
  **L590 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsC:`。
- **L591 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsI:`.
  **L591 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsI:`。
- **L592 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCeilF:`.
  **L592 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCeilF:`。
- **L593 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kFloorF:`.
  **L593 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kFloorF:`。
- **L594 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtF:`.
  **L594 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtF:`。
- **L595 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtC:`.
  **L595 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtC:`。
- **L596 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1F:`.
  **L596 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1F:`。
- **L597 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1C:`.
  **L597 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1C:`。
- **L598 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pF:`.
  **L598 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pF:`。
- **L599 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pC:`.
  **L599 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pC:`。
- **L600 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kRelu:`.
  **L600 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kRelu:`。

### Lines 601-624

````cpp
  case TensorExp::Kind::kSinF:
  case TensorExp::Kind::kSinC:
  case TensorExp::Kind::kTanhF:
  case TensorExp::Kind::kTanhC:
  case TensorExp::Kind::kNegF:
  case TensorExp::Kind::kNegC:
  case TensorExp::Kind::kNegI:
  case TensorExp::Kind::kTruncF:
  case TensorExp::Kind::kExtF:
  case TensorExp::Kind::kCastFS:
  case TensorExp::Kind::kCastFU:
  case TensorExp::Kind::kCastSF:
  case TensorExp::Kind::kCastUF:
  case TensorExp::Kind::kCastS:
  case TensorExp::Kind::kCastU:
  case TensorExp::Kind::kCastIdx:
  case TensorExp::Kind::kTruncI:
  case TensorExp::Kind::kCIm:
  case TensorExp::Kind::kCRe:
  case TensorExp::Kind::kBitCast:
  case TensorExp::Kind::kUnary:
    return isSingleCondition(t, expr.children.e0);
  case TensorExp::Kind::kBinaryBranch:
  case TensorExp::Kind::kSelect:
````
- **L601 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinF:`.
  **L601 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinF:`。
- **L602 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinC:`.
  **L602 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinC:`。
- **L603 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhF:`.
  **L603 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhF:`。
- **L604 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhC:`.
  **L604 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhC:`。
- **L605 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegF:`.
  **L605 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegF:`。
- **L606 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegC:`.
  **L606 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegC:`。
- **L607 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegI:`.
  **L607 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegI:`。
- **L608 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncF:`.
  **L608 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncF:`。
- **L609 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExtF:`.
  **L609 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExtF:`。
- **L610 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFS:`.
  **L610 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFS:`。
- **L611 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFU:`.
  **L611 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFU:`。
- **L612 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastSF:`.
  **L612 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastSF:`。
- **L613 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastUF:`.
  **L613 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastUF:`。
- **L614 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastS:`.
  **L614 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastS:`。
- **L615 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastU:`.
  **L615 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastU:`。
- **L616 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastIdx:`.
  **L616 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastIdx:`。
- **L617 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncI:`.
  **L617 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncI:`。
- **L618 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCIm:`.
  **L618 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCIm:`。
- **L619 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCRe:`.
  **L619 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCRe:`。
- **L620 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBitCast:`.
  **L620 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBitCast:`。
- **L621 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kUnary:`.
  **L621 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kUnary:`。
- **L622 EN**: Returns from the current function with `isSingleCondition(t, expr.children.e0)`.
  **L622 CN**: 以 `isSingleCondition(t, expr.children.e0)` 从当前函数返回。
- **L623 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinaryBranch:`.
  **L623 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinaryBranch:`。
- **L624 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSelect:`.
  **L624 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSelect:`。

### Lines 625-648

````cpp
    return false;
  // Binary operations.
  case TensorExp::Kind::kDivF: // note: x / c only
  case TensorExp::Kind::kDivC:
  case TensorExp::Kind::kDivS:
  case TensorExp::Kind::kDivU:
    assert(!maybeZero(expr.children.e1));
    return isSingleCondition(t, expr.children.e0);
  case TensorExp::Kind::kShrS: // note: x >> inv only
  case TensorExp::Kind::kShrU:
  case TensorExp::Kind::kShlI:
    assert(isInvariant(expr.children.e1));
    return isSingleCondition(t, expr.children.e0);
  case TensorExp::Kind::kMulF:
  case TensorExp::Kind::kMulC:
  case TensorExp::Kind::kMulI:
  case TensorExp::Kind::kAndI:
  case TensorExp::Kind::kReduce:
    if (isSingleCondition(t, expr.children.e0))
      return isSingleCondition(t, expr.children.e1) ||
             isInvariant(expr.children.e1);
    if (isSingleCondition(t, expr.children.e1))
      return isInvariant(expr.children.e0);
    return false;
````
- **L625 EN**: Returns from the current function with `false`.
  **L625 CN**: 以 `false` 从当前函数返回。
- **L626 EN**: Comment explains nearby logic, invariants, or intent: `Binary operations.`.
  **L626 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Binary operations.`。
- **L627 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivF: // note: x / c only`.
  **L627 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivF: // note: x / c only`。
- **L628 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivC:`.
  **L628 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivC:`。
- **L629 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivS:`.
  **L629 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivS:`。
- **L630 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivU:`.
  **L630 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivU:`。
- **L631 EN**: Checks an internal invariant in debug builds.
  **L631 CN**: 在调试构建中检查内部不变式。
- **L632 EN**: Returns from the current function with `isSingleCondition(t, expr.children.e0)`.
  **L632 CN**: 以 `isSingleCondition(t, expr.children.e0)` 从当前函数返回。
- **L633 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrS: // note: x >> inv only`.
  **L633 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrS: // note: x >> inv only`。
- **L634 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrU:`.
  **L634 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrU:`。
- **L635 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShlI:`.
  **L635 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShlI:`。
- **L636 EN**: Checks an internal invariant in debug builds.
  **L636 CN**: 在调试构建中检查内部不变式。
- **L637 EN**: Returns from the current function with `isSingleCondition(t, expr.children.e0)`.
  **L637 CN**: 以 `isSingleCondition(t, expr.children.e0)` 从当前函数返回。
- **L638 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulF:`.
  **L638 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulF:`。
- **L639 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulC:`.
  **L639 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulC:`。
- **L640 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulI:`.
  **L640 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulI:`。
- **L641 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAndI:`.
  **L641 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAndI:`。
- **L642 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kReduce:`.
  **L642 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kReduce:`。
- **L643 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L643 CN**: 开始 `if` 控制流语句并计算其条件。
- **L644 EN**: Returns from the current function with `isSingleCondition(t, expr.children.e1) ||`.
  **L644 CN**: 以 `isSingleCondition(t, expr.children.e1) ||` 从当前函数返回。
- **L645 EN**: Executes a call or declaration centered on `isInvariant`.
  **L645 CN**: 执行以 `isInvariant` 为核心的调用或声明。
- **L646 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L646 CN**: 开始 `if` 控制流语句并计算其条件。
- **L647 EN**: Returns from the current function with `isInvariant(expr.children.e0)`.
  **L647 CN**: 以 `isInvariant(expr.children.e0)` 从当前函数返回。
- **L648 EN**: Returns from the current function with `false`.
  **L648 CN**: 以 `false` 从当前函数返回。

### Lines 649-672

````cpp
  case TensorExp::Kind::kAddF:
  case TensorExp::Kind::kAddC:
  case TensorExp::Kind::kAddI:
    return isSingleCondition(t, expr.children.e0) &&
           isSingleCondition(t, expr.children.e1);
  case TensorExp::Kind::kSubF:
  case TensorExp::Kind::kSubC:
  case TensorExp::Kind::kSubI:
  case TensorExp::Kind::kOrI:
  case TensorExp::Kind::kXorI:
  case TensorExp::Kind::kCmpF:
  case TensorExp::Kind::kCmpI:
  case TensorExp::Kind::kBinary:
    return false;
  case TensorExp::Kind::kDenseOp:
    // Since Merger guarantees all the operands of the kDenseOp to be dense, the
    // operation must be single-condition.
    return true;
  }
  llvm_unreachable("unexpected kind");
}

bool Merger::hasAnySparse(const BitVector &bits) const {
  for (TensorLoopId b : bits.set_bits()) {
````
- **L649 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddF:`.
  **L649 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddF:`。
- **L650 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddC:`.
  **L650 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddC:`。
- **L651 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddI:`.
  **L651 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddI:`。
- **L652 EN**: Returns from the current function with `isSingleCondition(t, expr.children.e0) &&`.
  **L652 CN**: 以 `isSingleCondition(t, expr.children.e0) &&` 从当前函数返回。
- **L653 EN**: Executes a call or declaration centered on `isSingleCondition`.
  **L653 CN**: 执行以 `isSingleCondition` 为核心的调用或声明。
- **L654 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubF:`.
  **L654 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubF:`。
- **L655 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubC:`.
  **L655 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubC:`。
- **L656 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubI:`.
  **L656 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubI:`。
- **L657 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kOrI:`.
  **L657 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kOrI:`。
- **L658 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kXorI:`.
  **L658 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kXorI:`。
- **L659 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpF:`.
  **L659 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpF:`。
- **L660 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpI:`.
  **L660 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpI:`。
- **L661 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinary:`.
  **L661 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinary:`。
- **L662 EN**: Returns from the current function with `false`.
  **L662 CN**: 以 `false` 从当前函数返回。
- **L663 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDenseOp:`.
  **L663 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDenseOp:`。
- **L664 EN**: Comment explains nearby logic, invariants, or intent: `Since Merger guarantees all the operands of the kDenseOp to be dense, the`.
  **L664 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Since Merger guarantees all the operands of the kDenseOp to be dense, the`。
- **L665 EN**: Comment explains nearby logic, invariants, or intent: `operation must be single-condition.`.
  **L665 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operation must be single-condition.`。
- **L666 EN**: Returns from the current function with `true`.
  **L666 CN**: 以 `true` 从当前函数返回。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Marks this control path as unreachable.
  **L668 CN**: 将该控制路径标记为不可达。
- **L669 EN**: Closes the current lexical scope or compound statement.
  **L669 CN**: 结束当前词法作用域或复合语句块。
- **L670 EN**: Blank line separating nearby declarations or logic blocks.
  **L670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L671 EN**: Starts a function, method, lambda, or structured scope: `bool Merger::hasAnySparse(const BitVector &bits) const {`.
  **L671 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Merger::hasAnySparse(const BitVector &bits) const {`。
- **L672 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L672 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 673-696

````cpp
    const auto lt = getLvlType(b);
    if (lt.hasSparseSemantic())
      return true;
  }
  return hasSparseIdxReduction(bits);
}

bool Merger::hasSparseIdxReduction(const BitVector &bits) const {
  for (TensorLoopId b : bits.set_bits())
    if (isSparseLvlWithNonTrivialIdxExp(b))
      return true;
  return false;
}

#ifndef NDEBUG

//===----------------------------------------------------------------------===//
// Print methods (for debugging).
//===----------------------------------------------------------------------===//

static const char *kindToOpSymbol(TensorExp::Kind kind) {
  switch (kind) {
  // Leaf.
  case TensorExp::Kind::kTensor:
````
- **L673 EN**: Initializes variable `lt` from the right-hand expression.
  **L673 CN**: 使用右侧表达式初始化变量 `lt`。
- **L674 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L674 CN**: 开始 `if` 控制流语句并计算其条件。
- **L675 EN**: Returns from the current function with `true`.
  **L675 CN**: 以 `true` 从当前函数返回。
- **L676 EN**: Closes the current lexical scope or compound statement.
  **L676 CN**: 结束当前词法作用域或复合语句块。
- **L677 EN**: Returns from the current function with `hasSparseIdxReduction(bits)`.
  **L677 CN**: 以 `hasSparseIdxReduction(bits)` 从当前函数返回。
- **L678 EN**: Closes the current lexical scope or compound statement.
  **L678 CN**: 结束当前词法作用域或复合语句块。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L680 EN**: Starts a function, method, lambda, or structured scope: `bool Merger::hasSparseIdxReduction(const BitVector &bits) const {`.
  **L680 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Merger::hasSparseIdxReduction(const BitVector &bits) const {`。
- **L681 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L681 CN**: 开始 `for` 控制流语句并计算其条件。
- **L682 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L682 CN**: 开始 `if` 控制流语句并计算其条件。
- **L683 EN**: Returns from the current function with `true`.
  **L683 CN**: 以 `true` 从当前函数返回。
- **L684 EN**: Returns from the current function with `false`.
  **L684 CN**: 以 `false` 从当前函数返回。
- **L685 EN**: Closes the current lexical scope or compound statement.
  **L685 CN**: 结束当前词法作用域或复合语句块。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L687 EN**: Starts a preprocessor conditional block: `#ifndef NDEBUG`.
  **L687 CN**: 开始一个预处理条件块：`#ifndef NDEBUG`。
- **L688 EN**: Blank line separating nearby declarations or logic blocks.
  **L688 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L689 EN**: Banner comment marking a file or section boundary.
  **L689 CN**: 横幅注释，用于标记文件或章节边界。
- **L690 EN**: Comment explains nearby logic, invariants, or intent: `Print methods (for debugging).`.
  **L690 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Print methods (for debugging).`。
- **L691 EN**: Banner comment marking a file or section boundary.
  **L691 CN**: 横幅注释，用于标记文件或章节边界。
- **L692 EN**: Blank line separating nearby declarations or logic blocks.
  **L692 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L693 EN**: Starts a function, method, lambda, or structured scope: `static const char *kindToOpSymbol(TensorExp::Kind kind) {`.
  **L693 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static const char *kindToOpSymbol(TensorExp::Kind kind) {`。
- **L694 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L694 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L695 EN**: Comment explains nearby logic, invariants, or intent: `Leaf.`.
  **L695 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Leaf.`。
- **L696 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTensor:`.
  **L696 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTensor:`。

### Lines 697-720

````cpp
    return "tensor";
  case TensorExp::Kind::kInvariant:
    return "invariant";
  case TensorExp::Kind::kLoopVar:
    return "index";
  case TensorExp::Kind::kSynZero:
    return "0";
  // Unary operations.
  case TensorExp::Kind::kAbsF:
  case TensorExp::Kind::kAbsC:
  case TensorExp::Kind::kAbsI:
    return "abs";
  case TensorExp::Kind::kCeilF:
    return "ceil";
  case TensorExp::Kind::kFloorF:
    return "floor";
  case TensorExp::Kind::kSqrtF:
  case TensorExp::Kind::kSqrtC:
    return "sqrt";
  case TensorExp::Kind::kExpm1F:
  case TensorExp::Kind::kExpm1C:
    return "expm1";
  case TensorExp::Kind::kLog1pF:
  case TensorExp::Kind::kLog1pC:
````
- **L697 EN**: Returns from the current function with `"tensor"`.
  **L697 CN**: 以 `"tensor"` 从当前函数返回。
- **L698 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kInvariant:`.
  **L698 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kInvariant:`。
- **L699 EN**: Returns from the current function with `"invariant"`.
  **L699 CN**: 以 `"invariant"` 从当前函数返回。
- **L700 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLoopVar:`.
  **L700 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLoopVar:`。
- **L701 EN**: Returns from the current function with `"index"`.
  **L701 CN**: 以 `"index"` 从当前函数返回。
- **L702 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSynZero:`.
  **L702 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSynZero:`。
- **L703 EN**: Returns from the current function with `"0"`.
  **L703 CN**: 以 `"0"` 从当前函数返回。
- **L704 EN**: Comment explains nearby logic, invariants, or intent: `Unary operations.`.
  **L704 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unary operations.`。
- **L705 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsF:`.
  **L705 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsF:`。
- **L706 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsC:`.
  **L706 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsC:`。
- **L707 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsI:`.
  **L707 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsI:`。
- **L708 EN**: Returns from the current function with `"abs"`.
  **L708 CN**: 以 `"abs"` 从当前函数返回。
- **L709 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCeilF:`.
  **L709 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCeilF:`。
- **L710 EN**: Returns from the current function with `"ceil"`.
  **L710 CN**: 以 `"ceil"` 从当前函数返回。
- **L711 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kFloorF:`.
  **L711 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kFloorF:`。
- **L712 EN**: Returns from the current function with `"floor"`.
  **L712 CN**: 以 `"floor"` 从当前函数返回。
- **L713 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtF:`.
  **L713 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtF:`。
- **L714 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtC:`.
  **L714 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtC:`。
- **L715 EN**: Returns from the current function with `"sqrt"`.
  **L715 CN**: 以 `"sqrt"` 从当前函数返回。
- **L716 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1F:`.
  **L716 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1F:`。
- **L717 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1C:`.
  **L717 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1C:`。
- **L718 EN**: Returns from the current function with `"expm1"`.
  **L718 CN**: 以 `"expm1"` 从当前函数返回。
- **L719 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pF:`.
  **L719 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pF:`。
- **L720 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pC:`.
  **L720 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pC:`。

### Lines 721-744

````cpp
    return "log1p";
  case TensorExp::Kind::kRelu:
    return "relu";
  case TensorExp::Kind::kSinF:
  case TensorExp::Kind::kSinC:
    return "sin";
  case TensorExp::Kind::kTanhF:
  case TensorExp::Kind::kTanhC:
    return "tanh";
  case TensorExp::Kind::kNegF:
  case TensorExp::Kind::kNegC:
  case TensorExp::Kind::kNegI:
    return "-";
  case TensorExp::Kind::kTruncF:
  case TensorExp::Kind::kExtF:
  case TensorExp::Kind::kCastFS:
  case TensorExp::Kind::kCastFU:
  case TensorExp::Kind::kCastSF:
  case TensorExp::Kind::kCastUF:
  case TensorExp::Kind::kCastS:
  case TensorExp::Kind::kCastU:
  case TensorExp::Kind::kCastIdx:
  case TensorExp::Kind::kTruncI:
  case TensorExp::Kind::kCIm:
````
- **L721 EN**: Returns from the current function with `"log1p"`.
  **L721 CN**: 以 `"log1p"` 从当前函数返回。
- **L722 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kRelu:`.
  **L722 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kRelu:`。
- **L723 EN**: Returns from the current function with `"relu"`.
  **L723 CN**: 以 `"relu"` 从当前函数返回。
- **L724 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinF:`.
  **L724 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinF:`。
- **L725 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinC:`.
  **L725 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinC:`。
- **L726 EN**: Returns from the current function with `"sin"`.
  **L726 CN**: 以 `"sin"` 从当前函数返回。
- **L727 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhF:`.
  **L727 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhF:`。
- **L728 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhC:`.
  **L728 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhC:`。
- **L729 EN**: Returns from the current function with `"tanh"`.
  **L729 CN**: 以 `"tanh"` 从当前函数返回。
- **L730 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegF:`.
  **L730 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegF:`。
- **L731 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegC:`.
  **L731 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegC:`。
- **L732 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegI:`.
  **L732 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegI:`。
- **L733 EN**: Returns from the current function with `"-"`.
  **L733 CN**: 以 `"-"` 从当前函数返回。
- **L734 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncF:`.
  **L734 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncF:`。
- **L735 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExtF:`.
  **L735 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExtF:`。
- **L736 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFS:`.
  **L736 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFS:`。
- **L737 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFU:`.
  **L737 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFU:`。
- **L738 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastSF:`.
  **L738 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastSF:`。
- **L739 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastUF:`.
  **L739 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastUF:`。
- **L740 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastS:`.
  **L740 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastS:`。
- **L741 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastU:`.
  **L741 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastU:`。
- **L742 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastIdx:`.
  **L742 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastIdx:`。
- **L743 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncI:`.
  **L743 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncI:`。
- **L744 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCIm:`.
  **L744 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCIm:`。

### Lines 745-768

````cpp
    return "complex.im";
  case TensorExp::Kind::kCRe:
    return "complex.re";
  case TensorExp::Kind::kBitCast:
    return "cast";
  case TensorExp::Kind::kBinaryBranch:
    return "binary_branch";
  case TensorExp::Kind::kUnary:
    return "unary";
  case TensorExp::Kind::kSelect:
    return "select";
  // Binary operations.
  case TensorExp::Kind::kMulF:
  case TensorExp::Kind::kMulC:
  case TensorExp::Kind::kMulI:
    return "*";
  case TensorExp::Kind::kDivF:
  case TensorExp::Kind::kDivC:
  case TensorExp::Kind::kDivS:
  case TensorExp::Kind::kDivU:
    return "/";
  case TensorExp::Kind::kAddF:
  case TensorExp::Kind::kAddC:
  case TensorExp::Kind::kAddI:
````
- **L745 EN**: Returns from the current function with `"complex.im"`.
  **L745 CN**: 以 `"complex.im"` 从当前函数返回。
- **L746 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCRe:`.
  **L746 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCRe:`。
- **L747 EN**: Returns from the current function with `"complex.re"`.
  **L747 CN**: 以 `"complex.re"` 从当前函数返回。
- **L748 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBitCast:`.
  **L748 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBitCast:`。
- **L749 EN**: Returns from the current function with `"cast"`.
  **L749 CN**: 以 `"cast"` 从当前函数返回。
- **L750 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinaryBranch:`.
  **L750 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinaryBranch:`。
- **L751 EN**: Returns from the current function with `"binary_branch"`.
  **L751 CN**: 以 `"binary_branch"` 从当前函数返回。
- **L752 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kUnary:`.
  **L752 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kUnary:`。
- **L753 EN**: Returns from the current function with `"unary"`.
  **L753 CN**: 以 `"unary"` 从当前函数返回。
- **L754 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSelect:`.
  **L754 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSelect:`。
- **L755 EN**: Returns from the current function with `"select"`.
  **L755 CN**: 以 `"select"` 从当前函数返回。
- **L756 EN**: Comment explains nearby logic, invariants, or intent: `Binary operations.`.
  **L756 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Binary operations.`。
- **L757 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulF:`.
  **L757 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulF:`。
- **L758 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulC:`.
  **L758 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulC:`。
- **L759 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulI:`.
  **L759 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulI:`。
- **L760 EN**: Returns from the current function with `"*"`.
  **L760 CN**: 以 `"*"` 从当前函数返回。
- **L761 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivF:`.
  **L761 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivF:`。
- **L762 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivC:`.
  **L762 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivC:`。
- **L763 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivS:`.
  **L763 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivS:`。
- **L764 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivU:`.
  **L764 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivU:`。
- **L765 EN**: Returns from the current function with `"/"`.
  **L765 CN**: 以 `"/"` 从当前函数返回。
- **L766 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddF:`.
  **L766 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddF:`。
- **L767 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddC:`.
  **L767 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddC:`。
- **L768 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddI:`.
  **L768 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddI:`。

### Lines 769-792

````cpp
    return "+";
  case TensorExp::Kind::kSubF:
  case TensorExp::Kind::kSubC:
  case TensorExp::Kind::kSubI:
    return "-";
  case TensorExp::Kind::kAndI:
    return "&";
  case TensorExp::Kind::kOrI:
    return "|";
  case TensorExp::Kind::kXorI:
    return "^";
  case TensorExp::Kind::kShrS:
    return "a>>";
  case TensorExp::Kind::kShrU:
    return ">>";
  case TensorExp::Kind::kShlI:
    return "<<";
  case TensorExp::Kind::kCmpF:
  case TensorExp::Kind::kCmpI:
    return "cmp";
  case TensorExp::Kind::kBinary:
    return "binary";
  case TensorExp::Kind::kReduce:
    return "reduce";
````
- **L769 EN**: Returns from the current function with `"+"`.
  **L769 CN**: 以 `"+"` 从当前函数返回。
- **L770 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubF:`.
  **L770 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubF:`。
- **L771 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubC:`.
  **L771 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubC:`。
- **L772 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubI:`.
  **L772 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubI:`。
- **L773 EN**: Returns from the current function with `"-"`.
  **L773 CN**: 以 `"-"` 从当前函数返回。
- **L774 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAndI:`.
  **L774 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAndI:`。
- **L775 EN**: Returns from the current function with `"&"`.
  **L775 CN**: 以 `"&"` 从当前函数返回。
- **L776 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kOrI:`.
  **L776 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kOrI:`。
- **L777 EN**: Returns from the current function with `"|"`.
  **L777 CN**: 以 `"|"` 从当前函数返回。
- **L778 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kXorI:`.
  **L778 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kXorI:`。
- **L779 EN**: Returns from the current function with `"^"`.
  **L779 CN**: 以 `"^"` 从当前函数返回。
- **L780 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrS:`.
  **L780 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrS:`。
- **L781 EN**: Returns from the current function with `"a>>"`.
  **L781 CN**: 以 `"a>>"` 从当前函数返回。
- **L782 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrU:`.
  **L782 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrU:`。
- **L783 EN**: Returns from the current function with `">>"`.
  **L783 CN**: 以 `">>"` 从当前函数返回。
- **L784 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShlI:`.
  **L784 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShlI:`。
- **L785 EN**: Returns from the current function with `"<<"`.
  **L785 CN**: 以 `"<<"` 从当前函数返回。
- **L786 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpF:`.
  **L786 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpF:`。
- **L787 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpI:`.
  **L787 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpI:`。
- **L788 EN**: Returns from the current function with `"cmp"`.
  **L788 CN**: 以 `"cmp"` 从当前函数返回。
- **L789 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinary:`.
  **L789 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinary:`。
- **L790 EN**: Returns from the current function with `"binary"`.
  **L790 CN**: 以 `"binary"` 从当前函数返回。
- **L791 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kReduce:`.
  **L791 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kReduce:`。
- **L792 EN**: Returns from the current function with `"reduce"`.
  **L792 CN**: 以 `"reduce"` 从当前函数返回。

### Lines 793-816

````cpp
  case TensorExp::Kind::kDenseOp:
    return "dense";
  }
  llvm_unreachable("unexpected kind for symbol");
}

void Merger::dumpExp(ExprId e) const {
  const auto &expr = exp(e);
  switch (expr.kind) {
  // Leaf.
  case TensorExp::Kind::kTensor:
    if (expr.tensor == syntheticTensor)
      llvm::dbgs() << "synthetic_";
    else if (expr.tensor == outTensor)
      llvm::dbgs() << "output_";
    llvm::dbgs() << "tensor_" << expr.tensor;
    break;
  case TensorExp::Kind::kInvariant:
    llvm::dbgs() << "invariant";
    break;
  case TensorExp::Kind::kSynZero:
    llvm::dbgs() << "0";
    break;
  case TensorExp::Kind::kLoopVar:
````
- **L793 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDenseOp:`.
  **L793 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDenseOp:`。
- **L794 EN**: Returns from the current function with `"dense"`.
  **L794 CN**: 以 `"dense"` 从当前函数返回。
- **L795 EN**: Closes the current lexical scope or compound statement.
  **L795 CN**: 结束当前词法作用域或复合语句块。
- **L796 EN**: Marks this control path as unreachable.
  **L796 CN**: 将该控制路径标记为不可达。
- **L797 EN**: Closes the current lexical scope or compound statement.
  **L797 CN**: 结束当前词法作用域或复合语句块。
- **L798 EN**: Blank line separating nearby declarations or logic blocks.
  **L798 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L799 EN**: Starts a function, method, lambda, or structured scope: `void Merger::dumpExp(ExprId e) const {`.
  **L799 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Merger::dumpExp(ExprId e) const {`。
- **L800 EN**: Executes a call or declaration centered on `exp`.
  **L800 CN**: 执行以 `exp` 为核心的调用或声明。
- **L801 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L801 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L802 EN**: Comment explains nearby logic, invariants, or intent: `Leaf.`.
  **L802 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Leaf.`。
- **L803 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTensor:`.
  **L803 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTensor:`。
- **L804 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L804 CN**: 开始 `if` 控制流语句并计算其条件。
- **L805 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L805 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L806 EN**: Starts the alternative branch of the preceding conditional.
  **L806 CN**: 开始前一个条件语句的备选分支。
- **L807 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L807 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L808 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L808 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L809 EN**: Exits the nearest loop or switch statement.
  **L809 CN**: 退出最近的循环或 switch 语句。
- **L810 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kInvariant:`.
  **L810 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kInvariant:`。
- **L811 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L811 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L812 EN**: Exits the nearest loop or switch statement.
  **L812 CN**: 退出最近的循环或 switch 语句。
- **L813 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSynZero:`.
  **L813 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSynZero:`。
- **L814 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L814 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L815 EN**: Exits the nearest loop or switch statement.
  **L815 CN**: 退出最近的循环或 switch 语句。
- **L816 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLoopVar:`.
  **L816 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLoopVar:`。

### Lines 817-840

````cpp
    llvm::dbgs() << "loopvar_" << expr.loop;
    break;
  // Unary operations.
  case TensorExp::Kind::kAbsF:
  case TensorExp::Kind::kAbsC:
  case TensorExp::Kind::kAbsI:
  case TensorExp::Kind::kCeilF:
  case TensorExp::Kind::kFloorF:
  case TensorExp::Kind::kSqrtF:
  case TensorExp::Kind::kSqrtC:
  case TensorExp::Kind::kExpm1F:
  case TensorExp::Kind::kExpm1C:
  case TensorExp::Kind::kLog1pF:
  case TensorExp::Kind::kLog1pC:
  case TensorExp::Kind::kRelu:
  case TensorExp::Kind::kSinF:
  case TensorExp::Kind::kSinC:
  case TensorExp::Kind::kTanhF:
  case TensorExp::Kind::kTanhC:
  case TensorExp::Kind::kNegF:
  case TensorExp::Kind::kNegC:
  case TensorExp::Kind::kNegI:
  case TensorExp::Kind::kTruncF:
  case TensorExp::Kind::kExtF:
````
- **L817 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L817 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L818 EN**: Exits the nearest loop or switch statement.
  **L818 CN**: 退出最近的循环或 switch 语句。
- **L819 EN**: Comment explains nearby logic, invariants, or intent: `Unary operations.`.
  **L819 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unary operations.`。
- **L820 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsF:`.
  **L820 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsF:`。
- **L821 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsC:`.
  **L821 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsC:`。
- **L822 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsI:`.
  **L822 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsI:`。
- **L823 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCeilF:`.
  **L823 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCeilF:`。
- **L824 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kFloorF:`.
  **L824 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kFloorF:`。
- **L825 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtF:`.
  **L825 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtF:`。
- **L826 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtC:`.
  **L826 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtC:`。
- **L827 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1F:`.
  **L827 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1F:`。
- **L828 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1C:`.
  **L828 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1C:`。
- **L829 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pF:`.
  **L829 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pF:`。
- **L830 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pC:`.
  **L830 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pC:`。
- **L831 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kRelu:`.
  **L831 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kRelu:`。
- **L832 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinF:`.
  **L832 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinF:`。
- **L833 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinC:`.
  **L833 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinC:`。
- **L834 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhF:`.
  **L834 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhF:`。
- **L835 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhC:`.
  **L835 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhC:`。
- **L836 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegF:`.
  **L836 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegF:`。
- **L837 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegC:`.
  **L837 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegC:`。
- **L838 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegI:`.
  **L838 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegI:`。
- **L839 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncF:`.
  **L839 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncF:`。
- **L840 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExtF:`.
  **L840 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExtF:`。

### Lines 841-864

````cpp
  case TensorExp::Kind::kCastFS:
  case TensorExp::Kind::kCastFU:
  case TensorExp::Kind::kCastSF:
  case TensorExp::Kind::kCastUF:
  case TensorExp::Kind::kCastS:
  case TensorExp::Kind::kCastU:
  case TensorExp::Kind::kCastIdx:
  case TensorExp::Kind::kTruncI:
  case TensorExp::Kind::kCIm:
  case TensorExp::Kind::kCRe:
  case TensorExp::Kind::kBitCast:
  case TensorExp::Kind::kBinaryBranch:
  case TensorExp::Kind::kUnary:
  case TensorExp::Kind::kSelect:
    llvm::dbgs() << kindToOpSymbol(expr.kind) << " ";
    dumpExp(expr.children.e0);
    break;
  // Binary operations.
  case TensorExp::Kind::kMulF:
  case TensorExp::Kind::kMulC:
  case TensorExp::Kind::kMulI:
  case TensorExp::Kind::kDivF:
  case TensorExp::Kind::kDivC:
  case TensorExp::Kind::kDivS:
````
- **L841 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFS:`.
  **L841 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFS:`。
- **L842 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFU:`.
  **L842 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFU:`。
- **L843 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastSF:`.
  **L843 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastSF:`。
- **L844 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastUF:`.
  **L844 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastUF:`。
- **L845 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastS:`.
  **L845 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastS:`。
- **L846 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastU:`.
  **L846 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastU:`。
- **L847 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastIdx:`.
  **L847 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastIdx:`。
- **L848 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncI:`.
  **L848 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncI:`。
- **L849 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCIm:`.
  **L849 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCIm:`。
- **L850 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCRe:`.
  **L850 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCRe:`。
- **L851 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBitCast:`.
  **L851 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBitCast:`。
- **L852 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinaryBranch:`.
  **L852 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinaryBranch:`。
- **L853 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kUnary:`.
  **L853 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kUnary:`。
- **L854 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSelect:`.
  **L854 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSelect:`。
- **L855 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L855 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L856 EN**: Executes a call or declaration centered on `dumpExp`.
  **L856 CN**: 执行以 `dumpExp` 为核心的调用或声明。
- **L857 EN**: Exits the nearest loop or switch statement.
  **L857 CN**: 退出最近的循环或 switch 语句。
- **L858 EN**: Comment explains nearby logic, invariants, or intent: `Binary operations.`.
  **L858 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Binary operations.`。
- **L859 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulF:`.
  **L859 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulF:`。
- **L860 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulC:`.
  **L860 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulC:`。
- **L861 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulI:`.
  **L861 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulI:`。
- **L862 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivF:`.
  **L862 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivF:`。
- **L863 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivC:`.
  **L863 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivC:`。
- **L864 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivS:`.
  **L864 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivS:`。

### Lines 865-888

````cpp
  case TensorExp::Kind::kDivU:
  case TensorExp::Kind::kAddF:
  case TensorExp::Kind::kAddC:
  case TensorExp::Kind::kAddI:
  case TensorExp::Kind::kSubF:
  case TensorExp::Kind::kSubC:
  case TensorExp::Kind::kSubI:
  case TensorExp::Kind::kAndI:
  case TensorExp::Kind::kOrI:
  case TensorExp::Kind::kXorI:
  case TensorExp::Kind::kShrS:
  case TensorExp::Kind::kShrU:
  case TensorExp::Kind::kShlI:
  case TensorExp::Kind::kCmpF:
  case TensorExp::Kind::kCmpI:
  case TensorExp::Kind::kBinary:
  case TensorExp::Kind::kReduce:
  case TensorExp::Kind::kDenseOp:
    llvm::dbgs() << "(";
    dumpExp(expr.children.e0);
    llvm::dbgs() << " " << kindToOpSymbol(expr.kind);
    if (expr.attr)
      llvm::dbgs() << "{" << expr.attr << "}";
    if (expr.children.e1 != detail::kInvalidId) {
````
- **L865 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivU:`.
  **L865 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivU:`。
- **L866 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddF:`.
  **L866 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddF:`。
- **L867 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddC:`.
  **L867 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddC:`。
- **L868 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddI:`.
  **L868 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddI:`。
- **L869 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubF:`.
  **L869 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubF:`。
- **L870 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubC:`.
  **L870 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubC:`。
- **L871 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubI:`.
  **L871 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubI:`。
- **L872 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAndI:`.
  **L872 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAndI:`。
- **L873 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kOrI:`.
  **L873 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kOrI:`。
- **L874 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kXorI:`.
  **L874 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kXorI:`。
- **L875 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrS:`.
  **L875 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrS:`。
- **L876 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrU:`.
  **L876 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrU:`。
- **L877 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShlI:`.
  **L877 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShlI:`。
- **L878 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpF:`.
  **L878 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpF:`。
- **L879 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpI:`.
  **L879 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpI:`。
- **L880 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinary:`.
  **L880 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinary:`。
- **L881 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kReduce:`.
  **L881 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kReduce:`。
- **L882 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDenseOp:`.
  **L882 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDenseOp:`。
- **L883 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L883 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L884 EN**: Executes a call or declaration centered on `dumpExp`.
  **L884 CN**: 执行以 `dumpExp` 为核心的调用或声明。
- **L885 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L885 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L886 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L886 CN**: 开始 `if` 控制流语句并计算其条件。
- **L887 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L887 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L888 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L888 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 889-912

````cpp
      llvm::dbgs() << " ";
      dumpExp(expr.children.e1);
      llvm::dbgs() << ")";
    } else {
      assert(expr.kind == TensorExp::Kind::kDenseOp);
    }
    break;
  }
}

void Merger::dumpLat(LatPointId p) const {
  const auto &point = lat(p);
  llvm::dbgs() << "lat(";
  dumpBits(point.bits);
  llvm::dbgs() << " :";
  dumpBits(point.simple);
  llvm::dbgs() << " : ";
  dumpExp(point.exp);
  llvm::dbgs() << " )\n";
}

void Merger::dumpSet(LatSetId s) const {
  const auto &ss = set(s);
  llvm::dbgs() << "{ #" << ss.size() << "\n";
````
- **L889 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L889 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L890 EN**: Executes a call or declaration centered on `dumpExp`.
  **L890 CN**: 执行以 `dumpExp` 为核心的调用或声明。
- **L891 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L891 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L892 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L892 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L893 EN**: Checks an internal invariant in debug builds.
  **L893 CN**: 在调试构建中检查内部不变式。
- **L894 EN**: Closes the current lexical scope or compound statement.
  **L894 CN**: 结束当前词法作用域或复合语句块。
- **L895 EN**: Exits the nearest loop or switch statement.
  **L895 CN**: 退出最近的循环或 switch 语句。
- **L896 EN**: Closes the current lexical scope or compound statement.
  **L896 CN**: 结束当前词法作用域或复合语句块。
- **L897 EN**: Closes the current lexical scope or compound statement.
  **L897 CN**: 结束当前词法作用域或复合语句块。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L899 EN**: Starts a function, method, lambda, or structured scope: `void Merger::dumpLat(LatPointId p) const {`.
  **L899 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Merger::dumpLat(LatPointId p) const {`。
- **L900 EN**: Executes a call or declaration centered on `lat`.
  **L900 CN**: 执行以 `lat` 为核心的调用或声明。
- **L901 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L901 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L902 EN**: Executes a call or declaration centered on `dumpBits`.
  **L902 CN**: 执行以 `dumpBits` 为核心的调用或声明。
- **L903 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L903 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L904 EN**: Executes a call or declaration centered on `dumpBits`.
  **L904 CN**: 执行以 `dumpBits` 为核心的调用或声明。
- **L905 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L905 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L906 EN**: Executes a call or declaration centered on `dumpExp`.
  **L906 CN**: 执行以 `dumpExp` 为核心的调用或声明。
- **L907 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L907 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L908 EN**: Closes the current lexical scope or compound statement.
  **L908 CN**: 结束当前词法作用域或复合语句块。
- **L909 EN**: Blank line separating nearby declarations or logic blocks.
  **L909 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L910 EN**: Starts a function, method, lambda, or structured scope: `void Merger::dumpSet(LatSetId s) const {`.
  **L910 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Merger::dumpSet(LatSetId s) const {`。
- **L911 EN**: Executes a call or declaration centered on `set`.
  **L911 CN**: 执行以 `set` 为核心的调用或声明。
- **L912 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L912 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。

### Lines 913-936

````cpp
  for (const LatPointId p : ss) {
    llvm::dbgs() << "  ";
    dumpLat(p);
  }
  llvm::dbgs() << "}\n";
}

void Merger::dumpBits(const BitVector &bits) const {
  for (TensorLoopId b = 0, be = bits.size(); b < be; b++) {
    if (bits[b]) {
      const TensorId t = tensor(b);
      const LoopId i = loop(b);
      const auto lt = lvlTypes[t][i];
      if (isLvlWithNonTrivialIdxExp(b))
        llvm::dbgs() << " DEP_" << t << "_" << i;
      else
        llvm::dbgs() << " i_" << t << "_" << i << "_" << toMLIRString(lt);
    }
  }
}

#endif // NDEBUG

//===----------------------------------------------------------------------===//
````
- **L913 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L913 CN**: 开始 `for` 控制流语句并计算其条件。
- **L914 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L914 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L915 EN**: Executes a call or declaration centered on `dumpLat`.
  **L915 CN**: 执行以 `dumpLat` 为核心的调用或声明。
- **L916 EN**: Closes the current lexical scope or compound statement.
  **L916 CN**: 结束当前词法作用域或复合语句块。
- **L917 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L917 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L918 EN**: Closes the current lexical scope or compound statement.
  **L918 CN**: 结束当前词法作用域或复合语句块。
- **L919 EN**: Blank line separating nearby declarations or logic blocks.
  **L919 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L920 EN**: Starts a function, method, lambda, or structured scope: `void Merger::dumpBits(const BitVector &bits) const {`.
  **L920 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Merger::dumpBits(const BitVector &bits) const {`。
- **L921 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L921 CN**: 开始 `for` 控制流语句并计算其条件。
- **L922 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L922 CN**: 开始 `if` 控制流语句并计算其条件。
- **L923 EN**: Initializes variable `t` from the right-hand expression.
  **L923 CN**: 使用右侧表达式初始化变量 `t`。
- **L924 EN**: Initializes variable `i` from the right-hand expression.
  **L924 CN**: 使用右侧表达式初始化变量 `i`。
- **L925 EN**: Initializes variable `lt` from the right-hand expression.
  **L925 CN**: 使用右侧表达式初始化变量 `lt`。
- **L926 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L926 CN**: 开始 `if` 控制流语句并计算其条件。
- **L927 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L927 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L928 EN**: Starts the alternative branch of the preceding conditional.
  **L928 CN**: 开始前一个条件语句的备选分支。
- **L929 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L929 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L930 EN**: Closes the current lexical scope or compound statement.
  **L930 CN**: 结束当前词法作用域或复合语句块。
- **L931 EN**: Closes the current lexical scope or compound statement.
  **L931 CN**: 结束当前词法作用域或复合语句块。
- **L932 EN**: Closes the current lexical scope or compound statement.
  **L932 CN**: 结束当前词法作用域或复合语句块。
- **L933 EN**: Blank line separating nearby declarations or logic blocks.
  **L933 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L934 EN**: Closes the current preprocessor conditional block.
  **L934 CN**: 结束当前预处理条件块。
- **L935 EN**: Blank line separating nearby declarations or logic blocks.
  **L935 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L936 EN**: Banner comment marking a file or section boundary.
  **L936 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 937-960

````cpp
// Builder methods.
//===----------------------------------------------------------------------===//

LatSetId Merger::buildLattices(ExprId e, LoopId i) {
  // NOTE: The `expr` reference will be invalidated by recursive calls
  // (and any other method that may add new expressions); therefore, the
  // code below must make sure to copy fields of `expr` into local variables
  // before making any recursive calls.
  const auto &expr = exp(e);
  const TensorExp::Kind kind = expr.kind;
  switch (kind) {
  // Leaf.
  case TensorExp::Kind::kTensor:
  case TensorExp::Kind::kInvariant:
  case TensorExp::Kind::kSynZero:
  case TensorExp::Kind::kLoopVar: {
    // Either the loop-var is really used in the tensor expression, or it is
    // set to the undefined loop-var in that level. An invariant expression,
    // a proper index value, and a truly dynamic sparse output tensor are set
    // to a synthetic tensor with undefined indices only to ensure the
    // iteration space is not skipped as a result of their contents.
    const LatSetId s = addSet();
    TensorId t = syntheticTensor;
    if (kind == TensorExp::Kind::kTensor) {
````
- **L937 EN**: Comment explains nearby logic, invariants, or intent: `Builder methods.`.
  **L937 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Builder methods.`。
- **L938 EN**: Banner comment marking a file or section boundary.
  **L938 CN**: 横幅注释，用于标记文件或章节边界。
- **L939 EN**: Blank line separating nearby declarations or logic blocks.
  **L939 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L940 EN**: Starts a function, method, lambda, or structured scope: `LatSetId Merger::buildLattices(ExprId e, LoopId i) {`.
  **L940 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LatSetId Merger::buildLattices(ExprId e, LoopId i) {`。
- **L941 EN**: Comment highlights an implementation note: `NOTE: The `expr` reference will be invalidated by recursive calls`.
  **L941 CN**: 注释强调了一条实现说明：`NOTE: The `expr` reference will be invalidated by recursive calls`。
- **L942 EN**: Comment explains nearby logic, invariants, or intent: `(and any other method that may add new expressions); therefore, the`.
  **L942 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(and any other method that may add new expressions); therefore, the`。
- **L943 EN**: Comment explains nearby logic, invariants, or intent: `code below must make sure to copy fields of `expr` into local variables`.
  **L943 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`code below must make sure to copy fields of `expr` into local variables`。
- **L944 EN**: Comment explains nearby logic, invariants, or intent: `before making any recursive calls.`.
  **L944 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`before making any recursive calls.`。
- **L945 EN**: Executes a call or declaration centered on `exp`.
  **L945 CN**: 执行以 `exp` 为核心的调用或声明。
- **L946 EN**: Initializes variable `kind` from the right-hand expression.
  **L946 CN**: 使用右侧表达式初始化变量 `kind`。
- **L947 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L947 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L948 EN**: Comment explains nearby logic, invariants, or intent: `Leaf.`.
  **L948 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Leaf.`。
- **L949 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTensor:`.
  **L949 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTensor:`。
- **L950 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kInvariant:`.
  **L950 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kInvariant:`。
- **L951 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSynZero:`.
  **L951 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSynZero:`。
- **L952 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLoopVar: {`.
  **L952 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLoopVar: {`。
- **L953 EN**: Comment explains nearby logic, invariants, or intent: `Either the loop-var is really used in the tensor expression, or it is`.
  **L953 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Either the loop-var is really used in the tensor expression, or it is`。
- **L954 EN**: Comment explains nearby logic, invariants, or intent: `set to the undefined loop-var in that level. An invariant expression,`.
  **L954 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`set to the undefined loop-var in that level. An invariant expression,`。
- **L955 EN**: Comment explains nearby logic, invariants, or intent: `a proper index value, and a truly dynamic sparse output tensor are set`.
  **L955 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a proper index value, and a truly dynamic sparse output tensor are set`。
- **L956 EN**: Comment explains nearby logic, invariants, or intent: `to a synthetic tensor with undefined indices only to ensure the`.
  **L956 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to a synthetic tensor with undefined indices only to ensure the`。
- **L957 EN**: Comment explains nearby logic, invariants, or intent: `iteration space is not skipped as a result of their contents.`.
  **L957 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`iteration space is not skipped as a result of their contents.`。
- **L958 EN**: Initializes variable `s` from the right-hand expression.
  **L958 CN**: 使用右侧表达式初始化变量 `s`。
- **L959 EN**: Initializes variable `t` from the right-hand expression.
  **L959 CN**: 使用右侧表达式初始化变量 `t`。
- **L960 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L960 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 961-984

````cpp
      t = expr.tensor;
      if (hasSparseOut && t == outTensor)
        t = syntheticTensor;
    }
    latSets[s].push_back(addLat(t, i, e));
    return s;
  }
  // Unary operations.
  case TensorExp::Kind::kAbsF:
  case TensorExp::Kind::kAbsC:
  case TensorExp::Kind::kAbsI:
  case TensorExp::Kind::kCeilF:
  case TensorExp::Kind::kFloorF:
  case TensorExp::Kind::kSqrtF:
  case TensorExp::Kind::kSqrtC:
  case TensorExp::Kind::kExpm1F:
  case TensorExp::Kind::kExpm1C:
  case TensorExp::Kind::kLog1pF:
  case TensorExp::Kind::kLog1pC:
  case TensorExp::Kind::kRelu:
  case TensorExp::Kind::kSinF:
  case TensorExp::Kind::kSinC:
  case TensorExp::Kind::kTanhF:
  case TensorExp::Kind::kTanhC:
````
- **L961 EN**: Executes a standalone statement or declaration: `t = expr.tensor;`.
  **L961 CN**: 执行一条独立语句或声明：`t = expr.tensor;`。
- **L962 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L962 CN**: 开始 `if` 控制流语句并计算其条件。
- **L963 EN**: Executes a standalone statement or declaration: `t = syntheticTensor;`.
  **L963 CN**: 执行一条独立语句或声明：`t = syntheticTensor;`。
- **L964 EN**: Closes the current lexical scope or compound statement.
  **L964 CN**: 结束当前词法作用域或复合语句块。
- **L965 EN**: Executes a call or declaration centered on `latSets[s].push_back`.
  **L965 CN**: 执行以 `latSets[s].push_back` 为核心的调用或声明。
- **L966 EN**: Returns from the current function with `s`.
  **L966 CN**: 以 `s` 从当前函数返回。
- **L967 EN**: Closes the current lexical scope or compound statement.
  **L967 CN**: 结束当前词法作用域或复合语句块。
- **L968 EN**: Comment explains nearby logic, invariants, or intent: `Unary operations.`.
  **L968 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unary operations.`。
- **L969 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsF:`.
  **L969 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsF:`。
- **L970 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsC:`.
  **L970 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsC:`。
- **L971 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsI:`.
  **L971 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsI:`。
- **L972 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCeilF:`.
  **L972 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCeilF:`。
- **L973 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kFloorF:`.
  **L973 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kFloorF:`。
- **L974 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtF:`.
  **L974 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtF:`。
- **L975 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtC:`.
  **L975 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtC:`。
- **L976 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1F:`.
  **L976 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1F:`。
- **L977 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1C:`.
  **L977 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1C:`。
- **L978 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pF:`.
  **L978 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pF:`。
- **L979 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pC:`.
  **L979 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pC:`。
- **L980 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kRelu:`.
  **L980 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kRelu:`。
- **L981 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinF:`.
  **L981 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinF:`。
- **L982 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinC:`.
  **L982 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinC:`。
- **L983 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhF:`.
  **L983 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhF:`。
- **L984 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhC:`.
  **L984 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhC:`。

### Lines 985-1008

````cpp
  case TensorExp::Kind::kNegF:
  case TensorExp::Kind::kNegC:
  case TensorExp::Kind::kNegI:
  case TensorExp::Kind::kTruncF:
  case TensorExp::Kind::kExtF:
  case TensorExp::Kind::kCastFS:
  case TensorExp::Kind::kCastFU:
  case TensorExp::Kind::kCastSF:
  case TensorExp::Kind::kCastUF:
  case TensorExp::Kind::kCastS:
  case TensorExp::Kind::kCastU:
  case TensorExp::Kind::kCastIdx:
  case TensorExp::Kind::kTruncI:
  case TensorExp::Kind::kCIm:
  case TensorExp::Kind::kCRe:
  case TensorExp::Kind::kBitCast:
    // A zero preserving operation (viz. f(0) = 0, [Bik96,Ch5]) maps the
    // lattice set of the operand through the operator into a new set.
    //
    //  -y|!y | y |
    //  --+---+---+
    //    | 0 |-y |
    {
      const ExprId e0 = expr.children.e0;
````
- **L985 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegF:`.
  **L985 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegF:`。
- **L986 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegC:`.
  **L986 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegC:`。
- **L987 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegI:`.
  **L987 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegI:`。
- **L988 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncF:`.
  **L988 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncF:`。
- **L989 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExtF:`.
  **L989 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExtF:`。
- **L990 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFS:`.
  **L990 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFS:`。
- **L991 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFU:`.
  **L991 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFU:`。
- **L992 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastSF:`.
  **L992 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastSF:`。
- **L993 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastUF:`.
  **L993 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastUF:`。
- **L994 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastS:`.
  **L994 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastS:`。
- **L995 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastU:`.
  **L995 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastU:`。
- **L996 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastIdx:`.
  **L996 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastIdx:`。
- **L997 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncI:`.
  **L997 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncI:`。
- **L998 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCIm:`.
  **L998 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCIm:`。
- **L999 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCRe:`.
  **L999 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCRe:`。
- **L1000 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBitCast:`.
  **L1000 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBitCast:`。
- **L1001 EN**: Comment explains nearby logic, invariants, or intent: `A zero preserving operation (viz. f(0) = 0, [Bik96,Ch5]) maps the`.
  **L1001 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A zero preserving operation (viz. f(0) = 0, [Bik96,Ch5]) maps the`。
- **L1002 EN**: Comment explains nearby logic, invariants, or intent: `lattice set of the operand through the operator into a new set.`.
  **L1002 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lattice set of the operand through the operator into a new set.`。
- **L1003 EN**: Separator comment used for visual grouping.
  **L1003 CN**: 用于视觉分组的分隔注释。
- **L1004 EN**: Comment explains nearby logic, invariants, or intent: `y|!y | y |`.
  **L1004 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`y|!y | y |`。
- **L1005 EN**: Comment explains nearby logic, invariants, or intent: `+---+---+`.
  **L1005 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+---+---+`。
- **L1006 EN**: Comment explains nearby logic, invariants, or intent: `| 0 |-y |`.
  **L1006 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| 0 |-y |`。
- **L1007 EN**: Opens a new lexical scope or compound statement.
  **L1007 CN**: 打开一个新的词法作用域或复合语句块。
- **L1008 EN**: Initializes variable `e0` from the right-hand expression.
  **L1008 CN**: 使用右侧表达式初始化变量 `e0`。

### Lines 1009-1032

````cpp
      const Value v = expr.val;
      Attribute a = expr.attr;
      return mapSet(kind, buildLattices(e0, i), v, nullptr, a);
    }
  case TensorExp::Kind::kBinaryBranch:
  case TensorExp::Kind::kSelect:
    // The left or right half of a binary operation which has already
    // been split into separate operations for each region.
    {
      const ExprId e0 = expr.children.e0;
      Operation *const op = expr.op;
      return mapSet(kind, buildLattices(e0, i), Value(), op);
    }
  case TensorExp::Kind::kUnary:
    // A custom unary operation.
    //
    //  op y|    !y    |     y      |
    //  ----+----------+------------+
    //      | absent() | present(y) |
    {
      const ExprId e0 = expr.children.e0;
      UnaryOp unop = cast<UnaryOp>(expr.op);
      const LatSetId child0 = buildLattices(e0, i);
      Region &absentRegion = unop.getAbsentRegion();
````
- **L1009 EN**: Initializes variable `v` from the right-hand expression.
  **L1009 CN**: 使用右侧表达式初始化变量 `v`。
- **L1010 EN**: Initializes variable `a` from the right-hand expression.
  **L1010 CN**: 使用右侧表达式初始化变量 `a`。
- **L1011 EN**: Returns from the current function with `mapSet(kind, buildLattices(e0, i), v, nullptr, a)`.
  **L1011 CN**: 以 `mapSet(kind, buildLattices(e0, i), v, nullptr, a)` 从当前函数返回。
- **L1012 EN**: Closes the current lexical scope or compound statement.
  **L1012 CN**: 结束当前词法作用域或复合语句块。
- **L1013 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinaryBranch:`.
  **L1013 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinaryBranch:`。
- **L1014 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSelect:`.
  **L1014 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSelect:`。
- **L1015 EN**: Comment explains nearby logic, invariants, or intent: `The left or right half of a binary operation which has already`.
  **L1015 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The left or right half of a binary operation which has already`。
- **L1016 EN**: Comment explains nearby logic, invariants, or intent: `been split into separate operations for each region.`.
  **L1016 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`been split into separate operations for each region.`。
- **L1017 EN**: Opens a new lexical scope or compound statement.
  **L1017 CN**: 打开一个新的词法作用域或复合语句块。
- **L1018 EN**: Initializes variable `e0` from the right-hand expression.
  **L1018 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1019 EN**: Initializes variable `op` from the right-hand expression.
  **L1019 CN**: 使用右侧表达式初始化变量 `op`。
- **L1020 EN**: Returns from the current function with `mapSet(kind, buildLattices(e0, i), Value(), op)`.
  **L1020 CN**: 以 `mapSet(kind, buildLattices(e0, i), Value(), op)` 从当前函数返回。
- **L1021 EN**: Closes the current lexical scope or compound statement.
  **L1021 CN**: 结束当前词法作用域或复合语句块。
- **L1022 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kUnary:`.
  **L1022 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kUnary:`。
- **L1023 EN**: Comment explains nearby logic, invariants, or intent: `A custom unary operation.`.
  **L1023 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A custom unary operation.`。
- **L1024 EN**: Separator comment used for visual grouping.
  **L1024 CN**: 用于视觉分组的分隔注释。
- **L1025 EN**: Comment explains nearby logic, invariants, or intent: `op y|    !y    |     y      |`.
  **L1025 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`op y|    !y    |     y      |`。
- **L1026 EN**: Comment explains nearby logic, invariants, or intent: `+----------+------------+`.
  **L1026 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+----------+------------+`。
- **L1027 EN**: Comment explains nearby logic, invariants, or intent: `| absent() | present(y) |`.
  **L1027 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| absent() | present(y) |`。
- **L1028 EN**: Opens a new lexical scope or compound statement.
  **L1028 CN**: 打开一个新的词法作用域或复合语句块。
- **L1029 EN**: Initializes variable `e0` from the right-hand expression.
  **L1029 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1030 EN**: Initializes variable `unop` from the right-hand expression.
  **L1030 CN**: 使用右侧表达式初始化变量 `unop`。
- **L1031 EN**: Initializes variable `child0` from the right-hand expression.
  **L1031 CN**: 使用右侧表达式初始化变量 `child0`。
- **L1032 EN**: Executes a call or declaration centered on `unop.getAbsentRegion`.
  **L1032 CN**: 执行以 `unop.getAbsentRegion` 为核心的调用或声明。

### Lines 1033-1056

````cpp
      if (absentRegion.empty()) {
        // Simple mapping over existing values.
        return mapSet(kind, child0, Value(), unop);
      }
      // Use a disjunction with `unop` on the left and the absent value as an
      // invariant on the right.
      Block &absentBlock = absentRegion.front();
      YieldOp absentYield = cast<YieldOp>(absentBlock.getTerminator());
      const Value absentVal = absentYield.getSingleResult();
      const ExprId rhs = addInvariantExp(absentVal);
      return disjSet(e, child0, buildLattices(rhs, i), unop);
    }
  // Binary operations.
  case TensorExp::Kind::kMulF:
  case TensorExp::Kind::kMulC:
  case TensorExp::Kind::kMulI:
  case TensorExp::Kind::kAndI:
    // A multiplicative operation only needs to be performed
    // for the conjunction of sparse iteration spaces.
    //
    //  x*y|!y | y |
    //  ---+---+---+
    //  !x | 0 | 0 |
    //   x | 0 |x*y|
````
- **L1033 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1033 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1034 EN**: Comment explains nearby logic, invariants, or intent: `Simple mapping over existing values.`.
  **L1034 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Simple mapping over existing values.`。
- **L1035 EN**: Returns from the current function with `mapSet(kind, child0, Value(), unop)`.
  **L1035 CN**: 以 `mapSet(kind, child0, Value(), unop)` 从当前函数返回。
- **L1036 EN**: Closes the current lexical scope or compound statement.
  **L1036 CN**: 结束当前词法作用域或复合语句块。
- **L1037 EN**: Comment explains nearby logic, invariants, or intent: `Use a disjunction with `unop` on the left and the absent value as an`.
  **L1037 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Use a disjunction with `unop` on the left and the absent value as an`。
- **L1038 EN**: Comment explains nearby logic, invariants, or intent: `invariant on the right.`.
  **L1038 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`invariant on the right.`。
- **L1039 EN**: Executes a call or declaration centered on `absentRegion.front`.
  **L1039 CN**: 执行以 `absentRegion.front` 为核心的调用或声明。
- **L1040 EN**: Initializes variable `absentYield` from the right-hand expression.
  **L1040 CN**: 使用右侧表达式初始化变量 `absentYield`。
- **L1041 EN**: Initializes variable `absentVal` from the right-hand expression.
  **L1041 CN**: 使用右侧表达式初始化变量 `absentVal`。
- **L1042 EN**: Initializes variable `rhs` from the right-hand expression.
  **L1042 CN**: 使用右侧表达式初始化变量 `rhs`。
- **L1043 EN**: Returns from the current function with `disjSet(e, child0, buildLattices(rhs, i), unop)`.
  **L1043 CN**: 以 `disjSet(e, child0, buildLattices(rhs, i), unop)` 从当前函数返回。
- **L1044 EN**: Closes the current lexical scope or compound statement.
  **L1044 CN**: 结束当前词法作用域或复合语句块。
- **L1045 EN**: Comment explains nearby logic, invariants, or intent: `Binary operations.`.
  **L1045 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Binary operations.`。
- **L1046 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulF:`.
  **L1046 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulF:`。
- **L1047 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulC:`.
  **L1047 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulC:`。
- **L1048 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulI:`.
  **L1048 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulI:`。
- **L1049 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAndI:`.
  **L1049 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAndI:`。
- **L1050 EN**: Comment explains nearby logic, invariants, or intent: `A multiplicative operation only needs to be performed`.
  **L1050 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A multiplicative operation only needs to be performed`。
- **L1051 EN**: Comment explains nearby logic, invariants, or intent: `for the conjunction of sparse iteration spaces.`.
  **L1051 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for the conjunction of sparse iteration spaces.`。
- **L1052 EN**: Separator comment used for visual grouping.
  **L1052 CN**: 用于视觉分组的分隔注释。
- **L1053 EN**: Comment explains nearby logic, invariants, or intent: `x*y|!y | y |`.
  **L1053 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x*y|!y | y |`。
- **L1054 EN**: Comment explains nearby logic, invariants, or intent: `+---+---+`.
  **L1054 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+---+---+`。
- **L1055 EN**: Comment explains nearby logic, invariants, or intent: `x | 0 | 0 |`.
  **L1055 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x | 0 | 0 |`。
- **L1056 EN**: Comment explains nearby logic, invariants, or intent: `x | 0 |x*y|`.
  **L1056 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x | 0 |x*y|`。

### Lines 1057-1080

````cpp
    //
    // Note even here, 0*NaN=NaN and 0*Inf=NaN, but that is ignored.
    {
      const ExprId e0 = expr.children.e0;
      const ExprId e1 = expr.children.e1;
      return conjSet(e, buildLattices(e0, i), buildLattices(e1, i));
    }
  case TensorExp::Kind::kDivF:
  case TensorExp::Kind::kDivC:
  case TensorExp::Kind::kDivS:
  case TensorExp::Kind::kDivU:
    // A division is tricky, since 0/0, 0/c, c/0 all have
    // specific outcomes for floating-point and integers.
    // Thus, we need to traverse the full iteration space.
    //
    //  x/y|!y | y |
    //  ---+---+---+
    //  !x |0/0|0/y|   FP: 0/0=NaN,c/0=Inf,0/c=0 with c true nonzero
    //   x |x/0|x/y|  INT: x/0=exception for any x
    //
    // TODO: for now we "fixed" this by only accepting x/c cases
    //       during expression building, so that the conjunction
    //       rules applies (viz. x/c = x*(1/c) as far as lattice
    //       construction is concerned).
````
- **L1057 EN**: Separator comment used for visual grouping.
  **L1057 CN**: 用于视觉分组的分隔注释。
- **L1058 EN**: Comment explains nearby logic, invariants, or intent: `Note even here, 0*NaN=NaN and 0*Inf=NaN, but that is ignored.`.
  **L1058 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note even here, 0*NaN=NaN and 0*Inf=NaN, but that is ignored.`。
- **L1059 EN**: Opens a new lexical scope or compound statement.
  **L1059 CN**: 打开一个新的词法作用域或复合语句块。
- **L1060 EN**: Initializes variable `e0` from the right-hand expression.
  **L1060 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1061 EN**: Initializes variable `e1` from the right-hand expression.
  **L1061 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1062 EN**: Returns from the current function with `conjSet(e, buildLattices(e0, i), buildLattices(e1, i))`.
  **L1062 CN**: 以 `conjSet(e, buildLattices(e0, i), buildLattices(e1, i))` 从当前函数返回。
- **L1063 EN**: Closes the current lexical scope or compound statement.
  **L1063 CN**: 结束当前词法作用域或复合语句块。
- **L1064 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivF:`.
  **L1064 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivF:`。
- **L1065 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivC:`.
  **L1065 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivC:`。
- **L1066 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivS:`.
  **L1066 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivS:`。
- **L1067 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivU:`.
  **L1067 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivU:`。
- **L1068 EN**: Comment explains nearby logic, invariants, or intent: `A division is tricky, since 0/0, 0/c, c/0 all have`.
  **L1068 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A division is tricky, since 0/0, 0/c, c/0 all have`。
- **L1069 EN**: Comment explains nearby logic, invariants, or intent: `specific outcomes for floating-point and integers.`.
  **L1069 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`specific outcomes for floating-point and integers.`。
- **L1070 EN**: Comment explains nearby logic, invariants, or intent: `Thus, we need to traverse the full iteration space.`.
  **L1070 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Thus, we need to traverse the full iteration space.`。
- **L1071 EN**: Separator comment used for visual grouping.
  **L1071 CN**: 用于视觉分组的分隔注释。
- **L1072 EN**: Comment explains nearby logic, invariants, or intent: `x/y|!y | y |`.
  **L1072 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x/y|!y | y |`。
- **L1073 EN**: Comment explains nearby logic, invariants, or intent: `+---+---+`.
  **L1073 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+---+---+`。
- **L1074 EN**: Comment explains nearby logic, invariants, or intent: `x |0/0|0/y|   FP: 0/0=NaN,c/0=Inf,0/c=0 with c true nonzero`.
  **L1074 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x |0/0|0/y|   FP: 0/0=NaN,c/0=Inf,0/c=0 with c true nonzero`。
- **L1075 EN**: Comment explains nearby logic, invariants, or intent: `x |x/0|x/y|  INT: x/0=exception for any x`.
  **L1075 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x |x/0|x/y|  INT: x/0=exception for any x`。
- **L1076 EN**: Separator comment used for visual grouping.
  **L1076 CN**: 用于视觉分组的分隔注释。
- **L1077 EN**: Comment records a pending task or caution: `TODO: for now we "fixed" this by only accepting x/c cases`.
  **L1077 CN**: 注释记录了待办事项或注意点：`TODO: for now we "fixed" this by only accepting x/c cases`。
- **L1078 EN**: Comment explains nearby logic, invariants, or intent: `during expression building, so that the conjunction`.
  **L1078 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`during expression building, so that the conjunction`。
- **L1079 EN**: Comment explains nearby logic, invariants, or intent: `rules applies (viz. x/c = x*(1/c) as far as lattice`.
  **L1079 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`rules applies (viz. x/c = x*(1/c) as far as lattice`。
- **L1080 EN**: Comment explains nearby logic, invariants, or intent: `construction is concerned).`.
  **L1080 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`construction is concerned).`。

### Lines 1081-1104

````cpp
    {
      const ExprId e0 = expr.children.e0;
      const ExprId e1 = expr.children.e1;
      assert(!maybeZero(e1));
      return conjSet(e, buildLattices(e0, i), buildLattices(e1, i));
    }
  case TensorExp::Kind::kAddF:
  case TensorExp::Kind::kAddC:
  case TensorExp::Kind::kAddI:
  case TensorExp::Kind::kSubF:
  case TensorExp::Kind::kSubC:
  case TensorExp::Kind::kSubI:
  case TensorExp::Kind::kOrI:
  case TensorExp::Kind::kXorI:
    // An additive operation needs to be performed
    // for the disjunction of sparse iteration spaces.
    //
    //  x+y|!y | y |    x-y|!y | y |
    //  ---+---+---+    ---+---+---+
    //  !x | 0 | y |    !x | 0 |-y |
    //   x | x |x+y|     x | x |x-y|
    {
      const ExprId e0 = expr.children.e0;
      const ExprId e1 = expr.children.e1;
````
- **L1081 EN**: Opens a new lexical scope or compound statement.
  **L1081 CN**: 打开一个新的词法作用域或复合语句块。
- **L1082 EN**: Initializes variable `e0` from the right-hand expression.
  **L1082 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1083 EN**: Initializes variable `e1` from the right-hand expression.
  **L1083 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1084 EN**: Checks an internal invariant in debug builds.
  **L1084 CN**: 在调试构建中检查内部不变式。
- **L1085 EN**: Returns from the current function with `conjSet(e, buildLattices(e0, i), buildLattices(e1, i))`.
  **L1085 CN**: 以 `conjSet(e, buildLattices(e0, i), buildLattices(e1, i))` 从当前函数返回。
- **L1086 EN**: Closes the current lexical scope or compound statement.
  **L1086 CN**: 结束当前词法作用域或复合语句块。
- **L1087 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddF:`.
  **L1087 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddF:`。
- **L1088 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddC:`.
  **L1088 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddC:`。
- **L1089 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddI:`.
  **L1089 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddI:`。
- **L1090 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubF:`.
  **L1090 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubF:`。
- **L1091 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubC:`.
  **L1091 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubC:`。
- **L1092 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubI:`.
  **L1092 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubI:`。
- **L1093 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kOrI:`.
  **L1093 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kOrI:`。
- **L1094 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kXorI:`.
  **L1094 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kXorI:`。
- **L1095 EN**: Comment explains nearby logic, invariants, or intent: `An additive operation needs to be performed`.
  **L1095 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`An additive operation needs to be performed`。
- **L1096 EN**: Comment explains nearby logic, invariants, or intent: `for the disjunction of sparse iteration spaces.`.
  **L1096 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for the disjunction of sparse iteration spaces.`。
- **L1097 EN**: Separator comment used for visual grouping.
  **L1097 CN**: 用于视觉分组的分隔注释。
- **L1098 EN**: Comment explains nearby logic, invariants, or intent: `x+y|!y | y |    x-y|!y | y |`.
  **L1098 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x+y|!y | y |    x-y|!y | y |`。
- **L1099 EN**: Comment explains nearby logic, invariants, or intent: `+---+---+    ---+---+---+`.
  **L1099 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+---+---+    ---+---+---+`。
- **L1100 EN**: Comment explains nearby logic, invariants, or intent: `x | 0 | y |    !x | 0 |-y |`.
  **L1100 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x | 0 | y |    !x | 0 |-y |`。
- **L1101 EN**: Comment explains nearby logic, invariants, or intent: `x | x |x+y|     x | x |x-y|`.
  **L1101 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x | x |x+y|     x | x |x-y|`。
- **L1102 EN**: Opens a new lexical scope or compound statement.
  **L1102 CN**: 打开一个新的词法作用域或复合语句块。
- **L1103 EN**: Initializes variable `e0` from the right-hand expression.
  **L1103 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1104 EN**: Initializes variable `e1` from the right-hand expression.
  **L1104 CN**: 使用右侧表达式初始化变量 `e1`。

### Lines 1105-1128

````cpp
      return disjSet(e, buildLattices(e0, i), buildLattices(e1, i));
    }
  case TensorExp::Kind::kCmpF:
  case TensorExp::Kind::kCmpI:
    // A comparison operation needs to be performed
    // for the disjunction of sparse iteration spaces.
    //
    //   x < y |  !y   |   y   |
    //  -------+-------+-------+
    //     !x  |   0   | 0 < y |
    //      x  | x < 0 | x < y |
    {
      const ExprId e0 = expr.children.e0;
      const ExprId e1 = expr.children.e1;
      return disjSetWithZero(e, buildLattices(e0, i), buildLattices(e1, i));
    }
  case TensorExp::Kind::kShrS:
  case TensorExp::Kind::kShrU:
  case TensorExp::Kind::kShlI:
    // A shift operation by an invariant amount (viz. tensor expressions
    // can only occur at the left-hand-side of the operator) can be handled
    // with the conjunction rule.
    {
      const ExprId e0 = expr.children.e0;
````
- **L1105 EN**: Returns from the current function with `disjSet(e, buildLattices(e0, i), buildLattices(e1, i))`.
  **L1105 CN**: 以 `disjSet(e, buildLattices(e0, i), buildLattices(e1, i))` 从当前函数返回。
- **L1106 EN**: Closes the current lexical scope or compound statement.
  **L1106 CN**: 结束当前词法作用域或复合语句块。
- **L1107 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpF:`.
  **L1107 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpF:`。
- **L1108 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpI:`.
  **L1108 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpI:`。
- **L1109 EN**: Comment explains nearby logic, invariants, or intent: `A comparison operation needs to be performed`.
  **L1109 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A comparison operation needs to be performed`。
- **L1110 EN**: Comment explains nearby logic, invariants, or intent: `for the disjunction of sparse iteration spaces.`.
  **L1110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for the disjunction of sparse iteration spaces.`。
- **L1111 EN**: Separator comment used for visual grouping.
  **L1111 CN**: 用于视觉分组的分隔注释。
- **L1112 EN**: Comment explains nearby logic, invariants, or intent: `x < y |  !y   |   y   |`.
  **L1112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x < y |  !y   |   y   |`。
- **L1113 EN**: Comment explains nearby logic, invariants, or intent: `+-------+-------+`.
  **L1113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+-------+-------+`。
- **L1114 EN**: Comment explains nearby logic, invariants, or intent: `x  |   0   | 0 < y |`.
  **L1114 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x  |   0   | 0 < y |`。
- **L1115 EN**: Comment explains nearby logic, invariants, or intent: `x  | x < 0 | x < y |`.
  **L1115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x  | x < 0 | x < y |`。
- **L1116 EN**: Opens a new lexical scope or compound statement.
  **L1116 CN**: 打开一个新的词法作用域或复合语句块。
- **L1117 EN**: Initializes variable `e0` from the right-hand expression.
  **L1117 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1118 EN**: Initializes variable `e1` from the right-hand expression.
  **L1118 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1119 EN**: Returns from the current function with `disjSetWithZero(e, buildLattices(e0, i), buildLattices(e1, i))`.
  **L1119 CN**: 以 `disjSetWithZero(e, buildLattices(e0, i), buildLattices(e1, i))` 从当前函数返回。
- **L1120 EN**: Closes the current lexical scope or compound statement.
  **L1120 CN**: 结束当前词法作用域或复合语句块。
- **L1121 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrS:`.
  **L1121 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrS:`。
- **L1122 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrU:`.
  **L1122 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrU:`。
- **L1123 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShlI:`.
  **L1123 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShlI:`。
- **L1124 EN**: Comment explains nearby logic, invariants, or intent: `A shift operation by an invariant amount (viz. tensor expressions`.
  **L1124 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A shift operation by an invariant amount (viz. tensor expressions`。
- **L1125 EN**: Comment explains nearby logic, invariants, or intent: `can only occur at the left-hand-side of the operator) can be handled`.
  **L1125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`can only occur at the left-hand-side of the operator) can be handled`。
- **L1126 EN**: Comment explains nearby logic, invariants, or intent: `with the conjunction rule.`.
  **L1126 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`with the conjunction rule.`。
- **L1127 EN**: Opens a new lexical scope or compound statement.
  **L1127 CN**: 打开一个新的词法作用域或复合语句块。
- **L1128 EN**: Initializes variable `e0` from the right-hand expression.
  **L1128 CN**: 使用右侧表达式初始化变量 `e0`。

### Lines 1129-1152

````cpp
      const ExprId e1 = expr.children.e1;
      assert(isInvariant(e1));
      return conjSet(e, buildLattices(e0, i), buildLattices(e1, i));
    }
  case TensorExp::Kind::kBinary:
    // A custom binary operation.
    //
    //  x op y|   !y    |       y      |
    //  ------+---------+--------------+
    //    !x  |  empty  |   right(y)   |
    //     x  | left(x) | overlap(x,y) |
    {
      const ExprId e0 = expr.children.e0;
      const ExprId e1 = expr.children.e1;
      BinaryOp binop = cast<BinaryOp>(expr.op);
      const LatSetId child0 = buildLattices(e0, i);
      const LatSetId child1 = buildLattices(e1, i);
      Region &leftRegion = binop.getLeftRegion();
      Region &rightRegion = binop.getRightRegion();
      // Left Region.
      Operation *leftYield = nullptr;
      if (!leftRegion.empty()) {
        Block &leftBlock = leftRegion.front();
        leftYield = leftBlock.getTerminator();
````
- **L1129 EN**: Initializes variable `e1` from the right-hand expression.
  **L1129 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1130 EN**: Checks an internal invariant in debug builds.
  **L1130 CN**: 在调试构建中检查内部不变式。
- **L1131 EN**: Returns from the current function with `conjSet(e, buildLattices(e0, i), buildLattices(e1, i))`.
  **L1131 CN**: 以 `conjSet(e, buildLattices(e0, i), buildLattices(e1, i))` 从当前函数返回。
- **L1132 EN**: Closes the current lexical scope or compound statement.
  **L1132 CN**: 结束当前词法作用域或复合语句块。
- **L1133 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinary:`.
  **L1133 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinary:`。
- **L1134 EN**: Comment explains nearby logic, invariants, or intent: `A custom binary operation.`.
  **L1134 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A custom binary operation.`。
- **L1135 EN**: Separator comment used for visual grouping.
  **L1135 CN**: 用于视觉分组的分隔注释。
- **L1136 EN**: Comment explains nearby logic, invariants, or intent: `x op y|   !y    |       y      |`.
  **L1136 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x op y|   !y    |       y      |`。
- **L1137 EN**: Comment explains nearby logic, invariants, or intent: `+---------+--------------+`.
  **L1137 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+---------+--------------+`。
- **L1138 EN**: Comment explains nearby logic, invariants, or intent: `x  |  empty  |   right(y)   |`.
  **L1138 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x  |  empty  |   right(y)   |`。
- **L1139 EN**: Comment explains nearby logic, invariants, or intent: `x  | left(x) | overlap(x,y) |`.
  **L1139 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x  | left(x) | overlap(x,y) |`。
- **L1140 EN**: Opens a new lexical scope or compound statement.
  **L1140 CN**: 打开一个新的词法作用域或复合语句块。
- **L1141 EN**: Initializes variable `e0` from the right-hand expression.
  **L1141 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1142 EN**: Initializes variable `e1` from the right-hand expression.
  **L1142 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1143 EN**: Initializes variable `binop` from the right-hand expression.
  **L1143 CN**: 使用右侧表达式初始化变量 `binop`。
- **L1144 EN**: Initializes variable `child0` from the right-hand expression.
  **L1144 CN**: 使用右侧表达式初始化变量 `child0`。
- **L1145 EN**: Initializes variable `child1` from the right-hand expression.
  **L1145 CN**: 使用右侧表达式初始化变量 `child1`。
- **L1146 EN**: Executes a call or declaration centered on `binop.getLeftRegion`.
  **L1146 CN**: 执行以 `binop.getLeftRegion` 为核心的调用或声明。
- **L1147 EN**: Executes a call or declaration centered on `binop.getRightRegion`.
  **L1147 CN**: 执行以 `binop.getRightRegion` 为核心的调用或声明。
- **L1148 EN**: Comment explains nearby logic, invariants, or intent: `Left Region.`.
  **L1148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Left Region.`。
- **L1149 EN**: Executes a standalone statement or declaration: `Operation *leftYield = nullptr;`.
  **L1149 CN**: 执行一条独立语句或声明：`Operation *leftYield = nullptr;`。
- **L1150 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1150 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1151 EN**: Executes a call or declaration centered on `leftRegion.front`.
  **L1151 CN**: 执行以 `leftRegion.front` 为核心的调用或声明。
- **L1152 EN**: Executes a call or declaration centered on `leftBlock.getTerminator`.
  **L1152 CN**: 执行以 `leftBlock.getTerminator` 为核心的调用或声明。

### Lines 1153-1176

````cpp
      }
      // Right Region.
      Operation *rightYield = nullptr;
      if (!rightRegion.empty()) {
        Block &rightBlock = rightRegion.front();
        rightYield = rightBlock.getTerminator();
      }
      bool includeLeft = binop.getLeftIdentity() || !leftRegion.empty();
      bool includeRight = binop.getRightIdentity() || !rightRegion.empty();
      return combiSet(e, child0, child1, binop, includeLeft,
                      TensorExp::Kind::kBinaryBranch, leftYield, includeRight,
                      TensorExp::Kind::kBinaryBranch, rightYield);
    }
  case TensorExp::Kind::kReduce:
    // A custom reduce operation.
    {
      const ExprId e0 = expr.children.e0;
      const ExprId e1 = expr.children.e1;
      Operation *const op = expr.op;
      return conjSet(e, buildLattices(e0, i), buildLattices(e1, i), op);
    }
  case TensorExp::Kind::kDenseOp: {
    // It does not really matter whether we use conjunctive/disjunctive set
    // here, as all the operands of kDenseOp must be dense, the disjunctive set
````
- **L1153 EN**: Closes the current lexical scope or compound statement.
  **L1153 CN**: 结束当前词法作用域或复合语句块。
- **L1154 EN**: Comment explains nearby logic, invariants, or intent: `Right Region.`.
  **L1154 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Right Region.`。
- **L1155 EN**: Executes a standalone statement or declaration: `Operation *rightYield = nullptr;`.
  **L1155 CN**: 执行一条独立语句或声明：`Operation *rightYield = nullptr;`。
- **L1156 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1156 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1157 EN**: Executes a call or declaration centered on `rightRegion.front`.
  **L1157 CN**: 执行以 `rightRegion.front` 为核心的调用或声明。
- **L1158 EN**: Executes a call or declaration centered on `rightBlock.getTerminator`.
  **L1158 CN**: 执行以 `rightBlock.getTerminator` 为核心的调用或声明。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Initializes variable `includeLeft` from the right-hand expression.
  **L1160 CN**: 使用右侧表达式初始化变量 `includeLeft`。
- **L1161 EN**: Initializes variable `includeRight` from the right-hand expression.
  **L1161 CN**: 使用右侧表达式初始化变量 `includeRight`。
- **L1162 EN**: Returns from the current function with `combiSet(e, child0, child1, binop, includeLeft,`.
  **L1162 CN**: 以 `combiSet(e, child0, child1, binop, includeLeft,` 从当前函数返回。
- **L1163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TensorExp::Kind::kBinaryBranch, leftYield, includeRight,`.
  **L1163 CN**: 继续一个多行参数列表、初始化器或聚合项：`TensorExp::Kind::kBinaryBranch, leftYield, includeRight,`。
- **L1164 EN**: Executes a standalone statement or declaration: `TensorExp::Kind::kBinaryBranch, rightYield);`.
  **L1164 CN**: 执行一条独立语句或声明：`TensorExp::Kind::kBinaryBranch, rightYield);`。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kReduce:`.
  **L1166 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kReduce:`。
- **L1167 EN**: Comment explains nearby logic, invariants, or intent: `A custom reduce operation.`.
  **L1167 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A custom reduce operation.`。
- **L1168 EN**: Opens a new lexical scope or compound statement.
  **L1168 CN**: 打开一个新的词法作用域或复合语句块。
- **L1169 EN**: Initializes variable `e0` from the right-hand expression.
  **L1169 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1170 EN**: Initializes variable `e1` from the right-hand expression.
  **L1170 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1171 EN**: Initializes variable `op` from the right-hand expression.
  **L1171 CN**: 使用右侧表达式初始化变量 `op`。
- **L1172 EN**: Returns from the current function with `conjSet(e, buildLattices(e0, i), buildLattices(e1, i), op)`.
  **L1172 CN**: 以 `conjSet(e, buildLattices(e0, i), buildLattices(e1, i), op)` 从当前函数返回。
- **L1173 EN**: Closes the current lexical scope or compound statement.
  **L1173 CN**: 结束当前词法作用域或复合语句块。
- **L1174 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDenseOp: {`.
  **L1174 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDenseOp: {`。
- **L1175 EN**: Comment explains nearby logic, invariants, or intent: `It does not really matter whether we use conjunctive/disjunctive set`.
  **L1175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It does not really matter whether we use conjunctive/disjunctive set`。
- **L1176 EN**: Comment explains nearby logic, invariants, or intent: `here, as all the operands of kDenseOp must be dense, the disjunctive set`.
  **L1176 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`here, as all the operands of kDenseOp must be dense, the disjunctive set`。

### Lines 1177-1200

````cpp
    // will be optimized into conjunctive set eventually.
    if (expr.children.e1 == detail::kInvalidId) {
      const ExprId e0 = expr.children.e0;
      Operation *const op = expr.op;
      return mapSet(kind, buildLattices(e0, i), Value(), op);
    }

    const ExprId e0 = expr.children.e0;
    const ExprId e1 = expr.children.e1;
    Operation *const op = expr.op;
    return conjSet(e, buildLattices(e0, i), buildLattices(e1, i), op);
  }
  }
  llvm_unreachable("unexpected expression kind");
}

std::optional<ExprId> Merger::buildTensorExpFromLinalg(linalg::GenericOp op) {
  // Build the linalg semantics backward from yield.
  Operation *yield = op.getRegion().front().getTerminator();
  assert(isa<linalg::YieldOp>(yield));
  return buildTensorExp(op, yield->getOperand(0)).first;
}

/// Only returns true if we are certain this is a zero.
````
- **L1177 EN**: Comment explains nearby logic, invariants, or intent: `will be optimized into conjunctive set eventually.`.
  **L1177 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`will be optimized into conjunctive set eventually.`。
- **L1178 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1178 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1179 EN**: Initializes variable `e0` from the right-hand expression.
  **L1179 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1180 EN**: Initializes variable `op` from the right-hand expression.
  **L1180 CN**: 使用右侧表达式初始化变量 `op`。
- **L1181 EN**: Returns from the current function with `mapSet(kind, buildLattices(e0, i), Value(), op)`.
  **L1181 CN**: 以 `mapSet(kind, buildLattices(e0, i), Value(), op)` 从当前函数返回。
- **L1182 EN**: Closes the current lexical scope or compound statement.
  **L1182 CN**: 结束当前词法作用域或复合语句块。
- **L1183 EN**: Blank line separating nearby declarations or logic blocks.
  **L1183 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1184 EN**: Initializes variable `e0` from the right-hand expression.
  **L1184 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1185 EN**: Initializes variable `e1` from the right-hand expression.
  **L1185 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1186 EN**: Initializes variable `op` from the right-hand expression.
  **L1186 CN**: 使用右侧表达式初始化变量 `op`。
- **L1187 EN**: Returns from the current function with `conjSet(e, buildLattices(e0, i), buildLattices(e1, i), op)`.
  **L1187 CN**: 以 `conjSet(e, buildLattices(e0, i), buildLattices(e1, i), op)` 从当前函数返回。
- **L1188 EN**: Closes the current lexical scope or compound statement.
  **L1188 CN**: 结束当前词法作用域或复合语句块。
- **L1189 EN**: Closes the current lexical scope or compound statement.
  **L1189 CN**: 结束当前词法作用域或复合语句块。
- **L1190 EN**: Marks this control path as unreachable.
  **L1190 CN**: 将该控制路径标记为不可达。
- **L1191 EN**: Closes the current lexical scope or compound statement.
  **L1191 CN**: 结束当前词法作用域或复合语句块。
- **L1192 EN**: Blank line separating nearby declarations or logic blocks.
  **L1192 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1193 EN**: Starts a function, method, lambda, or structured scope: `std::optional<ExprId> Merger::buildTensorExpFromLinalg(linalg::GenericOp op) {`.
  **L1193 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::optional<ExprId> Merger::buildTensorExpFromLinalg(linalg::GenericOp op) {`。
- **L1194 EN**: Comment explains nearby logic, invariants, or intent: `Build the linalg semantics backward from yield.`.
  **L1194 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Build the linalg semantics backward from yield.`。
- **L1195 EN**: Executes a call or declaration centered on `op.getRegion`.
  **L1195 CN**: 执行以 `op.getRegion` 为核心的调用或声明。
- **L1196 EN**: Checks an internal invariant in debug builds.
  **L1196 CN**: 在调试构建中检查内部不变式。
- **L1197 EN**: Returns from the current function with `buildTensorExp(op, yield->getOperand(0)).first`.
  **L1197 CN**: 以 `buildTensorExp(op, yield->getOperand(0)).first` 从当前函数返回。
- **L1198 EN**: Closes the current lexical scope or compound statement.
  **L1198 CN**: 结束当前词法作用域或复合语句块。
- **L1199 EN**: Blank line separating nearby declarations or logic blocks.
  **L1199 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1200 EN**: Comment explains nearby logic, invariants, or intent: `Only returns true if we are certain this is a zero.`.
  **L1200 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only returns true if we are certain this is a zero.`。

### Lines 1201-1224

````cpp
static bool isCertainZero(Value val) {
  if (auto c = val.getDefiningOp<complex::ConstantOp>()) {
    ArrayAttr arrayAttr = c.getValue();
    return cast<FloatAttr>(arrayAttr[0]).getValue().isZero() &&
           cast<FloatAttr>(arrayAttr[1]).getValue().isZero();
  }
  if (auto c = val.getDefiningOp<arith::ConstantIntOp>())
    return c.value() == 0;
  if (auto c = val.getDefiningOp<arith::ConstantFloatOp>())
    return c.value().isZero();
  return false;
}

/// Only returns false if we are certain this is a nonzero.
bool Merger::maybeZero(ExprId e) const {
  const auto &expr = exp(e);
  if (expr.kind == TensorExp::Kind::kInvariant) {
    // Note that this is different from isCertainZero() in a subtle
    // way by always returning true for non-constants.
    if (auto c = expr.val.getDefiningOp<complex::ConstantOp>()) {
      ArrayAttr arrayAttr = c.getValue();
      return cast<FloatAttr>(arrayAttr[0]).getValue().isZero() &&
             cast<FloatAttr>(arrayAttr[1]).getValue().isZero();
    }
````
- **L1201 EN**: Starts a function, method, lambda, or structured scope: `static bool isCertainZero(Value val) {`.
  **L1201 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isCertainZero(Value val) {`。
- **L1202 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1202 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1203 EN**: Initializes variable `arrayAttr` from the right-hand expression.
  **L1203 CN**: 使用右侧表达式初始化变量 `arrayAttr`。
- **L1204 EN**: Returns from the current function with `cast<FloatAttr>(arrayAttr[0]).getValue().isZero() &&`.
  **L1204 CN**: 以 `cast<FloatAttr>(arrayAttr[0]).getValue().isZero() &&` 从当前函数返回。
- **L1205 EN**: Executes a call or declaration centered on `cast<FloatAttr>`.
  **L1205 CN**: 执行以 `cast<FloatAttr>` 为核心的调用或声明。
- **L1206 EN**: Closes the current lexical scope or compound statement.
  **L1206 CN**: 结束当前词法作用域或复合语句块。
- **L1207 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1207 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1208 EN**: Returns from the current function with `c.value() == 0`.
  **L1208 CN**: 以 `c.value() == 0` 从当前函数返回。
- **L1209 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1209 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1210 EN**: Returns from the current function with `c.value().isZero()`.
  **L1210 CN**: 以 `c.value().isZero()` 从当前函数返回。
- **L1211 EN**: Returns from the current function with `false`.
  **L1211 CN**: 以 `false` 从当前函数返回。
- **L1212 EN**: Closes the current lexical scope or compound statement.
  **L1212 CN**: 结束当前词法作用域或复合语句块。
- **L1213 EN**: Blank line separating nearby declarations or logic blocks.
  **L1213 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1214 EN**: Comment explains nearby logic, invariants, or intent: `Only returns false if we are certain this is a nonzero.`.
  **L1214 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only returns false if we are certain this is a nonzero.`。
- **L1215 EN**: Starts a function, method, lambda, or structured scope: `bool Merger::maybeZero(ExprId e) const {`.
  **L1215 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Merger::maybeZero(ExprId e) const {`。
- **L1216 EN**: Executes a call or declaration centered on `exp`.
  **L1216 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1217 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1217 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1218 EN**: Comment explains nearby logic, invariants, or intent: `Note that this is different from isCertainZero() in a subtle`.
  **L1218 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note that this is different from isCertainZero() in a subtle`。
- **L1219 EN**: Comment explains nearby logic, invariants, or intent: `way by always returning true for non-constants.`.
  **L1219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`way by always returning true for non-constants.`。
- **L1220 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1220 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1221 EN**: Initializes variable `arrayAttr` from the right-hand expression.
  **L1221 CN**: 使用右侧表达式初始化变量 `arrayAttr`。
- **L1222 EN**: Returns from the current function with `cast<FloatAttr>(arrayAttr[0]).getValue().isZero() &&`.
  **L1222 CN**: 以 `cast<FloatAttr>(arrayAttr[0]).getValue().isZero() &&` 从当前函数返回。
- **L1223 EN**: Executes a call or declaration centered on `cast<FloatAttr>`.
  **L1223 CN**: 执行以 `cast<FloatAttr>` 为核心的调用或声明。
- **L1224 EN**: Closes the current lexical scope or compound statement.
  **L1224 CN**: 结束当前词法作用域或复合语句块。

### Lines 1225-1248

````cpp
    if (auto c = expr.val.getDefiningOp<arith::ConstantIntOp>())
      return c.value() == 0;
    if (auto c = expr.val.getDefiningOp<arith::ConstantFloatOp>())
      return c.value().isZero();
  }
  return true;
}

Type Merger::inferType(ExprId e, Value src) const {
  // Obtain the destination type from the cast node.
  Type dtp = exp(e).val.getType();
  // Inspect source type. For vector types, apply the same
  // vectorization to the destination type.
  if (auto vtp = dyn_cast<VectorType>(src.getType()))
    return VectorType::get(vtp.getNumElements(), dtp, vtp.getScalableDims());
  return dtp;
}

/// Ensures that the sparsifier can generate code for expression.
static bool isAdmissibleBranchExp(Operation *op, Block *block, Value v) {
  // Arguments are always admissible.
  if (isa<BlockArgument>(v))
    return true;
  // Accept index anywhere.
````
- **L1225 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1225 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1226 EN**: Returns from the current function with `c.value() == 0`.
  **L1226 CN**: 以 `c.value() == 0` 从当前函数返回。
- **L1227 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1227 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1228 EN**: Returns from the current function with `c.value().isZero()`.
  **L1228 CN**: 以 `c.value().isZero()` 从当前函数返回。
- **L1229 EN**: Closes the current lexical scope or compound statement.
  **L1229 CN**: 结束当前词法作用域或复合语句块。
- **L1230 EN**: Returns from the current function with `true`.
  **L1230 CN**: 以 `true` 从当前函数返回。
- **L1231 EN**: Closes the current lexical scope or compound statement.
  **L1231 CN**: 结束当前词法作用域或复合语句块。
- **L1232 EN**: Blank line separating nearby declarations or logic blocks.
  **L1232 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1233 EN**: Starts a function, method, lambda, or structured scope: `Type Merger::inferType(ExprId e, Value src) const {`.
  **L1233 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Type Merger::inferType(ExprId e, Value src) const {`。
- **L1234 EN**: Comment explains nearby logic, invariants, or intent: `Obtain the destination type from the cast node.`.
  **L1234 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Obtain the destination type from the cast node.`。
- **L1235 EN**: Initializes variable `dtp` from the right-hand expression.
  **L1235 CN**: 使用右侧表达式初始化变量 `dtp`。
- **L1236 EN**: Comment explains nearby logic, invariants, or intent: `Inspect source type. For vector types, apply the same`.
  **L1236 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Inspect source type. For vector types, apply the same`。
- **L1237 EN**: Comment explains nearby logic, invariants, or intent: `vectorization to the destination type.`.
  **L1237 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectorization to the destination type.`。
- **L1238 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1238 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1239 EN**: Returns from the current function with `VectorType::get(vtp.getNumElements(), dtp, vtp.getScalableDims())`.
  **L1239 CN**: 以 `VectorType::get(vtp.getNumElements(), dtp, vtp.getScalableDims())` 从当前函数返回。
- **L1240 EN**: Returns from the current function with `dtp`.
  **L1240 CN**: 以 `dtp` 从当前函数返回。
- **L1241 EN**: Closes the current lexical scope or compound statement.
  **L1241 CN**: 结束当前词法作用域或复合语句块。
- **L1242 EN**: Blank line separating nearby declarations or logic blocks.
  **L1242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1243 EN**: Comment explains nearby logic, invariants, or intent: `Ensures that the sparsifier can generate code for expression.`.
  **L1243 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Ensures that the sparsifier can generate code for expression.`。
- **L1244 EN**: Starts a function, method, lambda, or structured scope: `static bool isAdmissibleBranchExp(Operation *op, Block *block, Value v) {`.
  **L1244 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isAdmissibleBranchExp(Operation *op, Block *block, Value v) {`。
- **L1245 EN**: Comment explains nearby logic, invariants, or intent: `Arguments are always admissible.`.
  **L1245 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Arguments are always admissible.`。
- **L1246 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1246 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1247 EN**: Returns from the current function with `true`.
  **L1247 CN**: 以 `true` 从当前函数返回。
- **L1248 EN**: Comment explains nearby logic, invariants, or intent: `Accept index anywhere.`.
  **L1248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Accept index anywhere.`。

### Lines 1249-1272

````cpp
  Operation *def = v.getDefiningOp();
  if (isa<linalg::IndexOp>(def))
    return true;
  // Operation defined outside branch.
  if (def->getBlock() != block)
    return def->getBlock() != op->getBlock(); // invariant?
  // Operation defined within branch. Anything is accepted,
  // as long as all subexpressions are admissible.
  for (unsigned i = 0, n = def->getNumOperands(); i < n; i++)
    if (!isAdmissibleBranchExp(op, block, def->getOperand(i)))
      return false;
  return true;
}

/// Ensures that the sparsifier can generate code for branch.
static bool isAdmissibleBranch(Operation *op, Region &region) {
  if (region.empty())
    return true;
  // Build the semi-ring branch semantics backward from yield.
  Operation *yield = region.front().getTerminator();
  assert(isa<YieldOp>(yield));
  return isAdmissibleBranchExp(op, &region.front(), yield->getOperand(0));
}

````
- **L1249 EN**: Executes a call or declaration centered on `v.getDefiningOp`.
  **L1249 CN**: 执行以 `v.getDefiningOp` 为核心的调用或声明。
- **L1250 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1250 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1251 EN**: Returns from the current function with `true`.
  **L1251 CN**: 以 `true` 从当前函数返回。
- **L1252 EN**: Comment explains nearby logic, invariants, or intent: `Operation defined outside branch.`.
  **L1252 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Operation defined outside branch.`。
- **L1253 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1253 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1254 EN**: Returns from the current function with `def->getBlock() != op->getBlock(); // invariant?`.
  **L1254 CN**: 以 `def->getBlock() != op->getBlock(); // invariant?` 从当前函数返回。
- **L1255 EN**: Comment explains nearby logic, invariants, or intent: `Operation defined within branch. Anything is accepted,`.
  **L1255 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Operation defined within branch. Anything is accepted,`。
- **L1256 EN**: Comment explains nearby logic, invariants, or intent: `as long as all subexpressions are admissible.`.
  **L1256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`as long as all subexpressions are admissible.`。
- **L1257 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1257 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1258 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1258 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1259 EN**: Returns from the current function with `false`.
  **L1259 CN**: 以 `false` 从当前函数返回。
- **L1260 EN**: Returns from the current function with `true`.
  **L1260 CN**: 以 `true` 从当前函数返回。
- **L1261 EN**: Closes the current lexical scope or compound statement.
  **L1261 CN**: 结束当前词法作用域或复合语句块。
- **L1262 EN**: Blank line separating nearby declarations or logic blocks.
  **L1262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1263 EN**: Comment explains nearby logic, invariants, or intent: `Ensures that the sparsifier can generate code for branch.`.
  **L1263 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Ensures that the sparsifier can generate code for branch.`。
- **L1264 EN**: Starts a function, method, lambda, or structured scope: `static bool isAdmissibleBranch(Operation *op, Region &region) {`.
  **L1264 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isAdmissibleBranch(Operation *op, Region &region) {`。
- **L1265 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1265 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1266 EN**: Returns from the current function with `true`.
  **L1266 CN**: 以 `true` 从当前函数返回。
- **L1267 EN**: Comment explains nearby logic, invariants, or intent: `Build the semi-ring branch semantics backward from yield.`.
  **L1267 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Build the semi-ring branch semantics backward from yield.`。
- **L1268 EN**: Executes a call or declaration centered on `region.front`.
  **L1268 CN**: 执行以 `region.front` 为核心的调用或声明。
- **L1269 EN**: Checks an internal invariant in debug builds.
  **L1269 CN**: 在调试构建中检查内部不变式。
- **L1270 EN**: Returns from the current function with `isAdmissibleBranchExp(op, &region.front(), yield->getOperand(0))`.
  **L1270 CN**: 以 `isAdmissibleBranchExp(op, &region.front(), yield->getOperand(0))` 从当前函数返回。
- **L1271 EN**: Closes the current lexical scope or compound statement.
  **L1271 CN**: 结束当前词法作用域或复合语句块。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1273-1296

````cpp
// Recognizes a direct GT comparison.
static bool isGreater(TensorExp::Kind kind, Attribute attr) {
  if (kind == TensorExp::Kind::kCmpI) {
    auto pred = llvm::cast<arith::CmpIPredicateAttr>(attr).getValue();
    return pred == arith::CmpIPredicate::ugt ||
           pred == arith::CmpIPredicate::sgt;
  }
  if (kind == TensorExp::Kind::kCmpF) {
    auto pred = llvm::cast<arith::CmpFPredicateAttr>(attr).getValue();
    return pred == arith::CmpFPredicate::UGT ||
           pred == arith::CmpFPredicate::OGT;
  }
  return false;
}

std::pair<std::optional<ExprId>, bool>
Merger::buildTensorExp(linalg::GenericOp op, Value v) {
  // Recursion leaves.
  if (auto arg = dyn_cast<BlockArgument>(v)) {
    const TensorId tid = makeTensorId(arg.getArgNumber());
    // Any argument of the generic op that is not marked as a scalar
    // argument is considered a tensor, indexed by the implicit loop
    // bounds. This includes rank-0 tensor arguments.
    if (arg.getOwner()->getParentOp() == op) {
````
- **L1273 EN**: Comment explains nearby logic, invariants, or intent: `Recognizes a direct GT comparison.`.
  **L1273 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Recognizes a direct GT comparison.`。
- **L1274 EN**: Starts a function, method, lambda, or structured scope: `static bool isGreater(TensorExp::Kind kind, Attribute attr) {`.
  **L1274 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isGreater(TensorExp::Kind kind, Attribute attr) {`。
- **L1275 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1275 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1276 EN**: Initializes variable `pred` from the right-hand expression.
  **L1276 CN**: 使用右侧表达式初始化变量 `pred`。
- **L1277 EN**: Returns from the current function with `pred == arith::CmpIPredicate::ugt ||`.
  **L1277 CN**: 以 `pred == arith::CmpIPredicate::ugt ||` 从当前函数返回。
- **L1278 EN**: Executes a standalone statement or declaration: `pred == arith::CmpIPredicate::sgt;`.
  **L1278 CN**: 执行一条独立语句或声明：`pred == arith::CmpIPredicate::sgt;`。
- **L1279 EN**: Closes the current lexical scope or compound statement.
  **L1279 CN**: 结束当前词法作用域或复合语句块。
- **L1280 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1280 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1281 EN**: Initializes variable `pred` from the right-hand expression.
  **L1281 CN**: 使用右侧表达式初始化变量 `pred`。
- **L1282 EN**: Returns from the current function with `pred == arith::CmpFPredicate::UGT ||`.
  **L1282 CN**: 以 `pred == arith::CmpFPredicate::UGT ||` 从当前函数返回。
- **L1283 EN**: Executes a standalone statement or declaration: `pred == arith::CmpFPredicate::OGT;`.
  **L1283 CN**: 执行一条独立语句或声明：`pred == arith::CmpFPredicate::OGT;`。
- **L1284 EN**: Closes the current lexical scope or compound statement.
  **L1284 CN**: 结束当前词法作用域或复合语句块。
- **L1285 EN**: Returns from the current function with `false`.
  **L1285 CN**: 以 `false` 从当前函数返回。
- **L1286 EN**: Closes the current lexical scope or compound statement.
  **L1286 CN**: 结束当前词法作用域或复合语句块。
- **L1287 EN**: Blank line separating nearby declarations or logic blocks.
  **L1287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1288 EN**: Continues the surrounding expression or declaration: `std::pair<std::optional<ExprId>, bool>`.
  **L1288 CN**: 继续构造周围的表达式或声明：`std::pair<std::optional<ExprId>, bool>`。
- **L1289 EN**: Starts a function, method, lambda, or structured scope: `Merger::buildTensorExp(linalg::GenericOp op, Value v) {`.
  **L1289 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Merger::buildTensorExp(linalg::GenericOp op, Value v) {`。
- **L1290 EN**: Comment explains nearby logic, invariants, or intent: `Recursion leaves.`.
  **L1290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Recursion leaves.`。
- **L1291 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1291 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1292 EN**: Initializes variable `tid` from the right-hand expression.
  **L1292 CN**: 使用右侧表达式初始化变量 `tid`。
- **L1293 EN**: Comment explains nearby logic, invariants, or intent: `Any argument of the generic op that is not marked as a scalar`.
  **L1293 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Any argument of the generic op that is not marked as a scalar`。
- **L1294 EN**: Comment explains nearby logic, invariants, or intent: `argument is considered a tensor, indexed by the implicit loop`.
  **L1294 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`argument is considered a tensor, indexed by the implicit loop`。
- **L1295 EN**: Comment explains nearby logic, invariants, or intent: `bounds. This includes rank-0 tensor arguments.`.
  **L1295 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bounds. This includes rank-0 tensor arguments.`。
- **L1296 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1296 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1297-1320

````cpp
      OpOperand &t = op->getOpOperand(tid);
      bool hasSpDep = getSparseTensorEncoding(t.get().getType()) != nullptr;
      if (!op.isScalar(&t))
        return {addTensorExp(tid), hasSpDep};
      v = t.get(); // get scalar value
    }
    // Any other argument (marked as scalar argument for the generic op
    // or belonging to an enveloping op) is considered invariant.
    return {addInvariantExp(v), /*hasSpDep=*/false};
  }

  // Something defined outside is invariant.
  Operation *def = v.getDefiningOp();
  if (def->getBlock() != &op.getRegion().front())
    return {addInvariantExp(v), /*hasSpDep=*/false};
  // Construct index operations.
  if (def->getNumOperands() == 0) {
    if (auto indexOp = dyn_cast<linalg::IndexOp>(def))
      return {addLoopVarExp(makeLoopId(indexOp.getDim())), /*hasSpDep=*/false};
  }

  // Construct unary operations if subexpression can be built.
  if (def->getNumOperands() == 1) {
    const auto [x, hasSpDep] = buildTensorExp(op, def->getOperand(0));
````
- **L1297 EN**: Executes a call or declaration centered on `op->getOpOperand`.
  **L1297 CN**: 执行以 `op->getOpOperand` 为核心的调用或声明。
- **L1298 EN**: Initializes variable `hasSpDep` from the right-hand expression.
  **L1298 CN**: 使用右侧表达式初始化变量 `hasSpDep`。
- **L1299 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1299 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1300 EN**: Returns from the current function with `{addTensorExp(tid), hasSpDep}`.
  **L1300 CN**: 以 `{addTensorExp(tid), hasSpDep}` 从当前函数返回。
- **L1301 EN**: Continues logic associated with callable symbol `get`.
  **L1301 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L1302 EN**: Closes the current lexical scope or compound statement.
  **L1302 CN**: 结束当前词法作用域或复合语句块。
- **L1303 EN**: Comment explains nearby logic, invariants, or intent: `Any other argument (marked as scalar argument for the generic op`.
  **L1303 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Any other argument (marked as scalar argument for the generic op`。
- **L1304 EN**: Comment explains nearby logic, invariants, or intent: `or belonging to an enveloping op) is considered invariant.`.
  **L1304 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`or belonging to an enveloping op) is considered invariant.`。
- **L1305 EN**: Returns from the current function with `{addInvariantExp(v), /*hasSpDep=*/false}`.
  **L1305 CN**: 以 `{addInvariantExp(v), /*hasSpDep=*/false}` 从当前函数返回。
- **L1306 EN**: Closes the current lexical scope or compound statement.
  **L1306 CN**: 结束当前词法作用域或复合语句块。
- **L1307 EN**: Blank line separating nearby declarations or logic blocks.
  **L1307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1308 EN**: Comment explains nearby logic, invariants, or intent: `Something defined outside is invariant.`.
  **L1308 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Something defined outside is invariant.`。
- **L1309 EN**: Executes a call or declaration centered on `v.getDefiningOp`.
  **L1309 CN**: 执行以 `v.getDefiningOp` 为核心的调用或声明。
- **L1310 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1310 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1311 EN**: Returns from the current function with `{addInvariantExp(v), /*hasSpDep=*/false}`.
  **L1311 CN**: 以 `{addInvariantExp(v), /*hasSpDep=*/false}` 从当前函数返回。
- **L1312 EN**: Comment explains nearby logic, invariants, or intent: `Construct index operations.`.
  **L1312 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Construct index operations.`。
- **L1313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1314 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1314 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1315 EN**: Returns from the current function with `{addLoopVarExp(makeLoopId(indexOp.getDim())), /*hasSpDep=*/false}`.
  **L1315 CN**: 以 `{addLoopVarExp(makeLoopId(indexOp.getDim())), /*hasSpDep=*/false}` 从当前函数返回。
- **L1316 EN**: Closes the current lexical scope or compound statement.
  **L1316 CN**: 结束当前词法作用域或复合语句块。
- **L1317 EN**: Blank line separating nearby declarations or logic blocks.
  **L1317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1318 EN**: Comment explains nearby logic, invariants, or intent: `Construct unary operations if subexpression can be built.`.
  **L1318 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Construct unary operations if subexpression can be built.`。
- **L1319 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1319 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1320 EN**: Executes a call or declaration centered on `buildTensorExp`.
  **L1320 CN**: 执行以 `buildTensorExp` 为核心的调用或声明。

### Lines 1321-1344

````cpp
    if (x.has_value()) {
      const ExprId e = *x;
      if (isa<math::AbsFOp>(def))
        return {addExp(TensorExp::Kind::kAbsF, e), hasSpDep};
      if (isa<complex::AbsOp>(def))
        return {addExp(TensorExp::Kind::kAbsC, e), hasSpDep};
      if (isa<math::AbsIOp>(def))
        return {addExp(TensorExp::Kind::kAbsI, e), hasSpDep};
      if (isa<math::CeilOp>(def))
        return {addExp(TensorExp::Kind::kCeilF, e), hasSpDep};
      if (isa<math::FloorOp>(def))
        return {addExp(TensorExp::Kind::kFloorF, e), hasSpDep};
      if (isa<math::SqrtOp>(def))
        return {addExp(TensorExp::Kind::kSqrtF, e), hasSpDep};
      if (isa<complex::SqrtOp>(def))
        return {addExp(TensorExp::Kind::kSqrtC, e), hasSpDep};
      if (isa<math::ExpM1Op>(def))
        return {addExp(TensorExp::Kind::kExpm1F, e), hasSpDep};
      if (isa<complex::Expm1Op>(def))
        return {addExp(TensorExp::Kind::kExpm1C, e), hasSpDep};
      if (isa<math::Log1pOp>(def))
        return {addExp(TensorExp::Kind::kLog1pF, e), hasSpDep};
      if (isa<complex::Log1pOp>(def))
        return {addExp(TensorExp::Kind::kLog1pC, e), hasSpDep};
````
- **L1321 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1321 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1322 EN**: Initializes variable `e` from the right-hand expression.
  **L1322 CN**: 使用右侧表达式初始化变量 `e`。
- **L1323 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1323 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1324 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kAbsF, e), hasSpDep}`.
  **L1324 CN**: 以 `{addExp(TensorExp::Kind::kAbsF, e), hasSpDep}` 从当前函数返回。
- **L1325 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1325 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1326 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kAbsC, e), hasSpDep}`.
  **L1326 CN**: 以 `{addExp(TensorExp::Kind::kAbsC, e), hasSpDep}` 从当前函数返回。
- **L1327 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1327 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1328 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kAbsI, e), hasSpDep}`.
  **L1328 CN**: 以 `{addExp(TensorExp::Kind::kAbsI, e), hasSpDep}` 从当前函数返回。
- **L1329 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1329 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1330 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCeilF, e), hasSpDep}`.
  **L1330 CN**: 以 `{addExp(TensorExp::Kind::kCeilF, e), hasSpDep}` 从当前函数返回。
- **L1331 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1331 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1332 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kFloorF, e), hasSpDep}`.
  **L1332 CN**: 以 `{addExp(TensorExp::Kind::kFloorF, e), hasSpDep}` 从当前函数返回。
- **L1333 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1333 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1334 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kSqrtF, e), hasSpDep}`.
  **L1334 CN**: 以 `{addExp(TensorExp::Kind::kSqrtF, e), hasSpDep}` 从当前函数返回。
- **L1335 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1335 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1336 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kSqrtC, e), hasSpDep}`.
  **L1336 CN**: 以 `{addExp(TensorExp::Kind::kSqrtC, e), hasSpDep}` 从当前函数返回。
- **L1337 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1337 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1338 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kExpm1F, e), hasSpDep}`.
  **L1338 CN**: 以 `{addExp(TensorExp::Kind::kExpm1F, e), hasSpDep}` 从当前函数返回。
- **L1339 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1339 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1340 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kExpm1C, e), hasSpDep}`.
  **L1340 CN**: 以 `{addExp(TensorExp::Kind::kExpm1C, e), hasSpDep}` 从当前函数返回。
- **L1341 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1341 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1342 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kLog1pF, e), hasSpDep}`.
  **L1342 CN**: 以 `{addExp(TensorExp::Kind::kLog1pF, e), hasSpDep}` 从当前函数返回。
- **L1343 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1343 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1344 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kLog1pC, e), hasSpDep}`.
  **L1344 CN**: 以 `{addExp(TensorExp::Kind::kLog1pC, e), hasSpDep}` 从当前函数返回。

### Lines 1345-1368

````cpp
      if (isa<math::SinOp>(def))
        return {addExp(TensorExp::Kind::kSinF, e), hasSpDep};
      if (isa<complex::SinOp>(def))
        return {addExp(TensorExp::Kind::kSinC, e), hasSpDep};
      if (isa<math::TanhOp>(def))
        return {addExp(TensorExp::Kind::kTanhF, e), hasSpDep};
      if (isa<complex::TanhOp>(def))
        return {addExp(TensorExp::Kind::kTanhC, e), hasSpDep};
      if (isa<arith::NegFOp>(def))
        return {addExp(TensorExp::Kind::kNegF, e), hasSpDep}; // no negi in std
      if (isa<complex::NegOp>(def))
        return {addExp(TensorExp::Kind::kNegC, e), hasSpDep};
      if (isa<arith::TruncFOp>(def))
        return {addExp(TensorExp::Kind::kTruncF, e, v), hasSpDep};
      if (isa<arith::ExtFOp>(def))
        return {addExp(TensorExp::Kind::kExtF, e, v), hasSpDep};
      if (isa<arith::FPToSIOp>(def))
        return {addExp(TensorExp::Kind::kCastFS, e, v), hasSpDep};
      if (isa<arith::FPToUIOp>(def))
        return {addExp(TensorExp::Kind::kCastFU, e, v), hasSpDep};
      if (isa<arith::SIToFPOp>(def))
        return {addExp(TensorExp::Kind::kCastSF, e, v), hasSpDep};
      if (isa<arith::UIToFPOp>(def))
        return {addExp(TensorExp::Kind::kCastUF, e, v), hasSpDep};
````
- **L1345 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1345 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1346 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kSinF, e), hasSpDep}`.
  **L1346 CN**: 以 `{addExp(TensorExp::Kind::kSinF, e), hasSpDep}` 从当前函数返回。
- **L1347 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1347 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1348 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kSinC, e), hasSpDep}`.
  **L1348 CN**: 以 `{addExp(TensorExp::Kind::kSinC, e), hasSpDep}` 从当前函数返回。
- **L1349 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1349 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1350 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kTanhF, e), hasSpDep}`.
  **L1350 CN**: 以 `{addExp(TensorExp::Kind::kTanhF, e), hasSpDep}` 从当前函数返回。
- **L1351 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1351 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1352 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kTanhC, e), hasSpDep}`.
  **L1352 CN**: 以 `{addExp(TensorExp::Kind::kTanhC, e), hasSpDep}` 从当前函数返回。
- **L1353 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1353 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1354 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kNegF, e), hasSpDep}; // no negi in std`.
  **L1354 CN**: 以 `{addExp(TensorExp::Kind::kNegF, e), hasSpDep}; // no negi in std` 从当前函数返回。
- **L1355 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1355 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1356 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kNegC, e), hasSpDep}`.
  **L1356 CN**: 以 `{addExp(TensorExp::Kind::kNegC, e), hasSpDep}` 从当前函数返回。
- **L1357 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1357 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1358 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kTruncF, e, v), hasSpDep}`.
  **L1358 CN**: 以 `{addExp(TensorExp::Kind::kTruncF, e, v), hasSpDep}` 从当前函数返回。
- **L1359 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1359 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1360 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kExtF, e, v), hasSpDep}`.
  **L1360 CN**: 以 `{addExp(TensorExp::Kind::kExtF, e, v), hasSpDep}` 从当前函数返回。
- **L1361 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1361 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1362 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCastFS, e, v), hasSpDep}`.
  **L1362 CN**: 以 `{addExp(TensorExp::Kind::kCastFS, e, v), hasSpDep}` 从当前函数返回。
- **L1363 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1363 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1364 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCastFU, e, v), hasSpDep}`.
  **L1364 CN**: 以 `{addExp(TensorExp::Kind::kCastFU, e, v), hasSpDep}` 从当前函数返回。
- **L1365 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1365 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1366 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCastSF, e, v), hasSpDep}`.
  **L1366 CN**: 以 `{addExp(TensorExp::Kind::kCastSF, e, v), hasSpDep}` 从当前函数返回。
- **L1367 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1367 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1368 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCastUF, e, v), hasSpDep}`.
  **L1368 CN**: 以 `{addExp(TensorExp::Kind::kCastUF, e, v), hasSpDep}` 从当前函数返回。

### Lines 1369-1392

````cpp
      if (isa<arith::ExtSIOp>(def))
        return {addExp(TensorExp::Kind::kCastS, e, v), hasSpDep};
      if (isa<arith::ExtUIOp>(def))
        return {addExp(TensorExp::Kind::kCastU, e, v), hasSpDep};
      if (isa<arith::IndexCastOp>(def))
        return {addExp(TensorExp::Kind::kCastIdx, e, v), hasSpDep};
      if (isa<arith::TruncIOp>(def))
        return {addExp(TensorExp::Kind::kTruncI, e, v), hasSpDep};
      if (isa<complex::ImOp>(def))
        return {addExp(TensorExp::Kind::kCIm, e), hasSpDep};
      if (isa<complex::ReOp>(def))
        return {addExp(TensorExp::Kind::kCRe, e), hasSpDep};
      if (isa<arith::BitcastOp>(def))
        return {addExp(TensorExp::Kind::kBitCast, e, v), hasSpDep};
      if (auto unop = dyn_cast<sparse_tensor::UnaryOp>(def)) {
        if (isAdmissibleBranch(unop, unop.getPresentRegion()) &&
            isAdmissibleBranch(unop, unop.getAbsentRegion()))
          return {addExp(TensorExp::Kind::kUnary, e, Value(), def), hasSpDep};
      }
      if (auto selop = dyn_cast<sparse_tensor::SelectOp>(def)) {
        if (isAdmissibleBranch(selop, selop.getRegion()))
          return {addExp(TensorExp::Kind::kSelect, e, Value(), def), hasSpDep};
      }
    }
````
- **L1369 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1369 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1370 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCastS, e, v), hasSpDep}`.
  **L1370 CN**: 以 `{addExp(TensorExp::Kind::kCastS, e, v), hasSpDep}` 从当前函数返回。
- **L1371 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1371 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1372 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCastU, e, v), hasSpDep}`.
  **L1372 CN**: 以 `{addExp(TensorExp::Kind::kCastU, e, v), hasSpDep}` 从当前函数返回。
- **L1373 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1373 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1374 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCastIdx, e, v), hasSpDep}`.
  **L1374 CN**: 以 `{addExp(TensorExp::Kind::kCastIdx, e, v), hasSpDep}` 从当前函数返回。
- **L1375 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1375 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1376 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kTruncI, e, v), hasSpDep}`.
  **L1376 CN**: 以 `{addExp(TensorExp::Kind::kTruncI, e, v), hasSpDep}` 从当前函数返回。
- **L1377 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1377 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1378 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCIm, e), hasSpDep}`.
  **L1378 CN**: 以 `{addExp(TensorExp::Kind::kCIm, e), hasSpDep}` 从当前函数返回。
- **L1379 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1379 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1380 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kCRe, e), hasSpDep}`.
  **L1380 CN**: 以 `{addExp(TensorExp::Kind::kCRe, e), hasSpDep}` 从当前函数返回。
- **L1381 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1381 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1382 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kBitCast, e, v), hasSpDep}`.
  **L1382 CN**: 以 `{addExp(TensorExp::Kind::kBitCast, e, v), hasSpDep}` 从当前函数返回。
- **L1383 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1383 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1384 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1384 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1385 EN**: Continues logic associated with callable symbol `isAdmissibleBranch`.
  **L1385 CN**: 继续与可调用符号 `isAdmissibleBranch` 相关的逻辑。
- **L1386 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kUnary, e, Value(), def), hasSpDep}`.
  **L1386 CN**: 以 `{addExp(TensorExp::Kind::kUnary, e, Value(), def), hasSpDep}` 从当前函数返回。
- **L1387 EN**: Closes the current lexical scope or compound statement.
  **L1387 CN**: 结束当前词法作用域或复合语句块。
- **L1388 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1388 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1389 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1389 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1390 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kSelect, e, Value(), def), hasSpDep}`.
  **L1390 CN**: 以 `{addExp(TensorExp::Kind::kSelect, e, Value(), def), hasSpDep}` 从当前函数返回。
- **L1391 EN**: Closes the current lexical scope or compound statement.
  **L1391 CN**: 结束当前词法作用域或复合语句块。
- **L1392 EN**: Closes the current lexical scope or compound statement.
  **L1392 CN**: 结束当前词法作用域或复合语句块。

### Lines 1393-1416

````cpp
  }

  // Construct binary operations if subexpressions can be built.
  // See buildLattices() for an explanation of rejecting certain
  // division and shift operations.
  if (def->getNumOperands() == 2) {
    const auto [x, xSpVals] = buildTensorExp(op, def->getOperand(0));
    const auto [y, ySpVals] = buildTensorExp(op, def->getOperand(1));
    // For a conjunctive operation, it yields a "sparse" result if any operand
    // is sparse. For a disjunctive operation, it yields a "sparse" result if
    // all operands are sparse.
    bool conjSpVals = xSpVals || ySpVals;
    bool disjSpVals = xSpVals && ySpVals;
    if (x.has_value() && y.has_value()) {
      const ExprId e0 = *x;
      const ExprId e1 = *y;
      if (isa<arith::MulFOp>(def))
        return {addExp(TensorExp::Kind::kMulF, e0, e1), conjSpVals};
      if (isa<complex::MulOp>(def))
        return {addExp(TensorExp::Kind::kMulC, e0, e1), conjSpVals};
      if (isa<arith::MulIOp>(def))
        return {addExp(TensorExp::Kind::kMulI, e0, e1), conjSpVals};
      if (isa<arith::DivFOp>(def) && !maybeZero(e1))
        return {addExp(TensorExp::Kind::kDivF, e0, e1), conjSpVals};
````
- **L1393 EN**: Closes the current lexical scope or compound statement.
  **L1393 CN**: 结束当前词法作用域或复合语句块。
- **L1394 EN**: Blank line separating nearby declarations or logic blocks.
  **L1394 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1395 EN**: Comment explains nearby logic, invariants, or intent: `Construct binary operations if subexpressions can be built.`.
  **L1395 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Construct binary operations if subexpressions can be built.`。
- **L1396 EN**: Comment explains nearby logic, invariants, or intent: `See buildLattices() for an explanation of rejecting certain`.
  **L1396 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See buildLattices() for an explanation of rejecting certain`。
- **L1397 EN**: Comment explains nearby logic, invariants, or intent: `division and shift operations.`.
  **L1397 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`division and shift operations.`。
- **L1398 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1398 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1399 EN**: Executes a call or declaration centered on `buildTensorExp`.
  **L1399 CN**: 执行以 `buildTensorExp` 为核心的调用或声明。
- **L1400 EN**: Executes a call or declaration centered on `buildTensorExp`.
  **L1400 CN**: 执行以 `buildTensorExp` 为核心的调用或声明。
- **L1401 EN**: Comment explains nearby logic, invariants, or intent: `For a conjunctive operation, it yields a "sparse" result if any operand`.
  **L1401 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For a conjunctive operation, it yields a "sparse" result if any operand`。
- **L1402 EN**: Comment explains nearby logic, invariants, or intent: `is sparse. For a disjunctive operation, it yields a "sparse" result if`.
  **L1402 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is sparse. For a disjunctive operation, it yields a "sparse" result if`。
- **L1403 EN**: Comment explains nearby logic, invariants, or intent: `all operands are sparse.`.
  **L1403 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all operands are sparse.`。
- **L1404 EN**: Initializes variable `conjSpVals` from the right-hand expression.
  **L1404 CN**: 使用右侧表达式初始化变量 `conjSpVals`。
- **L1405 EN**: Initializes variable `disjSpVals` from the right-hand expression.
  **L1405 CN**: 使用右侧表达式初始化变量 `disjSpVals`。
- **L1406 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1406 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1407 EN**: Initializes variable `e0` from the right-hand expression.
  **L1407 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1408 EN**: Initializes variable `e1` from the right-hand expression.
  **L1408 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1409 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1409 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1410 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kMulF, e0, e1), conjSpVals}`.
  **L1410 CN**: 以 `{addExp(TensorExp::Kind::kMulF, e0, e1), conjSpVals}` 从当前函数返回。
- **L1411 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1411 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1412 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kMulC, e0, e1), conjSpVals}`.
  **L1412 CN**: 以 `{addExp(TensorExp::Kind::kMulC, e0, e1), conjSpVals}` 从当前函数返回。
- **L1413 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1413 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1414 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kMulI, e0, e1), conjSpVals}`.
  **L1414 CN**: 以 `{addExp(TensorExp::Kind::kMulI, e0, e1), conjSpVals}` 从当前函数返回。
- **L1415 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1415 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1416 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kDivF, e0, e1), conjSpVals}`.
  **L1416 CN**: 以 `{addExp(TensorExp::Kind::kDivF, e0, e1), conjSpVals}` 从当前函数返回。

### Lines 1417-1440

````cpp
      if (isa<complex::DivOp>(def) && !maybeZero(e1))
        return {addExp(TensorExp::Kind::kDivC, e0, e1), conjSpVals};
      if (isa<arith::DivSIOp>(def) && !maybeZero(e1))
        return {addExp(TensorExp::Kind::kDivS, e0, e1), conjSpVals};
      if (isa<arith::DivUIOp>(def) && !maybeZero(e1))
        return {addExp(TensorExp::Kind::kDivU, e0, e1), conjSpVals};
      if (isa<arith::AddFOp>(def))
        return {addExp(TensorExp::Kind::kAddF, e0, e1), disjSpVals};
      if (isa<complex::AddOp>(def))
        return {addExp(TensorExp::Kind::kAddC, e0, e1), disjSpVals};
      if (isa<arith::AddIOp>(def))
        return {addExp(TensorExp::Kind::kAddI, e0, e1), disjSpVals};
      if (isa<arith::SubFOp>(def))
        return {addExp(TensorExp::Kind::kSubF, e0, e1), disjSpVals};
      if (isa<complex::SubOp>(def))
        return {addExp(TensorExp::Kind::kSubC, e0, e1), disjSpVals};
      if (isa<arith::SubIOp>(def))
        return {addExp(TensorExp::Kind::kSubI, e0, e1), disjSpVals};
      if (isa<arith::AndIOp>(def))
        return {addExp(TensorExp::Kind::kAndI, e0, e1), conjSpVals};
      if (isa<arith::OrIOp>(def))
        return {addExp(TensorExp::Kind::kOrI, e0, e1), disjSpVals};
      if (isa<arith::XOrIOp>(def))
        return {addExp(TensorExp::Kind::kXorI, e0, e1), disjSpVals};
````
- **L1417 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1417 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1418 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kDivC, e0, e1), conjSpVals}`.
  **L1418 CN**: 以 `{addExp(TensorExp::Kind::kDivC, e0, e1), conjSpVals}` 从当前函数返回。
- **L1419 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1419 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1420 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kDivS, e0, e1), conjSpVals}`.
  **L1420 CN**: 以 `{addExp(TensorExp::Kind::kDivS, e0, e1), conjSpVals}` 从当前函数返回。
- **L1421 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1421 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1422 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kDivU, e0, e1), conjSpVals}`.
  **L1422 CN**: 以 `{addExp(TensorExp::Kind::kDivU, e0, e1), conjSpVals}` 从当前函数返回。
- **L1423 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1423 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1424 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kAddF, e0, e1), disjSpVals}`.
  **L1424 CN**: 以 `{addExp(TensorExp::Kind::kAddF, e0, e1), disjSpVals}` 从当前函数返回。
- **L1425 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1425 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1426 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kAddC, e0, e1), disjSpVals}`.
  **L1426 CN**: 以 `{addExp(TensorExp::Kind::kAddC, e0, e1), disjSpVals}` 从当前函数返回。
- **L1427 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1427 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1428 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kAddI, e0, e1), disjSpVals}`.
  **L1428 CN**: 以 `{addExp(TensorExp::Kind::kAddI, e0, e1), disjSpVals}` 从当前函数返回。
- **L1429 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1429 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1430 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kSubF, e0, e1), disjSpVals}`.
  **L1430 CN**: 以 `{addExp(TensorExp::Kind::kSubF, e0, e1), disjSpVals}` 从当前函数返回。
- **L1431 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1431 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1432 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kSubC, e0, e1), disjSpVals}`.
  **L1432 CN**: 以 `{addExp(TensorExp::Kind::kSubC, e0, e1), disjSpVals}` 从当前函数返回。
- **L1433 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1433 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1434 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kSubI, e0, e1), disjSpVals}`.
  **L1434 CN**: 以 `{addExp(TensorExp::Kind::kSubI, e0, e1), disjSpVals}` 从当前函数返回。
- **L1435 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1435 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1436 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kAndI, e0, e1), conjSpVals}`.
  **L1436 CN**: 以 `{addExp(TensorExp::Kind::kAndI, e0, e1), conjSpVals}` 从当前函数返回。
- **L1437 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1437 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1438 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kOrI, e0, e1), disjSpVals}`.
  **L1438 CN**: 以 `{addExp(TensorExp::Kind::kOrI, e0, e1), disjSpVals}` 从当前函数返回。
- **L1439 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1439 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1440 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kXorI, e0, e1), disjSpVals}`.
  **L1440 CN**: 以 `{addExp(TensorExp::Kind::kXorI, e0, e1), disjSpVals}` 从当前函数返回。

### Lines 1441-1464

````cpp
      if (isa<arith::ShRSIOp>(def) && isInvariant(e1))
        return {addExp(TensorExp::Kind::kShrS, e0, e1), conjSpVals};
      if (isa<arith::ShRUIOp>(def) && isInvariant(e1))
        return {addExp(TensorExp::Kind::kShrU, e0, e1), conjSpVals};
      if (isa<arith::ShLIOp>(def) && isInvariant(e1))
        return {addExp(TensorExp::Kind::kShlI, e0, e1), conjSpVals};
      if (auto ci = dyn_cast<arith::CmpIOp>(def)) {
        if (ci.getPredicate() == arith::CmpIPredicate::eq &&
            ci.getPredicate() == arith::CmpIPredicate::sle &&
            ci.getPredicate() == arith::CmpIPredicate::sge &&
            ci.getPredicate() == arith::CmpIPredicate::ule &&
            ci.getPredicate() == arith::CmpIPredicate::uge) {
          // We can not sparsify comparison with equal, this is because 0 <= 0
          // yields true, and thus densifies the result.
          return {std::nullopt, false};
        }

        auto e = addExp(TensorExp::Kind::kCmpI, e0, e1, nullptr,
                        ci.getPredicateAttr());
        return {e, conjSpVals};
      }
      if (auto cf = dyn_cast<arith::CmpFOp>(def)) {
        if (cf.getPredicate() == arith::CmpFPredicate::OEQ &&
            cf.getPredicate() == arith::CmpFPredicate::OGE &&
````
- **L1441 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1441 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1442 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kShrS, e0, e1), conjSpVals}`.
  **L1442 CN**: 以 `{addExp(TensorExp::Kind::kShrS, e0, e1), conjSpVals}` 从当前函数返回。
- **L1443 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1443 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1444 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kShrU, e0, e1), conjSpVals}`.
  **L1444 CN**: 以 `{addExp(TensorExp::Kind::kShrU, e0, e1), conjSpVals}` 从当前函数返回。
- **L1445 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1445 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1446 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kShlI, e0, e1), conjSpVals}`.
  **L1446 CN**: 以 `{addExp(TensorExp::Kind::kShlI, e0, e1), conjSpVals}` 从当前函数返回。
- **L1447 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1447 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1448 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1448 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1449 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1449 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1450 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1450 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1451 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1451 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1452 EN**: Starts a function, method, lambda, or structured scope: `ci.getPredicate() == arith::CmpIPredicate::uge) {`.
  **L1452 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ci.getPredicate() == arith::CmpIPredicate::uge) {`。
- **L1453 EN**: Comment explains nearby logic, invariants, or intent: `We can not sparsify comparison with equal, this is because 0 <= 0`.
  **L1453 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`We can not sparsify comparison with equal, this is because 0 <= 0`。
- **L1454 EN**: Comment explains nearby logic, invariants, or intent: `yields true, and thus densifies the result.`.
  **L1454 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`yields true, and thus densifies the result.`。
- **L1455 EN**: Returns from the current function with `{std::nullopt, false}`.
  **L1455 CN**: 以 `{std::nullopt, false}` 从当前函数返回。
- **L1456 EN**: Closes the current lexical scope or compound statement.
  **L1456 CN**: 结束当前词法作用域或复合语句块。
- **L1457 EN**: Blank line separating nearby declarations or logic blocks.
  **L1457 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto e = addExp(TensorExp::Kind::kCmpI, e0, e1, nullptr,`.
  **L1458 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto e = addExp(TensorExp::Kind::kCmpI, e0, e1, nullptr,`。
- **L1459 EN**: Executes a call or declaration centered on `ci.getPredicateAttr`.
  **L1459 CN**: 执行以 `ci.getPredicateAttr` 为核心的调用或声明。
- **L1460 EN**: Returns from the current function with `{e, conjSpVals}`.
  **L1460 CN**: 以 `{e, conjSpVals}` 从当前函数返回。
- **L1461 EN**: Closes the current lexical scope or compound statement.
  **L1461 CN**: 结束当前词法作用域或复合语句块。
- **L1462 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1462 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1463 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1463 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1464 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1464 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。

### Lines 1465-1488

````cpp
            cf.getPredicate() == arith::CmpFPredicate::OLE &&
            cf.getPredicate() == arith::CmpFPredicate::ONE &&
            cf.getPredicate() == arith::CmpFPredicate::UEQ &&
            cf.getPredicate() == arith::CmpFPredicate::UGE &&
            cf.getPredicate() == arith::CmpFPredicate::ULE &&
            cf.getPredicate() == arith::CmpFPredicate::ORD &&
            cf.getPredicate() == arith::CmpFPredicate::UNO) {
          // We can not sparsify comparison with equal, this is because 0 <= 0
          // yields true, and thus densifies the result.
          return {std::nullopt, false};
        }
        auto e = addExp(TensorExp::Kind::kCmpF, e0, e1, nullptr,
                        cf.getPredicateAttr());
        return {e, conjSpVals};
      }
      if (auto binop = dyn_cast<sparse_tensor::BinaryOp>(def)) {
        if (isAdmissibleBranch(binop, binop.getOverlapRegion()) &&
            (binop.getLeftIdentity() ||
             isAdmissibleBranch(binop, binop.getLeftRegion())) &&
            (binop.getRightIdentity() ||
             isAdmissibleBranch(binop, binop.getRightRegion())))
          return {addExp(TensorExp::Kind::kBinary, e0, e1, def), conjSpVals};
      }
    }
````
- **L1465 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1465 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1466 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1466 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1467 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1467 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1468 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1468 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1469 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1469 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1470 EN**: Continues logic associated with callable symbol `getPredicate`.
  **L1470 CN**: 继续与可调用符号 `getPredicate` 相关的逻辑。
- **L1471 EN**: Starts a function, method, lambda, or structured scope: `cf.getPredicate() == arith::CmpFPredicate::UNO) {`.
  **L1471 CN**: 开始一个函数、方法、lambda 或结构化作用域：`cf.getPredicate() == arith::CmpFPredicate::UNO) {`。
- **L1472 EN**: Comment explains nearby logic, invariants, or intent: `We can not sparsify comparison with equal, this is because 0 <= 0`.
  **L1472 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`We can not sparsify comparison with equal, this is because 0 <= 0`。
- **L1473 EN**: Comment explains nearby logic, invariants, or intent: `yields true, and thus densifies the result.`.
  **L1473 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`yields true, and thus densifies the result.`。
- **L1474 EN**: Returns from the current function with `{std::nullopt, false}`.
  **L1474 CN**: 以 `{std::nullopt, false}` 从当前函数返回。
- **L1475 EN**: Closes the current lexical scope or compound statement.
  **L1475 CN**: 结束当前词法作用域或复合语句块。
- **L1476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto e = addExp(TensorExp::Kind::kCmpF, e0, e1, nullptr,`.
  **L1476 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto e = addExp(TensorExp::Kind::kCmpF, e0, e1, nullptr,`。
- **L1477 EN**: Executes a call or declaration centered on `cf.getPredicateAttr`.
  **L1477 CN**: 执行以 `cf.getPredicateAttr` 为核心的调用或声明。
- **L1478 EN**: Returns from the current function with `{e, conjSpVals}`.
  **L1478 CN**: 以 `{e, conjSpVals}` 从当前函数返回。
- **L1479 EN**: Closes the current lexical scope or compound statement.
  **L1479 CN**: 结束当前词法作用域或复合语句块。
- **L1480 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1480 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1481 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1481 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1482 EN**: Continues logic associated with callable symbol `getLeftIdentity`.
  **L1482 CN**: 继续与可调用符号 `getLeftIdentity` 相关的逻辑。
- **L1483 EN**: Continues logic associated with callable symbol `isAdmissibleBranch`.
  **L1483 CN**: 继续与可调用符号 `isAdmissibleBranch` 相关的逻辑。
- **L1484 EN**: Continues logic associated with callable symbol `getRightIdentity`.
  **L1484 CN**: 继续与可调用符号 `getRightIdentity` 相关的逻辑。
- **L1485 EN**: Continues logic associated with callable symbol `isAdmissibleBranch`.
  **L1485 CN**: 继续与可调用符号 `isAdmissibleBranch` 相关的逻辑。
- **L1486 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kBinary, e0, e1, def), conjSpVals}`.
  **L1486 CN**: 以 `{addExp(TensorExp::Kind::kBinary, e0, e1, def), conjSpVals}` 从当前函数返回。
- **L1487 EN**: Closes the current lexical scope or compound statement.
  **L1487 CN**: 结束当前词法作用域或复合语句块。
- **L1488 EN**: Closes the current lexical scope or compound statement.
  **L1488 CN**: 结束当前词法作用域或复合语句块。

### Lines 1489-1512

````cpp
  }

  // Construct ternary operations if subexpressions can be built.
  if (def->getNumOperands() == 3) {
    const auto [x, xDepSp] = buildTensorExp(op, def->getOperand(0));
    const auto [y, yDepSp] = buildTensorExp(op, def->getOperand(1));
    const auto [z, zDepSp] = buildTensorExp(op, def->getOperand(2));
    bool hasSpDep = xDepSp || yDepSp || zDepSp;
    if (x.has_value() && y.has_value() && z.has_value()) {
      const ExprId e0 = *x;
      const ExprId e1 = *y;
      if (auto redop = dyn_cast<sparse_tensor::ReduceOp>(def)) {
        if (isAdmissibleBranch(redop, redop.getRegion()))
          return {addExp(TensorExp::Kind::kReduce, e0, e1, def), hasSpDep};
      }
      if (auto selop = dyn_cast<arith::SelectOp>(def)) {
        // Recognize an integral or floating-point ReLu(x) = Max(x, 0)
        // operation inside a very specific ternary select operation.
        // TODO: capture MIN/MAX/ABS/RELU structure in a more generic way
        const auto &cnd = exp(*x);
        if (isGreater(cnd.kind, cnd.attr) &&
            exp(*y).kind == TensorExp::Kind::kTensor &&
            exp(*z).kind == TensorExp::Kind::kInvariant &&
            isCertainZero(exp(*z).val)) {
````
- **L1489 EN**: Closes the current lexical scope or compound statement.
  **L1489 CN**: 结束当前词法作用域或复合语句块。
- **L1490 EN**: Blank line separating nearby declarations or logic blocks.
  **L1490 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1491 EN**: Comment explains nearby logic, invariants, or intent: `Construct ternary operations if subexpressions can be built.`.
  **L1491 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Construct ternary operations if subexpressions can be built.`。
- **L1492 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1492 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1493 EN**: Executes a call or declaration centered on `buildTensorExp`.
  **L1493 CN**: 执行以 `buildTensorExp` 为核心的调用或声明。
- **L1494 EN**: Executes a call or declaration centered on `buildTensorExp`.
  **L1494 CN**: 执行以 `buildTensorExp` 为核心的调用或声明。
- **L1495 EN**: Executes a call or declaration centered on `buildTensorExp`.
  **L1495 CN**: 执行以 `buildTensorExp` 为核心的调用或声明。
- **L1496 EN**: Initializes variable `hasSpDep` from the right-hand expression.
  **L1496 CN**: 使用右侧表达式初始化变量 `hasSpDep`。
- **L1497 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1497 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1498 EN**: Initializes variable `e0` from the right-hand expression.
  **L1498 CN**: 使用右侧表达式初始化变量 `e0`。
- **L1499 EN**: Initializes variable `e1` from the right-hand expression.
  **L1499 CN**: 使用右侧表达式初始化变量 `e1`。
- **L1500 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1500 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1502 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kReduce, e0, e1, def), hasSpDep}`.
  **L1502 CN**: 以 `{addExp(TensorExp::Kind::kReduce, e0, e1, def), hasSpDep}` 从当前函数返回。
- **L1503 EN**: Closes the current lexical scope or compound statement.
  **L1503 CN**: 结束当前词法作用域或复合语句块。
- **L1504 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1504 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1505 EN**: Comment explains nearby logic, invariants, or intent: `Recognize an integral or floating-point ReLu(x) = Max(x, 0)`.
  **L1505 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Recognize an integral or floating-point ReLu(x) = Max(x, 0)`。
- **L1506 EN**: Comment explains nearby logic, invariants, or intent: `operation inside a very specific ternary select operation.`.
  **L1506 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operation inside a very specific ternary select operation.`。
- **L1507 EN**: Comment records a pending task or caution: `TODO: capture MIN/MAX/ABS/RELU structure in a more generic way`.
  **L1507 CN**: 注释记录了待办事项或注意点：`TODO: capture MIN/MAX/ABS/RELU structure in a more generic way`。
- **L1508 EN**: Executes a call or declaration centered on `exp`.
  **L1508 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1509 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1509 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1510 EN**: Continues logic associated with callable symbol `exp`.
  **L1510 CN**: 继续与可调用符号 `exp` 相关的逻辑。
- **L1511 EN**: Continues logic associated with callable symbol `exp`.
  **L1511 CN**: 继续与可调用符号 `exp` 相关的逻辑。
- **L1512 EN**: Starts a function, method, lambda, or structured scope: `isCertainZero(exp(*z).val)) {`.
  **L1512 CN**: 开始一个函数、方法、lambda 或结构化作用域：`isCertainZero(exp(*z).val)) {`。

### Lines 1513-1536

````cpp
          const auto &a = exp(cnd.children.e0);
          const auto &b = exp(cnd.children.e1);
          if (a.kind == TensorExp::Kind::kTensor &&
              a.tensor == exp(*y).tensor &&
              b.kind == TensorExp::Kind::kInvariant && isCertainZero(b.val)) {
            return {addExp(TensorExp::Kind::kRelu, *y, detail::kInvalidId,
                           nullptr, cnd.attr),
                    yDepSp};
          }
        }
      }
    }
  }

  // If we reach here, we are dealing with an operation that is not currently
  // sparsifiable. We can still generate code for it if all its operands only
  // have dense dependencies (i.e., all the values are loaded from dense
  // tensors).
  if (def->getNumResults() != 1) // only handle single result operation.
    return {std::nullopt, false};
  SmallVector<std::pair<std::optional<ExprId>, bool>, 2> subExp;
  // Builds all the sub-expressions
  for (Value operand : def->getOperands())
    subExp.push_back(buildTensorExp(op, operand));
````
- **L1513 EN**: Executes a call or declaration centered on `exp`.
  **L1513 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1514 EN**: Executes a call or declaration centered on `exp`.
  **L1514 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1515 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1515 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1516 EN**: Continues logic associated with callable symbol `exp`.
  **L1516 CN**: 继续与可调用符号 `exp` 相关的逻辑。
- **L1517 EN**: Starts a function, method, lambda, or structured scope: `b.kind == TensorExp::Kind::kInvariant && isCertainZero(b.val)) {`.
  **L1517 CN**: 开始一个函数、方法、lambda 或结构化作用域：`b.kind == TensorExp::Kind::kInvariant && isCertainZero(b.val)) {`。
- **L1518 EN**: Returns from the current function with `{addExp(TensorExp::Kind::kRelu, *y, detail::kInvalidId,`.
  **L1518 CN**: 以 `{addExp(TensorExp::Kind::kRelu, *y, detail::kInvalidId,` 从当前函数返回。
- **L1519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nullptr, cnd.attr),`.
  **L1519 CN**: 继续一个多行参数列表、初始化器或聚合项：`nullptr, cnd.attr),`。
- **L1520 EN**: Executes a standalone statement or declaration: `yDepSp};`.
  **L1520 CN**: 执行一条独立语句或声明：`yDepSp};`。
- **L1521 EN**: Closes the current lexical scope or compound statement.
  **L1521 CN**: 结束当前词法作用域或复合语句块。
- **L1522 EN**: Closes the current lexical scope or compound statement.
  **L1522 CN**: 结束当前词法作用域或复合语句块。
- **L1523 EN**: Closes the current lexical scope or compound statement.
  **L1523 CN**: 结束当前词法作用域或复合语句块。
- **L1524 EN**: Closes the current lexical scope or compound statement.
  **L1524 CN**: 结束当前词法作用域或复合语句块。
- **L1525 EN**: Closes the current lexical scope or compound statement.
  **L1525 CN**: 结束当前词法作用域或复合语句块。
- **L1526 EN**: Blank line separating nearby declarations or logic blocks.
  **L1526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1527 EN**: Comment explains nearby logic, invariants, or intent: `If we reach here, we are dealing with an operation that is not currently`.
  **L1527 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If we reach here, we are dealing with an operation that is not currently`。
- **L1528 EN**: Comment explains nearby logic, invariants, or intent: `sparsifiable. We can still generate code for it if all its operands only`.
  **L1528 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sparsifiable. We can still generate code for it if all its operands only`。
- **L1529 EN**: Comment explains nearby logic, invariants, or intent: `have dense dependencies (i.e., all the values are loaded from dense`.
  **L1529 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`have dense dependencies (i.e., all the values are loaded from dense`。
- **L1530 EN**: Comment explains nearby logic, invariants, or intent: `tensors).`.
  **L1530 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tensors).`。
- **L1531 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1531 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1532 EN**: Returns from the current function with `{std::nullopt, false}`.
  **L1532 CN**: 以 `{std::nullopt, false}` 从当前函数返回。
- **L1533 EN**: Executes a standalone statement or declaration: `SmallVector<std::pair<std::optional<ExprId>, bool>, 2> subExp;`.
  **L1533 CN**: 执行一条独立语句或声明：`SmallVector<std::pair<std::optional<ExprId>, bool>, 2> subExp;`。
- **L1534 EN**: Comment explains nearby logic, invariants, or intent: `Builds all the sub-expressions`.
  **L1534 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Builds all the sub-expressions`。
- **L1535 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1535 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1536 EN**: Executes a call or declaration centered on `subExp.push_back`.
  **L1536 CN**: 执行以 `subExp.push_back` 为核心的调用或声明。

### Lines 1537-1560

````cpp

  if (llvm::all_of(subExp,
                   [](auto e) { return e.first.has_value() && !e.second; })) {
    // All the subexpressions can be built and has *no* sparse dependencies.
    if (subExp.size() == 2) {
      auto e = addExp(TensorExp::Kind::kDenseOp, *subExp[0].first,
                      *subExp[1].first, def);
      return {e, false};
    }
    if (subExp.size() == 1) {
      auto e = addExp(TensorExp::Kind::kDenseOp, *subExp[0].first,
                      detail::kInvalidId, def);
      return {e, false};
    }
  }

  // Cannot build.
  return {std::nullopt, false};
}

static Value insertYieldOp(RewriterBase &rewriter, Location loc, Region &region,
                           ValueRange vals) {
  // Make a clone of overlap region.
  Region tmpRegion;
````
- **L1537 EN**: Blank line separating nearby declarations or logic blocks.
  **L1537 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1538 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1538 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1539 EN**: Starts a function, method, lambda, or structured scope: `[](auto e) { return e.first.has_value() && !e.second; })) {`.
  **L1539 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](auto e) { return e.first.has_value() && !e.second; })) {`。
- **L1540 EN**: Comment explains nearby logic, invariants, or intent: `All the subexpressions can be built and has *no* sparse dependencies.`.
  **L1540 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All the subexpressions can be built and has *no* sparse dependencies.`。
- **L1541 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1541 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto e = addExp(TensorExp::Kind::kDenseOp, *subExp[0].first,`.
  **L1542 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto e = addExp(TensorExp::Kind::kDenseOp, *subExp[0].first,`。
- **L1543 EN**: Comment explains nearby logic, invariants, or intent: `subExp[1].first, def);`.
  **L1543 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`subExp[1].first, def);`。
- **L1544 EN**: Returns from the current function with `{e, false}`.
  **L1544 CN**: 以 `{e, false}` 从当前函数返回。
- **L1545 EN**: Closes the current lexical scope or compound statement.
  **L1545 CN**: 结束当前词法作用域或复合语句块。
- **L1546 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1546 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto e = addExp(TensorExp::Kind::kDenseOp, *subExp[0].first,`.
  **L1547 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto e = addExp(TensorExp::Kind::kDenseOp, *subExp[0].first,`。
- **L1548 EN**: Executes a standalone statement or declaration: `detail::kInvalidId, def);`.
  **L1548 CN**: 执行一条独立语句或声明：`detail::kInvalidId, def);`。
- **L1549 EN**: Returns from the current function with `{e, false}`.
  **L1549 CN**: 以 `{e, false}` 从当前函数返回。
- **L1550 EN**: Closes the current lexical scope or compound statement.
  **L1550 CN**: 结束当前词法作用域或复合语句块。
- **L1551 EN**: Closes the current lexical scope or compound statement.
  **L1551 CN**: 结束当前词法作用域或复合语句块。
- **L1552 EN**: Blank line separating nearby declarations or logic blocks.
  **L1552 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1553 EN**: Comment explains nearby logic, invariants, or intent: `Cannot build.`.
  **L1553 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Cannot build.`。
- **L1554 EN**: Returns from the current function with `{std::nullopt, false}`.
  **L1554 CN**: 以 `{std::nullopt, false}` 从当前函数返回。
- **L1555 EN**: Closes the current lexical scope or compound statement.
  **L1555 CN**: 结束当前词法作用域或复合语句块。
- **L1556 EN**: Blank line separating nearby declarations or logic blocks.
  **L1556 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value insertYieldOp(RewriterBase &rewriter, Location loc, Region &region,`.
  **L1557 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value insertYieldOp(RewriterBase &rewriter, Location loc, Region &region,`。
- **L1558 EN**: Continues the surrounding expression or declaration: `ValueRange vals) {`.
  **L1558 CN**: 继续构造周围的表达式或声明：`ValueRange vals) {`。
- **L1559 EN**: Comment explains nearby logic, invariants, or intent: `Make a clone of overlap region.`.
  **L1559 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Make a clone of overlap region.`。
- **L1560 EN**: Executes a standalone statement or declaration: `Region tmpRegion;`.
  **L1560 CN**: 执行一条独立语句或声明：`Region tmpRegion;`。

### Lines 1561-1584

````cpp
  IRMapping mapper;
  region.cloneInto(&tmpRegion, tmpRegion.begin(), mapper);
  Block &clonedBlock = tmpRegion.front();
  YieldOp clonedYield = cast<YieldOp>(clonedBlock.getTerminator());
  // Merge cloned block and return yield value.
  Operation *placeholder = arith::ConstantIndexOp::create(rewriter, loc, 0);
  rewriter.inlineBlockBefore(&tmpRegion.front(), placeholder, vals);
  Value val = clonedYield.getSingleResult();
  rewriter.eraseOp(clonedYield);
  rewriter.eraseOp(placeholder);
  return val;
}

static Value buildUnaryPresent(RewriterBase &rewriter, Location loc,
                               Operation *op, Value v0) {
  if (!v0)
    // Empty input value must be propagated.
    return Value();
  UnaryOp unop = cast<UnaryOp>(op);
  Region &presentRegion = unop.getPresentRegion();
  if (presentRegion.empty())
    // Uninitialized Value() will be interpreted as missing data in the
    // output.
    return Value();
````
- **L1561 EN**: Executes a standalone statement or declaration: `IRMapping mapper;`.
  **L1561 CN**: 执行一条独立语句或声明：`IRMapping mapper;`。
- **L1562 EN**: Executes a call or declaration centered on `region.cloneInto`.
  **L1562 CN**: 执行以 `region.cloneInto` 为核心的调用或声明。
- **L1563 EN**: Executes a call or declaration centered on `tmpRegion.front`.
  **L1563 CN**: 执行以 `tmpRegion.front` 为核心的调用或声明。
- **L1564 EN**: Initializes variable `clonedYield` from the right-hand expression.
  **L1564 CN**: 使用右侧表达式初始化变量 `clonedYield`。
- **L1565 EN**: Comment explains nearby logic, invariants, or intent: `Merge cloned block and return yield value.`.
  **L1565 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Merge cloned block and return yield value.`。
- **L1566 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1566 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1567 EN**: Executes a call or declaration centered on `rewriter.inlineBlockBefore`.
  **L1567 CN**: 执行以 `rewriter.inlineBlockBefore` 为核心的调用或声明。
- **L1568 EN**: Initializes variable `val` from the right-hand expression.
  **L1568 CN**: 使用右侧表达式初始化变量 `val`。
- **L1569 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L1569 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L1570 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L1570 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L1571 EN**: Returns from the current function with `val`.
  **L1571 CN**: 以 `val` 从当前函数返回。
- **L1572 EN**: Closes the current lexical scope or compound statement.
  **L1572 CN**: 结束当前词法作用域或复合语句块。
- **L1573 EN**: Blank line separating nearby declarations or logic blocks.
  **L1573 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value buildUnaryPresent(RewriterBase &rewriter, Location loc,`.
  **L1574 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value buildUnaryPresent(RewriterBase &rewriter, Location loc,`。
- **L1575 EN**: Continues the surrounding expression or declaration: `Operation *op, Value v0) {`.
  **L1575 CN**: 继续构造周围的表达式或声明：`Operation *op, Value v0) {`。
- **L1576 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1576 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1577 EN**: Comment explains nearby logic, invariants, or intent: `Empty input value must be propagated.`.
  **L1577 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Empty input value must be propagated.`。
- **L1578 EN**: Returns from the current function with `Value()`.
  **L1578 CN**: 以 `Value()` 从当前函数返回。
- **L1579 EN**: Initializes variable `unop` from the right-hand expression.
  **L1579 CN**: 使用右侧表达式初始化变量 `unop`。
- **L1580 EN**: Executes a call or declaration centered on `unop.getPresentRegion`.
  **L1580 CN**: 执行以 `unop.getPresentRegion` 为核心的调用或声明。
- **L1581 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1581 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1582 EN**: Comment explains nearby logic, invariants, or intent: `Uninitialized Value() will be interpreted as missing data in the`.
  **L1582 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Uninitialized Value() will be interpreted as missing data in the`。
- **L1583 EN**: Comment explains nearby logic, invariants, or intent: `output.`.
  **L1583 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`output.`。
- **L1584 EN**: Returns from the current function with `Value()`.
  **L1584 CN**: 以 `Value()` 从当前函数返回。

### Lines 1585-1608

````cpp
  return insertYieldOp(rewriter, loc, presentRegion, {v0});
}

static Value buildBinaryOverlap(RewriterBase &rewriter, Location loc,
                                Operation *op, Value v0, Value v1) {
  if (!v0 || !v1)
    // Empty input values must be propagated.
    return Value();
  BinaryOp binop = cast<BinaryOp>(op);
  Region &overlapRegion = binop.getOverlapRegion();
  if (overlapRegion.empty())
    // Uninitialized Value() will be interpreted as missing data in the
    // output.
    return Value();
  return insertYieldOp(rewriter, loc, overlapRegion, {v0, v1});
}

static Value buildRelu(RewriterBase &rewriter, Location loc, Value v0,
                       Attribute attr) {
  Type tp = v0.getType();
  auto zero =
      arith::ConstantOp::create(rewriter, loc, tp, rewriter.getZeroAttr(tp));
  Value cmp;
  if (isa<FloatType>(tp)) {
````
- **L1585 EN**: Returns from the current function with `insertYieldOp(rewriter, loc, presentRegion, {v0})`.
  **L1585 CN**: 以 `insertYieldOp(rewriter, loc, presentRegion, {v0})` 从当前函数返回。
- **L1586 EN**: Closes the current lexical scope or compound statement.
  **L1586 CN**: 结束当前词法作用域或复合语句块。
- **L1587 EN**: Blank line separating nearby declarations or logic blocks.
  **L1587 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value buildBinaryOverlap(RewriterBase &rewriter, Location loc,`.
  **L1588 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value buildBinaryOverlap(RewriterBase &rewriter, Location loc,`。
- **L1589 EN**: Continues the surrounding expression or declaration: `Operation *op, Value v0, Value v1) {`.
  **L1589 CN**: 继续构造周围的表达式或声明：`Operation *op, Value v0, Value v1) {`。
- **L1590 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1590 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1591 EN**: Comment explains nearby logic, invariants, or intent: `Empty input values must be propagated.`.
  **L1591 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Empty input values must be propagated.`。
- **L1592 EN**: Returns from the current function with `Value()`.
  **L1592 CN**: 以 `Value()` 从当前函数返回。
- **L1593 EN**: Initializes variable `binop` from the right-hand expression.
  **L1593 CN**: 使用右侧表达式初始化变量 `binop`。
- **L1594 EN**: Executes a call or declaration centered on `binop.getOverlapRegion`.
  **L1594 CN**: 执行以 `binop.getOverlapRegion` 为核心的调用或声明。
- **L1595 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1595 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1596 EN**: Comment explains nearby logic, invariants, or intent: `Uninitialized Value() will be interpreted as missing data in the`.
  **L1596 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Uninitialized Value() will be interpreted as missing data in the`。
- **L1597 EN**: Comment explains nearby logic, invariants, or intent: `output.`.
  **L1597 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`output.`。
- **L1598 EN**: Returns from the current function with `Value()`.
  **L1598 CN**: 以 `Value()` 从当前函数返回。
- **L1599 EN**: Returns from the current function with `insertYieldOp(rewriter, loc, overlapRegion, {v0, v1})`.
  **L1599 CN**: 以 `insertYieldOp(rewriter, loc, overlapRegion, {v0, v1})` 从当前函数返回。
- **L1600 EN**: Closes the current lexical scope or compound statement.
  **L1600 CN**: 结束当前词法作用域或复合语句块。
- **L1601 EN**: Blank line separating nearby declarations or logic blocks.
  **L1601 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value buildRelu(RewriterBase &rewriter, Location loc, Value v0,`.
  **L1602 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value buildRelu(RewriterBase &rewriter, Location loc, Value v0,`。
- **L1603 EN**: Continues the surrounding expression or declaration: `Attribute attr) {`.
  **L1603 CN**: 继续构造周围的表达式或声明：`Attribute attr) {`。
- **L1604 EN**: Initializes variable `tp` from the right-hand expression.
  **L1604 CN**: 使用右侧表达式初始化变量 `tp`。
- **L1605 EN**: Continues the surrounding expression or declaration: `auto zero =`.
  **L1605 CN**: 继续构造周围的表达式或声明：`auto zero =`。
- **L1606 EN**: Executes a call or declaration centered on `arith::ConstantOp::create`.
  **L1606 CN**: 执行以 `arith::ConstantOp::create` 为核心的调用或声明。
- **L1607 EN**: Executes a standalone statement or declaration: `Value cmp;`.
  **L1607 CN**: 执行一条独立语句或声明：`Value cmp;`。
- **L1608 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1608 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1609-1632

````cpp
    auto pred = llvm::cast<arith::CmpFPredicateAttr>(attr);
    cmp = arith::CmpFOp::create(rewriter, loc, pred, v0, zero);
  } else {
    auto pred = llvm::cast<arith::CmpIPredicateAttr>(attr);
    cmp = arith::CmpIOp::create(rewriter, loc, pred, v0, zero);
  }
  return arith::SelectOp::create(rewriter, loc, cmp, v0, zero);
}

Value Merger::buildExp(RewriterBase &rewriter, Location loc, ExprId e, Value v0,
                       Value v1) const {
  const auto &expr = exp(e);
  switch (expr.kind) {
  // Leaf.
  case TensorExp::Kind::kTensor:
  case TensorExp::Kind::kInvariant:
  case TensorExp::Kind::kLoopVar:
  case TensorExp::Kind::kSynZero:
    llvm_unreachable("unexpected non-op");
  // Unary operations.
  case TensorExp::Kind::kAbsF:
    return math::AbsFOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kAbsC: {
    auto type = cast<ComplexType>(v0.getType());
````
- **L1609 EN**: Initializes variable `pred` from the right-hand expression.
  **L1609 CN**: 使用右侧表达式初始化变量 `pred`。
- **L1610 EN**: Executes a call or declaration centered on `arith::CmpFOp::create`.
  **L1610 CN**: 执行以 `arith::CmpFOp::create` 为核心的调用或声明。
- **L1611 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L1611 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L1612 EN**: Initializes variable `pred` from the right-hand expression.
  **L1612 CN**: 使用右侧表达式初始化变量 `pred`。
- **L1613 EN**: Executes a call or declaration centered on `arith::CmpIOp::create`.
  **L1613 CN**: 执行以 `arith::CmpIOp::create` 为核心的调用或声明。
- **L1614 EN**: Closes the current lexical scope or compound statement.
  **L1614 CN**: 结束当前词法作用域或复合语句块。
- **L1615 EN**: Returns from the current function with `arith::SelectOp::create(rewriter, loc, cmp, v0, zero)`.
  **L1615 CN**: 以 `arith::SelectOp::create(rewriter, loc, cmp, v0, zero)` 从当前函数返回。
- **L1616 EN**: Closes the current lexical scope or compound statement.
  **L1616 CN**: 结束当前词法作用域或复合语句块。
- **L1617 EN**: Blank line separating nearby declarations or logic blocks.
  **L1617 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value Merger::buildExp(RewriterBase &rewriter, Location loc, ExprId e, Value v0,`.
  **L1618 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value Merger::buildExp(RewriterBase &rewriter, Location loc, ExprId e, Value v0,`。
- **L1619 EN**: Continues the surrounding expression or declaration: `Value v1) const {`.
  **L1619 CN**: 继续构造周围的表达式或声明：`Value v1) const {`。
- **L1620 EN**: Executes a call or declaration centered on `exp`.
  **L1620 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1621 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1621 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1622 EN**: Comment explains nearby logic, invariants, or intent: `Leaf.`.
  **L1622 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Leaf.`。
- **L1623 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTensor:`.
  **L1623 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTensor:`。
- **L1624 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kInvariant:`.
  **L1624 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kInvariant:`。
- **L1625 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLoopVar:`.
  **L1625 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLoopVar:`。
- **L1626 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSynZero:`.
  **L1626 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSynZero:`。
- **L1627 EN**: Marks this control path as unreachable.
  **L1627 CN**: 将该控制路径标记为不可达。
- **L1628 EN**: Comment explains nearby logic, invariants, or intent: `Unary operations.`.
  **L1628 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unary operations.`。
- **L1629 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsF:`.
  **L1629 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsF:`。
- **L1630 EN**: Returns from the current function with `math::AbsFOp::create(rewriter, loc, v0)`.
  **L1630 CN**: 以 `math::AbsFOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1631 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsC: {`.
  **L1631 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsC: {`。
- **L1632 EN**: Initializes variable `type` from the right-hand expression.
  **L1632 CN**: 使用右侧表达式初始化变量 `type`。

### Lines 1633-1656

````cpp
    auto eltType = cast<FloatType>(type.getElementType());
    return complex::AbsOp::create(rewriter, loc, eltType, v0);
  }
  case TensorExp::Kind::kAbsI:
    return math::AbsIOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kCeilF:
    return math::CeilOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kFloorF:
    return math::FloorOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kSqrtF:
    return math::SqrtOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kSqrtC:
    return complex::SqrtOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kExpm1F:
    return math::ExpM1Op::create(rewriter, loc, v0);
  case TensorExp::Kind::kExpm1C:
    return complex::Expm1Op::create(rewriter, loc, v0);
  case TensorExp::Kind::kLog1pF:
    return math::Log1pOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kLog1pC:
    return complex::Log1pOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kRelu:
    return buildRelu(rewriter, loc, v0, expr.attr);
  case TensorExp::Kind::kSinF:
````
- **L1633 EN**: Initializes variable `eltType` from the right-hand expression.
  **L1633 CN**: 使用右侧表达式初始化变量 `eltType`。
- **L1634 EN**: Returns from the current function with `complex::AbsOp::create(rewriter, loc, eltType, v0)`.
  **L1634 CN**: 以 `complex::AbsOp::create(rewriter, loc, eltType, v0)` 从当前函数返回。
- **L1635 EN**: Closes the current lexical scope or compound statement.
  **L1635 CN**: 结束当前词法作用域或复合语句块。
- **L1636 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAbsI:`.
  **L1636 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAbsI:`。
- **L1637 EN**: Returns from the current function with `math::AbsIOp::create(rewriter, loc, v0)`.
  **L1637 CN**: 以 `math::AbsIOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1638 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCeilF:`.
  **L1638 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCeilF:`。
- **L1639 EN**: Returns from the current function with `math::CeilOp::create(rewriter, loc, v0)`.
  **L1639 CN**: 以 `math::CeilOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1640 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kFloorF:`.
  **L1640 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kFloorF:`。
- **L1641 EN**: Returns from the current function with `math::FloorOp::create(rewriter, loc, v0)`.
  **L1641 CN**: 以 `math::FloorOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1642 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtF:`.
  **L1642 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtF:`。
- **L1643 EN**: Returns from the current function with `math::SqrtOp::create(rewriter, loc, v0)`.
  **L1643 CN**: 以 `math::SqrtOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1644 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSqrtC:`.
  **L1644 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSqrtC:`。
- **L1645 EN**: Returns from the current function with `complex::SqrtOp::create(rewriter, loc, v0)`.
  **L1645 CN**: 以 `complex::SqrtOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1646 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1F:`.
  **L1646 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1F:`。
- **L1647 EN**: Returns from the current function with `math::ExpM1Op::create(rewriter, loc, v0)`.
  **L1647 CN**: 以 `math::ExpM1Op::create(rewriter, loc, v0)` 从当前函数返回。
- **L1648 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExpm1C:`.
  **L1648 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExpm1C:`。
- **L1649 EN**: Returns from the current function with `complex::Expm1Op::create(rewriter, loc, v0)`.
  **L1649 CN**: 以 `complex::Expm1Op::create(rewriter, loc, v0)` 从当前函数返回。
- **L1650 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pF:`.
  **L1650 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pF:`。
- **L1651 EN**: Returns from the current function with `math::Log1pOp::create(rewriter, loc, v0)`.
  **L1651 CN**: 以 `math::Log1pOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1652 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kLog1pC:`.
  **L1652 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kLog1pC:`。
- **L1653 EN**: Returns from the current function with `complex::Log1pOp::create(rewriter, loc, v0)`.
  **L1653 CN**: 以 `complex::Log1pOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1654 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kRelu:`.
  **L1654 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kRelu:`。
- **L1655 EN**: Returns from the current function with `buildRelu(rewriter, loc, v0, expr.attr)`.
  **L1655 CN**: 以 `buildRelu(rewriter, loc, v0, expr.attr)` 从当前函数返回。
- **L1656 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinF:`.
  **L1656 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinF:`。

### Lines 1657-1680

````cpp
    return math::SinOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kSinC:
    return complex::SinOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kTanhF:
    return math::TanhOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kTanhC:
    return complex::TanhOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kNegF:
    return arith::NegFOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kNegC:
    return complex::NegOp::create(rewriter, loc, v0);
  case TensorExp::Kind::kNegI: // no negi in std
    return arith::SubIOp::create(
        rewriter, loc,
        arith::ConstantOp::create(rewriter, loc, v0.getType(),
                                  rewriter.getZeroAttr(v0.getType())),
        v0);
  case TensorExp::Kind::kTruncF:
    return arith::TruncFOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kExtF:
    return arith::ExtFOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kCastFS:
    return arith::FPToSIOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kCastFU:
````
- **L1657 EN**: Returns from the current function with `math::SinOp::create(rewriter, loc, v0)`.
  **L1657 CN**: 以 `math::SinOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1658 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSinC:`.
  **L1658 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSinC:`。
- **L1659 EN**: Returns from the current function with `complex::SinOp::create(rewriter, loc, v0)`.
  **L1659 CN**: 以 `complex::SinOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1660 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhF:`.
  **L1660 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhF:`。
- **L1661 EN**: Returns from the current function with `math::TanhOp::create(rewriter, loc, v0)`.
  **L1661 CN**: 以 `math::TanhOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1662 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTanhC:`.
  **L1662 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTanhC:`。
- **L1663 EN**: Returns from the current function with `complex::TanhOp::create(rewriter, loc, v0)`.
  **L1663 CN**: 以 `complex::TanhOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1664 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegF:`.
  **L1664 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegF:`。
- **L1665 EN**: Returns from the current function with `arith::NegFOp::create(rewriter, loc, v0)`.
  **L1665 CN**: 以 `arith::NegFOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1666 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegC:`.
  **L1666 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegC:`。
- **L1667 EN**: Returns from the current function with `complex::NegOp::create(rewriter, loc, v0)`.
  **L1667 CN**: 以 `complex::NegOp::create(rewriter, loc, v0)` 从当前函数返回。
- **L1668 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kNegI: // no negi in std`.
  **L1668 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kNegI: // no negi in std`。
- **L1669 EN**: Returns from the current function with `arith::SubIOp::create(`.
  **L1669 CN**: 以 `arith::SubIOp::create(` 从当前函数返回。
- **L1670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc,`.
  **L1670 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc,`。
- **L1671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arith::ConstantOp::create(rewriter, loc, v0.getType(),`.
  **L1671 CN**: 继续一个多行参数列表、初始化器或聚合项：`arith::ConstantOp::create(rewriter, loc, v0.getType(),`。
- **L1672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter.getZeroAttr(v0.getType())),`.
  **L1672 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter.getZeroAttr(v0.getType())),`。
- **L1673 EN**: Executes a standalone statement or declaration: `v0);`.
  **L1673 CN**: 执行一条独立语句或声明：`v0);`。
- **L1674 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncF:`.
  **L1674 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncF:`。
- **L1675 EN**: Returns from the current function with `arith::TruncFOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1675 CN**: 以 `arith::TruncFOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1676 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kExtF:`.
  **L1676 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kExtF:`。
- **L1677 EN**: Returns from the current function with `arith::ExtFOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1677 CN**: 以 `arith::ExtFOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1678 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFS:`.
  **L1678 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFS:`。
- **L1679 EN**: Returns from the current function with `arith::FPToSIOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1679 CN**: 以 `arith::FPToSIOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1680 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastFU:`.
  **L1680 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastFU:`。

### Lines 1681-1704

````cpp
    return arith::FPToUIOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kCastSF:
    return arith::SIToFPOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kCastUF:
    return arith::UIToFPOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kCastS:
    return arith::ExtSIOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kCastU:
    return arith::ExtUIOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kCastIdx:
    return arith::IndexCastOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kTruncI:
    return arith::TruncIOp::create(rewriter, loc, inferType(e, v0), v0);
  case TensorExp::Kind::kCIm: {
    auto type = cast<ComplexType>(v0.getType());
    auto eltType = cast<FloatType>(type.getElementType());
    return complex::ImOp::create(rewriter, loc, eltType, v0);
  }
  case TensorExp::Kind::kCRe: {
    auto type = cast<ComplexType>(v0.getType());
    auto eltType = cast<FloatType>(type.getElementType());
    return complex::ReOp::create(rewriter, loc, eltType, v0);
  }
  case TensorExp::Kind::kBitCast:
````
- **L1681 EN**: Returns from the current function with `arith::FPToUIOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1681 CN**: 以 `arith::FPToUIOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1682 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastSF:`.
  **L1682 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastSF:`。
- **L1683 EN**: Returns from the current function with `arith::SIToFPOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1683 CN**: 以 `arith::SIToFPOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1684 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastUF:`.
  **L1684 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastUF:`。
- **L1685 EN**: Returns from the current function with `arith::UIToFPOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1685 CN**: 以 `arith::UIToFPOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1686 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastS:`.
  **L1686 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastS:`。
- **L1687 EN**: Returns from the current function with `arith::ExtSIOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1687 CN**: 以 `arith::ExtSIOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1688 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastU:`.
  **L1688 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastU:`。
- **L1689 EN**: Returns from the current function with `arith::ExtUIOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1689 CN**: 以 `arith::ExtUIOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1690 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCastIdx:`.
  **L1690 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCastIdx:`。
- **L1691 EN**: Returns from the current function with `arith::IndexCastOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1691 CN**: 以 `arith::IndexCastOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1692 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kTruncI:`.
  **L1692 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kTruncI:`。
- **L1693 EN**: Returns from the current function with `arith::TruncIOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1693 CN**: 以 `arith::TruncIOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1694 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCIm: {`.
  **L1694 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCIm: {`。
- **L1695 EN**: Initializes variable `type` from the right-hand expression.
  **L1695 CN**: 使用右侧表达式初始化变量 `type`。
- **L1696 EN**: Initializes variable `eltType` from the right-hand expression.
  **L1696 CN**: 使用右侧表达式初始化变量 `eltType`。
- **L1697 EN**: Returns from the current function with `complex::ImOp::create(rewriter, loc, eltType, v0)`.
  **L1697 CN**: 以 `complex::ImOp::create(rewriter, loc, eltType, v0)` 从当前函数返回。
- **L1698 EN**: Closes the current lexical scope or compound statement.
  **L1698 CN**: 结束当前词法作用域或复合语句块。
- **L1699 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCRe: {`.
  **L1699 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCRe: {`。
- **L1700 EN**: Initializes variable `type` from the right-hand expression.
  **L1700 CN**: 使用右侧表达式初始化变量 `type`。
- **L1701 EN**: Initializes variable `eltType` from the right-hand expression.
  **L1701 CN**: 使用右侧表达式初始化变量 `eltType`。
- **L1702 EN**: Returns from the current function with `complex::ReOp::create(rewriter, loc, eltType, v0)`.
  **L1702 CN**: 以 `complex::ReOp::create(rewriter, loc, eltType, v0)` 从当前函数返回。
- **L1703 EN**: Closes the current lexical scope or compound statement.
  **L1703 CN**: 结束当前词法作用域或复合语句块。
- **L1704 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBitCast:`.
  **L1704 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBitCast:`。

### Lines 1705-1728

````cpp
    return arith::BitcastOp::create(rewriter, loc, inferType(e, v0), v0);
  // Binary operations.
  case TensorExp::Kind::kMulF:
    return arith::MulFOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kMulC:
    return complex::MulOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kMulI:
    return arith::MulIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kDivF:
    return arith::DivFOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kDivC:
    return complex::DivOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kDivS:
    return arith::DivSIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kDivU:
    return arith::DivUIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kAddF:
    return arith::AddFOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kAddC:
    return complex::AddOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kAddI:
    return arith::AddIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kSubF:
    return arith::SubFOp::create(rewriter, loc, v0, v1);
````
- **L1705 EN**: Returns from the current function with `arith::BitcastOp::create(rewriter, loc, inferType(e, v0), v0)`.
  **L1705 CN**: 以 `arith::BitcastOp::create(rewriter, loc, inferType(e, v0), v0)` 从当前函数返回。
- **L1706 EN**: Comment explains nearby logic, invariants, or intent: `Binary operations.`.
  **L1706 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Binary operations.`。
- **L1707 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulF:`.
  **L1707 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulF:`。
- **L1708 EN**: Returns from the current function with `arith::MulFOp::create(rewriter, loc, v0, v1)`.
  **L1708 CN**: 以 `arith::MulFOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1709 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulC:`.
  **L1709 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulC:`。
- **L1710 EN**: Returns from the current function with `complex::MulOp::create(rewriter, loc, v0, v1)`.
  **L1710 CN**: 以 `complex::MulOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1711 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kMulI:`.
  **L1711 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kMulI:`。
- **L1712 EN**: Returns from the current function with `arith::MulIOp::create(rewriter, loc, v0, v1)`.
  **L1712 CN**: 以 `arith::MulIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1713 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivF:`.
  **L1713 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivF:`。
- **L1714 EN**: Returns from the current function with `arith::DivFOp::create(rewriter, loc, v0, v1)`.
  **L1714 CN**: 以 `arith::DivFOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1715 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivC:`.
  **L1715 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivC:`。
- **L1716 EN**: Returns from the current function with `complex::DivOp::create(rewriter, loc, v0, v1)`.
  **L1716 CN**: 以 `complex::DivOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1717 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivS:`.
  **L1717 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivS:`。
- **L1718 EN**: Returns from the current function with `arith::DivSIOp::create(rewriter, loc, v0, v1)`.
  **L1718 CN**: 以 `arith::DivSIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1719 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDivU:`.
  **L1719 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDivU:`。
- **L1720 EN**: Returns from the current function with `arith::DivUIOp::create(rewriter, loc, v0, v1)`.
  **L1720 CN**: 以 `arith::DivUIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1721 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddF:`.
  **L1721 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddF:`。
- **L1722 EN**: Returns from the current function with `arith::AddFOp::create(rewriter, loc, v0, v1)`.
  **L1722 CN**: 以 `arith::AddFOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1723 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddC:`.
  **L1723 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddC:`。
- **L1724 EN**: Returns from the current function with `complex::AddOp::create(rewriter, loc, v0, v1)`.
  **L1724 CN**: 以 `complex::AddOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1725 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAddI:`.
  **L1725 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAddI:`。
- **L1726 EN**: Returns from the current function with `arith::AddIOp::create(rewriter, loc, v0, v1)`.
  **L1726 CN**: 以 `arith::AddIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1727 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubF:`.
  **L1727 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubF:`。
- **L1728 EN**: Returns from the current function with `arith::SubFOp::create(rewriter, loc, v0, v1)`.
  **L1728 CN**: 以 `arith::SubFOp::create(rewriter, loc, v0, v1)` 从当前函数返回。

### Lines 1729-1752

````cpp
  case TensorExp::Kind::kSubC:
    return complex::SubOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kSubI:
    return arith::SubIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kAndI:
    return arith::AndIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kOrI:
    return arith::OrIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kXorI:
    return arith::XOrIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kShrS:
    return arith::ShRSIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kShrU:
    return arith::ShRUIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kShlI:
    return arith::ShLIOp::create(rewriter, loc, v0, v1);
  case TensorExp::Kind::kCmpI: {
    auto predicate = llvm::cast<arith::CmpIPredicateAttr>(expr.attr);
    return arith::CmpIOp::create(rewriter, loc, predicate, v0, v1);
  }
  case TensorExp::Kind::kCmpF: {
    auto predicate = llvm::cast<arith::CmpFPredicateAttr>(expr.attr);
    return arith::CmpFOp::create(rewriter, loc, predicate, v0, v1);
  }
````
- **L1729 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubC:`.
  **L1729 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubC:`。
- **L1730 EN**: Returns from the current function with `complex::SubOp::create(rewriter, loc, v0, v1)`.
  **L1730 CN**: 以 `complex::SubOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1731 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSubI:`.
  **L1731 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSubI:`。
- **L1732 EN**: Returns from the current function with `arith::SubIOp::create(rewriter, loc, v0, v1)`.
  **L1732 CN**: 以 `arith::SubIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1733 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kAndI:`.
  **L1733 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kAndI:`。
- **L1734 EN**: Returns from the current function with `arith::AndIOp::create(rewriter, loc, v0, v1)`.
  **L1734 CN**: 以 `arith::AndIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1735 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kOrI:`.
  **L1735 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kOrI:`。
- **L1736 EN**: Returns from the current function with `arith::OrIOp::create(rewriter, loc, v0, v1)`.
  **L1736 CN**: 以 `arith::OrIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1737 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kXorI:`.
  **L1737 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kXorI:`。
- **L1738 EN**: Returns from the current function with `arith::XOrIOp::create(rewriter, loc, v0, v1)`.
  **L1738 CN**: 以 `arith::XOrIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1739 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrS:`.
  **L1739 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrS:`。
- **L1740 EN**: Returns from the current function with `arith::ShRSIOp::create(rewriter, loc, v0, v1)`.
  **L1740 CN**: 以 `arith::ShRSIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1741 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShrU:`.
  **L1741 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShrU:`。
- **L1742 EN**: Returns from the current function with `arith::ShRUIOp::create(rewriter, loc, v0, v1)`.
  **L1742 CN**: 以 `arith::ShRUIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1743 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kShlI:`.
  **L1743 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kShlI:`。
- **L1744 EN**: Returns from the current function with `arith::ShLIOp::create(rewriter, loc, v0, v1)`.
  **L1744 CN**: 以 `arith::ShLIOp::create(rewriter, loc, v0, v1)` 从当前函数返回。
- **L1745 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpI: {`.
  **L1745 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpI: {`。
- **L1746 EN**: Initializes variable `predicate` from the right-hand expression.
  **L1746 CN**: 使用右侧表达式初始化变量 `predicate`。
- **L1747 EN**: Returns from the current function with `arith::CmpIOp::create(rewriter, loc, predicate, v0, v1)`.
  **L1747 CN**: 以 `arith::CmpIOp::create(rewriter, loc, predicate, v0, v1)` 从当前函数返回。
- **L1748 EN**: Closes the current lexical scope or compound statement.
  **L1748 CN**: 结束当前词法作用域或复合语句块。
- **L1749 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kCmpF: {`.
  **L1749 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kCmpF: {`。
- **L1750 EN**: Initializes variable `predicate` from the right-hand expression.
  **L1750 CN**: 使用右侧表达式初始化变量 `predicate`。
- **L1751 EN**: Returns from the current function with `arith::CmpFOp::create(rewriter, loc, predicate, v0, v1)`.
  **L1751 CN**: 以 `arith::CmpFOp::create(rewriter, loc, predicate, v0, v1)` 从当前函数返回。
- **L1752 EN**: Closes the current lexical scope or compound statement.
  **L1752 CN**: 结束当前词法作用域或复合语句块。

### Lines 1753-1776

````cpp
  case TensorExp::Kind::kBinaryBranch: // semi-ring ops with custom logic.
    return insertYieldOp(rewriter, loc, *expr.op->getBlock()->getParent(),
                         {v0});
  case TensorExp::Kind::kUnary:
    return buildUnaryPresent(rewriter, loc, expr.op, v0);
  case TensorExp::Kind::kSelect:
    return insertYieldOp(rewriter, loc,
                         cast<sparse_tensor::SelectOp>(expr.op).getRegion(),
                         {v0});
  case TensorExp::Kind::kBinary:
    return buildBinaryOverlap(rewriter, loc, expr.op, v0, v1);
  case TensorExp::Kind::kReduce: {
    ReduceOp redOp = cast<ReduceOp>(expr.op);
    return insertYieldOp(rewriter, loc, redOp.getRegion(), {v0, v1});
  }
  case TensorExp::Kind::kDenseOp: {
    Operation *actualOp = expr.op;
    IRMapping mapping;
    mapping.map(actualOp->getOperand(0), v0);
    if (actualOp->getNumOperands() == 2)
      mapping.map(actualOp->getOperand(1), v1);
    return rewriter.clone(*actualOp, mapping)->getResult(0);
  }
  }
````
- **L1753 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinaryBranch: // semi-ring ops with custom logic.`.
  **L1753 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinaryBranch: // semi-ring ops with custom logic.`。
- **L1754 EN**: Returns from the current function with `insertYieldOp(rewriter, loc, *expr.op->getBlock()->getParent(),`.
  **L1754 CN**: 以 `insertYieldOp(rewriter, loc, *expr.op->getBlock()->getParent(),` 从当前函数返回。
- **L1755 EN**: Executes a standalone statement or declaration: `{v0});`.
  **L1755 CN**: 执行一条独立语句或声明：`{v0});`。
- **L1756 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kUnary:`.
  **L1756 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kUnary:`。
- **L1757 EN**: Returns from the current function with `buildUnaryPresent(rewriter, loc, expr.op, v0)`.
  **L1757 CN**: 以 `buildUnaryPresent(rewriter, loc, expr.op, v0)` 从当前函数返回。
- **L1758 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kSelect:`.
  **L1758 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kSelect:`。
- **L1759 EN**: Returns from the current function with `insertYieldOp(rewriter, loc,`.
  **L1759 CN**: 以 `insertYieldOp(rewriter, loc,` 从当前函数返回。
- **L1760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `cast<sparse_tensor::SelectOp>(expr.op).getRegion(),`.
  **L1760 CN**: 继续一个多行参数列表、初始化器或聚合项：`cast<sparse_tensor::SelectOp>(expr.op).getRegion(),`。
- **L1761 EN**: Executes a standalone statement or declaration: `{v0});`.
  **L1761 CN**: 执行一条独立语句或声明：`{v0});`。
- **L1762 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kBinary:`.
  **L1762 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kBinary:`。
- **L1763 EN**: Returns from the current function with `buildBinaryOverlap(rewriter, loc, expr.op, v0, v1)`.
  **L1763 CN**: 以 `buildBinaryOverlap(rewriter, loc, expr.op, v0, v1)` 从当前函数返回。
- **L1764 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kReduce: {`.
  **L1764 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kReduce: {`。
- **L1765 EN**: Initializes variable `redOp` from the right-hand expression.
  **L1765 CN**: 使用右侧表达式初始化变量 `redOp`。
- **L1766 EN**: Returns from the current function with `insertYieldOp(rewriter, loc, redOp.getRegion(), {v0, v1})`.
  **L1766 CN**: 以 `insertYieldOp(rewriter, loc, redOp.getRegion(), {v0, v1})` 从当前函数返回。
- **L1767 EN**: Closes the current lexical scope or compound statement.
  **L1767 CN**: 结束当前词法作用域或复合语句块。
- **L1768 EN**: Introduces a switch dispatch label: `case TensorExp::Kind::kDenseOp: {`.
  **L1768 CN**: 引入一个 switch 分发标签：`case TensorExp::Kind::kDenseOp: {`。
- **L1769 EN**: Executes a standalone statement or declaration: `Operation *actualOp = expr.op;`.
  **L1769 CN**: 执行一条独立语句或声明：`Operation *actualOp = expr.op;`。
- **L1770 EN**: Executes a standalone statement or declaration: `IRMapping mapping;`.
  **L1770 CN**: 执行一条独立语句或声明：`IRMapping mapping;`。
- **L1771 EN**: Executes a call or declaration centered on `mapping.map`.
  **L1771 CN**: 执行以 `mapping.map` 为核心的调用或声明。
- **L1772 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1772 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1773 EN**: Executes a call or declaration centered on `mapping.map`.
  **L1773 CN**: 执行以 `mapping.map` 为核心的调用或声明。
- **L1774 EN**: Returns from the current function with `rewriter.clone(*actualOp, mapping)->getResult(0)`.
  **L1774 CN**: 以 `rewriter.clone(*actualOp, mapping)->getResult(0)` 从当前函数返回。
- **L1775 EN**: Closes the current lexical scope or compound statement.
  **L1775 CN**: 结束当前词法作用域或复合语句块。
- **L1776 EN**: Closes the current lexical scope or compound statement.
  **L1776 CN**: 结束当前词法作用域或复合语句块。

### Lines 1777-1781

````cpp
  llvm_unreachable("unexpected expression kind in build");
}

} // namespace sparse_tensor
} // namespace mlir
````
- **L1777 EN**: Marks this control path as unreachable.
  **L1777 CN**: 将该控制路径标记为不可达。
- **L1778 EN**: Closes the current lexical scope or compound statement.
  **L1778 CN**: 结束当前词法作用域或复合语句块。
- **L1779 EN**: Blank line separating nearby declarations or logic blocks.
  **L1779 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1780 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace sparse_tensor`.
  **L1780 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace sparse_tensor`。
- **L1781 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace mlir`.
  **L1781 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace mlir`。

## Key Concepts / 关键概念

- **Sparse tensor compilation / 稀疏张量编译**
- **Shared helper utilities / 共享辅助工具**
- **Dialect definition and registration / 方言定义与注册**
- **Operation semantics and verification / 操作语义与验证**
- **Attribute representation / 属性表示**
- **Type-system modeling / 类型系统建模**
- **Region-based control structure / 基于 Region 的控制结构**
- **Basic block ownership and traversal / 基本块拥有关系与遍历**
- **SSA value representation / SSA 值表示**
- **Vector type semantics / 向量类型语义**

## Dependencies / 依赖关系

- `mlir/Dialect/SparseTensor/Utils/Merger.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Arith/IR/Arith.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Complex/IR/Complex.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Math/IR/Math.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SparseTensor/IR/SparseTensor.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/Operation.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `llvm/Support/Debug.h`: Provides LLVM support-library facilities such as diagnostics and casting. / 提供LLVM 支持库设施，如诊断与类型转换。
- `optional`: Provides supporting declarations used by the current translation unit. / 提供当前编译单元使用的辅助声明。
