# ControlFlowOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/ControlFlow/IR/ControlFlowOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains definitions for the operations within the ControlFlow dialect.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/ControlFlow/IR`，围绕 ControlFlow 方言公开 `ControlFlowOps` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```tablegen
//===- ControlFlowOps.td - ControlFlow operations ----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains definitions for the operations within the ControlFlow
// dialect.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 13-21
```tablegen

#ifndef MLIR_DIALECTS_CONTROLFLOW_IR_CONTROLFLOWOPS_TD
#define MLIR_DIALECTS_CONTROLFLOW_IR_CONTROLFLOWOPS_TD

include "mlir/IR/EnumAttr.td"
include "mlir/IR/OpAsmInterface.td"
include "mlir/Interfaces/ControlFlowInterfaces.td"
include "mlir/Interfaces/SideEffectInterfaces.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 22-32
```tablegen
def ControlFlow_Dialect : Dialect {
  let name = "cf";
  let cppNamespace = "::mlir::cf";
  let dependentDialects = ["arith::ArithDialect"];
  let description = [{
    This dialect contains low-level, i.e. non-region based, control flow
    constructs. These constructs generally represent control flow directly
    on SSA blocks of a control flow graph.
  }];
}

```
- **EN**: Introduces declarations for `ControlFlow_Dialect`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ControlFlow_Dialect` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 33-40
```tablegen
class CF_Op<string mnemonic, list<Trait> traits = []> :
    Op<ControlFlow_Dialect, mnemonic, traits>;

//===----------------------------------------------------------------------===//
// AssertOp
//===----------------------------------------------------------------------===//

def AssertOp : CF_Op<"assert",
```
- **EN**: Introduces declarations for `CF_Op`, `AssertOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CF_Op`, `AssertOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 41-49
```tablegen
    [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
  let summary = "Assert operation with message attribute";
  let description = [{
    Assert operation at runtime with single boolean operand and an error
    message attribute.
    If the argument is `true` this operation has no effect. Otherwise, the
    program execution will abort. The provided error message may be used by a
    runtime to propagate the error to the user.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 50-58
```tablegen
    Example:

    ```mlir
    cf.assert %b, "Expected ... to be true"
    ```
  }];

  let arguments = (ins I1:$arg, StrAttr:$msg);

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 59-66
```tablegen
  let assemblyFormat = "$arg `,` $msg attr-dict";
  let hasCanonicalizeMethod = 1;
}

//===----------------------------------------------------------------------===//
// BranchOp
//===----------------------------------------------------------------------===//

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 67-77
```tablegen
def BranchOp : CF_Op<"br", [
    DeclareOpInterfaceMethods<BranchOpInterface, ["getSuccessorForOperands"]>,
    Pure, Terminator
  ]> {
  let summary = "Branch operation";
  let description = [{
    The `cf.br` operation represents a direct branch operation to a given
    block. The operands of this operation are forwarded to the successor block,
    and the number and type of the operands must match the arguments of the
    target block.

```
- **EN**: Introduces declarations for `BranchOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `BranchOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 78-87
```tablegen
    Example:

    ```mlir
    ^bb2:
      %2 = call @someFn()
      cf.br ^bb3(%2 : tensor<*xf32>)
    ^bb3(%3: tensor<*xf32>):
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `someFn`, `bb3`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `someFn`, `bb3` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 88-97
```tablegen
  let arguments = (ins Variadic<AnyType>:$destOperands);
  let successors = (successor AnySuccessor:$dest);

  let builders = [
    OpBuilder<(ins "Block *":$dest,
                   CArg<"ValueRange", "{}">:$destOperands), [{
      $_state.addSuccessors(dest);
      $_state.addOperands(destOperands);
    }]>];

```
- **EN**: Implements logic around `OpBuilder`, `addSuccessors`, `addOperands`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder`, `addSuccessors`, `addOperands` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 98-110
```tablegen
  let extraClassDeclaration = [{
    void setDest(Block *block);

    /// Erase the operand at 'index' from the operand list.
    void eraseOperand(unsigned index);
  }];

  let hasCanonicalizeMethod = 1;
  let assemblyFormat = [{
    $dest (`(` $destOperands^ `:` type($destOperands) `)`)? attr-dict
  }];
}

```
- **EN**: Implements logic around `setDest`, `eraseOperand`, `dest`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setDest`, `eraseOperand`, `dest` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 111-126
```tablegen
//===----------------------------------------------------------------------===//
// CondBranchOp
//===----------------------------------------------------------------------===//

def CondBranchOp
    : CF_Op<"cond_br", [AttrSizedOperandSegments,
                        DeclareOpInterfaceMethods<
                            BranchOpInterface, ["getSuccessorForOperands"]>,
                        WeightedBranchOpInterface, Pure, Terminator]> {
  let summary = "Conditional branch operation";
  let description = [{
    The `cf.cond_br` terminator operation represents a conditional branch on a
    boolean (1-bit integer) value. If the bit is set, then the first destination
    is jumped to; if it is false, the second destination is chosen. The count
    and types of operands must align with the arguments in the corresponding
    target blocks.
```
- **EN**: Introduces declarations for `CondBranchOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CondBranchOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 127-134
```tablegen

    The MLIR conditional branch operation is not allowed to target the entry
    block for a region. The two destinations of the conditional branch operation
    are allowed to be the same.

    The following example illustrates a function with a conditional branch
    operation that targets the same block.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 135-147
```tablegen
    Example:

    ```mlir
    func.func @select(%a: i32, %b: i32, %flag: i1) -> i32 {
      // Both targets are the same, operands differ
      cf.cond_br %flag, ^bb1(%a : i32), ^bb1(%b : i32)

    ^bb1(%x : i32) :
      return %x : i32
    }
    ```
  }];

```
- **EN**: Implements logic around `select`, `bb1`.
- **CN**: 围绕 `select`, `bb1` 实现具体逻辑。

### Lines 148-163
```tablegen
  let arguments = (ins I1:$condition, Variadic<AnyType>:$trueDestOperands,
      Variadic<AnyType>:$falseDestOperands,
      OptionalAttr<DenseI32ArrayAttr>:$branch_weights);
  let successors = (successor AnySuccessor:$trueDest, AnySuccessor:$falseDest);

  let builders = [OpBuilder<(ins "Value":$condition, "Block *":$trueDest,
                                "ValueRange":$trueOperands,
                                "Block *":$falseDest,
                                "ValueRange":$falseOperands,
                                CArg<"ArrayRef<int32_t>", "{}">:$branchWeights),
                            [{
      DenseI32ArrayAttr weights;
      if (!branchWeights.empty())
        weights = $_builder.getDenseI32ArrayAttr(branchWeights);
      build($_builder, $_state, condition, trueOperands, falseOperands,
            weights, trueDest, falseDest);
```
- **EN**: Implements logic around `OpBuilder`, `empty`, `getDenseI32ArrayAttr`, `build`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder`, `empty`, `getDenseI32ArrayAttr`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 164-176
```tablegen
    }]>,
                  OpBuilder<(ins "Value":$condition, "Block *":$trueDest,
                                "Block *":$falseDest,
                                CArg<"ValueRange", "{}">:$falseOperands,
                                CArg<"ArrayRef<int32_t>", "{}">:$branchWeights),
                            [{
      DenseI32ArrayAttr weights;
      if (!branchWeights.empty())
        weights = $_builder.getDenseI32ArrayAttr(branchWeights);
      build($_builder, $_state, condition, ValueRange(), falseOperands,
            weights, trueDest, falseDest);
    }]>];

```
- **EN**: Implements logic around `OpBuilder`, `empty`, `getDenseI32ArrayAttr`, `build`.
- **CN**: 围绕 `OpBuilder`, `empty`, `getDenseI32ArrayAttr`, `build` 实现具体逻辑。

### Lines 177-186
```tablegen
  let extraClassDeclaration = [{
    // These are the indices into the dests list.
    enum { trueIndex = 0, falseIndex = 1 };

    // Accessors for operands to the 'true' destination.
    Value getTrueOperand(unsigned idx) {
      assert(idx < getNumTrueOperands());
      return getOperand(getTrueDestOperandIndex() + idx);
    }

```
- **EN**: Implements logic around `getTrueOperand`, `assert`, `getOperand`.
- **CN**: 围绕 `getTrueOperand`, `assert`, `getOperand` 实现具体逻辑。

### Lines 187-194
```tablegen
    void setTrueOperand(unsigned idx, Value value) {
      assert(idx < getNumTrueOperands());
      setOperand(getTrueDestOperandIndex() + idx, value);
    }

    unsigned getNumTrueOperands()  { return getTrueOperands().size(); }

    /// Erase the operand at 'index' from the true operand list.
```
- **EN**: Implements logic around `setTrueOperand`, `assert`, `setOperand`, `getNumTrueOperands`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setTrueOperand`, `assert`, `setOperand`, `getNumTrueOperands` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 195-208
```tablegen
    void eraseTrueOperand(unsigned index)  {
      getTrueDestOperandsMutable().erase(index);
    }

    // Accessors for operands to the 'false' destination.
    Value getFalseOperand(unsigned idx) {
      assert(idx < getNumFalseOperands());
      return getOperand(getFalseDestOperandIndex() + idx);
    }
    void setFalseOperand(unsigned idx, Value value) {
      assert(idx < getNumFalseOperands());
      setOperand(getFalseDestOperandIndex() + idx, value);
    }

```
- **EN**: Implements logic around `eraseTrueOperand`, `getTrueDestOperandsMutable`, `getFalseOperand`, `assert`, and 3 more symbols.
- **CN**: 围绕 `eraseTrueOperand`, `getTrueDestOperandsMutable`, `getFalseOperand`, `assert`, and 3 more symbols 实现具体逻辑。

### Lines 209-218
```tablegen
    operand_range getTrueOperands() { return getTrueDestOperands(); }
    operand_range getFalseOperands() { return getFalseDestOperands(); }

    unsigned getNumFalseOperands() { return getFalseOperands().size(); }

    /// Erase the operand at 'index' from the false operand list.
    void eraseFalseOperand(unsigned index) {
      getFalseDestOperandsMutable().erase(index);
    }

```
- **EN**: Implements logic around `getTrueOperands`, `getFalseOperands`, `getNumFalseOperands`, `eraseFalseOperand`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getTrueOperands`, `getFalseOperands`, `getNumFalseOperands`, `eraseFalseOperand`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 219-228
```tablegen
  private:
    /// Get the index of the first true destination operand.
    unsigned getTrueDestOperandIndex() { return 1; }

    /// Get the index of the first false destination operand.
    unsigned getFalseDestOperandIndex() {
      return getTrueDestOperandIndex() + getNumTrueOperands();
    }
  }];

```
- **EN**: Implements logic around `getTrueDestOperandIndex`, `getFalseDestOperandIndex`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getTrueDestOperandIndex`, `getFalseDestOperandIndex` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 229-237
```tablegen
  let hasCanonicalizer = 1;
  let assemblyFormat = [{
    $condition (`weights` `(` $branch_weights^ `)` )? `,`
    $trueDest (`(` $trueDestOperands^ `:` type($trueDestOperands) `)`)? `,`
    $falseDest (`(` $falseDestOperands^ `:` type($falseDestOperands) `)`)?
    attr-dict
  }];
}

```
- **EN**: Implements logic around `condition`, `trueDest`, `falseDest`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `condition`, `trueDest`, `falseDest` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 238-253
```tablegen
//===----------------------------------------------------------------------===//
// SwitchOp
//===----------------------------------------------------------------------===//

def SwitchOp : CF_Op<"switch",
    [AttrSizedOperandSegments,
     DeclareOpInterfaceMethods<BranchOpInterface, ["getSuccessorForOperands"]>,
     Pure, Terminator]> {
  let summary = "Switch operation";
  let description = [{
    The `cf.switch` terminator operation represents a switch on a signless integer
    value. If the flag matches one of the specified cases, then the
    corresponding destination is jumped to. If the flag does not match any of
    the cases, the default destination is jumped to. The count and types of
    operands must align with the arguments in the corresponding target blocks.

```
- **EN**: Introduces declarations for `SwitchOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SwitchOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 254-264
```tablegen
    Example:

    ```mlir
    cf.switch %flag : i32, [
      default: ^bb1(%a : i32),
      42: ^bb1(%b : i32),
      43: ^bb3(%c : i32)
    ]
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `bb1`, `bb3`.
- **CN**: 声明与 `bb1`, `bb3` 相关的 API 或声明式规则。

### Lines 265-280
```tablegen
  let arguments = (ins
    AnyInteger:$flag,
    Variadic<AnyType>:$defaultOperands,
    VariadicOfVariadic<AnyType, "case_operand_segments">:$caseOperands,
    OptionalAttr<AnyIntElementsAttr>:$case_values,
    DenseI32ArrayAttr:$case_operand_segments
  );
  let successors = (successor
    AnySuccessor:$defaultDestination,
    VariadicSuccessor<AnySuccessor>:$caseDestinations
  );
  let builders = [
    OpBuilder<(ins "Value":$flag,
      "Block *":$defaultDestination,
      "ValueRange":$defaultOperands,
      CArg<"ArrayRef<APInt>", "{}">:$caseValues,
```
- **EN**: Implements logic around `OpBuilder`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 281-296
```tablegen
      CArg<"BlockRange", "{}">:$caseDestinations,
      CArg<"ArrayRef<ValueRange>", "{}">:$caseOperands)>,
    OpBuilder<(ins "Value":$flag,
      "Block *":$defaultDestination,
      "ValueRange":$defaultOperands,
      CArg<"ArrayRef<int32_t>", "{}">:$caseValues,
      CArg<"BlockRange", "{}">:$caseDestinations,
      CArg<"ArrayRef<ValueRange>", "{}">:$caseOperands)>,
    OpBuilder<(ins "Value":$flag,
      "Block *":$defaultDestination,
      "ValueRange":$defaultOperands,
      CArg<"DenseIntElementsAttr", "{}">:$caseValues,
      CArg<"BlockRange", "{}">:$caseDestinations,
      CArg<"ArrayRef<ValueRange>", "{}">:$caseOperands)>
  ];

```
- **EN**: Implements logic around `OpBuilder`.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑。

### Lines 297-309
```tablegen
  let assemblyFormat = [{
    $flag `:` type($flag) `,` `[` `\n`
      custom<SwitchOpCases>(ref(type($flag)),$defaultDestination,
                            $defaultOperands,
                            type($defaultOperands),
                            $case_values,
                            $caseDestinations,
                            $caseOperands,
                            type($caseOperands))
   `]`
    attr-dict
  }];

```
- **EN**: Implements logic around `type`, `custom`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type`, `custom` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 310-317
```tablegen
  let extraClassDeclaration = [{
    /// Return the operands for the case destination block at the given index.
    OperandRange getCaseOperands(unsigned index) {
      return getCaseOperands()[index];
    }

    /// Return a mutable range of operands for the case destination block at the
    /// given index.
```
- **EN**: Implements logic around `getCaseOperands`.
- **CN**: 围绕 `getCaseOperands` 实现具体逻辑。

### Lines 318-326
```tablegen
    MutableOperandRange getCaseOperandsMutable(unsigned index) {
      return getCaseOperandsMutable()[index];
    }
  }];

  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `getCaseOperandsMutable`.
- **CN**: 围绕 `getCaseOperandsMutable` 实现具体逻辑。

### Lines 327-327
```tablegen
#endif // MLIR_DIALECTS_CONTROLFLOW_IR_CONTROLFLOWOPS_TD
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

- **TableGen includes / TableGen 包含**: `mlir/IR/EnumAttr.td`, `mlir/IR/OpAsmInterface.td`, `mlir/Interfaces/ControlFlowInterfaces.td`, `mlir/Interfaces/SideEffectInterfaces.td`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (2)
