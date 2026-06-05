# AffineOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/IR/AffineOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines MLIR affine operations.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine/IR`，围绕 Affine 方言公开 `AffineOps` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===- AffineOps.td - Affine operation definitions ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Defines MLIR affine operations.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 12-23
```tablegen

#ifndef AFFINE_OPS
#define AFFINE_OPS

include "mlir/Dialect/Arith/IR/ArithBase.td"
include "mlir/Dialect/Affine/IR/AffineMemoryOpInterfaces.td"
include "mlir/Interfaces/ControlFlowInterfaces.td"
include "mlir/Interfaces/InferIntRangeInterface.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/Interfaces/LoopLikeInterface.td"
include "mlir/Interfaces/SideEffectInterfaces.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 24-34
```tablegen
def Affine_Dialect : Dialect {
  let name = "affine";
  let cppNamespace = "::mlir::affine";
  let hasConstantMaterializer = 1;
  let dependentDialects = ["arith::ArithDialect", "ub::UBDialect"];
}

// Base class for Affine dialect ops.
class Affine_Op<string mnemonic, list<Trait> traits = []> :
    Op<Affine_Dialect, mnemonic, traits>;

```
- **EN**: Introduces declarations for `Affine_Dialect`, `for`, `Affine_Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Affine_Dialect`, `for`, `Affine_Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 35-44
```tablegen
// Type constraint for index-like types: index or vector of index.
def Affine_IndexOrVectorOfIndex :
    Type<Or<[Index.predicate,
             VectorOfAnyRankOf<[Index]>.predicate]>,
         "index or vector of index">;

// Require regions to have affine.yield.
def ImplicitAffineTerminator
    : SingleBlockImplicitTerminator<"AffineYieldOp">;

```
- **EN**: Introduces declarations for `Affine_IndexOrVectorOfIndex`, `ImplicitAffineTerminator`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Affine_IndexOrVectorOfIndex`, `ImplicitAffineTerminator` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 45-56
```tablegen
def AffineApplyOp : Affine_Op<"apply",
    [Pure, DeclareOpInterfaceMethods<InferIntRangeInterface, ["inferResultRanges"]>]> {
  let summary = "affine apply operation";
  let description = [{
    The `affine.apply` operation applies an [affine mapping](#affine-maps)
    to a list of SSA values, yielding a single SSA value. The number of
    dimension and symbol operands to `affine.apply` must be equal to the
    respective number of dimensional and symbolic inputs to the affine mapping;
    the affine mapping has to be one-dimensional, and so the `affine.apply`
    operation always returns one value. The input operands and result must all
    have ‘index’ type.

```
- **EN**: Introduces declarations for `AffineApplyOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineApplyOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 57-67
```tablegen
    An operand that is a valid dimension as per the [rules on valid affine
    dimensions and symbols](#restrictions-on-dimensions-and-symbols)
    cannot be used as a symbolic operand.

    Example:

    ```mlir
    #map = affine_map<(d0, d1) -> (d0 floordiv 8 + d1 floordiv 128)>
    ...
    %1 = affine.apply #map (%s, %t)

```
- **EN**: Implements logic around `map`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `map` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 68-86
```tablegen
    // Inline example.
    %2 = affine.apply affine_map<(i)[s0] -> (i + s0)> (%42)[%n]
    ```
  }];
  let arguments = (ins AffineMapAttr:$map, Variadic<Index>:$mapOperands);
  let results = (outs Index);

  // TODO: The auto-generated builders should check to see if the return type
  // has a constant builder. That way we wouldn't need to explicitly specify the
  // result types here.
  let builders = [
    OpBuilder<(ins "ArrayRef<AffineExpr> ":$exprList,"ValueRange":$mapOperands),
    [{
      build($_builder, $_state, $_builder.getIndexType(),
            AffineMap::inferFromExprList(exprList, $_builder.getContext())
                                        .front(), mapOperands);
    }]>
  ];

```
- **EN**: Implements logic around `affine_map`, `OpBuilder`, `build`, `inferFromExprList`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `affine_map`, `OpBuilder`, `build`, `inferFromExprList`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 87-97
```tablegen
  let extraClassDeclaration = [{
    /// Returns the affine map to be applied by this operation.
    AffineMap getAffineMap() { return getMap(); }

    /// Returns the affine value map computed from this operation.
    AffineValueMap getAffineValueMap();

    /// Returns true if the result of this operation can be used as dimension id
    /// in the region of the closest surrounding op with trait AffineScope.
    bool isValidDim();

```
- **EN**: Implements logic around `getAffineMap`, `getAffineValueMap`, `isValidDim`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getAffineMap`, `getAffineValueMap`, `isValidDim` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 98-107
```tablegen
    /// Returns true if the result of this operation can be used as dimension id
    /// within 'region', i.e., for all its uses with `region`.
    bool isValidDim(Region *region);

    /// Returns true if the result of this operation is a symbol in the region
    /// of the closest surrounding op that has the trait AffineScope.
    bool isValidSymbol();

    /// Returns true if the result of this operation is a symbol for all its
    /// uses in `region`.
```
- **EN**: Declares APIs or declarative rules around `isValidDim`, `isValidSymbol`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `isValidDim`, `isValidSymbol` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 108-122
```tablegen
    bool isValidSymbol(Region *region);

    /// Returns all dimension operands.
    ValueRange getDimOperands() {
      return OperandRange{getOperands().begin(),
                          getOperands().begin() + getMap().getNumDims()};
    }

    /// Returns all symbol operands.
    ValueRange getSymbolOperands() {
      return OperandRange{getOperands().begin() + getMap().getNumDims(),
                          getOperands().end()};
    }
  }];

```
- **EN**: Implements logic around `isValidSymbol`, `getDimOperands`, `getOperands`, `getSymbolOperands`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isValidSymbol`, `getDimOperands`, `getOperands`, `getSymbolOperands` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 123-141
```tablegen
  let hasCanonicalizer = 1;
  let hasCustomAssemblyFormat = 1;
  let hasFolder = 1;
  let hasVerifier = 1;
}

def AffineForOp : Affine_Op<"for",
    [AttrSizedOperandSegments, AutomaticAllocationScope,
     ImplicitAffineTerminator, ConditionallySpeculatable,
     RecursiveMemoryEffects, DeclareOpInterfaceMethods<LoopLikeOpInterface,
     ["getLoopInductionVars", "getLoopLowerBounds", "getLoopSteps",
      "getLoopUpperBounds", "getYieldedValuesMutable",
      "replaceWithAdditionalYields"]>,
     DeclareOpInterfaceMethods<RegionBranchOpInterface,
     ["getEntrySuccessorOperands", "getSuccessorInputs"]>]> {
  let summary = "for operation";
  let description = [{
    Syntax:

```
- **EN**: Introduces declarations for `AffineForOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineForOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 142-157
```tablegen
    ```
    operation   ::= `affine.for` ssa-id `=` lower-bound `to` upper-bound
                    (`step` integer-literal)? `{` op* `}`

    lower-bound ::= `max`? affine-map-attribute dim-and-symbol-use-list | shorthand-bound
    upper-bound ::= `min`? affine-map-attribute dim-and-symbol-use-list | shorthand-bound
    shorthand-bound ::= ssa-id | `-`? integer-literal
    ```

    The `affine.for` operation represents an affine loop nest. It has one region
    containing its body. This region must contain one block that terminates with
    [`affine.yield`](#affineyield-mliraffineyieldop). *Note:* when
    `affine.for` is printed in custom format, the terminator is omitted. The
    block has one argument of [`index`](Builtin.md/#indextype) type that
    represents the induction variable of the loop.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 158-169
```tablegen
    The `affine.for` operation executes its body a number of times iterating
    from a lower bound to an upper bound by a stride. The stride, represented by
    `step`, is a positive constant integer which defaults to "1" if not present.
    The lower and upper bounds specify a half-open range: the range includes the
    lower bound but does not include the upper bound.

    The lower and upper bounds of a `affine.for` operation are represented as an
    application of an affine mapping to a list of SSA values passed to the map.
    The [same restrictions](#restrictions-on-dimensions-and-symbols) hold for
    these SSA values as for all bindings of SSA values to dimensions and
    symbols.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 170-182
```tablegen
    The affine mappings for the bounds may return multiple results, in which
    case the `max`/`min` keywords are required (for the lower/upper bound
    respectively), and the bound is the maximum/minimum of the returned values.
    There is no semantic ambiguity, but MLIR syntax requires the use of these
    keywords to make things more obvious to human readers.

    Many upper and lower bounds are simple, so MLIR accepts two custom form
    syntaxes: the form that accepts a single 'ssa-id' (e.g. `%N`) is shorthand
    for applying that SSA value to a function that maps a single symbol to
    itself, e.g., `()[s]->(s)()[%N]`. The integer literal form (e.g. `-42`) is
    shorthand for a nullary mapping function that returns the constant value
    (e.g. `()->(-42)()`).

```
- **EN**: Implements logic around `required`, `form`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `required`, `form` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 183-202
```tablegen
    Example showing reverse iteration of the inner loop:

    ```mlir
    #map57 = affine_map<(d0)[s0] -> (s0 - d0 - 1)>

    func.func @simple_example(%A: memref<?x?xf32>, %B: memref<?x?xf32>) {
      %N = dim %A, 0 : memref<?x?xf32>
      affine.for %i = 0 to %N step 1 {
        affine.for %j = 0 to %N {   // implicitly steps by 1
          %0 = affine.apply #map57(%j)[%N]
          %tmp = call @F1(%A, %i, %0) : (memref<?x?xf32>, index, index)->(f32)
          call @F2(%tmp, %B, %i, %0) : (f32, memref<?x?xf32>, index, index)->()
        }
      }
      return
    }
    ```
    `affine.for` can also operate on loop-carried variables (`iter_args`) and
    return the final values after loop termination. The initial values of the
    variables are passed as additional SSA operands to the `affine.for`
```
- **EN**: Implements logic around `simple_example`, `map57`, `F1`, `F2`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `simple_example`, `map57`, `F1`, `F2`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 203-212
```tablegen
    following the operands for the loop's lower and upper bounds. The
    operation's region has equivalent arguments for each variable representing
    the value of the variable at the current iteration.

    The region must terminate with an `affine.yield` that passes all the current
    iteration variables to the next iteration, or to the `affine.for`'s results
    if at the last iteration. For `affine.for`'s that execute zero iterations, the
    initial values of the loop-carried variables (corresponding to the SSA
    operands) will be the op's results.

```
- **EN**: Implements logic around `variables`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `variables` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 213-231
```tablegen
    For example, to sum-reduce a memref:

     ```mlir
    func.func @reduce(%buffer: memref<1024xf32>) -> (f32) {
      // Initial sum set to 0.
      %sum_0 = arith.constant 0.0 : f32
      // iter_args binds initial values to the loop's region arguments.
      %sum = affine.for %i = 0 to 10 step 2
          iter_args(%sum_iter = %sum_0) -> (f32) {
        %t = affine.load %buffer[%i] : memref<1024xf32>
        %sum_next = arith.addf %sum_iter, %t : f32
        // Yield current iteration sum to next iteration %sum_iter or to %sum
        // if final iteration.
        affine.yield %sum_next : f32
      }
      return %sum : f32
    }
    ```

```
- **EN**: Implements logic around `reduce`, `iter_args`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `reduce`, `iter_args` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 232-251
```tablegen
    ```mlir
    %res:2 = affine.for %i = 0 to 128 iter_args(%arg0 = %init0, %arg1 = %init1)
               -> (index, index) {
      %y0 = arith.addi %arg0, %c1 : index
      %y1 = arith.addi %arg1, %c2 : index
      affine.yield %y0, %y1 : index, index
    }
    ```
    If the `affine.for` defines any values, a yield terminator must be
    explicitly present. The number and types of the "affine.for" results must
    match the initial values in the `iter_args` binding and the yield operands.
  }];
  let arguments = (ins Variadic<Index>:$lowerBoundOperands,
                       Variadic<Index>:$upperBoundOperands,
                       Variadic<AnyType>:$inits,
                       AffineMapAttr:$lowerBoundMap,
                       AffineMapAttr:$upperBoundMap,
                       IndexAttr:$step);
  let results = (outs Variadic<AnyType>:$results);
  let regions = (region SizedRegion<1>:$region);
```
- **EN**: Implements logic around `iter_args`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `iter_args` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 252-264
```tablegen

  let skipDefaultBuilders = 1;
  let builders =
      [OpBuilder<(ins "int64_t":$lowerBound, "int64_t":$upperBound,
           CArg<"int64_t", "1">:$step, CArg<"ValueRange", "{}">:$iterArgs,
           CArg<"function_ref<void(OpBuilder &, Location, Value, ValueRange)>",
                "nullptr">:$bodyBuilder)>,
       OpBuilder<(ins "ValueRange":$lbOperands, "AffineMap":$lbMap,
           "ValueRange":$ubOperands, "AffineMap":$ubMap,
           CArg<"int64_t", "1">:$step, CArg<"ValueRange", "{}">:$iterArgs,
           CArg<"function_ref<void(OpBuilder &, Location, Value, ValueRange)>",
                "nullptr">:$bodyBuilder)>];

```
- **EN**: Implements logic around `OpBuilder`, `function_ref`.
- **CN**: 围绕 `OpBuilder`, `function_ref` 实现具体逻辑。

### Lines 265-274
```tablegen
  let extraClassDeclaration = [{
    /// Defining the function type we use for building the body of affine.for.
    using BodyBuilderFn =
        function_ref<void(OpBuilder &, Location, Value, ValueRange)>;

    BlockArgument getInductionVar() { return getBody()->getArgument(0); }
    Block::BlockArgListType getRegionIterArgs() {
      return getBody()->getArguments().drop_front();
    }

```
- **EN**: Implements logic around `function_ref`, `getInductionVar`, `getRegionIterArgs`, `getBody`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `function_ref`, `getInductionVar`, `getRegionIterArgs`, `getBody` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 275-284
```tablegen
    /// Returns operands for the lower and upper bound maps with the operands
    /// for the lower bound map in front of those for the upper bound map.
    operand_range getControlOperands();

    /// Returns information about the lower bound as a single object.
    AffineBound getLowerBound();

    /// Returns information about the upper bound as a single object.
    AffineBound getUpperBound();

```
- **EN**: Declares APIs or declarative rules around `getControlOperands`, `getLowerBound`, `getUpperBound`.
- **CN**: 声明与 `getControlOperands`, `getLowerBound`, `getUpperBound` 相关的 API 或声明式规则。

### Lines 285-294
```tablegen
    /// Returns loop step.
    int64_t getStepAsInt() { return getStep().getSExtValue(); }

    /// Set lower bound. The new bound must have the same number of operands as
    /// the current bound map. Otherwise, 'replaceForLowerBound' should be used.
    void setLowerBound(ValueRange operands, AffineMap map);
    /// Set upper bound. The new bound must not have more operands than the
    /// current bound map. Otherwise, 'replaceForUpperBound' should be used.
    void setUpperBound(ValueRange operands, AffineMap map);

```
- **EN**: Implements logic around `getStepAsInt`, `setLowerBound`, `setUpperBound`.
- **CN**: 围绕 `getStepAsInt`, `setLowerBound`, `setUpperBound` 实现具体逻辑。

### Lines 295-305
```tablegen
    /// Set loop step.
    void setStep(int64_t step) {
      assert(step > 0 && "step has to be a positive integer constant");
      setStep(APInt(/*numBits=*/64, step, /*isSigned=*/true));
    }

    /// Returns number of region arguments for loop-carried values.
    unsigned getNumRegionIterArgs() {
      return getBody()->getNumArguments() - 1;
    }

```
- **EN**: Implements logic around `setStep`, `assert`, `getNumRegionIterArgs`, `getBody`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setStep`, `assert`, `getNumRegionIterArgs`, `getBody` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 306-316
```tablegen
    /// Number of operands controlling the loop: lb and ub.
    unsigned getNumControlOperands() {
      return getOperation()->getNumOperands() - getNumIterOperands();
    }

    /// Get the number of loop-carried values.
    unsigned getNumIterOperands();

    /// Returns true if the lower bound is constant.
    bool hasConstantLowerBound();
    /// Returns true if the upper bound is constant.
```
- **EN**: Implements logic around `getNumControlOperands`, `getOperation`, `getNumIterOperands`, `hasConstantLowerBound`.
- **CN**: 围绕 `getNumControlOperands`, `getOperation`, `getNumIterOperands`, `hasConstantLowerBound` 实现具体逻辑。

### Lines 317-326
```tablegen
    bool hasConstantUpperBound();
    /// Returns true if both bounds are constant.
    bool hasConstantBounds() {
      return hasConstantLowerBound() && hasConstantUpperBound();
    }
    /// Returns the value of the constant lower bound.
    /// Fails assertion if the bound is non-constant.
    int64_t getConstantLowerBound();
    /// Returns the value of the constant upper bound. The upper bound is
    /// exclusive. Fails assertion if the bound is non-constant.
```
- **EN**: Implements logic around `hasConstantUpperBound`, `hasConstantBounds`, `hasConstantLowerBound`, `getConstantLowerBound`.
- **CN**: 围绕 `hasConstantUpperBound`, `hasConstantBounds`, `hasConstantLowerBound`, `getConstantLowerBound` 实现具体逻辑。

### Lines 327-336
```tablegen
    int64_t getConstantUpperBound();
    /// Sets the lower bound to the given constant value.
    void setConstantLowerBound(int64_t value);
    /// Sets the upper bound to the given constant value.
    void setConstantUpperBound(int64_t value);

    /// Returns true if both the lower and upper bound have the same operand
    /// lists (same operands in the same order).
    bool matchingBoundOperandList();

```
- **EN**: Declares APIs or declarative rules around `getConstantUpperBound`, `setConstantLowerBound`, `setConstantUpperBound`, `matchingBoundOperandList`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `getConstantUpperBound`, `setConstantLowerBound`, `setConstantUpperBound`, `matchingBoundOperandList` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 337-346
```tablegen
    /// Interface method for ConditionallySpeculatable.
    Speculation::Speculatability getSpeculatability();
  }];

  let hasCustomAssemblyFormat = 1;
  let hasFolder = 1;
  let hasRegionVerifier = 1;
}

def AffineIfOp : Affine_Op<"if",
```
- **EN**: Introduces declarations for `AffineIfOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineIfOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 347-360
```tablegen
                           [ImplicitAffineTerminator, RecursivelySpeculatable,
                            RecursiveMemoryEffects, NoRegionArguments,
                            DeclareOpInterfaceMethods<RegionBranchOpInterface,
                                ["getSuccessorInputs"]>
                           ]> {
  let summary = "if-then-else operation";
  let description = [{
    Syntax:

    ```
    operation  ::= `affine.if` if-op-cond `{` op* `}` (`else` `{` op* `}`)?
    if-op-cond ::= integer-set-attr dim-and-symbol-use-list
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 361-371
```tablegen
    The `affine.if` operation restricts execution to a subset of the loop
    iteration space defined by an integer set (a conjunction of affine
    constraints). A single `affine.if` may end with an optional `else` clause.

    The condition of the `affine.if` is represented by an
    [integer set](#integer-sets) (a conjunction of affine constraints),
    and the SSA values bound to the dimensions and symbols in the integer set.
    The [same restrictions](#restrictions-on-dimensions-and-symbols) hold for
    these SSA values as for all bindings of SSA values to dimensions and
    symbols.

```
- **EN**: Implements logic around `set`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `set` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 372-382
```tablegen
    The `affine.if` operation contains two regions for the "then" and "else"
    clauses.  `affine.if` may return results that are defined in its regions.
    The values defined are determined by which execution path is taken.  Each
    region of the `affine.if` must contain a single block with no arguments,
    and be terminated by `affine.yield`.  If `affine.if` defines no values,
    the `affine.yield` can be left out, and will be inserted implicitly.
    Otherwise, it must be explicit.  If no values are defined, the else block
    may be empty (i.e. contain no blocks).

    Example:

```
- **EN**: Implements logic around `empty`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `empty` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 383-400
```tablegen
    ```mlir
    #set = affine_set<(d0, d1)[s0]: (d0 - 10 >= 0, s0 - d0 - 9 >= 0,
                                     d1 - 10 >= 0, s0 - d1 - 9 >= 0)>
    func.func @reduced_domain_example(%A, %X, %N) : (memref<10xi32>, i32, i32) {
      affine.for %i = 0 to %N {
         affine.for %j = 0 to %N {
           %0 = affine.apply #map42(%j)
           %tmp = call @S1(%X, %i, %0)
           affine.if #set(%i, %j)[%N] {
              %1 = affine.apply #map43(%i, %j)
              call @S2(%tmp, %A, %i, %1)
           }
        }
      }
      return
    }
    ```

```
- **EN**: Implements logic around `reduced_domain_example`, `map42`, `S1`, `set`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `reduced_domain_example`, `map42`, `S1`, `set`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 401-420
```tablegen
    Example with an explicit yield (initialization with edge padding):

    ```mlir
    #interior = affine_set<(i, j) : (i - 1 >= 0, j - 1 >= 0,  10 - i >= 0, 10 - j >= 0)> (%i, %j)
    func.func @pad_edges(%I : memref<10x10xf32>) -> (memref<12x12xf32) {
      %O = alloc memref<12x12xf32>
      affine.parallel (%i, %j) = (0, 0) to (12, 12) {
        %1 = affine.if #interior (%i, %j) {
          %2 = load %I[%i - 1, %j - 1] : memref<10x10xf32>
          affine.yield %2
        } else {
          %2 = arith.constant 0.0 : f32
          affine.yield %2 : f32
        }
        affine.store %1, %O[%i, %j] : memref<12x12xf32>
      }
      return %O
    }
    ```
  }];
```
- **EN**: Implements logic around `yield`, `pad_edges`, `parallel`, `interior`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `yield`, `pad_edges`, `parallel`, `interior` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 421-434
```tablegen
  let arguments = (ins Variadic<AnyType>,
                       IntegerSetAttr:$condition);
  let results = (outs Variadic<AnyType>:$results);
  let regions = (region SizedRegion<1>:$thenRegion, AnyRegion:$elseRegion);

  let skipDefaultBuilders = 1;

  let builders = [
    OpBuilder<(ins "IntegerSet":$set, "ValueRange":$args,
      "bool":$withElseRegion)>,
    OpBuilder<(ins "TypeRange":$resultTypes, "IntegerSet":$set,
      "ValueRange":$args, "bool":$withElseRegion)>,
  ];

```
- **EN**: Declares APIs or declarative rules around `OpBuilder`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `OpBuilder` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 435-444
```tablegen
  let extraClassDeclaration = [{
    static StringRef getConditionAttrStrName() { return "condition"; }

    IntegerSet getIntegerSet();
    void setIntegerSet(IntegerSet newSet);

    /// Sets the integer set with its operands.
    void setConditional(IntegerSet set, ValueRange operands);

    /// Returns true if an else block exists.
```
- **EN**: Implements logic around `getConditionAttrStrName`, `getIntegerSet`, `setIntegerSet`, `setConditional`.
- **CN**: 围绕 `getConditionAttrStrName`, `getIntegerSet`, `setIntegerSet`, `setConditional` 实现具体逻辑。

### Lines 445-456
```tablegen
    bool hasElse() { return !getElseRegion().empty(); }

    Block *getThenBlock() {
      assert(!getThenRegion().empty() && "Unexpected empty 'then' region.");
      return &getThenRegion().front();
    }

    Block *getElseBlock() {
      assert(hasElse() && "Empty 'else' region.");
      return &getElseRegion().front();
    }

```
- **EN**: Implements logic around `hasElse`, `getThenBlock`, `assert`, `getThenRegion`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `hasElse`, `getThenBlock`, `assert`, `getThenRegion`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 457-468
```tablegen
    OpBuilder getThenBodyBuilder() {
      assert(!getThenRegion().empty() && "Unexpected empty 'then' region.");
      Block &body = getThenRegion().front();
      return OpBuilder(&body, std::prev(body.end()));
    }
    OpBuilder getElseBodyBuilder() {
      assert(hasElse() && "No 'else' block");
      Block &body = getElseRegion().front();
      return OpBuilder(&body, std::prev(body.end()));
    }
  }];

```
- **EN**: Implements logic around `getThenBodyBuilder`, `assert`, `getThenRegion`, `OpBuilder`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getThenBodyBuilder`, `assert`, `getThenRegion`, `OpBuilder`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 469-484
```tablegen
  let hasCanonicalizer = 1;
  let hasCustomAssemblyFormat = 1;
  let hasFolder = 1;
  let hasVerifier = 1;
}

class AffineLoadOpBase<string mnemonic, list<Trait> traits = []> :
    Affine_Op<mnemonic, !listconcat(traits,
        [DeclareOpInterfaceMethods<AffineReadOpInterface>,
        DeclareOpInterfaceMethods<AffineMapAccessInterface>,
        MemRefsNormalizable])> {
  let arguments = (ins Arg<AnyMemRef, "the reference to load from",
      [MemRead]>:$memref,
      Variadic<Index>:$indices,
      AffineMapAttr:$map);

```
- **EN**: Introduces declarations for `AffineLoadOpBase`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineLoadOpBase` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 485-495
```tablegen
  code extraClassDeclarationBase = [{
    /// Returns the operand index of the memref.
    unsigned getMemRefOperandIndex() { return 0; }

    void setMemRef(Value value) { setOperand(getMemRefOperandIndex(), value); }

    /// Returns the affine map used to index the memref for this operation.
    AffineMapAttr getAffineMapAttr() {
      return getProperties().map;
    }

```
- **EN**: Implements logic around `getMemRefOperandIndex`, `setMemRef`, `getAffineMapAttr`, `getProperties`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMemRefOperandIndex`, `setMemRef`, `getAffineMapAttr`, `getProperties` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 496-508
```tablegen
    static StringRef getMapAttrStrName() { return "map"; }
  }];
}

def AffineLoadOp : AffineLoadOpBase<"load"> {
  let summary = "affine load operation";
  let description = [{
    Syntax:

    ```
    operation ::= ssa-id `=` `affine.load` ssa-use `[` multi-dim-affine-map-of-ssa-ids `]` `:` memref-type
    ```

```
- **EN**: Introduces declarations for `AffineLoadOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineLoadOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 509-521
```tablegen
    The `affine.load` op reads an element from a memref, where the index
    for each memref dimension is an affine expression of loop induction
    variables and symbols. The output of `affine.load` is a new value with the
    same type as the elements of the memref. An affine expression of loop IVs
    and symbols must be specified for each dimension of the memref. The keyword
    `symbol` can be used to indicate SSA identifiers which are symbolic.

    Example 1:

    ```mlir
    %1 = affine.load %0[%i0 + 3, %i1 + 7] : memref<100x100xf32>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 522-532
```tablegen
    Example 2: Uses `symbol` keyword for symbols `%n` and `%m`.

    ```mlir
    %1 = affine.load %0[%i0 + symbol(%n), %i1 + symbol(%m)] : memref<100x100xf32>
    ```
  }];

  let results = (outs AnyType:$result);

  let builders = [
    /// Builds an affine load op with the specified map and operands.
```
- **EN**: Declares APIs or declarative rules around `symbol`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `symbol` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 533-542
```tablegen
    OpBuilder<(ins "AffineMap":$map, "ValueRange":$operands)>,
    /// Builds an affine load op with an identity map and operands.
    OpBuilder<(ins "Value":$memref, CArg<"ValueRange", "{}">:$indices)>,
    /// Builds an affine load op with the specified map and its operands.
    OpBuilder<(ins "Value":$memref, "AffineMap":$map,
      "ValueRange":$mapOperands)>
  ];

  let extraClassDeclaration = extraClassDeclarationBase;

```
- **EN**: Implements logic around `OpBuilder`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 543-553
```tablegen
  let hasCanonicalizer = 1;
  let hasCustomAssemblyFormat = 1;
  let hasFolder = 1;
  let hasVerifier = 1;
}

class AffineMinMaxOpBase<string mnemonic, list<Trait> traits = []> :
    Op<Affine_Dialect, mnemonic, traits> {
  let arguments = (ins AffineMapAttr:$map, Variadic<Index>:$operands);
  let results = (outs Index);

```
- **EN**: Introduces declarations for `AffineMinMaxOpBase`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineMinMaxOpBase` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 554-572
```tablegen
  let extraClassDeclaration = [{
    static StringRef getMapAttrStrName() { return "map"; }
    AffineMap getAffineMap() { return getMap(); }
    ValueRange getMapOperands() { return getOperands(); }
    ValueRange getDimOperands() {
      return OperandRange{getOperands().begin(),
                          getOperands().begin() + getMap().getNumDims()};
    }
    ValueRange getSymbolOperands() {
      return OperandRange{getOperands().begin() + getMap().getNumDims(),
                          getOperands().end()};
    }
  }];
  let hasCustomAssemblyFormat = 1;
  let hasFolder = 1;
  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `getMapAttrStrName`, `getAffineMap`, `getMapOperands`, `getDimOperands`, and 2 more symbols.
- **CN**: 围绕 `getMapAttrStrName`, `getAffineMap`, `getMapOperands`, `getDimOperands`, and 2 more symbols 实现具体逻辑。

### Lines 573-588
```tablegen
def AffineMinOp : AffineMinMaxOpBase<"min", [Pure]> {
  let summary = "min operation";
  let description = [{
    Syntax:

    ```
    operation ::= ssa-id `=` `affine.min` affine-map-attribute dim-and-symbol-use-list
    ```

    The `affine.min` operation applies an [affine mapping](#affine-expressions)
    to a list of SSA values, and returns the minimum value of all result
    expressions. The number of dimension and symbol arguments to `affine.min`
    must be equal to the respective number of dimensional and symbolic inputs to
    the affine mapping; the `affine.min` operation always returns one value. The
    input operands and result must all have 'index' type.

```
- **EN**: Introduces declarations for `AffineMinOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineMinOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 589-602
```tablegen
    Example:

    ```mlir
    %0 = affine.min affine_map<(d0)[s0] -> (1000, d0 + 512, s0)> (%arg0)[%arg1]
    ```
  }];
}

def AffineMaxOp : AffineMinMaxOpBase<"max", [Pure]> {
  let summary = "max operation";
  let description = [{
    The `affine.max` operation computes the maximum value result from a multi-result
    affine map.

```
- **EN**: Introduces declarations for `AffineMaxOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineMaxOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 603-622
```tablegen
    Example:

    ```mlir
    %0 = affine.max (d0) -> (1000, d0 + 512) (%i0) : index
    ```
  }];
}

def AffineParallelOp : Affine_Op<"parallel",
    [AutomaticAllocationScope, ImplicitAffineTerminator, RecursivelySpeculatable,
     RecursiveMemoryEffects, DeclareOpInterfaceMethods<LoopLikeOpInterface>,
     MemRefsNormalizable]> {
  let summary = "multi-index parallel band operation";
  let description = [{
    The `affine.parallel` operation represents a hyper-rectangular affine
    parallel band, defining zero or more SSA values for its induction variables.
    It has one region capturing the parallel band body. The induction variables
    are represented as arguments of this region. These SSA values always have
    type index, which is the size of the machine word. The strides, represented
    by steps, are positive constant integers which defaults to "1" if not
```
- **EN**: Introduces declarations for `AffineParallelOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineParallelOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 623-637
```tablegen
    present. The lower and upper bounds specify a half-open range: the range
    includes the lower bound but does not include the upper bound. The body
    region must contain exactly one block that terminates with `affine.yield`.

    The lower and upper bounds of a parallel operation are represented as an
    application of an affine mapping to a list of SSA values passed to the map.
    The same restrictions hold for these SSA values as for all bindings of SSA
    values to dimensions and symbols. The list of expressions in each map is
    interpreted according to the respective bounds group attribute. If a single
    expression belongs to the group, then the result of this expression is taken
    as a lower(upper) bound of the corresponding loop induction variable. If
    multiple expressions belong to the group, then the lower(upper) bound is the
    max(min) of these values obtained from these expressions. The loop band has
    as many loops as elements in the group bounds attributes.

```
- **EN**: Implements logic around `lower`, `max`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `lower`, `max` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 638-649
```tablegen
    Each value yielded by `affine.yield` will be accumulated/reduced via one of
    the reduction methods defined in the AtomicRMWKind enum.  The order of
    reduction is unspecified, and lowering may produce any valid ordering.
    Loops with a 0 trip count will produce as a result the identity value
    associated with each reduction (i.e. 0.0 for addf, 1.0 for mulf).  Assign
    reductions for loops with a trip count != 1 produces undefined results.

    Note: Calling `AffineParallelOp::build` will create the required region and
    block, and insert the required terminator if it is trivial (i.e. no values
    are yielded).  Parsing will also create the required region, block, and
    terminator, even when they are missing from the textual representation.

```
- **EN**: Implements logic around `reduction`, `trivial`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `reduction`, `trivial` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 650-667
```tablegen
    Example (3x3 valid convolution):

    ```mlir
    func.func @conv_2d(%D : memref<100x100xf32>, %K : memref<3x3xf32>) -> (memref<98x98xf32>) {
      %O = memref.alloc() : memref<98x98xf32>
      affine.parallel (%x, %y) = (0, 0) to (98, 98) {
        %0 = affine.parallel (%kx, %ky) = (0, 0) to (2, 2) reduce ("addf") -> f32 {
          %1 = affine.load %D[%x + %kx, %y + %ky] : memref<100x100xf32>
          %2 = affine.load %K[%kx, %ky] : memref<3x3xf32>
          %3 = arith.mulf %1, %2 : f32
          affine.yield %3 : f32
        }
        affine.store %0, %O[%x, %y] : memref<98x98xf32>
      }
      return %O : memref<98x98xf32>
    }
    ```

```
- **EN**: Implements logic around `Example`, `conv_2d`, `alloc`, `parallel`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Example`, `conv_2d`, `alloc`, `parallel` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 668-679
```tablegen
    Example (tiling by potentially imperfectly dividing sizes):

    ```mlir
    affine.parallel (%ii, %jj) = (0, 0) to (%N, %M) step (32, 32) {
      affine.parallel (%i, %j) = (%ii, %jj)
                              to (min(%ii + 32, %N), min(%jj + 32, %M)) {
        call @f(%i, %j) : (index, index) -> ()
      }
    }
    ```
  }];

```
- **EN**: Implements logic around `Example`, `parallel`, `to`, `f`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Example`, `parallel`, `to`, `f` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 680-690
```tablegen
  let arguments = (ins
     TypedArrayAttrBase<AtomicRMWKindAttr, "Reduction ops">:$reductions,
     AffineMapAttr:$lowerBoundsMap,
     I32ElementsAttr:$lowerBoundsGroups,
     AffineMapAttr:$upperBoundsMap,
     I32ElementsAttr:$upperBoundsGroups,
     I64SmallVectorArrayAttr:$steps,
     Variadic<Index>:$mapOperands);
  let results = (outs Variadic<AnyType>:$results);
  let regions = (region SizedRegion<1>:$region);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 691-701
```tablegen
  let builders = [
    OpBuilder<(ins "TypeRange":$resultTypes,
      "ArrayRef<arith::AtomicRMWKind>":$reductions, "ArrayRef<int64_t>":$ranges)>,
    OpBuilder<(ins "TypeRange":$resultTypes,
      "ArrayRef<arith::AtomicRMWKind>":$reductions, "ArrayRef<AffineMap>":$lbMaps,
      "ValueRange":$lbArgs, "ArrayRef<AffineMap>":$ubMaps, "ValueRange":$ubArgs,
      "ArrayRef<int64_t>":$steps)>
  ];

  let extraClassDeclaration = [{
    /// Get the number of dimensions.
```
- **EN**: Implements logic around `OpBuilder`.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑。

### Lines 702-712
```tablegen
    unsigned getNumDims();

    /// Get ranges as constants, may fail in dynamic case.
    std::optional<SmallVector<int64_t, 8>> getConstantRanges();

    Block *getBody();
    OpBuilder getBodyBuilder();
    MutableArrayRef<BlockArgument> getIVs() {
      return getBody()->getArguments();
    }

```
- **EN**: Implements logic around `getNumDims`, `getConstantRanges`, `getBody`, `getBodyBuilder`, and 1 more symbols.
- **CN**: 围绕 `getNumDims`, `getConstantRanges`, `getBody`, `getBodyBuilder`, and 1 more symbols 实现具体逻辑。

### Lines 713-725
```tablegen
    /// Returns elements of the loop lower bound.
    AffineMap getLowerBoundMap(unsigned pos);
    operand_range getLowerBoundsOperands();
    AffineValueMap getLowerBoundsValueMap();

    /// Sets elements of the loop lower bound.
    void setLowerBounds(ValueRange operands, AffineMap map);

    /// Returns elements of the loop upper bound.
    AffineMap getUpperBoundMap(unsigned pos);
    operand_range getUpperBoundsOperands();
    AffineValueMap getUpperBoundsValueMap();

```
- **EN**: Declares APIs or declarative rules around `getLowerBoundMap`, `getLowerBoundsOperands`, `getLowerBoundsValueMap`, `setLowerBounds`, and 3 more symbols.
- **CN**: 声明与 `getLowerBoundMap`, `getLowerBoundsOperands`, `getLowerBoundsValueMap`, `setLowerBounds`, and 3 more symbols 相关的 API 或声明式规则。

### Lines 726-743
```tablegen
    /// Sets elements of the loop upper bound.
    void setUpperBounds(ValueRange operands, AffineMap map);

    void setSteps(ArrayRef<int64_t> newSteps);

    /// Returns attribute names to use in op construction. Not expected to be
    /// used directly.
    static StringRef getReductionsAttrStrName() { return "reductions"; }
    static StringRef getLowerBoundsMapAttrStrName() { return "lowerBoundsMap"; }
    static StringRef getLowerBoundsGroupsAttrStrName() {
      return "lowerBoundsGroups";
    }
    static StringRef getUpperBoundsMapAttrStrName() { return "upperBoundsMap"; }
    static StringRef getUpperBoundsGroupsAttrStrName() {
      return "upperBoundsGroups";
    }
    static StringRef getStepsAttrStrName() { return "steps"; }

```
- **EN**: Implements logic around `setUpperBounds`, `setSteps`, `getReductionsAttrStrName`, `getLowerBoundsMapAttrStrName`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setUpperBounds`, `setSteps`, `getReductionsAttrStrName`, `getLowerBoundsMapAttrStrName`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 744-755
```tablegen
    /// Returns `true` if the loop bounds have min/max expressions.
    bool hasMinMaxBounds() {
      return getLowerBoundsMap().getNumResults() != getNumDims() ||
             getUpperBoundsMap().getNumResults() != getNumDims();
    }
  }];

  let hasCustomAssemblyFormat = 1;
  let hasFolder = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `hasMinMaxBounds`, `getLowerBoundsMap`, `getUpperBoundsMap`.
- **CN**: 围绕 `hasMinMaxBounds`, `getLowerBoundsMap`, `getUpperBoundsMap` 实现具体逻辑。

### Lines 756-765
```tablegen
def AffinePrefetchOp : Affine_Op<"prefetch",
  [DeclareOpInterfaceMethods<AffineMapAccessInterface>,
   MemRefsNormalizable]> {
  let summary = "affine prefetch operation";
  let description = [{
    The `affine.prefetch` op prefetches data from a memref location described
    with an affine subscript similar to affine.load, and has three attributes:
    a read/write specifier, a locality hint, and a cache type specifier as shown
    below:

```
- **EN**: Introduces declarations for `AffinePrefetchOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffinePrefetchOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 766-776
```tablegen
    ```mlir
    affine.prefetch %0[%i, %j + 5], read, locality<3>, data : memref<400x400xi32>
    ```

    The read/write specifier is either 'read' or 'write', the locality hint
    specifier ranges from locality<0> (no locality) to locality<3> (extremely
    local keep in cache). The cache type specifier is either 'data' or 'instr'
    and specifies whether the prefetch is performed on data cache or on
    instruction cache.
  }];

```
- **EN**: Declares APIs or declarative rules around `locality`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `locality` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 777-796
```tablegen
  let arguments = (ins AnyMemRef:$memref, Variadic<Index>:$indices,
                   BoolAttr:$isWrite,
                   ConfinedAttr<I32Attr, [IntMinValue<0>,
                     IntMaxValue<3>]>:$localityHint,
                   BoolAttr:$isDataCache,
                   AffineMapAttr:$map);

  let builders = [
    OpBuilder<(ins "Value":$memref, "AffineMap":$map,
      "ArrayRef<Value>":$mapOperands, "bool":$isWrite, "unsigned":$localityHint,
      "bool":$isDataCache),
    [{
      assert(map.getNumInputs() == mapOperands.size()
             && "inconsistent index info");
      auto localityHintAttr = $_builder.getI32IntegerAttr(localityHint);
      auto isWriteAttr = $_builder.getBoolAttr(isWrite);
      auto isDataCacheAttr = $_builder.getBoolAttr(isDataCache);
      $_state.addOperands(memref);
      $_state.addOperands(mapOperands);
      Properties &prop = $_state.getOrAddProperties<Properties>();
```
- **EN**: Implements logic around `OpBuilder`, `assert`, `getI32IntegerAttr`, `getBoolAttr`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `OpBuilder`, `assert`, `getI32IntegerAttr`, `getBoolAttr`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 797-807
```tablegen
      prop.map = AffineMapAttr::get(map);
      prop.localityHint = localityHintAttr;
      prop.isWrite = isWriteAttr;
      prop.isDataCache = isDataCacheAttr;
    }]>];

  let extraClassDeclaration = [{
    MemRefType getMemRefType() {
      return ::llvm::cast<MemRefType>(getMemref().getType());
    }

```
- **EN**: Implements logic around `get`, `getMemRefType`, `getMemref`.
- **CN**: 围绕 `get`, `getMemRefType`, `getMemref` 实现具体逻辑。

### Lines 808-822
```tablegen
    /// Returns the affine map used to index the memref for this operation.
    AffineMap getAffineMap() { return getAffineMapAttr().getValue(); }
    AffineMapAttr getAffineMapAttr() {
      return getProperties().map;
    }

    /// Implements the AffineMapAccessInterface.
    /// Returns the AffineMapAttr associated with 'memref'.
    NamedAttribute getAffineMapAttrForMemRef(Value mref) {
      assert(mref == getMemref() &&
             "Expected mref argument to match memref operand");
      return {StringAttr::get(getContext(), getMapAttrStrName()),
        getAffineMapAttr()};
    }

```
- **EN**: Implements logic around `getAffineMap`, `getAffineMapAttr`, `getProperties`, `getAffineMapAttrForMemRef`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getAffineMap`, `getAffineMapAttr`, `getProperties`, `getAffineMapAttrForMemRef`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 823-833
```tablegen
    /// Get affine map operands.
    operand_range getMapOperands() {
      return {operand_begin() + 1, operand_end()};
    }

    static StringRef getMapAttrStrName() { return "map"; }
    static StringRef getLocalityHintAttrStrName() { return "localityHint"; }
    static StringRef getIsWriteAttrStrName() { return "isWrite"; }
    static StringRef getIsDataCacheAttrStrName() { return "isDataCache"; }
  }];

```
- **EN**: Implements logic around `getMapOperands`, `operand_begin`, `getMapAttrStrName`, `getLocalityHintAttrStrName`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMapOperands`, `operand_begin`, `getMapAttrStrName`, `getLocalityHintAttrStrName`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 834-846
```tablegen
  let hasCanonicalizer = 1;
  let hasCustomAssemblyFormat = 1;
  let hasFolder = 1;
  let hasVerifier = 1;
}

class AffineStoreOpBase<string mnemonic, list<Trait> traits = []> :
    Affine_Op<mnemonic, !listconcat(traits,
    [DeclareOpInterfaceMethods<AffineWriteOpInterface>,
    DeclareOpInterfaceMethods<AffineMapAccessInterface>,
    MemRefsNormalizable])> {
  code extraClassDeclarationBase = [{
    /// Returns the operand index of the value to be stored.
```
- **EN**: Introduces declarations for `AffineStoreOpBase`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineStoreOpBase` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 847-858
```tablegen
    unsigned getStoredValOperandIndex() { return 0; }

    /// Returns the operand index of the memref.
    unsigned getMemRefOperandIndex() { return 1; }

    void setMemRef(Value value) { setOperand(getMemRefOperandIndex(), value); }

    /// Returns the affine map used to index the memref for this operation.
    AffineMapAttr getAffineMapAttr() {
      return getProperties().map;
    }

```
- **EN**: Implements logic around `getStoredValOperandIndex`, `getMemRefOperandIndex`, `setMemRef`, `getAffineMapAttr`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getStoredValOperandIndex`, `getMemRefOperandIndex`, `setMemRef`, `getAffineMapAttr`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 859-871
```tablegen
    static StringRef getMapAttrStrName() { return "map"; }
  }];
}

def AffineStoreOp : AffineStoreOpBase<"store"> {
  let summary = "affine store operation";
  let description = [{
    Syntax:

    ```
    operation ::= `affine.store` ssa-use, ssa-use `[` multi-dim-affine-map-of-ssa-ids `]` `:` memref-type
    ```

```
- **EN**: Introduces declarations for `AffineStoreOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineStoreOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 872-884
```tablegen
    The `affine.store` op writes an element to a memref, where the index
    for each memref dimension is an affine expression of loop induction
    variables and symbols. The `affine.store` op stores a new value which is the
    same type as the elements of the memref. An affine expression of loop IVs
    and symbols must be specified for each dimension of the memref. The keyword
    `symbol` can be used to indicate SSA identifiers which are symbolic.

    Example 1:

    ```mlir
    affine.store %v0, %0[%i0 + 3, %i1 + 7] : memref<100x100xf32>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 885-896
```tablegen
    Example 2: Uses `symbol` keyword for symbols `%n` and `%m`.

    ```mlir
    affine.store %v0, %0[%i0 + symbol(%n), %i1 + symbol(%m)] : memref<100x100xf32>
    ```
  }];
  let arguments = (ins AnyType:$value,
      Arg<AnyMemRef, "the reference to store to",
      [MemWrite]>:$memref,
      Variadic<Index>:$indices,
      AffineMapAttr:$map);

```
- **EN**: Declares APIs or declarative rules around `symbol`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `symbol` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 897-906
```tablegen
  let skipDefaultBuilders = 1;
  let builders = [
    OpBuilder<(ins "Value":$valueToStore, "Value":$memref,
      "ValueRange":$indices)>,
    OpBuilder<(ins "Value":$valueToStore, "Value":$memref, "AffineMap":$map,
      "ValueRange":$mapOperands)>
  ];

  let extraClassDeclaration = extraClassDeclarationBase;

```
- **EN**: Declares APIs or declarative rules around `OpBuilder`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `OpBuilder` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 907-926
```tablegen
  let hasCanonicalizer = 1;
  let hasCustomAssemblyFormat = 1;
  let hasFolder = 1;
  let hasVerifier = 1;
}

def AffineYieldOp : Affine_Op<"yield", [Pure, Terminator, ReturnLike,
    MemRefsNormalizable]> {
  let summary = "Yield values to parent operation";
  let description = [{
    The `affine.yield` yields zero or more SSA values from an affine op region and
    terminates the region. The semantics of how the values yielded are used
    is defined by the parent operation.
    If `affine.yield` has any operands, the operands must match the parent
    operation's results.
    If the parent operation defines no values, then the `affine.yield` may be
    left out in the custom syntax and the builders will insert one implicitly.
    Otherwise, it has to be present in the syntax to indicate which values are
    yielded.
  }];
```
- **EN**: Introduces declarations for `AffineYieldOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineYieldOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 927-936
```tablegen

  let arguments = (ins Variadic<AnyType>:$operands);

  let builders = [OpBuilder<(ins), [{ build($_builder, $_state, {}); }]>];

  let assemblyFormat = "attr-dict ($operands^ `:` type($operands))?";
  let hasVerifier = 1;
}

def AffineVectorLoadOp : AffineLoadOpBase<"vector_load"> {
```
- **EN**: Introduces declarations for `AffineVectorLoadOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineVectorLoadOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 937-951
```tablegen
  let summary = "affine vector load operation";
  let description = [{
    The `affine.vector_load` is the vector counterpart of
    [affine.load](#affineload-mliraffineloadop). It reads a slice from a
    [MemRef](Builtin.md/#memreftype), supplied as its first operand,
    into a [vector](Builtin.md/#vectortype) of the same base elemental type.
    The index for each memref dimension is an affine expression of loop induction
    variables and symbols. These indices determine the start position of the read
    within the memref. The shape of the return vector type determines the shape of
    the slice read from the memref. This slice is contiguous along the respective
    dimensions of the shape. Strided vector loads will be supported in the future.
    An affine expression of loop IVs and symbols must be specified for each
    dimension of the memref. The keyword `symbol` can be used to indicate SSA
    identifiers which are symbolic.

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 952-963
```tablegen
    Example 1: 8-wide f32 vector load.

    ```mlir
    %1 = affine.vector_load %0[%i0 + 3, %i1 + 7] : memref<100x100xf32>, vector<8xf32>
    ```

    Example 2: 4-wide f32 vector load. Uses `symbol` keyword for symbols `%n` and `%m`.

    ```mlir
    %1 = affine.vector_load %0[%i0 + symbol(%n), %i1 + symbol(%m)] : memref<100x100xf32>, vector<4xf32>
    ```

```
- **EN**: Implements logic around `symbol`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `symbol` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 964-975
```tablegen
    Example 3: 2-dim f32 vector load.

    ```mlir
    %1 = affine.vector_load %0[%i0, %i1] : memref<100x100xf32>, vector<2x8xf32>
    ```

    TODOs:
    * Add support for strided vector loads.
    * Consider adding a permutation map to permute the slice that is read from memory
    (see [vector.transfer_read](../Vector/#vectortransfer_read-mlirvectortransferreadop)).
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 976-985
```tablegen
  let results = (outs AnyVectorOfNonZeroRank:$result);

  let builders = [
    /// Builds an affine vector load op with the specified map and operands.
    OpBuilder<(ins "VectorType":$resultType, "AffineMap":$map,
      "ValueRange":$operands)>,
    /// Builds an affine vector load op with an identity map and operands.
    OpBuilder<(ins "VectorType":$resultType, "Value":$memref,
      CArg<"ValueRange", "{}">:$indices)>,
    /// Builds an affine vector load op with the specified map and its operands.
```
- **EN**: Implements logic around `OpBuilder`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 986-995
```tablegen
    OpBuilder<(ins "VectorType":$resultType, "Value":$memref,
      "AffineMap":$map, "ValueRange":$mapOperands)>
  ];

  let extraClassDeclaration = extraClassDeclarationBase # [{
    VectorType getVectorType() {
      return ::llvm::cast<VectorType>(getResult().getType());
    }
  }];

```
- **EN**: Implements logic around `OpBuilder`, `getVectorType`, `getResult`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `OpBuilder`, `getVectorType`, `getResult` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 996-1015
```tablegen
  let hasCanonicalizer = 1;
  let hasCustomAssemblyFormat = 1;
  let hasVerifier = 1;
}

def AffineVectorStoreOp : AffineStoreOpBase<"vector_store"> {
  let summary = "affine vector store operation";
  let description = [{
    The `affine.vector_store` is the vector counterpart of
    [affine.store](#affinestore-mliraffinestoreop). It writes a
    [vector](Builtin.md/#vectortype), supplied as its first operand,
    into a slice within a [MemRef](Builtin.md/#memreftype) of the same base
    elemental type, supplied as its second operand.
    The index for each memref dimension is an affine expression of loop
    induction variables and symbols. These indices determine the start position
    of the write within the memref. The shape of the input vector determines the
    shape of the slice written to the memref. This slice is contiguous along the
    respective dimensions of the shape. Strided vector stores will be supported
    in the future.
    An affine expression of loop IVs and symbols must be specified for each
```
- **EN**: Introduces declarations for `AffineVectorStoreOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineVectorStoreOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1016-1026
```tablegen
    dimension of the memref. The keyword `symbol` can be used to indicate SSA
    identifiers which are symbolic.

    Example 1: 8-wide f32 vector store.

    ```mlir
    affine.vector_store %v0, %0[%i0 + 3, %i1 + 7] : memref<100x100xf32>, vector<8xf32>
    ```

    Example 2: 4-wide f32 vector store. Uses `symbol` keyword for symbols `%n` and `%m`.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1027-1036
```tablegen
    ```mlir
    affine.vector_store %v0, %0[%i0 + symbol(%n), %i1 + symbol(%m)] : memref<100x100xf32>, vector<4xf32>
    ```

    Example 3: 2-dim f32 vector store.

    ```mlir
    affine.vector_store %v0, %0[%i0, %i1] : memref<100x100xf32>, vector<2x8xf32>
    ```

```
- **EN**: Implements logic around `symbol`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `symbol` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1037-1048
```tablegen
    TODOs:
    * Add support for strided vector stores.
    * Consider adding a permutation map to permute the slice that is written to memory
    (see [vector.transfer_write](../Vector/#vectortransfer_write-mlirvectortransferwriteop)).
  }];

  let arguments = (ins AnyVectorOfNonZeroRank:$value,
      Arg<AnyMemRef, "the reference to store to",
      [MemWrite]>:$memref,
      Variadic<Index>:$indices,
      AffineMapAttr:$map);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1049-1062
```tablegen
  let skipDefaultBuilders = 1;
  let builders = [
    OpBuilder<(ins "Value":$valueToStore, "Value":$memref,
      "ValueRange":$indices)>,
    OpBuilder<(ins "Value":$valueToStore, "Value":$memref, "AffineMap":$map,
      "ValueRange":$mapOperands)>
  ];

  let extraClassDeclaration = extraClassDeclarationBase # [{
    VectorType getVectorType() {
      return ::llvm::cast<VectorType>(getValue().getType());
    }
  }];

```
- **EN**: Implements logic around `OpBuilder`, `getVectorType`, `getValue`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `OpBuilder`, `getVectorType`, `getValue` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1063-1072
```tablegen
  let hasCanonicalizer = 1;
  let hasCustomAssemblyFormat = 1;
  let hasVerifier = 1;
}

//===----------------------------------------------------------------------===//
// AffineDelinearizeIndexOp
//===----------------------------------------------------------------------===//

def AffineDelinearizeIndexOp : Affine_Op<"delinearize_index",
```
- **EN**: Introduces declarations for `AffineDelinearizeIndexOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineDelinearizeIndexOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1073-1082
```tablegen
    [Pure, Elementwise,
     // Infer linear_index type from the first result type during parsing.
     TypesMatchWith<"linear_index type must match result types",
                    "multi_index", "linear_index", "$_self[0]">
    ]> {
  let summary = "delinearize an index";
  let description = [{
    The `affine.delinearize_index` operation takes a single index value and
    calculates the multi-index according to the given basis.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1083-1099
```tablegen
    Example:

    ```
    %indices:3 = affine.delinearize_index %linear_index into (%c16, %c224, %c224) : index, index, index
    ```

    In the above example, `%indices:3` conceptually holds the following:

    ```
    #map0 = affine_map<()[s0] -> (s0 floordiv 50176)>
    #map1 = affine_map<()[s0] -> ((s0 mod 50176) floordiv 224)>
    #map2 = affine_map<()[s0] -> (s0 mod 224)>
    %indices_0 = affine.apply #map0()[%linear_index]
    %indices_1 = affine.apply #map1()[%linear_index]
    %indices_2 = affine.apply #map2()[%linear_index]
    ```

```
- **EN**: Implements logic around `into`, `map0`, `map1`, `map2`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `into`, `map0`, `map1`, `map2` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1100-1112
```tablegen
    In other words, `%0:3 = affine.delinearize_index %x into (B, C)` produces
    `%0 = {%x / (B * C), (%x mod (B * C)) / C, %x mod C}`.

    The basis may either contain `N` or `N-1` elements, where `N` is the number of results.
    If there are N basis elements, the first one will not be used during computations,
    but may be used during analysis and canonicalization to eliminate terms from
    the `affine.delinearize_index` or to enable conclusions about the total size of
    `%linear_index`.

    If the basis is fully provided, the delinearize_index operation is said to "have
    an outer bound". The builders assume that an `affine.delinearize_index` has
    an outer bound by default, as this is how the operation was initially defined.

```
- **EN**: Implements logic around `into`, `mod`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `into`, `mod` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1113-1122
```tablegen
    That is, the example above could also have been written
    ```mlir
    %0:3 = affine.delinearize_index %linear_index into (244, 244) : index, index
    ```

    Note that, for symmetry with `getPaddedBasis()`, if `hasOuterBound` is `true`
    when one of the `OpFoldResult` builders is called but the first element of the
    basis is `nullptr`, that first element is ignored and the builder proceeds as if
    there was no outer bound.

```
- **EN**: Implements logic around `into`, `getPaddedBasis`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `into`, `getPaddedBasis` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1123-1136
```tablegen
    Due to the constraints of affine maps, all the basis elements must
    be strictly positive. A dynamic basis element being 0 or negative causes
    undefined behavior.

    As with other affine operations, lowerings of delinearize_index may assume
    that the underlying computations do not overflow the index type in a signed sense
    - that is, the product of all basis elements is positive as an `index` as well.
  }];

  let arguments = (ins Affine_IndexOrVectorOfIndex:$linear_index,
    Variadic<Index>:$dynamic_basis,
    DenseI64ArrayAttr:$static_basis);
  let results = (outs Variadic<Affine_IndexOrVectorOfIndex>:$multi_index);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1137-1149
```tablegen
  let assemblyFormat = [{
    $linear_index `into`
    custom<DynamicIndexList>($dynamic_basis, $static_basis, "{}", "::mlir::AsmParser::Delimiter::Paren")
    attr-dict `:` type($multi_index)
  }];

  let builders = [
    OpBuilder<(ins "Value":$linear_index, "ValueRange":$dynamic_basis, "ArrayRef<int64_t>":$static_basis, CArg<"bool", "true">:$hasOuterBound)>,
    OpBuilder<(ins "Value":$linear_index, "ValueRange":$basis, CArg<"bool", "true">:$hasOuterBound)>,
    OpBuilder<(ins "Value":$linear_index, "ArrayRef<OpFoldResult>":$basis, CArg<"bool", "true">:$hasOuterBound)>,
    OpBuilder<(ins "Value":$linear_index, "ArrayRef<int64_t>":$basis, CArg<"bool", "true">:$hasOuterBound)>
  ];

```
- **EN**: Implements logic around `custom`, `type`, `OpBuilder`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type`, `OpBuilder` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1150-1161
```tablegen
  let extraClassDeclaration = [{
    /// Return true if the basis includes a bound on the first index input.
    bool hasOuterBound() {
      return getMultiIndex().size() == getStaticBasis().size();
    }

    /// Returns a vector with all the static and dynamic basis values.
    SmallVector<OpFoldResult> getMixedBasis() {
      OpBuilder builder(getContext());
      return ::mlir::getMixedValues(getStaticBasis(), getDynamicBasis(), builder);
    }

```
- **EN**: Implements logic around `hasOuterBound`, `getMultiIndex`, `getMixedBasis`, `builder`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasOuterBound`, `getMultiIndex`, `getMixedBasis`, `builder`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1162-1171
```tablegen
    /// Return a vector that contains the basis of the operation, removing
    /// the outer bound if one is present.
    SmallVector<OpFoldResult> getEffectiveBasis();

    /// Return the vector with one basis element per result of the operation. If
    /// there is no outer bound specified, the leading entry of this result will be
    /// nullptr.
    SmallVector<OpFoldResult> getPaddedBasis();
  }];

```
- **EN**: Declares APIs or declarative rules around `getEffectiveBasis`, `getPaddedBasis`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getEffectiveBasis`, `getPaddedBasis` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1172-1191
```tablegen
  let hasVerifier = 1;
  let hasFolder = 1;
  let hasCanonicalizer = 1;
}

//===----------------------------------------------------------------------===//
// AffineLinearizeIndexOp
//===----------------------------------------------------------------------===//
def AffineLinearizeIndexOp : Affine_Op<"linearize_index",
    [Pure, AttrSizedOperandSegments, Elementwise,
     // Infer multi_index types from result type during parsing.
     TypesMatchWith<"multi_index types must match result type",
                    "linear_index", "multi_index", "$_self",
                    "[](::mlir::Type a, ::mlir::TypeRange b) { "
                    "return llvm::all_of(b, [a](::mlir::Type t) { "
                    "return t == a; }); }">]> {
  let summary = "linearize an index";
  let description = [{
    The `affine.linearize_index` operation takes a sequence of index values and a
    basis of the same length and linearizes the indices using that basis.
```
- **EN**: Introduces declarations for `AffineLinearizeIndexOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineLinearizeIndexOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1192-1202
```tablegen

    That is, for indices `%idx_0` to `%idx_{N-1}` and basis elements `b_0`
    (or `b_1`) up to `b_{N-1}` it computes

    ```
    sum(i = 0 to N-1) %idx_i * product(j = i + 1 to N-1) B_j
    ```

    In other words, `%0 = affine.linearize_index [%z, %y, %x] by (Z, Y, X)`
    gives `%0 = %x + %y * X + %z * X * Y`, or `%0 = %x + X * (%y + Y * (%z))`.

```
- **EN**: Implements logic around `sum`, `by`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `sum`, `by` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1203-1214
```tablegen
    The basis may either have `N` or `N-1` elements, where `N` is the number of
    inputs to linearize_index. If `N` inputs are provided, the first one is not used
    in computation, but may be used during analysis or canonicalization as a bound
    on `%idx_0`.

    If all `N` basis elements are provided, the linearize_index operation is said to
    "have an outer bound".

    As a convenience, and for symmetry with `getPaddedBasis()`, if the first
    element of a set of `OpFoldResult`s passed to the builders of this operation is
    `nullptr`, that element is ignored.

```
- **EN**: Implements logic around `getPaddedBasis`.
- **CN**: 围绕 `getPaddedBasis` 实现具体逻辑。

### Lines 1215-1224
```tablegen
    If the `disjoint` property is present, this is an optimization hint that,
    for all `i`, `0 <= %idx_i < B_i` - that is, no index affects any other index,
    except that `%idx_0` may be negative to make the index as a whole negative.
    In addition, `disjoint` is an assertion that all bases elements are non-negative.

    Note that the outputs of `affine.delinearize_index` are, by definition, `disjoint`.

    As with other affine ops, undefined behavior occurs if the linearization
    computation overflows in the signed sense.

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1225-1234
```tablegen
    Example:

    ```mlir
    %linear_index = affine.linearize_index [%index_0, %index_1, %index_2] by (2, 3, 5) : index
    // Same effect
    %linear_index = affine.linearize_index [%index_0, %index_1, %index_2] by (3, 5) : index
    ```

    In the above example, `%linear_index` conceptually holds the following:

```
- **EN**: Implements logic around `by`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `by` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1235-1246
```tablegen
    ```mlir
    #map = affine_map<()[s0, s1, s2] -> (s0 * 15 + s1 * 5 + s2)>
    %linear_index = affine.apply #map()[%index_0, %index_1, %index_2]
    ```
  }];

  let arguments = (ins Variadic<Affine_IndexOrVectorOfIndex>:$multi_index,
    Variadic<Index>:$dynamic_basis,
    DenseI64ArrayAttr:$static_basis,
    UnitProp:$disjoint);
  let results = (outs Affine_IndexOrVectorOfIndex:$linear_index);

```
- **EN**: Declares APIs or declarative rules around `map`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `map` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1247-1259
```tablegen
  let assemblyFormat = [{
    (`disjoint` $disjoint^)? ` `
    `[` $multi_index `]` `by`
    custom<DynamicIndexList>($dynamic_basis, $static_basis, "{}", "::mlir::AsmParser::Delimiter::Paren")
    attr-dict `:` type($linear_index)
  }];

  let builders = [
    OpBuilder<(ins "ValueRange":$multi_index, "ValueRange":$basis, CArg<"bool", "false">:$disjoint)>,
    OpBuilder<(ins "ValueRange":$multi_index, "ArrayRef<OpFoldResult>":$basis, CArg<"bool", "false">:$disjoint)>,
    OpBuilder<(ins "ValueRange":$multi_index, "ArrayRef<int64_t>":$basis, CArg<"bool", "false">:$disjoint)>
  ];

```
- **EN**: Implements logic around `custom`, `type`, `OpBuilder`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type`, `OpBuilder` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1260-1271
```tablegen
  let extraClassDeclaration = [{
    /// Return true if the basis includes a bound on the first index input.
    bool hasOuterBound() {
      return getMultiIndex().size() == getStaticBasis().size();
    }

    /// Return a vector with all the static and dynamic basis values.
    SmallVector<OpFoldResult> getMixedBasis() {
      OpBuilder builder(getContext());
      return ::mlir::getMixedValues(getStaticBasis(), getDynamicBasis(), builder);
    }

```
- **EN**: Implements logic around `hasOuterBound`, `getMultiIndex`, `getMixedBasis`, `builder`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasOuterBound`, `getMultiIndex`, `getMixedBasis`, `builder`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1272-1281
```tablegen
    /// Return a vector that contains the basis of the operation, removing
    /// the outer bound if one is present.
    SmallVector<OpFoldResult> getEffectiveBasis();

    /// Return the vector with one basis element per index operand of the operation.
    /// If there is no outer bound specified, the leading entry of this basis will be
    /// nullptr.
    SmallVector<OpFoldResult> getPaddedBasis();
  }];

```
- **EN**: Declares APIs or declarative rules around `getEffectiveBasis`, `getPaddedBasis`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getEffectiveBasis`, `getPaddedBasis` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1282-1291
```tablegen
  let hasVerifier = 1;
  let hasFolder = 1;
  let hasCanonicalizer = 1;
}

//===----------------------------------------------------------------------===//
// AffineDmaStartOp
//===----------------------------------------------------------------------===//

def AffineDmaStartOp : Affine_Op<"dma_start", [
```
- **EN**: Introduces declarations for `AffineDmaStartOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineDmaStartOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1292-1309
```tablegen
    MemRefsNormalizable,
    DeclareOpInterfaceMethods<AffineMapAccessInterface>,
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
  let summary = "affine dma start operation";
  let description = [{
    The `affine.dma_start` op starts a non-blocking DMA operation that
    transfers data from a source memref to a destination memref. The source and
    destination memref need not be of the same dimensionality, but need to have
    the same elemental type. The operands include the source and destination
    memref's each followed by its indices, size of the data transfer in terms of
    the number of elements (of the elemental type of the memref), a tag memref
    with its indices, and optionally at the end, a stride and a
    number_of_elements_per_stride arguments. The tag location is used by an
    `affine.dma_wait` to check for completion. The indices of the source memref,
    destination memref, and the tag memref have the same restrictions as any
    affine.load/store. In particular, index for each memref dimension must be an
    affine expression of loop induction variables and symbols.

```
- **EN**: Implements logic around `elements`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `elements` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1310-1319
```tablegen
    The optional stride arguments should be of 'index' type, and specify a
    stride for the slower memory space (memory space with a lower memory space
    id), transferring chunks of number_of_elements_per_stride every stride until
    %num_elements are transferred. Either both or no stride arguments should be
    specified. The value of 'num_elements' must be a multiple of
    'number_of_elements_per_stride'. If the source and destination locations
    overlap the behavior of this operation is not defined.

    Example:

```
- **EN**: Implements logic around `space`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `space` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1320-1335
```tablegen
    ```mlir
    %num_elements = arith.constant 256
    %idx = arith.constant 0 : index
    %tag = memref.alloc() : memref<1xi32, 4>
    affine.dma_start %src[%i + 3, %j], %dst[%k + 7, %l], %tag[%idx],
      %num_elements :
        memref<40x128xf32, 0>, memref<2x1024xf32, 1>, memref<1xi32, 2>

    // If %stride and %num_elt_per_stride are specified, the DMA is expected to
    // transfer %num_elt_per_stride elements every %stride elements apart from
    // memory space 0 until %num_elements are transferred.
    affine.dma_start %src[%i, %j], %dst[%k, %l], %tag[%idx], %num_elements,
      %stride, %num_elt_per_stride : ...
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `alloc` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1336-1353
```tablegen
  let arguments = (ins
    Variadic<AnyType>,
    AffineMapAttr:$src_map,
    AffineMapAttr:$dst_map,
    AffineMapAttr:$tag_map);

  let results = (outs);

  let skipDefaultBuilders = 1;
  let builders = [
    OpBuilder<(ins "Value":$srcMemRef, "AffineMap":$srcMap,
      "ValueRange":$srcIndices, "Value":$destMemRef, "AffineMap":$dstMap,
      "ValueRange":$destIndices, "Value":$tagMemRef, "AffineMap":$tagMap,
      "ValueRange":$tagIndices, "Value":$numElements,
      CArg<"Value", "nullptr">:$stride,
      CArg<"Value", "nullptr">:$elementsPerStride)>
  ];

```
- **EN**: Declares APIs or declarative rules around `OpBuilder`.
- **CN**: 声明与 `OpBuilder` 相关的 API 或声明式规则。

### Lines 1354-1366
```tablegen
  let extraClassDeclaration = [{
    /// Returns the operand index of the source memref.
    unsigned getSrcMemRefOperandIndex() { return 0; }

    /// Returns the source MemRefType for this DMA operation.
    Value getSrcMemRef() { return getOperand(getSrcMemRefOperandIndex()); }
    OpOperand &getSrcMemRefMutable() {
      return getOperation()->getOpOperand(getSrcMemRefOperandIndex());
    }
    MemRefType getSrcMemRefType() {
      return ::llvm::cast<MemRefType>(getSrcMemRef().getType());
    }

```
- **EN**: Implements logic around `getSrcMemRefOperandIndex`, `getSrcMemRef`, `getSrcMemRefMutable`, `getOperation`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getSrcMemRefOperandIndex`, `getSrcMemRef`, `getSrcMemRefMutable`, `getOperation`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1367-1376
```tablegen
    /// Returns the rank (number of indices) of the source MemRefType.
    unsigned getSrcMemRefRank() { return getSrcMemRefType().getRank(); }

    /// Returns the source memref affine map indices for this DMA operation.
    operand_range getSrcIndices() {
      return {operand_begin() + getSrcMemRefOperandIndex() + 1,
              operand_begin() + getSrcMemRefOperandIndex() + 1 +
                  getSrcMap().getNumInputs()};
    }

```
- **EN**: Implements logic around `getSrcMemRefRank`, `getSrcIndices`, `operand_begin`, `getSrcMap`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getSrcMemRefRank`, `getSrcIndices`, `operand_begin`, `getSrcMap` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1377-1387
```tablegen
    /// Returns the memory space of the source memref.
    unsigned getSrcMemorySpace() {
      return ::llvm::cast<MemRefType>(getSrcMemRef().getType())
          .getMemorySpaceAsInt();
    }

    /// Returns the operand index of the destination memref.
    unsigned getDstMemRefOperandIndex() {
      return getSrcMemRefOperandIndex() + 1 + getSrcMap().getNumInputs();
    }

```
- **EN**: Implements logic around `getSrcMemorySpace`, `getSrcMemRef`, `getMemorySpaceAsInt`, `getDstMemRefOperandIndex`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getSrcMemorySpace`, `getSrcMemRef`, `getMemorySpaceAsInt`, `getDstMemRefOperandIndex`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1388-1397
```tablegen
    /// Returns the destination MemRefType for this DMA operation.
    Value getDstMemRef() { return getOperand(getDstMemRefOperandIndex()); }
    OpOperand &getDstMemRefMutable() {
      return getOperation()->getOpOperand(getDstMemRefOperandIndex());
    }
    MemRefType getDstMemRefType() {
      return ::llvm::cast<MemRefType>(getDstMemRef().getType());
    }

    /// Returns the rank (number of indices) of the destination MemRefType.
```
- **EN**: Implements logic around `getDstMemRef`, `getDstMemRefMutable`, `getOperation`, `getDstMemRefType`.
- **CN**: 围绕 `getDstMemRef`, `getDstMemRefMutable`, `getOperation`, `getDstMemRefType` 实现具体逻辑。

### Lines 1398-1407
```tablegen
    unsigned getDstMemRefRank() {
      return ::llvm::cast<MemRefType>(getDstMemRef().getType()).getRank();
    }

    /// Returns the memory space of the destination memref.
    unsigned getDstMemorySpace() {
      return ::llvm::cast<MemRefType>(getDstMemRef().getType())
          .getMemorySpaceAsInt();
    }

```
- **EN**: Implements logic around `getDstMemRefRank`, `getDstMemRef`, `getDstMemorySpace`, `getMemorySpaceAsInt`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDstMemRefRank`, `getDstMemRef`, `getDstMemorySpace`, `getMemorySpaceAsInt` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1408-1419
```tablegen
    /// Returns the destination memref indices for this DMA operation.
    operand_range getDstIndices() {
      return {operand_begin() + getDstMemRefOperandIndex() + 1,
              operand_begin() + getDstMemRefOperandIndex() + 1 +
                  getDstMap().getNumInputs()};
    }

    /// Returns the operand index of the tag memref.
    unsigned getTagMemRefOperandIndex() {
      return getDstMemRefOperandIndex() + 1 + getDstMap().getNumInputs();
    }

```
- **EN**: Implements logic around `getDstIndices`, `operand_begin`, `getDstMap`, `getTagMemRefOperandIndex`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDstIndices`, `operand_begin`, `getDstMap`, `getTagMemRefOperandIndex`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1420-1429
```tablegen
    /// Returns the tag MemRef for this DMA operation.
    Value getTagMemRef() { return getOperand(getTagMemRefOperandIndex()); }
    OpOperand &getTagMemRefMutable() {
      return getOperation()->getOpOperand(getTagMemRefOperandIndex());
    }
    MemRefType getTagMemRefType() {
      return ::llvm::cast<MemRefType>(getTagMemRef().getType());
    }

    /// Returns the rank (number of indices) of the tag MemRefType.
```
- **EN**: Implements logic around `getTagMemRef`, `getTagMemRefMutable`, `getOperation`, `getTagMemRefType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTagMemRef`, `getTagMemRefMutable`, `getOperation`, `getTagMemRefType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1430-1440
```tablegen
    unsigned getTagMemRefRank() {
      return ::llvm::cast<MemRefType>(getTagMemRef().getType()).getRank();
    }

    /// Returns the tag memref indices for this DMA operation.
    operand_range getTagIndices() {
      return {operand_begin() + getTagMemRefOperandIndex() + 1,
              operand_begin() + getTagMemRefOperandIndex() + 1 +
                  getTagMap().getNumInputs()};
    }

```
- **EN**: Implements logic around `getTagMemRefRank`, `getTagMemRef`, `getTagIndices`, `operand_begin`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTagMemRefRank`, `getTagMemRef`, `getTagIndices`, `operand_begin`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1441-1460
```tablegen
    /// Returns the number of elements being transferred by this DMA operation.
    Value getNumElements() {
      return getOperand(getTagMemRefOperandIndex() + 1 +
                        getTagMap().getNumInputs());
    }

    /// Implements the AffineMapAccessInterface.
    /// Returns the AffineMapAttr associated with 'memref'.
    NamedAttribute getAffineMapAttrForMemRef(Value memref) {
      if (memref == getSrcMemRef())
        return {StringAttr::get(getContext(), getSrcMapAttrStrName()),
                getSrcMapAttr()};
      if (memref == getDstMemRef())
        return {StringAttr::get(getContext(), getDstMapAttrStrName()),
                getDstMapAttr()};
      assert(memref == getTagMemRef() &&
             "DmaStartOp expected source, destination or tag memref");
      return {StringAttr::get(getContext(), getTagMapAttrStrName()),
              getTagMapAttr()};
    }
```
- **EN**: Implements logic around `getNumElements`, `getOperand`, `getTagMap`, `getAffineMapAttrForMemRef`, and 7 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumElements`, `getOperand`, `getTagMap`, `getAffineMapAttrForMemRef`, and 7 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1461-1471
```tablegen

    /// Returns true if this is a DMA from a faster memory space to a slower one.
    bool isDestMemorySpaceFaster() {
      return (getSrcMemorySpace() < getDstMemorySpace());
    }

    /// Returns true if this is a DMA from a slower memory space to a faster one.
    bool isSrcMemorySpaceFaster() {
      return (getDstMemorySpace() < getSrcMemorySpace());
    }

```
- **EN**: Implements logic around `isDestMemorySpaceFaster`, `getSrcMemorySpace`, `isSrcMemorySpaceFaster`, `getDstMemorySpace`.
- **CN**: 围绕 `isDestMemorySpaceFaster`, `getSrcMemorySpace`, `isSrcMemorySpaceFaster`, `getDstMemorySpace` 实现具体逻辑。

### Lines 1472-1482
```tablegen
    /// Returns the operand position of either the source or destination memref
    /// depending on which is at the higher level of the memory hierarchy.
    unsigned getFasterMemPos() {
      assert(isSrcMemorySpaceFaster() || isDestMemorySpaceFaster());
      return isSrcMemorySpaceFaster() ? 0 : getDstMemRefOperandIndex();
    }

    static StringRef getSrcMapAttrStrName() { return "src_map"; }
    static StringRef getDstMapAttrStrName() { return "dst_map"; }
    static StringRef getTagMapAttrStrName() { return "tag_map"; }

```
- **EN**: Implements logic around `getFasterMemPos`, `assert`, `isSrcMemorySpaceFaster`, `getSrcMapAttrStrName`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getFasterMemPos`, `assert`, `isSrcMemorySpaceFaster`, `getSrcMapAttrStrName`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1483-1495
```tablegen
    /// Returns true if this DMA operation is strided, returns false otherwise.
    bool isStrided() {
      return getNumOperands() !=
             getTagMemRefOperandIndex() + 1 + getTagMap().getNumInputs() + 1;
    }

    /// Returns the stride value for this DMA operation.
    Value getStride() {
      if (!isStrided())
        return nullptr;
      return getOperand(getNumOperands() - 1 - 1);
    }

```
- **EN**: Implements logic around `isStrided`, `getNumOperands`, `getTagMemRefOperandIndex`, `getStride`, and 1 more symbols.
- **CN**: 围绕 `isStrided`, `getNumOperands`, `getTagMemRefOperandIndex`, `getStride`, and 1 more symbols 实现具体逻辑。

### Lines 1496-1509
```tablegen
    /// Returns the number of elements to transfer per stride for this DMA op.
    Value getNumElementsPerStride() {
      if (!isStrided())
        return nullptr;
      return getOperand(getNumOperands() - 1);
    }

  }];

  let hasCustomAssemblyFormat = 1;
  let hasVerifier = 1;
  let hasFolder = 1;
}

```
- **EN**: Implements logic around `getNumElementsPerStride`, `isStrided`, `getOperand`.
- **CN**: 围绕 `getNumElementsPerStride`, `isStrided`, `getOperand` 实现具体逻辑。

### Lines 1510-1526
```tablegen
//===----------------------------------------------------------------------===//
// AffineDmaWaitOp
//===----------------------------------------------------------------------===//

def AffineDmaWaitOp : Affine_Op<"dma_wait", [
    MemRefsNormalizable,
    DeclareOpInterfaceMethods<AffineMapAccessInterface>,
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
  let summary = "affine dma wait operation";
  let description = [{
    The `affine.dma_wait` op blocks until the completion of a DMA operation
    associated with the tag element `%tag[%index]`. `%tag` is a memref, and
    `%index` has to be an index with the same restrictions as any load/store
    index. In particular, index for each memref dimension must be an affine
    expression of loop induction variables and symbols. `%num_elements` is the
    number of elements associated with the DMA operation.

```
- **EN**: Introduces declarations for `AffineDmaWaitOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineDmaWaitOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1527-1536
```tablegen
    Example:

    ```mlir
    affine.dma_start %src[%i, %j], %dst[%k, %l], %tag[%index], %num_elements :
      memref<2048xf32, 0>, memref<256xf32, 1>, memref<1xi32, 2>
    ...
    affine.dma_wait %tag[%index], %num_elements : memref<1xi32, 2>
    ```
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1537-1548
```tablegen
  let arguments = (ins
    Variadic<AnyType>,
    AffineMapAttr:$tag_map);

  let results = (outs);

  let skipDefaultBuilders = 1;
  let builders = [
    OpBuilder<(ins "Value":$tagMemRef, "AffineMap":$tagMap,
      "ValueRange":$tagIndices, "Value":$numElements)>
  ];

```
- **EN**: Declares APIs or declarative rules around `OpBuilder`.
- **CN**: 声明与 `OpBuilder` 相关的 API 或声明式规则。

### Lines 1549-1558
```tablegen
  let extraClassDeclaration = [{
    /// Returns the tag MemRef associated with the DMA operation being waited on.
    Value getTagMemRef() { return getOperand(0); }
    OpOperand &getTagMemRefMutable() {
      return getOperation()->getOpOperand(0);
    }
    MemRefType getTagMemRefType() {
      return ::llvm::cast<MemRefType>(getTagMemRef().getType());
    }

```
- **EN**: Implements logic around `getTagMemRef`, `getTagMemRefMutable`, `getOperation`, `getTagMemRefType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTagMemRef`, `getTagMemRefMutable`, `getOperation`, `getTagMemRefType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1559-1569
```tablegen
    /// Returns the tag memref index for this DMA operation.
    operand_range getTagIndices() {
      return {operand_begin() + 1,
              operand_begin() + 1 + getTagMap().getNumInputs()};
    }

    /// Returns the rank (number of indices) of the tag memref.
    unsigned getTagMemRefRank() {
      return ::llvm::cast<MemRefType>(getTagMemRef().getType()).getRank();
    }

```
- **EN**: Implements logic around `getTagIndices`, `operand_begin`, `getTagMemRefRank`, `getTagMemRef`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTagIndices`, `operand_begin`, `getTagMemRefRank`, `getTagMemRef` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1570-1582
```tablegen
    /// Implements the AffineMapAccessInterface. Returns the AffineMapAttr
    /// associated with 'memref'.
    NamedAttribute getAffineMapAttrForMemRef(Value memref) {
      assert(memref == getTagMemRef());
      return {StringAttr::get(getContext(), getTagMapAttrStrName()),
              getTagMapAttr()};
    }

    /// Returns the number of elements transferred by the associated DMA op.
    Value getNumElements() {
      return getOperand(1 + getTagMap().getNumInputs());
    }

```
- **EN**: Implements logic around `getAffineMapAttrForMemRef`, `assert`, `get`, `getTagMapAttr`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getAffineMapAttrForMemRef`, `assert`, `get`, `getTagMapAttr`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1583-1592
```tablegen
    static StringRef getTagMapAttrStrName() { return "tag_map"; }

  }];

  let hasCustomAssemblyFormat = 1;
  let hasVerifier = 1;
  let hasFolder = 1;
}

#endif // AFFINE_OPS
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Describes the declarative or C++ interface for dialect operations, attributes, types, and registration helpers.
  - **CN**: 描述方言操作、属性、类型及注册辅助逻辑的声明式或 C++ 接口。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Dialect/Arith/IR/ArithBase.td`, `mlir/Dialect/Affine/IR/AffineMemoryOpInterfaces.td`, `mlir/Interfaces/ControlFlowInterfaces.td`, `mlir/Interfaces/InferIntRangeInterface.td`, `mlir/Interfaces/InferTypeOpInterface.td`, `mlir/Interfaces/LoopLikeInterface.td`, `mlir/Interfaces/SideEffectInterfaces.td`
- **Subsystem categories / 子系统类别**: cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (5), dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (2)
