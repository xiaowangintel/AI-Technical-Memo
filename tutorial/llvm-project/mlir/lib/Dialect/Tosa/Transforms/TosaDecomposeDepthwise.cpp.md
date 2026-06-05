# TosaDecomposeDepthwise.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Tosa/Transforms/TosaDecomposeDepthwise.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Decompose TOSA Depthwise operation to a series of TOSA Ops specifically (1) Convert a 1x1 Depthwise to Reshape -> Mul -> Reshape -> Add.
- **Purpose (CN)**: 实现 TOSA 专用的优化与 lowering pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===- TosaDecomposeDepthwise.cpp -----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Decompose TOSA Depthwise operation to a series of TOSA Ops specifically
// (1) Convert a 1x1 Depthwise to Reshape -> Mul -> Reshape -> Add
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Tosa/IR/TosaOps.h"
#include "mlir/Dialect/Tosa/Transforms/Passes.h"
#include "mlir/Dialect/Tosa/Utils/ConversionUtils.h"
#include "mlir/IR/BuiltinTypes.h"

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
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `Decompose TOSA Depthwise operation to a series of TOSA Ops specifically`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Decompose TOSA Depthwise operation to a series of TOSA Ops specifically`。
- **L10 EN**: Comment explains nearby logic, invariants, or intent: `(1) Convert a 1x1 Depthwise to Reshape -> Mul -> Reshape -> Add`.
  **L10 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(1) Convert a 1x1 Depthwise to Reshape -> Mul -> Reshape -> Add`。
- **L11 EN**: Separator comment used for visual grouping.
  **L11 CN**: 用于视觉分组的分隔注释。
- **L12 EN**: Banner comment marking a file or section boundary.
  **L12 CN**: 横幅注释，用于标记文件或章节边界。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L14 EN**: Includes "mlir/Dialect/Tosa/IR/TosaOps.h" to access dialect-specific IR, transforms, or shared utilities.
  **L14 CN**: 引入 "mlir/Dialect/Tosa/IR/TosaOps.h" 以使用方言专用 IR、变换或共享工具。
- **L15 EN**: Includes "mlir/Dialect/Tosa/Transforms/Passes.h" to access dialect-specific IR, transforms, or shared utilities.
  **L15 CN**: 引入 "mlir/Dialect/Tosa/Transforms/Passes.h" 以使用方言专用 IR、变换或共享工具。
- **L16 EN**: Includes "mlir/Dialect/Tosa/Utils/ConversionUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L16 CN**: 引入 "mlir/Dialect/Tosa/Utils/ConversionUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L17 EN**: Includes "mlir/IR/BuiltinTypes.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L17 CN**: 引入 "mlir/IR/BuiltinTypes.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 19-36

````cpp
using namespace mlir;
using namespace mlir::tosa;

namespace {

struct DepthwiseConv2DIsMul : public OpRewritePattern<tosa::DepthwiseConv2DOp> {
  explicit DepthwiseConv2DIsMul(MLIRContext *context)
      : OpRewritePattern(context) {}

  LogicalResult matchAndRewrite(tosa::DepthwiseConv2DOp op,
                                PatternRewriter &rewriter) const override {
    Value input = op.getInput();
    Value weight = op.getWeight();
    ShapedType inputType = cast<ShapedType>(input.getType());
    ShapedType weightType = cast<ShapedType>(weight.getType());
    ShapedType resultType = cast<ShapedType>(op.getOutput().getType());

    // Any dimensions other than batchSize cannot be dynamic for input/output
````
- **L19 EN**: Brings namespace `mlir` into local scope.
  **L19 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L20 EN**: Brings namespace `mlir::tosa` into local scope.
  **L20 CN**: 将命名空间 `mlir::tosa` 引入当前作用域。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Opens namespace scope ``.
  **L22 CN**: 打开命名空间作用域 ``。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Declares struct `DepthwiseConv2DIsMul`.
  **L24 CN**: 声明 struct `DepthwiseConv2DIsMul`。
- **L25 EN**: Continues logic associated with callable symbol `DepthwiseConv2DIsMul`.
  **L25 CN**: 继续与可调用符号 `DepthwiseConv2DIsMul` 相关的逻辑。
- **L26 EN**: Continues logic associated with callable symbol `OpRewritePattern`.
  **L26 CN**: 继续与可调用符号 `OpRewritePattern` 相关的逻辑。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(tosa::DepthwiseConv2DOp op,`.
  **L28 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(tosa::DepthwiseConv2DOp op,`。
- **L29 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L29 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L30 EN**: Initializes variable `input` from the right-hand expression.
  **L30 CN**: 使用右侧表达式初始化变量 `input`。
- **L31 EN**: Initializes variable `weight` from the right-hand expression.
  **L31 CN**: 使用右侧表达式初始化变量 `weight`。
- **L32 EN**: Initializes variable `inputType` from the right-hand expression.
  **L32 CN**: 使用右侧表达式初始化变量 `inputType`。
- **L33 EN**: Initializes variable `weightType` from the right-hand expression.
  **L33 CN**: 使用右侧表达式初始化变量 `weightType`。
- **L34 EN**: Initializes variable `resultType` from the right-hand expression.
  **L34 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Comment explains nearby logic, invariants, or intent: `Any dimensions other than batchSize cannot be dynamic for input/output`.
  **L36 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Any dimensions other than batchSize cannot be dynamic for input/output`。

### Lines 37-54

````cpp
    for (unsigned int i = 1; i < 4; ++i) {
      if (inputType.isDynamicDim(i) || resultType.isDynamicDim(i))
        return failure();
    }

    if (!weightType.hasStaticShape()) {
      return failure();
    }

    if (!llvm::all_of(op.getStride(), [](int64_t v) { return v == 1; }))
      return failure();

    // Only works for a 1x1 kernel.
    ArrayRef<int64_t> weightShape = weightType.getShape();
    if (weightShape[0] != 1 || weightShape[1] != 1) {
      return failure();
    }

````
- **L37 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L37 CN**: 开始 `for` 控制流语句并计算其条件。
- **L38 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L38 CN**: 开始 `if` 控制流语句并计算其条件。
- **L39 EN**: Returns from the current function with `failure()`.
  **L39 CN**: 以 `failure()` 从当前函数返回。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L42 CN**: 开始 `if` 控制流语句并计算其条件。
- **L43 EN**: Returns from the current function with `failure()`.
  **L43 CN**: 以 `failure()` 从当前函数返回。
- **L44 EN**: Closes the current lexical scope or compound statement.
  **L44 CN**: 结束当前词法作用域或复合语句块。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L46 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L46 CN**: 开始 `if` 控制流语句并计算其条件。
- **L47 EN**: Returns from the current function with `failure()`.
  **L47 CN**: 以 `failure()` 从当前函数返回。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L49 EN**: Comment explains nearby logic, invariants, or intent: `Only works for a 1x1 kernel.`.
  **L49 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only works for a 1x1 kernel.`。
- **L50 EN**: Initializes variable `weightShape` from the right-hand expression.
  **L50 CN**: 使用右侧表达式初始化变量 `weightShape`。
- **L51 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L51 CN**: 开始 `if` 控制流语句并计算其条件。
- **L52 EN**: Returns from the current function with `failure()`.
  **L52 CN**: 以 `failure()` 从当前函数返回。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 55-72

````cpp
    Type inputETy = inputType.getElementType();
    Type weightETy = weightType.getElementType();
    if (!inputETy.isIntOrFloat() || !weightETy.isIntOrFloat())
      return rewriter.notifyMatchFailure(op, "unsupported type");

    // Get and verify zero points.
    FailureOr<int64_t> maybeIZp = op.getInputZeroPoint();
    if (failed(maybeIZp))
      return rewriter.notifyMatchFailure(
          op, "input zero point cannot be statically determined");

    FailureOr<int64_t> maybeWZp = op.getWeightZeroPoint();
    if (failed(maybeWZp))
      return rewriter.notifyMatchFailure(
          op, "weight zero point cannot be statically determined");

    int64_t iZp = *maybeIZp;
    int64_t wZp = *maybeWZp;
````
- **L55 EN**: Initializes variable `inputETy` from the right-hand expression.
  **L55 CN**: 使用右侧表达式初始化变量 `inputETy`。
- **L56 EN**: Initializes variable `weightETy` from the right-hand expression.
  **L56 CN**: 使用右侧表达式初始化变量 `weightETy`。
- **L57 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L57 CN**: 开始 `if` 控制流语句并计算其条件。
- **L58 EN**: Returns from the current function with `rewriter.notifyMatchFailure(op, "unsupported type")`.
  **L58 CN**: 以 `rewriter.notifyMatchFailure(op, "unsupported type")` 从当前函数返回。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L60 EN**: Comment explains nearby logic, invariants, or intent: `Get and verify zero points.`.
  **L60 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get and verify zero points.`。
- **L61 EN**: Initializes variable `maybeIZp` from the right-hand expression.
  **L61 CN**: 使用右侧表达式初始化变量 `maybeIZp`。
- **L62 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L62 CN**: 开始 `if` 控制流语句并计算其条件。
- **L63 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L63 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L64 EN**: Executes a standalone statement or declaration: `op, "input zero point cannot be statically determined");`.
  **L64 CN**: 执行一条独立语句或声明：`op, "input zero point cannot be statically determined");`。
- **L65 EN**: Blank line separating nearby declarations or logic blocks.
  **L65 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L66 EN**: Initializes variable `maybeWZp` from the right-hand expression.
  **L66 CN**: 使用右侧表达式初始化变量 `maybeWZp`。
- **L67 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L67 CN**: 开始 `if` 控制流语句并计算其条件。
- **L68 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L68 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L69 EN**: Executes a standalone statement or declaration: `op, "weight zero point cannot be statically determined");`.
  **L69 CN**: 执行一条独立语句或声明：`op, "weight zero point cannot be statically determined");`。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L71 EN**: Initializes variable `iZp` from the right-hand expression.
  **L71 CN**: 使用右侧表达式初始化变量 `iZp`。
- **L72 EN**: Initializes variable `wZp` from the right-hand expression.
  **L72 CN**: 使用右侧表达式初始化变量 `wZp`。

### Lines 73-90

````cpp
    if (op.verifyInputZeroPoint(iZp).failed())
      return rewriter.notifyMatchFailure(
          op, "input zero point must be zero for non-int8 integer types");
    if (op.verifyWeightZeroPoint(wZp).failed())
      return rewriter.notifyMatchFailure(
          op, "weight zero point must be zero for non-int8 integer types");

    // Reshape input to [N, H, W, C] -> [N, H, W, C, 1].
    ArrayRef<int64_t> inputShape = inputType.getShape();
    llvm::SmallVector<int64_t, 2> revisedInputShape{
        inputShape[0], inputShape[1], inputShape[2], inputShape[3], 1};
    inputType = RankedTensorType::get(
        revisedInputShape,
        dyn_cast<RankedTensorType>(input.getType()).getElementType());
    auto revisedInputShapeValue =
        getTosaConstShape(rewriter, op.getLoc(), revisedInputShape);
    input = tosa::ReshapeOp::create(rewriter, op.getLoc(), inputType, input,
                                    revisedInputShapeValue)
````
- **L73 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L73 CN**: 开始 `if` 控制流语句并计算其条件。
- **L74 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L74 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L75 EN**: Executes a standalone statement or declaration: `op, "input zero point must be zero for non-int8 integer types");`.
  **L75 CN**: 执行一条独立语句或声明：`op, "input zero point must be zero for non-int8 integer types");`。
- **L76 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L76 CN**: 开始 `if` 控制流语句并计算其条件。
- **L77 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L77 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L78 EN**: Executes a standalone statement or declaration: `op, "weight zero point must be zero for non-int8 integer types");`.
  **L78 CN**: 执行一条独立语句或声明：`op, "weight zero point must be zero for non-int8 integer types");`。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L80 EN**: Comment explains nearby logic, invariants, or intent: `Reshape input to [N, H, W, C] -> [N, H, W, C, 1].`.
  **L80 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reshape input to [N, H, W, C] -> [N, H, W, C, 1].`。
- **L81 EN**: Initializes variable `inputShape` from the right-hand expression.
  **L81 CN**: 使用右侧表达式初始化变量 `inputShape`。
- **L82 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<int64_t, 2> revisedInputShape{`.
  **L82 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<int64_t, 2> revisedInputShape{`。
- **L83 EN**: Executes a standalone statement or declaration: `inputShape[0], inputShape[1], inputShape[2], inputShape[3], 1};`.
  **L83 CN**: 执行一条独立语句或声明：`inputShape[0], inputShape[1], inputShape[2], inputShape[3], 1};`。
- **L84 EN**: Continues logic associated with callable symbol `get`.
  **L84 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `revisedInputShape,`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`revisedInputShape,`。
- **L86 EN**: Executes a call or declaration centered on `dyn_cast<RankedTensorType>`.
  **L86 CN**: 执行以 `dyn_cast<RankedTensorType>` 为核心的调用或声明。
- **L87 EN**: Continues the surrounding expression or declaration: `auto revisedInputShapeValue =`.
  **L87 CN**: 继续构造周围的表达式或声明：`auto revisedInputShapeValue =`。
- **L88 EN**: Executes a call or declaration centered on `getTosaConstShape`.
  **L88 CN**: 执行以 `getTosaConstShape` 为核心的调用或声明。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `input = tosa::ReshapeOp::create(rewriter, op.getLoc(), inputType, input,`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`input = tosa::ReshapeOp::create(rewriter, op.getLoc(), inputType, input,`。
- **L90 EN**: Continues the surrounding expression or declaration: `revisedInputShapeValue)`.
  **L90 CN**: 继续构造周围的表达式或声明：`revisedInputShapeValue)`。

### Lines 91-108

````cpp
                .getResult();

    Type resultETy = resultType.getElementType();

    if (inputETy != resultETy) {
      inputType = inputType.clone(resultETy);
      input = tosa::CastOp::create(rewriter, op.getLoc(), inputType, input);
    }

    if (weightETy != resultETy) {
      weightType = weightType.clone(resultETy);
      weight = tosa::CastOp::create(rewriter, op.getLoc(), weightType, weight);
    }

    if (iZp != 0 || wZp != 0) {

      auto applyZp = [&](Value val, int64_t zp) -> Value {
        if (zp == 0)
````
- **L91 EN**: Executes a call or declaration centered on `.getResult`.
  **L91 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Initializes variable `resultETy` from the right-hand expression.
  **L93 CN**: 使用右侧表达式初始化变量 `resultETy`。
- **L94 EN**: Blank line separating nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L95 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L95 CN**: 开始 `if` 控制流语句并计算其条件。
- **L96 EN**: Executes a call or declaration centered on `inputType.clone`.
  **L96 CN**: 执行以 `inputType.clone` 为核心的调用或声明。
- **L97 EN**: Executes a call or declaration centered on `tosa::CastOp::create`.
  **L97 CN**: 执行以 `tosa::CastOp::create` 为核心的调用或声明。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L100 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L100 CN**: 开始 `if` 控制流语句并计算其条件。
- **L101 EN**: Executes a call or declaration centered on `weightType.clone`.
  **L101 CN**: 执行以 `weightType.clone` 为核心的调用或声明。
- **L102 EN**: Executes a call or declaration centered on `tosa::CastOp::create`.
  **L102 CN**: 执行以 `tosa::CastOp::create` 为核心的调用或声明。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L105 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L105 CN**: 开始 `if` 控制流语句并计算其条件。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L107 EN**: Starts a function, method, lambda, or structured scope: `auto applyZp = [&](Value val, int64_t zp) -> Value {`.
  **L107 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto applyZp = [&](Value val, int64_t zp) -> Value {`。
- **L108 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L108 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 109-126

````cpp
          return val;
        auto ety = cast<ShapedType>(val.getType()).getElementType();
        std::vector<int64_t> shape(cast<ShapedType>(val.getType()).getRank(),
                                   1);
        auto zpTy = RankedTensorType::get(shape, ety);
        auto zpAttr =
            DenseElementsAttr::get(zpTy, rewriter.getIntegerAttr(ety, zp));
        auto zpVal = tosa::ConstOp::create(rewriter, op.getLoc(), zpTy, zpAttr);
        return tosa::SubOp::create(rewriter, op.getLoc(), val.getType(), val,
                                   zpVal);
      };

      input = applyZp(input, iZp);
      weight = applyZp(weight, wZp);
    }

    ArrayRef<int64_t> padAttr = op.getPad();
    llvm::SmallVector<int64_t> pad(10, 0);
````
- **L109 EN**: Returns from the current function with `val`.
  **L109 CN**: 以 `val` 从当前函数返回。
- **L110 EN**: Initializes variable `ety` from the right-hand expression.
  **L110 CN**: 使用右侧表达式初始化变量 `ety`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::vector<int64_t> shape(cast<ShapedType>(val.getType()).getRank(),`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::vector<int64_t> shape(cast<ShapedType>(val.getType()).getRank(),`。
- **L112 EN**: Executes a standalone statement or declaration: `1);`.
  **L112 CN**: 执行一条独立语句或声明：`1);`。
- **L113 EN**: Initializes variable `zpTy` from the right-hand expression.
  **L113 CN**: 使用右侧表达式初始化变量 `zpTy`。
- **L114 EN**: Continues the surrounding expression or declaration: `auto zpAttr =`.
  **L114 CN**: 继续构造周围的表达式或声明：`auto zpAttr =`。
- **L115 EN**: Executes a call or declaration centered on `DenseElementsAttr::get`.
  **L115 CN**: 执行以 `DenseElementsAttr::get` 为核心的调用或声明。
- **L116 EN**: Initializes variable `zpVal` from the right-hand expression.
  **L116 CN**: 使用右侧表达式初始化变量 `zpVal`。
- **L117 EN**: Returns from the current function with `tosa::SubOp::create(rewriter, op.getLoc(), val.getType(), val,`.
  **L117 CN**: 以 `tosa::SubOp::create(rewriter, op.getLoc(), val.getType(), val,` 从当前函数返回。
- **L118 EN**: Executes a standalone statement or declaration: `zpVal);`.
  **L118 CN**: 执行一条独立语句或声明：`zpVal);`。
- **L119 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L119 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L121 EN**: Executes a call or declaration centered on `applyZp`.
  **L121 CN**: 执行以 `applyZp` 为核心的调用或声明。
- **L122 EN**: Executes a call or declaration centered on `applyZp`.
  **L122 CN**: 执行以 `applyZp` 为核心的调用或声明。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Initializes variable `padAttr` from the right-hand expression.
  **L125 CN**: 使用右侧表达式初始化变量 `padAttr`。
- **L126 EN**: Executes a call or declaration centered on `pad`.
  **L126 CN**: 执行以 `pad` 为核心的调用或声明。

### Lines 127-144

````cpp
    for (const auto &it : llvm::enumerate(padAttr))
      pad[it.index() + 2] = it.value();

    if (llvm::any_of(pad, [](int64_t p) { return p != 0; })) {
      Attribute zeroAttr = rewriter.getZeroAttr(inputETy);

      llvm::SmallVector<int64_t> newShape(inputType.getShape());
      for (int i = 0, s = pad.size(); i < s; ++i) {
        if (newShape[i / 2] != ShapedType::kDynamic) {
          newShape[i / 2] += pad[i];
        }
      }

      Value padSizeVal = getTosaConstShape(rewriter, op->getLoc(), pad);

      auto padTy = RankedTensorType::get({1}, inputETy);
      auto padAttr = DenseElementsAttr::get(padTy, zeroAttr);
      Value padVal =
````
- **L127 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L127 CN**: 开始 `for` 控制流语句并计算其条件。
- **L128 EN**: Executes a call or declaration centered on `pad[it.index`.
  **L128 CN**: 执行以 `pad[it.index` 为核心的调用或声明。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L130 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L130 CN**: 开始 `if` 控制流语句并计算其条件。
- **L131 EN**: Initializes variable `zeroAttr` from the right-hand expression.
  **L131 CN**: 使用右侧表达式初始化变量 `zeroAttr`。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Executes a call or declaration centered on `newShape`.
  **L133 CN**: 执行以 `newShape` 为核心的调用或声明。
- **L134 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L134 CN**: 开始 `for` 控制流语句并计算其条件。
- **L135 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L135 CN**: 开始 `if` 控制流语句并计算其条件。
- **L136 EN**: Executes a standalone statement or declaration: `newShape[i / 2] += pad[i];`.
  **L136 CN**: 执行一条独立语句或声明：`newShape[i / 2] += pad[i];`。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Closes the current lexical scope or compound statement.
  **L138 CN**: 结束当前词法作用域或复合语句块。
- **L139 EN**: Blank line separating nearby declarations or logic blocks.
  **L139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L140 EN**: Initializes variable `padSizeVal` from the right-hand expression.
  **L140 CN**: 使用右侧表达式初始化变量 `padSizeVal`。
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L142 EN**: Initializes variable `padTy` from the right-hand expression.
  **L142 CN**: 使用右侧表达式初始化变量 `padTy`。
- **L143 EN**: Initializes variable `padAttr` from the right-hand expression.
  **L143 CN**: 使用右侧表达式初始化变量 `padAttr`。
- **L144 EN**: Continues the surrounding expression or declaration: `Value padVal =`.
  **L144 CN**: 继续构造周围的表达式或声明：`Value padVal =`。

### Lines 145-162

````cpp
          tosa::ConstOp::create(rewriter, op->getLoc(), padTy, padAttr);
      inputType = RankedTensorType::get(newShape, inputETy);
      input = tosa::PadOp::create(rewriter, op->getLoc(), inputType, input,
                                  padSizeVal, padVal);
    }

    // Perform an elementwise mul over the reshaped input and weight.
    llvm::SmallVector<int64_t, 2> mulShape{
        inputType.getDimSize(0), inputType.getDimSize(1),
        inputType.getDimSize(2), inputType.getDimSize(3), weightShape[3]};
    auto mulShapeType = RankedTensorType::get(
        mulShape,
        dyn_cast<RankedTensorType>(weight.getType()).getElementType());

    if (EqualizeRanks(rewriter, op.getLoc(), input, weight).failed()) {
      return failure();
    }

````
- **L145 EN**: Executes a call or declaration centered on `tosa::ConstOp::create`.
  **L145 CN**: 执行以 `tosa::ConstOp::create` 为核心的调用或声明。
- **L146 EN**: Executes a call or declaration centered on `RankedTensorType::get`.
  **L146 CN**: 执行以 `RankedTensorType::get` 为核心的调用或声明。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `input = tosa::PadOp::create(rewriter, op->getLoc(), inputType, input,`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`input = tosa::PadOp::create(rewriter, op->getLoc(), inputType, input,`。
- **L148 EN**: Executes a standalone statement or declaration: `padSizeVal, padVal);`.
  **L148 CN**: 执行一条独立语句或声明：`padSizeVal, padVal);`。
- **L149 EN**: Closes the current lexical scope or compound statement.
  **L149 CN**: 结束当前词法作用域或复合语句块。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L151 EN**: Comment explains nearby logic, invariants, or intent: `Perform an elementwise mul over the reshaped input and weight.`.
  **L151 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Perform an elementwise mul over the reshaped input and weight.`。
- **L152 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<int64_t, 2> mulShape{`.
  **L152 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<int64_t, 2> mulShape{`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `inputType.getDimSize(0), inputType.getDimSize(1),`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`inputType.getDimSize(0), inputType.getDimSize(1),`。
- **L154 EN**: Executes a call or declaration centered on `inputType.getDimSize`.
  **L154 CN**: 执行以 `inputType.getDimSize` 为核心的调用或声明。
- **L155 EN**: Continues logic associated with callable symbol `get`.
  **L155 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mulShape,`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`mulShape,`。
- **L157 EN**: Executes a call or declaration centered on `dyn_cast<RankedTensorType>`.
  **L157 CN**: 执行以 `dyn_cast<RankedTensorType>` 为核心的调用或声明。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L159 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L159 CN**: 开始 `if` 控制流语句并计算其条件。
- **L160 EN**: Returns from the current function with `failure()`.
  **L160 CN**: 以 `failure()` 从当前函数返回。
- **L161 EN**: Closes the current lexical scope or compound statement.
  **L161 CN**: 结束当前词法作用域或复合语句块。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 163-180

````cpp
    auto shiftElementType = IntegerType::get(rewriter.getContext(), 8);
    auto shiftType = RankedTensorType::get({1}, shiftElementType);
    auto shiftZeroAttr = DenseElementsAttr::get(
        shiftType, rewriter.getIntegerAttr(shiftElementType, 0));
    Value constZero =
        tosa::ConstOp::create(rewriter, op.getLoc(), shiftType, shiftZeroAttr);
    Value mulValue = tosa::MulOp::create(rewriter, op.getLoc(), mulShapeType,
                                         input, weight, constZero)
                         .getResult();

    // Reshape output to [N, H, W, C * M].
    auto outputShape = cast<ShapedType>(op.getOutput().getType()).getShape();
    auto outputShapeType = RankedTensorType::get(
        outputShape,
        dyn_cast<RankedTensorType>(input.getType()).getElementType());
    auto outputShapeValue =
        getTosaConstShape(rewriter, op->getLoc(), outputShape);
    Value outputValue = tosa::ReshapeOp::create(
````
- **L163 EN**: Initializes variable `shiftElementType` from the right-hand expression.
  **L163 CN**: 使用右侧表达式初始化变量 `shiftElementType`。
- **L164 EN**: Initializes variable `shiftType` from the right-hand expression.
  **L164 CN**: 使用右侧表达式初始化变量 `shiftType`。
- **L165 EN**: Continues logic associated with callable symbol `get`.
  **L165 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L166 EN**: Executes a call or declaration centered on `rewriter.getIntegerAttr`.
  **L166 CN**: 执行以 `rewriter.getIntegerAttr` 为核心的调用或声明。
- **L167 EN**: Continues the surrounding expression or declaration: `Value constZero =`.
  **L167 CN**: 继续构造周围的表达式或声明：`Value constZero =`。
- **L168 EN**: Executes a call or declaration centered on `tosa::ConstOp::create`.
  **L168 CN**: 执行以 `tosa::ConstOp::create` 为核心的调用或声明。
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value mulValue = tosa::MulOp::create(rewriter, op.getLoc(), mulShapeType,`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value mulValue = tosa::MulOp::create(rewriter, op.getLoc(), mulShapeType,`。
- **L170 EN**: Continues the surrounding expression or declaration: `input, weight, constZero)`.
  **L170 CN**: 继续构造周围的表达式或声明：`input, weight, constZero)`。
- **L171 EN**: Executes a call or declaration centered on `.getResult`.
  **L171 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L173 EN**: Comment explains nearby logic, invariants, or intent: `Reshape output to [N, H, W, C * M].`.
  **L173 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reshape output to [N, H, W, C * M].`。
- **L174 EN**: Initializes variable `outputShape` from the right-hand expression.
  **L174 CN**: 使用右侧表达式初始化变量 `outputShape`。
- **L175 EN**: Continues logic associated with callable symbol `get`.
  **L175 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `outputShape,`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`outputShape,`。
- **L177 EN**: Executes a call or declaration centered on `dyn_cast<RankedTensorType>`.
  **L177 CN**: 执行以 `dyn_cast<RankedTensorType>` 为核心的调用或声明。
- **L178 EN**: Continues the surrounding expression or declaration: `auto outputShapeValue =`.
  **L178 CN**: 继续构造周围的表达式或声明：`auto outputShapeValue =`。
- **L179 EN**: Executes a call or declaration centered on `getTosaConstShape`.
  **L179 CN**: 执行以 `getTosaConstShape` 为核心的调用或声明。
- **L180 EN**: Continues logic associated with callable symbol `create`.
  **L180 CN**: 继续与可调用符号 `create` 相关的逻辑。

### Lines 181-198

````cpp
        rewriter, op.getLoc(), outputShapeType, mulValue, outputShapeValue);

    Value bias = op.getBias();
    if (EqualizeRanks(rewriter, op.getLoc(), outputValue, bias).failed()) {
      return failure();
    }

    // Add in the bias.
    rewriter
        .replaceOpWithNewOp<tosa::AddOp>(op, outputShapeType, outputValue, bias)
        .getResult();
    return success();
  }
};

} // namespace

void mlir::tosa::populateTosaDecomposeDepthwise(MLIRContext *ctx,
````
- **L181 EN**: Executes a call or declaration centered on `op.getLoc`.
  **L181 CN**: 执行以 `op.getLoc` 为核心的调用或声明。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L183 EN**: Initializes variable `bias` from the right-hand expression.
  **L183 CN**: 使用右侧表达式初始化变量 `bias`。
- **L184 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L184 CN**: 开始 `if` 控制流语句并计算其条件。
- **L185 EN**: Returns from the current function with `failure()`.
  **L185 CN**: 以 `failure()` 从当前函数返回。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L188 EN**: Comment explains nearby logic, invariants, or intent: `Add in the bias.`.
  **L188 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Add in the bias.`。
- **L189 EN**: Continues the surrounding expression or declaration: `rewriter`.
  **L189 CN**: 继续构造周围的表达式或声明：`rewriter`。
- **L190 EN**: Continues logic associated with callable symbol `AddOp>`.
  **L190 CN**: 继续与可调用符号 `AddOp>` 相关的逻辑。
- **L191 EN**: Executes a call or declaration centered on `.getResult`.
  **L191 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L192 EN**: Returns from the current function with `success()`.
  **L192 CN**: 以 `success()` 从当前函数返回。
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L194 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L196 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L196 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void mlir::tosa::populateTosaDecomposeDepthwise(MLIRContext *ctx,`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`void mlir::tosa::populateTosaDecomposeDepthwise(MLIRContext *ctx,`。

### Lines 199-201

````cpp
                                                RewritePatternSet &patterns) {
  patterns.add<DepthwiseConv2DIsMul>(ctx);
}
````
- **L199 EN**: Continues the surrounding expression or declaration: `RewritePatternSet &patterns) {`.
  **L199 CN**: 继续构造周围的表达式或声明：`RewritePatternSet &patterns) {`。
- **L200 EN**: Executes a call or declaration centered on `patterns.add<DepthwiseConv2DIsMul>`.
  **L200 CN**: 执行以 `patterns.add<DepthwiseConv2DIsMul>` 为核心的调用或声明。
- **L201 EN**: Closes the current lexical scope or compound statement.
  **L201 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TOSA dialect semantics / TOSA 方言语义**
- **Rewrite-driven lowering / 基于重写的 lowering**
- **Dialect definition and registration / 方言定义与注册**
- **Pattern-based rewriting / 基于模式的重写**
- **Pattern collection management / 重写模式集合管理**
- **Failure-aware result propagation / 带失败语义的结果传播**
- **Optional success payloads / 带成功载荷的可失败结果**
- **IR mutation through rewriter helpers / 通过重写器辅助工具进行 IR 变换**
- **Attribute representation / 属性表示**
- **Type-system modeling / 类型系统建模**

## Dependencies / 依赖关系

- `mlir/Dialect/Tosa/IR/TosaOps.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Tosa/Transforms/Passes.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Tosa/Utils/ConversionUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/BuiltinTypes.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
