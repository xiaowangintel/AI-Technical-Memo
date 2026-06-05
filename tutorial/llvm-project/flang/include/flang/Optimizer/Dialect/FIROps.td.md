# FIROps.td — Code Analysis / 代码分析

## Source / 来源

- File: `flang/include/flang/Optimizer/Dialect/FIROps.td`
- Repository: `llvm-project` / `llvm-project`
- Purpose (EN): FIR operation definitions \file Definition of the FIR dialect operations.
- Purpose (CN): 声明与 FIROps 相关的接口、数据结构或工具。

## Line-by-Line Analysis / 逐行分析

### Line 1

~~~~text
//===-- FIROps.td - FIR operation definitions --------------*- tablegen -*-===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 2

~~~~text
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 3

~~~~text
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
~~~~
- EN: Records licensing and redistribution terms for this file.
- CN: 记录该文件的许可证与再分发条款。

### Line 4

~~~~text
// See https://llvm.org/LICENSE.txt for license information.
~~~~
- EN: Records licensing and redistribution terms for this file.
- CN: 记录该文件的许可证与再分发条款。

### Line 5

~~~~text
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
~~~~
- EN: Records licensing and redistribution terms for this file.
- CN: 记录该文件的许可证与再分发条款。

### Line 6

~~~~text
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 7

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 8

~~~~text
///
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 9

~~~~text
/// \file
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 10

~~~~text
/// Definition of the FIR dialect operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 11

~~~~text
///
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 12

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 13

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 14

~~~~text
#ifndef FORTRAN_DIALECT_FIR_OPS
~~~~
- EN: Controls conditional compilation with `#ifndef`.
- CN: 使用 `#ifndef` 控制条件编译。

### Line 15

~~~~text
#define FORTRAN_DIALECT_FIR_OPS
~~~~
- EN: Defines the preprocessor macro `FORTRAN_DIALECT_FIR_OPS`.
- CN: 定义预处理宏 `FORTRAN_DIALECT_FIR_OPS`。

### Line 16

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 17

~~~~text
include "mlir/Dialect/Arith/IR/ArithBase.td"
~~~~
- EN: Includes the TableGen file `mlir/Dialect/Arith/IR/ArithBase.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/Dialect/Arith/IR/ArithBase.td`，以便在此复用其中的记录。

### Line 18

~~~~text
include "mlir/Dialect/Arith/IR/ArithOpsInterfaces.td"
~~~~
- EN: Includes the TableGen file `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td`，以便在此复用其中的记录。

### Line 19

~~~~text
include "mlir/Dialect/LLVMIR/LLVMAttrDefs.td"
~~~~
- EN: Includes the TableGen file `mlir/Dialect/LLVMIR/LLVMAttrDefs.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/Dialect/LLVMIR/LLVMAttrDefs.td`，以便在此复用其中的记录。

### Line 20

~~~~text
include "mlir/Interfaces/MemorySlotInterfaces.td"
~~~~
- EN: Includes the TableGen file `mlir/Interfaces/MemorySlotInterfaces.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/Interfaces/MemorySlotInterfaces.td`，以便在此复用其中的记录。

### Line 21

~~~~text
include "mlir/Interfaces/ViewLikeInterface.td"
~~~~
- EN: Includes the TableGen file `mlir/Interfaces/ViewLikeInterface.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/Interfaces/ViewLikeInterface.td`，以便在此复用其中的记录。

### Line 22

~~~~text
include "flang/Optimizer/Dialect/CUF/Attributes/CUFAttr.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/CUF/Attributes/CUFAttr.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/CUF/Attributes/CUFAttr.td`，以便在此复用其中的记录。

### Line 23

~~~~text
include "flang/Optimizer/Dialect/FIRDialect.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FIRDialect.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FIRDialect.td`，以便在此复用其中的记录。

### Line 24

~~~~text
include "flang/Optimizer/Dialect/FIRTypes.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FIRTypes.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FIRTypes.td`，以便在此复用其中的记录。

### Line 25

~~~~text
include "flang/Optimizer/Dialect/FIRAttr.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FIRAttr.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FIRAttr.td`，以便在此复用其中的记录。

### Line 26

~~~~text
include "flang/Optimizer/Dialect/FortranVariableInterface.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FortranVariableInterface.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FortranVariableInterface.td`，以便在此复用其中的记录。

### Line 27

~~~~text
include "flang/Optimizer/Dialect/FirAliasTagOpInterface.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FirAliasTagOpInterface.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FirAliasTagOpInterface.td`，以便在此复用其中的记录。

### Line 28

~~~~text
include "flang/Optimizer/Dialect/SafeTempArrayCopyAttrInterface.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/SafeTempArrayCopyAttrInterface.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/SafeTempArrayCopyAttrInterface.td`，以便在此复用其中的记录。

### Line 29

~~~~text
include "mlir/IR/BuiltinAttributes.td"
~~~~
- EN: Includes the TableGen file `mlir/IR/BuiltinAttributes.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/IR/BuiltinAttributes.td`，以便在此复用其中的记录。

### Line 30

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 31

~~~~text
// Base class for FIR operations.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 32

~~~~text
// All operations automatically get a prefix of "fir.".
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 33

~~~~text
class fir_Op<string mnemonic, list<Trait> traits>
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 34

~~~~text
  : Op<FIROpsDialect, mnemonic, traits>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 35

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 36

~~~~text
// Base class for FIR operations that take a single argument
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 37

~~~~text
class fir_SimpleOp<string mnemonic, list<Trait> traits>
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 38

~~~~text
  : fir_Op<mnemonic, traits> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 39

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 40

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 41

~~~~text
    operands attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 42

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 43

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 44

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 45

~~~~text
def fir_OneResultOpBuilder : OpBuilder<(ins
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 46

~~~~text
    "mlir::Type":$resultType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 47

~~~~text
    "mlir::ValueRange":$operands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 48

~~~~text
    CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 49

~~~~text
  [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 50

~~~~text
    if (resultType)
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 51

~~~~text
      $_state.addTypes(resultType);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 52

~~~~text
    $_state.addOperands(operands);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 53

~~~~text
    $_state.addAttributes(attributes);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 54

~~~~text
  }]>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 55

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 56

~~~~text
// Base class of FIR operations that return 1 result
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 57

~~~~text
class fir_OneResultOp<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 58

~~~~text
    fir_Op<mnemonic, traits>, Results<(outs fir_Type:$res)> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 59

~~~~text
  let builders = [fir_OneResultOpBuilder];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 60

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 61

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 62

~~~~text
// Base class of FIR operations that have 1 argument and return 1 result
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 63

~~~~text
class fir_SimpleOneResultOp<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 64

~~~~text
    fir_SimpleOp<mnemonic, traits> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 65

~~~~text
  let builders = [fir_OneResultOpBuilder];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 66

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 67

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 68

~~~~text
// Whether a type is a BaseBoxType or a reference to a BaseBoxType.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 69

~~~~text
def IsBoxAddressOrValueTypePred
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 70

~~~~text
        : CPred<"::fir::isBoxAddressOrValue($_self)">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 71

~~~~text
def fir_BoxAddressOrValueType : Type<IsBoxAddressOrValueTypePred,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 72

~~~~text
    "fir.box or fir.class type or reference">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 73

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 74

~~~~text
def RefOfConstantSizeAggregateTypePred
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 75

~~~~text
        : CPred<"::fir::isRefOfConstantSizeAggregateType($_self)">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 76

~~~~text
def AnyRefOfConstantSizeAggregateType : TypeConstraint<
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 77

~~~~text
      RefOfConstantSizeAggregateTypePred,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 78

~~~~text
      "a reference type to a constant size fir.array, fir.char, or fir.type">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 79

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 80

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 81

~~~~text
// Memory SSA operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 82

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 83

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 84

~~~~text
def fir_AllocaOp : fir_Op<"alloca", [
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 85

~~~~text
  AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 86

~~~~text
  DeclareOpInterfaceMethods<PromotableAllocationOpInterface>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 87

~~~~text
]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 88

~~~~text
  let summary = "allocate storage for a temporary on the stack given a type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 89

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 90

~~~~text
    This primitive operation is used to allocate an object on the stack.  A
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 91

~~~~text
    reference to the object of type `!fir.ref<T>` is returned.  The returned
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 92

~~~~text
    object has an undefined/uninitialized state.  The allocation can be given
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 93

~~~~text
    an optional name.  The allocation may have a dynamic repetition count
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 94

~~~~text
    for allocating a sequence of locations for the specified type.
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 95

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 96

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 97

~~~~text
      %c = ... : i64
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 98

~~~~text
      %x = fir.alloca i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 99

~~~~text
      %y = fir.alloca !fir.array<8 x i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 100

~~~~text
      %z = fir.alloca f32, %c
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 101

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 102

~~~~text
      %i = ... : i16
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 103

~~~~text
      %j = ... : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 104

~~~~text
      %w = fir.alloca !fir.type<PT(len1:i16, len2:i32)> (%i, %j : i16, i32)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 105

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 106

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 107

~~~~text
    Note that in the case of `%z`, a contiguous block of memory is allocated
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 108

~~~~text
    and its size is a runtime multiple of a 32-bit REAL value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 109

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 110

~~~~text
    In the case of `%w`, the arguments `%i` and `%j` are LEN parameters
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 111

~~~~text
    (`len1`, `len2`) to the type `PT`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 112

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 113

~~~~text
    Finally, the operation is undefined if the ssa-value `%c` is negative.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 114

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 115

~~~~text
    Fortran Semantics:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 116

~~~~text
    There is no language mechanism in Fortran to allocate space on the stack
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 117

~~~~text
    like C's `alloca()` function. Therefore fir.alloca is not control-flow
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 118

~~~~text
    dependent. However, the lifetime of a stack allocation is often limited to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 119

~~~~text
    a small region and a legal implementation may reuse stack storage in other
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 120

~~~~text
    regions when there is no conflict. For example, take the following code
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 121

~~~~text
    fragment.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 122

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 123

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 124

~~~~text
      CALL foo(1)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 125

~~~~text
      CALL foo(2)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 126

~~~~text
      CALL foo(3)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 127

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 128

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 129

~~~~text
    A legal implementation can allocate a stack slot and initialize it with the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 130

~~~~text
    constant `1`, then pass that by reference to foo. Likewise for the second
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 131

~~~~text
    and third calls to foo, each stack slot being initialized accordingly. It is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 132

~~~~text
    also a conforming implementation to reuse the same stack slot for all three
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 133

~~~~text
    calls, just initializing each in turn. This is possible as the lifetime of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 134

~~~~text
    the copy of each constant need not exceed that of the CALL statement.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 135

~~~~text
    Indeed, a user would likely expect a good Fortran compiler to perform such
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 136

~~~~text
    an optimization.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 137

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 138

~~~~text
    Stack allocations have a maximum lifetime concept: their uses must not
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 139

~~~~text
    exceed the lifetime of the closest parent operation with the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 140

~~~~text
    AutomaticAllocationScope trait, IsIsolatedFromAbove trait, or
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 141

~~~~text
    LoopLikeOpInterface trait. This restriction is meant to ease the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 142

~~~~text
    insertion of stack save and restore operations, and to ease the conversion
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 143

~~~~text
    of stack allocation into heap allocation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 144

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 145

~~~~text
    Until Fortran 2018, procedures defaulted to non-recursive. A legal
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 146

~~~~text
    implementation could therefore convert stack allocations to global
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 147

~~~~text
    allocations. Such a conversion effectively adds the SAVE attribute to all
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 148

~~~~text
    variables.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 149

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 150

~~~~text
    Some temporary entities (large arrays) probably should not be stack
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 151

~~~~text
    allocated as stack space can often be limited. A legal implementation can
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 152

~~~~text
    convert these large stack allocations to heap allocations regardless of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 153

~~~~text
    whether the procedure is recursive or not.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 154

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 155

~~~~text
    The pinned attribute is used to flag fir.alloca operation in a specific
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 156

~~~~text
    region and avoid them being hoisted in an alloca hoisting pass.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 157

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 158

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 159

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 160

~~~~text
    TypeAttr:$in_type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 161

~~~~text
    OptionalAttr<StrAttr>:$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 162

~~~~text
    OptionalAttr<StrAttr>:$bindc_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 163

~~~~text
    UnitAttr:$pinned,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 164

~~~~text
    Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 165

~~~~text
    Variadic<AnyIntegerType>:$shape
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 166

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 167

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 168

~~~~text
  let results =
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 169

~~~~text
      (outs Res<fir_ReferenceType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 170

~~~~text
                "", [MemAlloc<AutomaticAllocationScopeResource>]>:$res);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 171

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 172

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 173

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 174

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 175

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 176

~~~~text
    OpBuilder<(ins "mlir::Type":$inType, "llvm::StringRef":$uniqName,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 177

~~~~text
      "llvm::StringRef":$bindcName, CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 178

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 179

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 180

~~~~text
    OpBuilder<(ins "mlir::Type":$inType, "llvm::StringRef":$uniqName,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 181

~~~~text
      "llvm::StringRef":$bindcName, "bool":$pinned,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 182

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 183

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 184

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 185

~~~~text
    OpBuilder<(ins "mlir::Type":$inType, "llvm::StringRef":$uniqName,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 186

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 187

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 188

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 189

~~~~text
    OpBuilder<(ins "mlir::Type":$inType, "llvm::StringRef":$uniqName,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 190

~~~~text
      "bool":$pinned, CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 191

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 192

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 193

~~~~text
    OpBuilder<(ins "mlir::Type":$inType, "bool":$pinned,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 194

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 195

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 196

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 197

~~~~text
    OpBuilder<(ins "mlir::Type":$inType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 198

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 199

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 200

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 201

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 202

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 203

~~~~text
    mlir::Type getAllocatedType();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 204

~~~~text
    bool hasLenParams() { return !getTypeparams().empty(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 205

~~~~text
    bool hasShapeOperands() { return !getShape().empty(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 206

~~~~text
    bool isDynamic() {return hasLenParams() || hasShapeOperands();}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 207

~~~~text
    unsigned numLenParams() { return getTypeparams().size(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 208

~~~~text
    operand_range getLenParams() { return getTypeparams(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 209

~~~~text
    unsigned numShapeOperands() { return getShape().size(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 210

~~~~text
    operand_range getShapeOperands() { return getShape(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 211

~~~~text
    static mlir::Type getRefTy(mlir::Type ty);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 212

~~~~text
    /// Is this an operation that owns the alloca directly made in its region?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 213

~~~~text
    static bool ownsNestedAlloca(mlir::Operation* op);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 214

~~~~text
    /// Get the parent region that owns this alloca. Nullptr if none can be
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 215

~~~~text
    /// identified.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 216

~~~~text
    mlir::Region* getOwnerRegion();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 217

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 218

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 219

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 220

~~~~text
def fir_AllocMemOp : fir_Op<"allocmem", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 221

~~~~text
  let summary = "allocate storage on the heap for an object of a given type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 222

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 223

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 224

~~~~text
    Creates a heap memory reference suitable for storing a value of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 225

~~~~text
    given type, T.  The heap refernce returned has type `!fir.heap<T>`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 226

~~~~text
    The memory object is in an undefined state.  `allocmem` operations must
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 227

~~~~text
    be paired with `freemem` operations to avoid memory leaks.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 228

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 229

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 230

~~~~text
      %0 = fir.allocmem !fir.array<10 x f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 231

~~~~text
      fir.freemem %0 : !fir.heap<!fir.array<10 x f32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 232

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 233

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 234

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 235

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 236

~~~~text
    TypeAttr:$in_type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 237

~~~~text
    OptionalAttr<StrAttr>:$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 238

~~~~text
    OptionalAttr<StrAttr>:$bindc_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 239

~~~~text
    Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 240

~~~~text
    Variadic<AnyIntegerType>:$shape
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 241

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 242

~~~~text
  let results = (outs Res<fir_HeapType, "", [MemAlloc<DefaultResource>]>:$res);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 243

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 244

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 245

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 246

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 247

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 248

~~~~text
    OpBuilder<(ins "mlir::Type":$in_type, "llvm::StringRef":$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 249

~~~~text
      "llvm::StringRef":$bindc_name, CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 250

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 251

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 252

~~~~text
    OpBuilder<(ins "mlir::Type":$in_type, "llvm::StringRef":$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 253

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 254

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 255

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 256

~~~~text
    OpBuilder<(ins "mlir::Type":$in_type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 257

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 258

~~~~text
      CArg<"mlir::ValueRange", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 259

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 260

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 261

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 262

~~~~text
    mlir::Type getAllocatedType();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 263

~~~~text
    bool hasLenParams() { return !getTypeparams().empty(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 264

~~~~text
    bool hasShapeOperands() { return !getShape().empty(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 265

~~~~text
    unsigned numLenParams() { return getTypeparams().size(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 266

~~~~text
    operand_range getLenParams() { return getTypeparams(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 267

~~~~text
    unsigned numShapeOperands() { return getShape().size(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 268

~~~~text
    operand_range getShapeOperands() { return getShape(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 269

~~~~text
    static mlir::Type getRefTy(mlir::Type ty);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 270

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 271

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 272

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 273

~~~~text
def fir_FreeMemOp : fir_Op<"freemem", []> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 274

~~~~text
  let summary = "free a heap object";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 275

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 276

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 277

~~~~text
    Deallocates a heap memory reference that was allocated by an `allocmem`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 278

~~~~text
    The memory object that is deallocated is placed in an undefined state
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 279

~~~~text
    after `fir.freemem`.  Optimizations may treat the loading of an object
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 280

~~~~text
    in the undefined state as undefined behavior.  This includes aliasing
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 281

~~~~text
    references, such as the result of an `fir.embox`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 282

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 283

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 284

~~~~text
      %21 = fir.allocmem !fir.type<ZT(p:i32){field:i32}>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 285

~~~~text
      ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 286

~~~~text
      fir.freemem %21 : !fir.heap<!fir.type<ZT>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 287

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 288

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 289

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 290

~~~~text
  let arguments = (ins Arg<fir_HeapType, "", [MemFree]>:$heapref);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 291

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 292

~~~~text
  let assemblyFormat = "$heapref attr-dict `:` qualified(type($heapref))";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 293

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 294

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 295

~~~~text
def fir_LoadOp : fir_OneResultOp<"load", [
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 296

~~~~text
  FirAliasTagOpInterface,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 297

~~~~text
  DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 298

~~~~text
  DeclareOpInterfaceMethods<PromotableMemOpInterface>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 299

~~~~text
]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 300

~~~~text
  let summary = "load a value from a memory reference";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 301

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 302

~~~~text
    Load a value from a memory reference into an ssa-value (virtual register).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 303

~~~~text
    Produces an immutable ssa-value of the referent type. A memory reference
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 304

~~~~text
    has type `!fir.ref<T>`, `!fir.heap<T>`, or `!fir.ptr<T>`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 305

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 306

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 307

~~~~text
      %a = fir.alloca i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 308

~~~~text
      %l = fir.load %a : !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 309

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 310

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 311

~~~~text
    The ssa-value has an undefined value if the memory reference is undefined
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 312

~~~~text
    or null.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 313

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 314

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 315

~~~~text
  let arguments = (ins AnyReferenceLike:$memref,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 316

~~~~text
      OptionalAttr<LLVM_TBAATagArrayAttr>:$tbaa, UnitAttr:$nontemporal,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 317

~~~~text
      OptionalAttr<LLVM_AccessGroupArrayAttr>:$accessGroups);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 318

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 319

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$refVal)>,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 320

~~~~text
                  OpBuilder<(ins "mlir::Type":$resTy, "mlir::Value":$refVal)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 321

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 322

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 323

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 324

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 325

~~~~text
    static mlir::ParseResult getElementOf(mlir::Type &ele, mlir::Type ref);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 326

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 327

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 328

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 329

~~~~text
def fir_StoreOp : fir_Op<"store", [
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 330

~~~~text
  FirAliasTagOpInterface,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 331

~~~~text
  DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 332

~~~~text
  DeclareOpInterfaceMethods<PromotableMemOpInterface>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 333

~~~~text
]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 334

~~~~text
  let summary = "store an SSA-value to a memory location";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 335

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 336

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 337

~~~~text
    Store an ssa-value (virtual register) to a memory reference.  The stored
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 338

~~~~text
    value must be of the same type as the referent type of the memory
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 339

~~~~text
    reference.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 340

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 341

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 342

~~~~text
      %v = ... : f64
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 343

~~~~text
      %p = ... : !fir.ptr<f64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 344

~~~~text
      fir.store %v to %p : !fir.ptr<f64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 345

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 346

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 347

~~~~text
    The above store changes the value to which the pointer is pointing and not
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 348

~~~~text
    the pointer itself. The operation is undefined if the memory reference,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 349

~~~~text
    `%p`, is undefined or null.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 350

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 351

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 352

~~~~text
  let arguments = (ins AnyType:$value, AnyReferenceLike:$memref,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 353

~~~~text
      OptionalAttr<LLVM_TBAATagArrayAttr>:$tbaa, UnitAttr:$nontemporal,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 354

~~~~text
      OptionalAttr<LLVM_AccessGroupArrayAttr>:$accessGroups);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 355

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 356

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$value, "mlir::Value":$memref)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 357

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 358

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 359

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 360

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 361

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 362

~~~~text
    static mlir::Type elementType(mlir::Type refType);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 363

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 364

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 365

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 366

~~~~text
// NOTE on memory effects: The allocation effect (MemAlloc) is to ensure
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 367

~~~~text
// this operation is not removed by dead code elimination as a dead read
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 368

~~~~text
// on the argument: the prefetch happens as a side effect.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 369

~~~~text
def fir_PrefetchOp : fir_Op<"prefetch", [MemoryEffects<[MemAlloc<DefaultResource>]>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 370

~~~~text
  let summary = "prefetch a memory reference";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 371

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 372

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 373

~~~~text
    The prefetch is a hint to the code generator that the memory reference will
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 374

~~~~text
    be used in the near future. The prefetch is not guaranteed to be executed.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 375

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 376

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 377

~~~~text
      %a = ... -> !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 378

~~~~text
      fir.prefetch %a {read, data, localityHint = 3 : i32} : !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 379

~~~~text
      // ...
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 380

~~~~text
      fir.load %a : !fir.ref<i32> // use the prefetched value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 381

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 382

~~~~text
    Here,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 383

~~~~text
    `memref'      : address to be prefetched
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 384

~~~~text
    `rw'          : rw specifier >
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 385

~~~~text
                     represented as read (default) or write with values
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 386

~~~~text
                     0 and 1, respectively
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 387

~~~~text
    `localityHint': temporal locality specifier >
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 388

~~~~text
                     value ranging from 0 - no locality to 3 - extremely local
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 389

~~~~text
    `cacheType'   : cache type specifier >
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 390

~~~~text
                     represented as instruction or data (default) with values
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 391

~~~~text
                     0 and 1, respectively
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 392

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 393

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 394

~~~~text
  /// NOTE: The numerical values used here is in reference to the LLVM LangRef
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 395

~~~~text
  let arguments =
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 396

~~~~text
      (ins AnyReferenceLike : $memref, UnitAttr : $rw,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 397

~~~~text
       ConfinedAttr<I32Attr, [IntMinValue<0>, IntMaxValue<3>]> : $localityHint,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 398

~~~~text
       UnitAttr : $cacheType);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 399

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 400

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 401

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 402

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 403

~~~~text
def fir_CopyOp : fir_Op<"copy", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 404

~~~~text
  let summary = "copy constant size memory";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 405

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 406

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 407

~~~~text
    Copy the memory from a source with compile time constant size to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 408

~~~~text
    a destination of the same type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 409

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 410

~~~~text
    This is meant to be used for aggregate types where load and store
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 411

~~~~text
    are not appropriate to make a copy because LLVM is not meant to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 412

~~~~text
    handle load and store of "big" aggregates.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 413

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 414

~~~~text
    Its "no_overlap" attribute allows indicating that the source and destination
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 415

~~~~text
    are known to not overlap at compile time.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 416

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 417

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 418

~~~~text
      !t =!fir.type<t{x:!fir.array<1000xi32>}>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 419

~~~~text
      fir.copy %x to %y : !fir.ref<!t>, !fir.ref<!t>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 420

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 421

~~~~text
    TODO: add FirAliasTagOpInterface to carry TBAA.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 422

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 423

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 424

~~~~text
  let arguments = (ins AnyRefOfConstantSizeAggregateType:$source,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 425

~~~~text
                       AnyRefOfConstantSizeAggregateType:$destination,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 426

~~~~text
                       OptionalAttr<UnitAttr>:$no_overlap);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 427

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 428

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$source,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 429

~~~~text
                                 "mlir::Value":$destination,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 430

~~~~text
                                  CArg<"bool", "false">:$no_overlap)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 431

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 432

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 433

~~~~text
    $source `to` $destination (`no_overlap` $no_overlap^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 434

~~~~text
    attr-dict `:` type(operands)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 435

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 436

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 437

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 438

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 439

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 440

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 441

~~~~text
def fir_SaveResultOp : fir_Op<"save_result", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 442

~~~~text
  let summary = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 443

~~~~text
    save an array, box, or record function result SSA-value to a memory location
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 444

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 445

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 446

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 447

~~~~text
    Save the result of a function returning an array, box, or record type value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 448

~~~~text
    into a memory location given the shape and LEN parameters of the result.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 449

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 450

~~~~text
    Function results of type fir.box, fir.array, or fir.rec are abstract values
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 451

~~~~text
    that require a storage to be manipulated on the caller side. This operation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 452

~~~~text
    allows associating such abstract result to a storage. In later lowering of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 453

~~~~text
    the function interfaces, this storage might be used to pass the result in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 454

~~~~text
    memory.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 455

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 456

~~~~text
    For arrays, result, it is required to provide the shape of the result. For
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 457

~~~~text
    character arrays and derived types with LEN parameters, the LEN parameter
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 458

~~~~text
    values must be provided.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 459

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 460

~~~~text
    The fir.save_result associated to a function call must immediately follow
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 461

~~~~text
    the call and be in the same block.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 462

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 463

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 464

~~~~text
      %buffer = fir.alloca fir.array<?xf32>, %c100
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 465

~~~~text
      %shape = fir.shape %c100
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 466

~~~~text
      %array_result = fir.call @foo() : () -> fir.array<?xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 467

~~~~text
      fir.save_result %array_result to %buffer(%shape)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 468

~~~~text
      %coor = fir.array_coor %buffer%(%shape), %c5
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 469

~~~~text
      %fifth_element = fir.load %coor : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 470

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 471

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 472

~~~~text
    The above fir.save_result allows saving a fir.array function result into
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 473

~~~~text
    a buffer to later access its 5th element.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 474

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 475

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 476

~~~~text
  let arguments = (ins ArrayOrBoxOrRecord:$value,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 477

~~~~text
                   Arg<AnyReferenceLike, "", [MemWrite]>:$memref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 478

~~~~text
                   Optional<AnyShapeType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 479

~~~~text
                   Variadic<AnyIntegerType>:$typeparams);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 480

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 481

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 482

~~~~text
    $value `to` $memref (`(` $shape^ `)`)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 483

~~~~text
    attr-dict `:` type(operands)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 484

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 485

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 486

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 487

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 488

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 489

~~~~text
def fir_CharConvertOp : fir_Op<"char_convert", []> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 490

~~~~text
  let summary = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 491

~~~~text
    Primitive to convert an entity of type CHARACTER from one KIND to a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 492

~~~~text
    different KIND.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 493

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 494

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 495

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 496

~~~~text
    Copy a CHARACTER (must be in memory) of KIND _k1_ to a CHARACTER (also must
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 497

~~~~text
    be in memory) of KIND _k2_ where _k1_ != _k2_ and the buffers do not
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 498

~~~~text
    overlap. This latter restriction is unchecked, as the Fortran language
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 499

~~~~text
    definition eliminates the overlapping in memory case.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 500

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 501

~~~~text
    The number of code points copied is specified explicitly as the second
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 502

~~~~text
    argument. The length of the !fir.char type is ignored.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 503

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 504

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 505

~~~~text
      fir.char_convert %1 for %2 to %3 : !fir.ref<!fir.char<1,?>>, i32,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 506

~~~~text
          !fir.ref<!fir.char<2,20>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 507

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 508

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 509

~~~~text
    Should future support for encodings other than ASCII be supported, codegen
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 510

~~~~text
    can generate a call to a runtime helper routine which will map the code
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 511

~~~~text
    points from UTF-8 to UCS-2, for example. Such remappings may not always
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 512

~~~~text
    be possible as they may involve the creation of more code points than the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 513

~~~~text
    `count` limit. These details are left as future to-dos.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 514

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 515

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 516

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 517

~~~~text
    Arg<AnyReferenceLike, "", [MemRead]>:$from,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 518

~~~~text
    AnyIntegerType:$count,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 519

~~~~text
    Arg<AnyReferenceLike, "", [MemWrite]>:$to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 520

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 521

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 522

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 523

~~~~text
    $from `for` $count `to` $to attr-dict `:` type(operands)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 524

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 525

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 526

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 527

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 528

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 529

~~~~text
def fir_UndefOp : fir_OneResultOp<"undefined", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 530

~~~~text
  let summary = "explicit undefined value of some type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 531

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 532

~~~~text
    Constructs an ssa-value of the specified type with an undefined value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 533

~~~~text
    This operation is typically created internally by the mem2reg conversion
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 534

~~~~text
    pass. An undefined value can be of any type except `!fir.ref<T>`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 535

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 536

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 537

~~~~text
      %a = fir.undefined !fir.array<10 x !fir.type<T>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 538

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 539

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 540

~~~~text
    The example creates an array shaped ssa-value. The array is rank 1, extent
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 541

~~~~text
    10, and each element has type `!fir.type<T>`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 542

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 543

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 544

~~~~text
  let results = (outs AnyType:$intype);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 545

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 546

~~~~text
  let assemblyFormat = "type($intype) attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 547

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 548

~~~~text
  // Note: we allow `undef : ref<T>` since it is a possible from transformations.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 549

~~~~text
  let hasVerifier = 0;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 550

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 551

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 552

~~~~text
def fir_ZeroOp : fir_OneResultOp<"zero_bits", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 553

~~~~text
  let summary = "explicit polymorphic zero value of some type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 554

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 555

~~~~text
    Constructs an ssa-value of the specified type with a value of zero for all
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 556

~~~~text
    bits.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 557

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 558

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 559

~~~~text
      %a = fir.zero_bits !fir.box<!fir.array<10 x !fir.type<T>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 560

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 561

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 562

~~~~text
    The example creates a value of type box where all bits are zero.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 563

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 564

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 565

~~~~text
  let results = (outs AnyType:$intype);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 566

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 567

~~~~text
  let assemblyFormat = "type($intype) attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 568

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 569

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 570

~~~~text
def fir_FakeUseOp : fir_Op<"fake_use", []> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 571

~~~~text
  let summary = "fake use of values to extend their lifetime";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 572

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 573

~~~~text
    This operation is used to keep values alive for debug information purposes.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 574

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 575

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 576

~~~~text
  let arguments = (ins Variadic<AnyType>:$args);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 577

~~~~text
  let assemblyFormat = "$args attr-dict `:` type($args)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 578

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 579

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 580

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 581

~~~~text
// Terminator operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 582

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 583

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 584

~~~~text
class fir_SwitchTerminatorOp<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 585

~~~~text
    fir_Op<mnemonic, !listconcat(traits, [AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 586

~~~~text
    DeclareOpInterfaceMethods<BranchOpInterface>, Terminator])> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 587

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 588

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 589

~~~~text
    AnyType:$selector,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 590

~~~~text
    Variadic<AnyType>:$compareArgs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 591

~~~~text
    Variadic<AnyType>:$targetArgs
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 592

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 593

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 594

~~~~text
  let results = (outs);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 595

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 596

~~~~text
  let successors = (successor VariadicSuccessor<AnySuccessor>:$targets);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 597

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 598

~~~~text
  string extraSwitchClassDeclaration = [{
~~~~
- EN: Declares a string-valued TableGen field.
- CN: 声明一个字符串类型的 TableGen 字段。

### Line 599

~~~~text
    using Conditions = mlir::Value;
~~~~
- EN: Creates the alias `Conditions` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `Conditions`。

### Line 600

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 601

~~~~text
    static constexpr llvm::StringRef getCasesAttr() { return "case_tags"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 602

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 603

~~~~text
    // The number of destination conditions that may be tested
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 604

~~~~text
    unsigned getNumConditions() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 605

~~~~text
      return getCases().size();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 606

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 607

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 608

~~~~text
    // The selector is the value being tested to determine the destination
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 609

~~~~text
    mlir::Value getSelector(llvm::ArrayRef<mlir::Value> operands) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 610

~~~~text
      return operands[0];
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 611

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 612

~~~~text
    mlir::Value getSelector(mlir::ValueRange operands) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 613

~~~~text
      return operands.front();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 614

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 615

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 616

~~~~text
    // The number of blocks that may be branched to
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 617

~~~~text
    unsigned getNumDest() { return (*this)->getNumSuccessors(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 618

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 619

~~~~text
    std::optional<mlir::OperandRange> getCompareOperands(unsigned cond);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 620

~~~~text
    std::optional<llvm::ArrayRef<mlir::Value>> getCompareOperands(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 621

~~~~text
        llvm::ArrayRef<mlir::Value> operands, unsigned cond);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 622

~~~~text
    std::optional<mlir::ValueRange> getCompareOperands(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 623

~~~~text
        mlir::ValueRange operands, unsigned cond);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 624

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 625

~~~~text
    std::optional<llvm::ArrayRef<mlir::Value>> getSuccessorOperands(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 626

~~~~text
        llvm::ArrayRef<mlir::Value> operands, unsigned cond);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 627

~~~~text
    std::optional<mlir::ValueRange> getSuccessorOperands(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 628

~~~~text
        mlir::ValueRange operands, unsigned cond);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 629

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 630

~~~~text
    // Helper function to deal with Optional operand forms
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 631

~~~~text
    void printSuccessorAtIndex(mlir::OpAsmPrinter &p, unsigned i) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 632

~~~~text
      auto *succ = getSuccessor(i);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 633

~~~~text
      auto ops = getSuccessorOperands(i);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 634

~~~~text
      p.printSuccessorAndUseList(succ, ops.getForwardedOperands());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 635

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 636

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 637

~~~~text
    mlir::ArrayAttr getCases() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 638

~~~~text
      return (*this)->getAttrOfType<mlir::ArrayAttr>(getCasesAttr());
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 639

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 640

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 641

~~~~text
    unsigned targetOffsetSize();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 642

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 643

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 644

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 645

~~~~text
class fir_IntegralSwitchTerminatorOp<string mnemonic,
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 646

~~~~text
    list<Trait> traits = []> : fir_SwitchTerminatorOp<mnemonic, traits> {
~~~~
- EN: Declares a list-valued TableGen field.
- CN: 声明一个列表类型的 TableGen 字段。

### Line 647

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 648

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 649

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$selector,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 650

~~~~text
    "llvm::ArrayRef<int64_t>":$compareOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 651

~~~~text
    "llvm::ArrayRef<mlir::Block *>":$destinations,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 652

~~~~text
    CArg<"llvm::ArrayRef<mlir::ValueRange>", "{}">:$destOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 653

~~~~text
    CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 654

~~~~text
    [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 655

~~~~text
      $_state.addOperands(selector);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 656

~~~~text
      llvm::SmallVector<mlir::Attribute> ivalues;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 657

~~~~text
      for (auto iv : compareOperands)
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 658

~~~~text
        ivalues.push_back($_builder.getI64IntegerAttr(iv));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 659

~~~~text
      ivalues.push_back($_builder.getUnitAttr());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 660

~~~~text
      $_state.addAttribute(getCasesAttr(), $_builder.getArrayAttr(ivalues));
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 661

~~~~text
      const auto count = destinations.size();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 662

~~~~text
      for (auto d : destinations)
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 663

~~~~text
        $_state.addSuccessors(d);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 664

~~~~text
      const auto opCount = destOperands.size();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 665

~~~~text
      llvm::SmallVector<int32_t> argOffs;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 666

~~~~text
      int32_t sumArgs = 0;
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 667

~~~~text
      for (std::remove_const_t<decltype(count)> i = 0; i != count; ++i) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 668

~~~~text
        if (i < opCount) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 669

~~~~text
          $_state.addOperands(destOperands[i]);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 670

~~~~text
          const auto argSz = destOperands[i].size();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 671

~~~~text
          argOffs.push_back(argSz);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 672

~~~~text
          sumArgs += argSz;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 673

~~~~text
        } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 674

~~~~text
          argOffs.push_back(0);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 675

~~~~text
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 676

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 677

~~~~text
      $_state.addAttribute(getOperandSegmentSizeAttr(),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 678

~~~~text
        $_builder.getDenseI32ArrayAttr({1, 0, sumArgs}));
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 679

~~~~text
      $_state.addAttribute(getTargetOffsetAttr(),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 680

~~~~text
        $_builder.getDenseI32ArrayAttr(argOffs));
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 681

~~~~text
      $_state.addAttributes(attributes);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 682

~~~~text
    }]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 683

~~~~text
  >];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 684

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 685

~~~~text
  let extraClassDeclaration = extraSwitchClassDeclaration;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 686

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 687

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 688

~~~~text
def fir_SelectOp : fir_IntegralSwitchTerminatorOp<"select"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 689

~~~~text
  let summary = "a multiway branch";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 690

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 691

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 692

~~~~text
    A multiway branch terminator with similar semantics to C's `switch`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 693

~~~~text
    statement.  A selector value is matched against a list of constants
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 694

~~~~text
    of the same type for a match.  When a match is found, control is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 695

~~~~text
    transferred to the corresponding basic block.  A `select` must have
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 696

~~~~text
    at least one basic block with a corresponding `unit` match, and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 697

~~~~text
    that block will be selected when all other conditions fail to match.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 698

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 699

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 700

~~~~text
      fir.select %arg:i32 [1, ^bb1(%0 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 701

~~~~text
                           2, ^bb2(%2,%arg,%arg2 : i32,i32,i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 702

~~~~text
                          -3, ^bb3(%arg2,%2 : i32,i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 703

~~~~text
                           4, ^bb4(%1 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 704

~~~~text
                        unit, ^bb5]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 705

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 706

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 707

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 708

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 709

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 710

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 711

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 712

~~~~text
def fir_SelectRankOp : fir_IntegralSwitchTerminatorOp<"select_rank"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 713

~~~~text
  let summary = "Fortran's SELECT RANK statement";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 714

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 715

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 716

~~~~text
    Similar to `select`, `select_rank` provides a way to express Fortran's
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 717

~~~~text
    SELECT RANK construct.  In this case, the rank of the selector value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 718

~~~~text
    is matched against constants of integer type.  The structure is the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 719

~~~~text
    same as `select`, but `select_rank` determines the rank of the selector
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 720

~~~~text
    variable at runtime to determine the best match.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 721

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 722

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 723

~~~~text
      fir.select_rank %arg:i32 [1, ^bb1(%0 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 724

~~~~text
                                2, ^bb2(%2,%arg,%arg2 : i32,i32,i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 725

~~~~text
                                3, ^bb3(%arg2,%2 : i32,i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 726

~~~~text
                               -1, ^bb4(%1 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 727

~~~~text
                             unit, ^bb5]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 728

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 729

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 730

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 731

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 732

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 733

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 734

~~~~text
def fir_SelectCaseOp : fir_SwitchTerminatorOp<"select_case"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 735

~~~~text
  let summary = "Fortran's SELECT CASE statement";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 736

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 737

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 738

~~~~text
    Similar to `select`, `select_case` provides a way to express Fortran's
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 739

~~~~text
    SELECT CASE construct.  In this case, the selector value is matched
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 740

~~~~text
    against variables (not just constants) and ranges.  The structure is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 741

~~~~text
    the same as `select`, but `select_case` allows for the expression of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 742

~~~~text
    more complex match conditions.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 743

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 744

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 745

~~~~text
      fir.select_case %arg : i32 [
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 746

~~~~text
            #fir.point, %0, ^bb1(%0 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 747

~~~~text
            #fir.lower, %1, ^bb2(%2,%arg,%arg2,%1 : i32,i32,i32,i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 748

~~~~text
            #fir.interval, %2, %3, ^bb3(%2,%arg2 : i32,i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 749

~~~~text
            #fir.upper, %arg, ^bb4(%1 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 750

~~~~text
            unit, ^bb5]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 751

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 752

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 753

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 754

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 755

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 756

~~~~text
    OpBuilder<(ins "mlir::Value":$selector,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 757

~~~~text
      "llvm::ArrayRef<mlir::Attribute>":$compareAttrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 758

~~~~text
      "llvm::ArrayRef<mlir::ValueRange>":$cmpOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 759

~~~~text
      "llvm::ArrayRef<mlir::Block *>":$destinations,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 760

~~~~text
      CArg<"llvm::ArrayRef<mlir::ValueRange>", "{}">:$destOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 761

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 762

~~~~text
    OpBuilder<(ins "mlir::Value":$selector,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 763

~~~~text
      "llvm::ArrayRef<mlir::Attribute>":$compareAttrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 764

~~~~text
      "llvm::ArrayRef<mlir::Value>":$cmpOpList,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 765

~~~~text
      "llvm::ArrayRef<mlir::Block *>":$destinations,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 766

~~~~text
      CArg<"llvm::ArrayRef<mlir::ValueRange>", "{}">:$destOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 767

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 768

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 769

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 770

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 771

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 772

~~~~text
  let extraClassDeclaration = extraSwitchClassDeclaration#[{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 773

~~~~text
    unsigned compareOffsetSize();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 774

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 775

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 776

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 777

~~~~text
def fir_SelectTypeOp : fir_SwitchTerminatorOp<"select_type"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 778

~~~~text
  let summary = "Fortran's SELECT TYPE statement";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 779

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 780

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 781

~~~~text
    Similar to `select`, `select_type` provides a way to express Fortran's
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 782

~~~~text
    SELECT TYPE construct.  In this case, the type of the selector value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 783

~~~~text
    is matched against a list of type descriptors.  The structure is the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 784

~~~~text
    same as `select`, but `select_type` determines the type of the selector
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 785

~~~~text
    variable at runtime to determine the best match.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 786

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 787

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 788

~~~~text
      fir.select_type %arg : !fir.box<()> [
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 789

~~~~text
          #fir.type_is<!fir.type<type1>>, ^bb1(%0 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 790

~~~~text
          #fir.type_is<!fir.type<type2>>, ^bb2(%2 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 791

~~~~text
          #fir.class_is<!fir.type<type3>>, ^bb3(%2 : i32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 792

~~~~text
          #fir.type_is<!fir.type<type4>>, ^bb4(%1,%3 : i32,f32),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 793

~~~~text
          unit, ^bb5]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 794

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 795

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 796

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 797

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 798

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$selector,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 799

~~~~text
    "llvm::ArrayRef<mlir::Attribute>":$typeOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 800

~~~~text
    "llvm::ArrayRef<mlir::Block *>":$destinations,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 801

~~~~text
    CArg<"llvm::ArrayRef<mlir::ValueRange>", "{}">:$destOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 802

~~~~text
    CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 803

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 804

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 805

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 806

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 807

~~~~text
  let extraClassDeclaration = extraSwitchClassDeclaration;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 808

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 809

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 810

~~~~text
def fir_UnreachableOp : fir_Op<"unreachable", [Terminator]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 811

~~~~text
  let summary = "the unreachable instruction";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 812

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 813

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 814

~~~~text
    Terminates a basic block with the assertion that the end of the block
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 815

~~~~text
    will never be reached at runtime.  This instruction can be used
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 816

~~~~text
    immediately after a call to the Fortran runtime to terminate the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 817

~~~~text
    program, for example.  This instruction corresponds to the LLVM IR
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 818

~~~~text
    instruction `unreachable`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 819

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 820

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 821

~~~~text
      fir.unreachable
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 822

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 823

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 824

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 825

~~~~text
  let assemblyFormat = [{ attr-dict }];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 826

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 827

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 828

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 829

~~~~text
def fir_FirEndOp : fir_Op<"end", [Terminator, NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 830

~~~~text
  let summary = "the end instruction";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 831

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 832

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 833

~~~~text
    The end terminator is a special terminator used inside various FIR
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 834

~~~~text
    operations that have regions.  End is thus the custom invisible terminator
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 835

~~~~text
    for these operations.  It is implicit and need not appear in the textual
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 836

~~~~text
    representation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 837

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 838

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 839

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 840

~~~~text
def fir_HasValueOp : fir_Op<"has_value", [Terminator, HasParent<"GlobalOp">]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 841

~~~~text
  let summary = "terminator for GlobalOp";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 842

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 843

~~~~text
    The terminator for a GlobalOp with a body.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 844

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 845

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 846

~~~~text
      global @variable : tuple<i32, f32> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 847

~~~~text
        %0 = arith.constant 45 : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 848

~~~~text
        %1 = arith.constant 100.0 : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 849

~~~~text
        %2 = fir.undefined tuple<i32, f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 850

~~~~text
        %3 = arith.constant 0 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 851

~~~~text
        %4 = fir.insert_value %2, %0, %3 : (tuple<i32, f32>, i32, index) -> tuple<i32, f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 852

~~~~text
        %5 = arith.constant 1 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 853

~~~~text
        %6 = fir.insert_value %4, %1, %5 : (tuple<i32, f32>, f32, index) -> tuple<i32, f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 854

~~~~text
        fir.has_value %6 : tuple<i32, f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 855

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 856

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 857

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 858

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 859

~~~~text
  let arguments = (ins AnyType:$resval);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 860

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 861

~~~~text
  let assemblyFormat = "$resval attr-dict `:` type($resval)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 862

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 863

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 864

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 865

~~~~text
// Operations on !fir.box<T> type objects
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 866

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 867

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 868

~~~~text
def fir_EmboxOp : fir_Op<"embox", [NoMemoryEffect, AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 869

~~~~text
                                   ConditionallySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 870

~~~~text
                                   fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 871

~~~~text
  let summary = "boxes a given reference and (optional) dimension information";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 872

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 873

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 874

~~~~text
    Create a boxed reference value. In Fortran, the implementation can require
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 875

~~~~text
    extra information about an entity, such as its type, rank, etc.  This
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 876

~~~~text
    auxiliary information is packaged and abstracted as a value with box type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 877

~~~~text
    by the calling routine. (In Fortran, these are called descriptors.)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 878

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 879

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 880

~~~~text
      %c1 = arith.constant 1 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 881

~~~~text
      %c10 = arith.constant 10 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 882

~~~~text
      %5 = ... : !fir.ref<!fir.array<10 x i32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 883

~~~~text
      %6 = fir.embox %5 : (!fir.ref<!fir.array<10 x i32>>) -> !fir.box<!fir.array<10 x i32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 884

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 885

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 886

~~~~text
    The descriptor tuple may contain additional implementation-specific
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 887

~~~~text
    information through the use of additional attributes.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 888

~~~~text
    Specifically,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 889

~~~~text
        - shape: emboxing an array may require shape information (an array's
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 890

~~~~text
          lower bounds and extents may not be known until runtime),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 891

~~~~text
        - slice: an array section can be described with a slice triple,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 892

~~~~text
        - typeparams: for emboxing a derived type with LEN type parameters,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 893

~~~~text
        - sourceBox: A box to read information from such as CFI type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 894

~~~~text
          type descriptor or element size to populate the new descriptor.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 895

~~~~text
        - accessMap: unused/experimental.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 896

~~~~text
        - allocator_idx: specify special allocator to use.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 897

~~~~text
        - optional: indicates that a null pointer `memref` must produce a absent
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 898

~~~~text
          fir.box as opposed to a fir.box containing a nullptr.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 899

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 900

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 901

~~~~text
  let arguments = (ins AnyReferenceLike:$memref, Optional<AnyShapeType>:$shape,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 902

~~~~text
      Optional<fir_SliceType>:$slice, Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 903

~~~~text
      Optional<BoxOrClassType>:$sourceBox,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 904

~~~~text
      OptionalAttr<AffineMapAttr>:$accessMap,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 905

~~~~text
      OptionalAttr<I32Attr>:$allocator_idx,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 906

~~~~text
      UnitAttr:$optional);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 907

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 908

~~~~text
  let results = (outs BoxOrClassType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 909

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 910

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 911

~~~~text
    OpBuilder<(ins "llvm::ArrayRef<mlir::Type>":$resultTypes,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 912

~~~~text
      "mlir::Value":$memref, CArg<"mlir::Value", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 913

~~~~text
      CArg<"mlir::Value", "{}">:$slice,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 914

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 915

~~~~text
      CArg<"mlir::Value", "{}">:$sourceBox,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 916

~~~~text
      CArg<"mlir::IntegerAttr", "{}">:$allocator_idx),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 917

~~~~text
    [{ return build($_builder, $_state, resultTypes, memref, shape, slice,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 918

~~~~text
                    typeparams, sourceBox, mlir::AffineMapAttr{},
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 919

~~~~text
                    allocator_idx, /*optional=*/mlir::UnitAttr{}); }]>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 920

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 921

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 922

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 923

~~~~text
    $memref (`(` $shape^ `)`)? (`[` $slice^ `]`)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 924

~~~~text
      (`source_box` $sourceBox^)? (`map` $accessMap^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 925

~~~~text
      (`optional` $optional^)? attr-dict `:`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 926

~~~~text
      functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 927

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 928

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 929

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 930

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 931

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 932

~~~~text
    bool hasLenParams() { return !getTypeparams().empty(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 933

~~~~text
    unsigned numLenParams() { return getTypeparams().size(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 934

~~~~text
    unsigned getSourceBoxOperandIndex() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 935

~~~~text
      return 1 + (getShape() ? 1 : 0) + (getSlice() ? 1 : 0)
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 936

~~~~text
          + numLenParams();
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 937

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 938

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 939

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getMemref(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 940

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 941

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 942

~~~~text
    // Interface method for ConditionallySpeculatable.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 943

~~~~text
    mlir::Speculation::Speculatability getSpeculatability();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 944

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 945

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 946

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 947

~~~~text
def fir_ReboxOp : fir_Op<"rebox", [NoMemoryEffect, AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 948

~~~~text
                                   ConditionallySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 949

~~~~text
                                   fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 950

~~~~text
  let summary =
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 951

~~~~text
      "create a box given another box and (optional) dimension information";
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 952

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 953

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 954

~~~~text
    Create a new boxed reference value from another box. This is meant to be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 955

~~~~text
    used when the taking a reference to part of a boxed value, or to an entire
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 956

~~~~text
    boxed value with new shape or type information.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 957

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 958

~~~~text
    The new extra information can be:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 959

~~~~text
      - new shape information (new lower bounds, new rank, or new extents.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 960

~~~~text
        New rank/extents can only be provided if the original fir.box is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 961

~~~~text
        contiguous in all dimension but maybe the first row). The shape
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 962

~~~~text
        operand must be provided to set new shape information.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 963

~~~~text
      - new type (only for derived types). It is possible to set the dynamic
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 964

~~~~text
        type of the new box to one of the parent types of the input box dynamic
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 965

~~~~text
        type. Type parameters cannot be changed. This change is reflected in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 966

~~~~text
        the requested result type of the new box.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 967

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 968

~~~~text
    A slice argument can be provided to build a reference to part of a boxed
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 969

~~~~text
    value. In this case, the shape operand must be absent or be a fir.shift
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 970

~~~~text
    that can be used to provide a non default origin for the slice.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 971

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 972

~~~~text
    The following example illustrates creating a fir.box for x(10:33:2)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 973

~~~~text
    where x is described by a fir.box and has non default lower bounds,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 974

~~~~text
    and then applying a new 2-dimension shape to this fir.box.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 975

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 976

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 977

~~~~text
      %0 = fir.slice %c10, %c33, %c2 : (index, index, index) -> !fir.slice<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 978

~~~~text
      %1 = fir.shift %c0 : (index) -> !fir.shift<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 979

~~~~text
      %2 = fir.rebox %x(%1) [%0] : (!fir.box<!fir.array<?xf32>>, !fir.shift<1>, !fir.slice<1>) -> !fir.box<!fir.array<?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 980

~~~~text
      %3 = fir.shape %c3, %c4 : (index, index) -> !fir.shape<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 981

~~~~text
      %4 = fir.rebox %2(%3) : (!fir.box<!fir.array<?xf32>>, !fir.shape<2>) -> !fir.box<!fir.array<?x?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 982

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 983

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 984

~~~~text
    When the `optional` unit attribute is set, `$box` may be an absent
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 985

~~~~text
    (null-descriptor) OPTIONAL. In such case, the produced fir.box will be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 986

~~~~text
    absent when the input one is absent. Without this flag, it is illegal
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 987

~~~~text
    to execute a fir.rebox where the input fir.box is absent.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 988

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 989

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 990

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 991

~~~~text
    BoxOrClassType:$box,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 992

~~~~text
    Optional<AnyShapeOrShiftType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 993

~~~~text
    Optional<fir_SliceType>:$slice,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 994

~~~~text
    UnitAttr:$optional
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 995

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 996

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 997

~~~~text
  let results = (outs BoxOrClassType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 998

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 999

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1000

~~~~text
    $box (`(` $shape^ `)`)? (`[` $slice^ `]`)? (`optional` $optional^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1001

~~~~text
        attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1002

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1003

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1004

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1005

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1006

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1007

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1008

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getBox(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1009

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1010

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1011

~~~~text
    // Interface method for ConditionallySpeculatable.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1012

~~~~text
    mlir::Speculation::Speculatability getSpeculatability();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1013

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1014

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1015

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1016

~~~~text
def fir_ReboxAssumedRankOp : fir_Op<"rebox_assumed_rank",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1017

~~~~text
  [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1018

~~~~text
  let summary = "create an assumed-rank box given another assumed-rank box";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1019

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1020

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1021

~~~~text
    Limited version of fir.rebox for assumed-rank. Only the lower bounds,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1022

~~~~text
    attribute, and element type may change.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1023

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1024

~~~~text
    The input may be a box or a reference to a box, in which case the operation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1025

~~~~text
    reads the incoming reference.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1026

~~~~text
    Since a fir.shift cannot be built without knowing the rank statically,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1027

~~~~text
    lower bound changes are encoded via a LowerBoundModifierAttribute.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1028

~~~~text
    Attribute and element type change are encoded in the result type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1029

~~~~text
    Changing the element type is only allowed if the input type is a derived
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1030

~~~~text
    type that extends the output element type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1031

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1032

~~~~text
    Example:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1033

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1034

~~~~text
      fir.rebox_assumed_rank %1 lbs zeroes : (!fir.box<!fir.array<*:f32>>) -> !fir.box<!fir.array<*:f32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1035

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1036

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1037

~~~~text
    When the `optional` unit attribute is set, `$box` may be an absent
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1038

~~~~text
    (null-descriptor) OPTIONAL. In such case, the produced fir.box will be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1039

~~~~text
    absent when the input one is absent. Without this flag, it is illegal
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1040

~~~~text
    to execute a fir.rebox where the input fir.box is absent.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1041

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1042

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1043

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1044

~~~~text
    AnyRefOrBoxType:$box,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1045

~~~~text
    fir_LowerBoundModifierAttribute:$lbs_modifier,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1046

~~~~text
    UnitAttr:$optional
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1047

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1048

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1049

~~~~text
  let results = (outs BoxOrClassType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1050

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1051

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1052

~~~~text
    $box `lbs` $lbs_modifier (`optional` $optional^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1053

~~~~text
        attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1054

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1055

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1056

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1057

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1058

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1059

~~~~text
def fir_EmboxCharOp : fir_Op<"emboxchar", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1060

~~~~text
  let summary = "boxes a given CHARACTER reference and its LEN parameter";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1061

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1062

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1063

~~~~text
    Create a boxed CHARACTER value. The CHARACTER type has the LEN type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1064

~~~~text
    parameter, the value of which may only be known at runtime.  Therefore,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1065

~~~~text
    a variable of type CHARACTER has both its data reference as well as a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1066

~~~~text
    LEN type parameter.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1067

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1068

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1069

~~~~text
      CHARACTER(LEN=10) :: var
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1070

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1071

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1072

~~~~text
      %4 = ...         : !fir.ref<!fir.array<10 x !fir.char<1>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1073

~~~~text
      %5 = arith.constant 10 : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1074

~~~~text
      %6 = fir.emboxchar %4, %5 : (!fir.ref<!fir.array<10 x !fir.char<1>>>, i32) -> !fir.boxchar<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1075

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1076

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1077

~~~~text
    In the above `%4` is a memory reference to a buffer of 10 CHARACTER units.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1078

~~~~text
    This buffer and its LEN value (10) are wrapped into a pair in `%6`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1079

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1080

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1081

~~~~text
  let arguments = (ins AnyReferenceLike:$memref, AnyIntegerLike:$len);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1082

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1083

~~~~text
  let results = (outs fir_BoxCharType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1084

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1085

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1086

~~~~text
    $memref `,` $len attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1087

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1088

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1089

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1090

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1091

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1092

~~~~text
def fir_EmboxProcOp : fir_Op<"emboxproc", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1093

~~~~text
  let summary = "boxes a given procedure and optional host context";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1094

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1095

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1096

~~~~text
    Creates an abstract encapsulation of a PROCEDURE POINTER along with an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1097

~~~~text
    optional pointer to a host instance context. If the pointer is not to an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1098

~~~~text
    internal procedure or the internal procedure does not need a host context
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1099

~~~~text
    then the form takes only the procedure's symbol.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1100

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1101

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1102

~~~~text
      %f = ... : (i32) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1103

~~~~text
      %0 = fir.emboxproc %f : ((i32) -> i32) -> !fir.boxproc<(i32) -> i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1104

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1105

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1106

~~~~text
    An internal procedure requiring a host instance for correct execution uses
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1107

~~~~text
    the second form. The closure of the host procedure's state is passed as a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1108

~~~~text
    reference to a tuple. It is the responsibility of the host to manage the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1109

~~~~text
    context's values accordingly, up to and including inhibiting register
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1110

~~~~text
    promotion of local values.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1111

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1112

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1113

~~~~text
      %4 = ... : !fir.ref<tuple<!fir.ref<i32>, !fir.ref<i32>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1114

~~~~text
      %g = ... : (i32) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1115

~~~~text
      %5 = fir.emboxproc %g, %4 : ((i32) -> i32, !fir.ref<tuple<!fir.ref<i32>, !fir.ref<i32>>>) -> !fir.boxproc<(i32) -> i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1116

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1117

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1118

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1119

~~~~text
  let arguments = (ins FuncType:$func, Optional<fir_ReferenceType>:$host);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1120

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1121

~~~~text
  let results = (outs fir_BoxProcType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1122

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1123

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1124

~~~~text
    $func (`,` $host^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1125

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1126

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1127

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1128

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1129

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1130

~~~~text
def fir_UnboxCharOp : fir_SimpleOp<"unboxchar", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1131

~~~~text
  let summary = "unbox a boxchar value into a pair value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1132

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1133

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1134

~~~~text
    Unboxes a value of `boxchar` type into a pair consisting of a memory
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1135

~~~~text
    reference to the CHARACTER data and the LEN type parameter.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1136

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1137

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1138

~~~~text
      %45   = ... : !fir.boxchar<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1139

~~~~text
      %46:2 = fir.unboxchar %45 : (!fir.boxchar<1>) -> (!fir.ref<!fir.char<1>>, i32)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1140

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1141

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1142

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1143

~~~~text
  let arguments = (ins fir_BoxCharType:$boxchar);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1144

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1145

~~~~text
  let results = (outs fir_ReferenceType, AnyIntegerLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1146

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1147

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1148

~~~~text
def fir_UnboxProcOp : fir_SimpleOp<"unboxproc", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1149

~~~~text
  let summary = "unbox a boxproc value into a pair value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1150

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1151

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1152

~~~~text
    Unboxes a value of `boxproc` type into a pair consisting of a procedure
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1153

~~~~text
    pointer and a pointer to a host context.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1154

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1155

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1156

~~~~text
      %47   = ... : !fir.boxproc<() -> i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1157

~~~~text
      %48:2 = fir.unboxproc %47 : (!fir.ref<() -> i32>, !fir.ref<tuple<f32, i32>>)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1158

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1159

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1160

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1161

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1162

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1163

~~~~text
  let arguments = (ins fir_BoxProcType:$boxproc);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1164

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1165

~~~~text
  let results = (outs FunctionType, fir_ReferenceType:$refTuple);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1166

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1167

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1168

~~~~text
def fir_BoxAddrOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1169

~~~~text
    : fir_SimpleOneResultOp<"box_addr", [NoMemoryEffect,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1170

~~~~text
                                         ConditionallySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1171

~~~~text
                                         fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1172

~~~~text
  let summary = "return a memory reference to the boxed value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1173

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1174

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1175

~~~~text
    This operator is overloaded to work with values of type `box`,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1176

~~~~text
    `boxchar`, and `boxproc`.  The result for each of these
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1177

~~~~text
    cases, respectively, is the address of the data, the address of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1178

~~~~text
    `CHARACTER` data, and the address of the procedure.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1179

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1180

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1181

~~~~text
      %51 = fir.box_addr %box : (!fir.box<f64>) -> !fir.ref<f64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1182

~~~~text
      %52 = fir.box_addr %boxchar : (!fir.boxchar<1>) -> !fir.ref<!fir.char<1>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1183

~~~~text
      %53 = fir.box_addr %boxproc : (!fir.boxproc<!P>) -> !P
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1184

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1185

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1186

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1187

~~~~text
  let arguments = (ins AnyBoxLike:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1188

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1189

~~~~text
  let results = (outs AnyCodeOrDataRefLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1190

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1191

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1192

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1193

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$val)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1194

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1195

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1196

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1197

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getVal(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1198

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1199

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1200

~~~~text
    // Interface method for ConditionallySpeculatable.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1201

~~~~text
    mlir::Speculation::Speculatability getSpeculatability();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1202

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1203

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1204

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1205

~~~~text
def fir_BoxCharLenOp : fir_SimpleOp<"boxchar_len", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1206

~~~~text
  let summary = "return the LEN type parameter from a boxchar value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1207

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1208

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1209

~~~~text
    Extracts the LEN type parameter from a `boxchar` value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1210

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1211

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1212

~~~~text
      %45 = ... : !boxchar<1>  // CHARACTER(20)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1213

~~~~text
      %59 = fir.boxchar_len %45 : (!fir.boxchar<1>) -> i64  // len=20
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1214

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1215

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1216

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1217

~~~~text
  let arguments = (ins fir_BoxCharType:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1218

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1219

~~~~text
  let results = (outs AnyIntegerLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1220

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1221

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1222

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1223

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1224

~~~~text
def fir_BoxDimsOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1225

~~~~text
    : fir_Op<"box_dims", [NoMemoryEffect, ConditionallySpeculatable]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1226

~~~~text
  let summary = "return the dynamic dimension information for the boxed value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1227

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1228

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1229

~~~~text
    Returns the triple of lower bound, extent, and stride for `dim` dimension
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1230

~~~~text
    of `val`, which must have a `box` type.  The dimensions are enumerated from
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1231

~~~~text
    left to right from 0 to rank-1. This operation has undefined behavior if
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1232

~~~~text
    `dim` is out of bounds.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1233

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1234

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1235

~~~~text
      %c1   = arith.constant 0 : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1236

~~~~text
      %52:3 = fir.box_dims %40, %c1 : (!fir.box<!fir.array<*:f64>>, i32) -> (index, index, index)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1237

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1238

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1239

~~~~text
    The above is a request to return the left most row (at index 0) triple from
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1240

~~~~text
    the box. The triple will be the lower bound, extent, and byte-stride, which
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1241

~~~~text
    are the values encoded in a standard descriptor.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1242

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1243

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1244

~~~~text
  let arguments = (ins BoxOrClassType:$val, AnyIntegerLike:$dim);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1245

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1246

~~~~text
  let results = (outs Index, Index, Index);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1247

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1248

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1249

~~~~text
    $val `,` $dim attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1250

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1251

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1252

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1253

~~~~text
    mlir::Type getTupleType();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1254

~~~~text
    mlir::Value getLowerBound() {return getResult(0);};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1255

~~~~text
    mlir::Value getExtent() {return getResult(1);};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1256

~~~~text
    mlir::Value getByteStride() {return getResult(2);};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1257

~~~~text
    // Interface method for ConditionallySpeculatable.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1258

~~~~text
    mlir::Speculation::Speculatability getSpeculatability();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1259

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1260

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1261

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1262

~~~~text
def fir_BoxEleSizeOp : fir_SimpleOneResultOp<"box_elesize", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1263

~~~~text
  let summary = "return the size of an element of the boxed value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1264

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1265

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1266

~~~~text
    Returns the size of an element in an entity of `box` type.  This size may
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1267

~~~~text
    not be known until runtime.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1268

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1269

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1270

~~~~text
      %53 = fir.box_elesize %40 : (!fir.box<f32>) -> i32  // size=4
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1271

~~~~text
      %54 = fir.box_elesize %40 : (!fir.box<!fir.array<*:f32>>) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1272

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1273

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1274

~~~~text
    In the above example, `%53` may box an array of REAL values while `%54`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1275

~~~~text
    must box an array of REAL values (with dynamic rank and extent).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1276

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1277

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1278

~~~~text
  let arguments = (ins BoxOrClassType:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1279

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1280

~~~~text
  let results = (outs AnyIntegerLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1281

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1282

~~~~text
  let hasCanonicalizer = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1283

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1284

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1285

~~~~text
def fir_BoxTypeCodeOp : fir_SimpleOneResultOp<"box_typecode", [NoMemoryEffect]>
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1286

~~~~text
{
~~~~
- EN: Opens a new scope block.
- CN: 打开一个新的作用域块。

### Line 1287

~~~~text
  let summary = "return the type code the boxed value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1288

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1289

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1290

~~~~text
    Returns the descriptor type code of an entity of `box` type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1291

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1292

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1293

~~~~text
      %1 = fir.box_typecode %0 : (!fir.box<T>) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1294

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1295

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1296

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1297

~~~~text
  let arguments = (ins BoxOrClassType:$box);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1298

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1299

~~~~text
  let results = (outs AnyIntegerLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1300

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1301

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1302

~~~~text
def fir_BoxIsAllocOp : fir_SimpleOp<"box_isalloc", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1303

~~~~text
  let summary = "is the boxed value an ALLOCATABLE?";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1304

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1305

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1306

~~~~text
    Determine if the boxed value was from an ALLOCATABLE entity. This will
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1307

~~~~text
    return true if the originating box value was from a `fir.embox` op
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1308

~~~~text
    with a mem-ref value that had the type !fir.heap<T>.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1309

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1310

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1311

~~~~text
      %r = ... : !fir.heap<i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1312

~~~~text
      %b = fir.embox %r : (!fir.heap<i64>) -> !fir.box<i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1313

~~~~text
      %a = fir.box_isalloc %b : (!fir.box<i64>) -> i1  // true
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1314

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1315

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1316

~~~~text
    The canonical descriptor implementation will carry a flag to record if the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1317

~~~~text
    variable is an `ALLOCATABLE`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1318

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1319

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1320

~~~~text
  let arguments = (ins fir_BoxType:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1321

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1322

~~~~text
  let results = (outs BoolLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1323

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1324

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1325

~~~~text
def fir_BoxIsArrayOp : fir_SimpleOp<"box_isarray", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1326

~~~~text
  let summary = "is the boxed value an array?";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1327

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1328

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1329

~~~~text
    Determine if the boxed value has a positive (> 0) rank. This will return
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1330

~~~~text
    true if the originating box value was from a fir.embox with a memory
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1331

~~~~text
    reference value that had the type !fir.array<T> and/or a shape argument.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1332

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1333

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1334

~~~~text
      %r = ... : !fir.ref<i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1335

~~~~text
      %c_100 = arith.constant 100 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1336

~~~~text
      %d = fir.shape %c_100 : (index) -> !fir.shape<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1337

~~~~text
      %b = fir.embox %r(%d) : (!fir.ref<i64>, !fir.shape<1>) -> !fir.box<i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1338

~~~~text
      %a = fir.box_isarray %b : (!fir.box<i64>) -> i1  // true
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1339

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1340

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1341

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1342

~~~~text
  let arguments = (ins fir_BoxType:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1343

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1344

~~~~text
  let results = (outs BoolLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1345

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1346

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1347

~~~~text
def fir_IsAssumedSizeOp : fir_SimpleOp<"is_assumed_size", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1348

~~~~text
  let summary = "detect if a boxed value is an assumed-size array";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1349

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1350

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1351

~~~~text
    Fir box SSA values may describe assumed-size arrays. This operation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1352

~~~~text
    allows detecting this, even for assumed-rank box.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1353

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1354

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1355

~~~~text
      %a = fir.is_assumed_size %b : (!fir.box<!fir.array<*:f64>>) -> i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1356

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1357

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1358

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1359

~~~~text
  let arguments = (ins BoxOrClassType:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1360

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1361

~~~~text
  let results = (outs BoolLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1362

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1363

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1364

~~~~text
def fir_AssumedSizeExtentOp : fir_SimpleOneResultOp<"assumed_size_extent", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1365

~~~~text
  let summary = "get the assumed-size last extent sentinel";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1366

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1367

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1368

~~~~text
    Returns the special extent value representing the last dimension of an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1369

~~~~text
    assumed-size array. This is used to model the semantics in FIR without
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1370

~~~~text
    directly materializing the sentinel value. The concrete encoding is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1371

~~~~text
    introduced during FIR to LLVM lowering.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1372

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1373

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1374

~~~~text
      %e = fir.assumed_size_extent : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1375

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1376

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1377

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1378

~~~~text
  let results = (outs Index);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1379

~~~~text
  let assemblyFormat = "attr-dict `:` type(results)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1380

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1381

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1382

~~~~text
def fir_IsAssumedSizeExtentOp : fir_SimpleOp<"is_assumed_size_extent", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1383

~~~~text
  let summary = "is value the assumed-size last extent sentinel";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1384

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1385

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1386

~~~~text
    Returns true iff the given integer equals the assumed-size extent sentinel.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1387

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1388

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1389

~~~~text
      %t = fir.is_assumed_size_extent %v : (index) -> i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1390

~~~~text
      %c = fir.is_assumed_size_extent %x : (i32) -> i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1391

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1392

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1393

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1394

~~~~text
  let arguments = (ins AnyIntegerLike:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1395

~~~~text
  let results = (outs BoolLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1396

~~~~text
  let hasCanonicalizer = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1397

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1398

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1399

~~~~text
def fir_BoxIsPtrOp : fir_SimpleOp<"box_isptr", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1400

~~~~text
  let summary = "is the boxed value a POINTER?";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1401

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1402

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1403

~~~~text
    Determine if the boxed value was from a POINTER entity.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1404

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1405

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1406

~~~~text
      %p = ... : !fir.ptr<i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1407

~~~~text
      %b = fir.embox %p : (!fir.ptr<i64>) -> !fir.box<i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1408

~~~~text
      %a = fir.box_isptr %b : (!fir.box<i64>) -> i1  // true
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1409

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1410

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1411

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1412

~~~~text
  let arguments = (ins fir_BoxType:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1413

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1414

~~~~text
  let results = (outs BoolLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1415

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1416

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1417

~~~~text
def fir_BoxProcHostOp : fir_SimpleOp<"boxproc_host", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1418

~~~~text
  let summary = "returns the host instance pointer (or null)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1419

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1420

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1421

~~~~text
    Extract the host context pointer from a boxproc value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1422

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1423

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1424

~~~~text
      %8 = ... : !fir.boxproc<(!fir.ref<!fir.type<T>>) -> i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1425

~~~~text
      %9 = fir.boxproc_host %8 : (!fir.boxproc<(!fir.ref<!fir.type<T>>) -> i32>) -> !fir.ref<tuple<i32, i32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1426

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1427

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1428

~~~~text
    In the example, the reference to the closure over the host procedure's
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1429

~~~~text
    variables is returned. This allows an internal procedure to access the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1430

~~~~text
    host's variables. It is up to lowering to determine the contract between
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1431

~~~~text
    the host and the internal procedure.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1432

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1433

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1434

~~~~text
  let arguments = (ins fir_BoxProcType:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1435

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1436

~~~~text
  let results = (outs fir_ReferenceType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1437

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1438

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1439

~~~~text
def fir_BoxRankOp : fir_SimpleOneResultOp<"box_rank",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1440

~~~~text
  [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1441

~~~~text
  let summary = "return the number of dimensions for the boxed value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1442

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1443

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1444

~~~~text
    Return the rank of a value of `box` type.  If the value is scalar, the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1445

~~~~text
    rank is 0.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1446

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1447

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1448

~~~~text
      %57 = fir.box_rank %40 : (!fir.box<!fir.array<*:f64>>) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1449

~~~~text
      %58 = fir.box_rank %41 : (!fir.box<f64>) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1450

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1451

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1452

~~~~text
    The example `%57` shows how one would determine the rank of an array that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1453

~~~~text
    has deferred rank at runtime. This rank should be at least 1. In %58, the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1454

~~~~text
    descriptor may be either an array or a scalar, so the value is nonnegative.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1455

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1456

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1457

~~~~text
  let arguments = (ins fir_BoxAddressOrValueType:$box);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1458

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1459

~~~~text
  let results = (outs AnyIntegerType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1460

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1461

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1462

~~~~text
def fir_BoxTypeDescOp : fir_SimpleOneResultOp<"box_tdesc", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1463

~~~~text
  let summary = "return the type descriptor for the boxed value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1464

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1465

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1466

~~~~text
    Return the opaque type descriptor of a value of `box` type. A type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1467

~~~~text
    descriptor is an implementation defined value that fully describes a type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1468

~~~~text
    to the Fortran runtime.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1469

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1470

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1471

~~~~text
      %7 = fir.box_tdesc %41 : (!fir.box<f64>) -> !fir.tdesc<f64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1472

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1473

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1474

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1475

~~~~text
  let arguments = (ins BoxOrClassType:$box);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1476

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1477

~~~~text
  let results = (outs fir_TypeDescType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1478

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1479

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1480

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 1481

~~~~text
// Array value operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1482

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 1483

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1484

~~~~text
// Array value operations are used to capture the semantics of
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1485

~~~~text
// Fortran's array expressions in FIR. An abstract array expression is
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1486

~~~~text
// evaluated in the following way.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1487

~~~~text
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 1488

~~~~text
//  1. Determination of the iteration space under which the assignment
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1489

~~~~text
//     expression is to be evaluated. The iteration space may be implicit
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1490

~~~~text
//     (from the shape of the result array) or explicit (defined by the user).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1491

~~~~text
//  2. If there are masking expressions, evaluate (and cache) the
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1492

~~~~text
//     masking expression for the iteration space (from 1).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1493

~~~~text
//  3. The rhs of the assignment is evaluated for the iteration space. If
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1494

~~~~text
//     masking expressions were present then the rhs is only evaluated where
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1495

~~~~text
//     the mask was computed to be true. The entire rhs is completely evaluated
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1496

~~~~text
//     before any results are stored to the lhs.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1497

~~~~text
//  4. Each of the result values computed in the previous step are merged back
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1498

~~~~text
//     to the lhs array's storage.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1499

~~~~text
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 1500

~~~~text
// The model (in pseudo-code) is thus:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1501

~~~~text
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 1502

~~~~text
//   !- Load the arrays in the expression
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1503

~~~~text
//   %10 = array_load A
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1504

~~~~text
//   %11 = array_load B
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1505

~~~~text
//   !- optional: compute mask values
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1506

~~~~text
//   %masks = allocmem array<??xlogical>
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1507

~~~~text
//   do_loop_nest %i = ... {
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1508

~~~~text
//     %masks[i] = ...
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1509

~~~~text
//   }
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1510

~~~~text
//   !- Compute every element value "A = B ..."
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1511

~~~~text
//   do_loop_nest %i = ... {
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1512

~~~~text
//     if (%masks[i]) {
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1513

~~~~text
//       array_fetch %11, ...       !- B(...)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1514

~~~~text
//       %20 = ...                  !- element-by-element computation
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1515

~~~~text
//       array_update %10, %20, ... !- A(...) = ...
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1516

~~~~text
//     }
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1517

~~~~text
//   }
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1518

~~~~text
//   !- Merge the new and old values into the memory for "A"
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1519

~~~~text
//   array_merge_store <updated A> to <A's address>
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1520

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1521

~~~~text
def fir_ArrayLoadOp : fir_Op<"array_load", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1522

~~~~text
                                            DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1523

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1524

~~~~text
  let summary = "Load an array as a value.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1525

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1526

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1527

~~~~text
    This operation taken with array_merge_store captures Fortran's
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1528

~~~~text
    copy-in/copy-out semantics. One way to think of this is that array_load
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1529

~~~~text
    creates a snapshot copy of the entire array. This copy can then be used
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1530

~~~~text
    as the "original value" of the array while the array's new value is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1531

~~~~text
    computed. The array_merge_store operation is the copy-out semantics, which
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1532

~~~~text
    merge the updates with the original array value to produce the final array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1533

~~~~text
    result. This abstracts the copy operations as opposed to always creating
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1534

~~~~text
    copies or requiring dependence analysis be performed on the syntax trees
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1535

~~~~text
    and before lowering to the IR.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1536

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1537

~~~~text
    Load an entire array as a single SSA value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1538

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1539

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1540

~~~~text
      real :: a(o:n,p:m)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1541

~~~~text
      ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1542

~~~~text
      ... = ... a ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1543

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1544

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1545

~~~~text
    One can use `fir.array_load` to produce an ssa-value that captures an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1546

~~~~text
    immutable value of the entire array `a`, as in the Fortran array expression
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1547

~~~~text
    shown above. Subsequent changes to the memory containing the array do not
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1548

~~~~text
    alter its composite value. This operation lets one load an array as a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1549

~~~~text
    value while applying a runtime shape, shift, or slice to the memory
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1550

~~~~text
    reference, and its semantics guarantee immutability.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1551

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1552

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1553

~~~~text
      %s = fir.shape_shift %o, %n, %p, %m : (index, index, index, index) -> !fir.shapeshift<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1554

~~~~text
      // load the entire array 'a'
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1555

~~~~text
      %v = fir.array_load %a(%s) : (!fir.ref<!fir.array<?x?xf32>>, !fir.shapeshift<2>) -> !fir.array<?x?xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1556

~~~~text
      // a fir.store here into array %a does not change %v
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1557

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1558

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1559

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1560

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1561

~~~~text
    AnyRefOrBox:$memref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1562

~~~~text
    Optional<AnyShapeOrShiftType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1563

~~~~text
    Optional<fir_SliceType>:$slice,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1564

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1565

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1566

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1567

~~~~text
  let results = (outs fir_SequenceType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1568

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1569

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1570

~~~~text
    $memref (`(`$shape^`)`)? (`[`$slice^`]`)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1571

~~~~text
        attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1572

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1573

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1574

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1575

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1576

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1577

~~~~text
    std::vector<mlir::Value> getExtents();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1578

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1579

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1580

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1581

~~~~text
def fir_ArrayFetchOp : fir_Op<"array_fetch", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1582

~~~~text
    NoMemoryEffect]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1583

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1584

~~~~text
  let summary = "Fetch the value of an element of an array value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1585

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1586

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1587

~~~~text
    Fetch the value of an element in an array value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1588

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1589

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1590

~~~~text
      real :: a(n,m)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1591

~~~~text
      ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1592

~~~~text
      ... a ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1593

~~~~text
      ... a(r,s+1) ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1594

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1595

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1596

~~~~text
    One can use `fir.array_fetch` to fetch the (implied) value of `a(i,j)` in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1597

~~~~text
    an array expression as shown above. It can also be used to extract the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1598

~~~~text
    element `a(r,s+1)` in the second expression.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1599

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1600

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1601

~~~~text
      %s = fir.shape %n, %m : (index, index) -> !fir.shape<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1602

~~~~text
      // load the entire array 'a'
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1603

~~~~text
      %v = fir.array_load %a(%s) : (!fir.ref<!fir.array<?x?xf32>>, !fir.shape<2>) -> !fir.array<?x?xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1604

~~~~text
      // fetch the value of one of the array value's elements
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1605

~~~~text
      %1 = fir.array_fetch %v, %i, %j : (!fir.array<?x?xf32>, index, index) -> f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1606

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1607

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1608

~~~~text
    It is only possible to use `array_fetch` on an `array_load` result value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1609

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1610

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1611

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1612

~~~~text
    fir_SequenceType:$sequence,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1613

~~~~text
    Variadic<AnyCoordinateType>:$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1614

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1615

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1616

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1617

~~~~text
  let results = (outs AnyType:$element);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1618

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1619

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1620

~~~~text
    $sequence `,` $indices (`typeparams` $typeparams^)? attr-dict `:`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1621

~~~~text
      functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1622

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1623

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1624

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1625

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1626

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1627

~~~~text
def fir_ArrayUpdateOp : fir_Op<"array_update", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1628

~~~~text
    NoMemoryEffect]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1629

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1630

~~~~text
  let summary = "Update the value of an element of an array value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1631

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1632

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1633

~~~~text
    Updates the value of an element in an array value. A new array value is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1634

~~~~text
    returned where all element values of the input array are identical except
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1635

~~~~text
    for the selected element which is the value passed in the update.
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 1636

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1637

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1638

~~~~text
      real :: a(n,m)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1639

~~~~text
      ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1640

~~~~text
      a = ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1641

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1642

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1643

~~~~text
    One can use `fir.array_update` to update the (implied) value of `a(i,j)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1644

~~~~text
    in an array expression as shown above.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1645

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1646

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1647

~~~~text
      %s = fir.shape %n, %m : (index, index) -> !fir.shape<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1648

~~~~text
      // load the entire array 'a'
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1649

~~~~text
      %v = fir.array_load %a(%s) : (!fir.ref<!fir.array<?x?xf32>>, !fir.shape<2>) -> !fir.array<?x?xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1650

~~~~text
      // update the value of one of the array value's elements
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1651

~~~~text
      // %r_{ij} = %f  if (i,j) = (%i,%j),   %v_{ij} otherwise
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1652

~~~~text
      %r = fir.array_update %v, %f, %i, %j : (!fir.array<?x?xf32>, f32, index, index) -> !fir.array<?x?xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1653

~~~~text
      fir.array_merge_store %v, %r to %a : !fir.ref<!fir.array<?x?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1654

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1655

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1656

~~~~text
    An array value update behaves as if a mapping function from the indices
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1657

~~~~text
    to the new value has been added, replacing the previous mapping. These
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1658

~~~~text
    mappings can be added to the ssa-value, but will not be materialized in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1659

~~~~text
    memory until the `fir.array_merge_store` is performed.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1660

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1661

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1662

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1663

~~~~text
    fir_SequenceType:$sequence,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1664

~~~~text
    AnyType:$merge,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1665

~~~~text
    Variadic<AnyCoordinateType>:$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1666

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1667

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1668

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1669

~~~~text
  let results = (outs fir_SequenceType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1670

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1671

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1672

~~~~text
    $sequence `,` $merge `,` $indices (`typeparams` $typeparams^)? attr-dict
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1673

~~~~text
      `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1674

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1675

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1676

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1677

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1678

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1679

~~~~text
def fir_ArrayModifyOp : fir_Op<"array_modify", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1680

~~~~text
    NoMemoryEffect]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1681

~~~~text
  let summary = "Get an address for an array value to modify it.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1682

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1683

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1684

~~~~text
    Modify the value of an element in an array value through actions done
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1685

~~~~text
    on the returned address. A new array value is also
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1686

~~~~text
    returned where all element values of the input array are identical except
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1687

~~~~text
    for the selected element which is the value after the modification done
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 1688

~~~~text
    on the element address.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1689

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1690

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1691

~~~~text
      real :: a(n)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1692

~~~~text
      ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1693

~~~~text
      ! Elemental user defined assignment from type(SomeType) to real.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1694

~~~~text
      a = value_of_some_type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1695

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1696

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1697

~~~~text
    One can use `fir.array_modify` to update the (implied) value of `a(i)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1698

~~~~text
    in an array expression as shown above.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1699

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1700

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1701

~~~~text
      %s = fir.shape %n : (index) -> !fir.shape<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1702

~~~~text
      // Load the entire array 'a'.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1703

~~~~text
      %v = fir.array_load %a(%s) : (!fir.ref<!fir.array<?xf32>>, !fir.shape<1>) -> !fir.array<?xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1704

~~~~text
      // Update the value of one of the array value's elements with a user
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1705

~~~~text
      // defined assignment from %rhs.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1706

~~~~text
      %new = fir.do_loop %i = ... (%inner = %v) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1707

~~~~text
        %rhs = ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1708

~~~~text
        %addr, %r = fir.array_modify %inner, %i : (!fir.array<?xf32>, index) -> (fir.ref<f32>, !fir.array<?xf32>)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1709

~~~~text
        fir.call @user_def_assign(%addr, %rhs) (fir.ref<f32>, fir.ref<!fir.type<SomeType>>) -> ()
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1710

~~~~text
        fir.result %r : !fir.ref<!fir.array<?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1711

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1712

~~~~text
      fir.array_merge_store %v, %new to %a : !fir.ref<!fir.array<?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1713

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1714

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1715

~~~~text
    An array value modification behaves as if a mapping function from the indices
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1716

~~~~text
    to the new value has been added, replacing the previous mapping. These
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1717

~~~~text
    mappings can be added to the ssa-value, but will not be materialized in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1718

~~~~text
    memory until the `fir.array_merge_store` is performed.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1719

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1720

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1721

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1722

~~~~text
    fir_SequenceType:$sequence,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1723

~~~~text
    Variadic<AnyCoordinateType>:$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1724

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1725

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1726

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1727

~~~~text
  let results = (outs fir_ReferenceType, fir_SequenceType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1728

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1729

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1730

~~~~text
    $sequence `,` $indices (`typeparams` $typeparams^)? attr-dict
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1731

~~~~text
      `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1732

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1733

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1734

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1735

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1736

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1737

~~~~text
def fir_ArrayAccessOp : fir_Op<"array_access", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1738

~~~~text
    NoMemoryEffect]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1739

~~~~text
  let summary = "Fetch the reference of an element of an array value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1740

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1741

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1742

~~~~text
    The `array_access` provides a reference to a single element from an array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1743

~~~~text
    value. This is *not* a view in the immutable array, otherwise it couldn't
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1744

~~~~text
    be stored to. It can be see as a logical copy of the element and its
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1745

~~~~text
    position in the array. This reference can be written to and modified without
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1746

~~~~text
    changing the original array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1747

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1748

~~~~text
    The `array_access` operation is used to fetch the memory reference of an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1749

~~~~text
    element in an array value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1750

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1751

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1752

~~~~text
      real :: a(n,m)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1753

~~~~text
      ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1754

~~~~text
      ... a ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1755

~~~~text
      ... a(r,s+1) ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1756

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1757

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1758

~~~~text
    One can use `fir.array_access` to recover the implied memory reference to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1759

~~~~text
    the element `a(i,j)` in an array expression `a` as shown above. It can also
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1760

~~~~text
    be used to recover the reference element `a(r,s+1)` in the second
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1761

~~~~text
    expression.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1762

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1763

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1764

~~~~text
      %s = fir.shape %n, %m : (index, index) -> !fir.shape<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1765

~~~~text
      // load the entire array 'a'
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1766

~~~~text
      %v = fir.array_load %a(%s) : (!fir.ref<!fir.array<?x?xf32>>, !fir.shape<2>) -> !fir.array<?x?xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1767

~~~~text
      // fetch the value of one of the array value's elements
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1768

~~~~text
      %1 = fir.array_access %v, %i, %j : (!fir.array<?x?xf32>, index, index) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1769

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1770

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1771

~~~~text
    It is only possible to use `array_access` on an `array_load` result value or
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1772

~~~~text
    a value that can be trace back transitively to an `array_load` as the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1773

~~~~text
    dominating source. Other array operations such as `array_amend` can be in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1774

~~~~text
    between.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1775

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1776

~~~~text
    TODO: The above restriction is not enforced. The design of the operation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1777

~~~~text
    might need to be revisited to avoid such restrictions.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1778

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1779

~~~~text
    More information about `array_access` and other array operations can be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1780

~~~~text
    found in flang/docs/FIRArrayOperations.md.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1781

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1782

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1783

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1784

~~~~text
    fir_SequenceType:$sequence,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1785

~~~~text
    Variadic<AnyCoordinateType>:$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1786

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1787

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1788

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1789

~~~~text
  let results = (outs fir_ReferenceType:$element);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1790

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1791

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1792

~~~~text
    $sequence `,` $indices (`typeparams` $typeparams^)? attr-dict `:`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1793

~~~~text
      functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1794

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1795

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1796

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1797

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1798

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1799

~~~~text
def fir_ArrayAmendOp : fir_Op<"array_amend", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1800

~~~~text
  let summary = "Mark an array value as having been changed by reference.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1801

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1802

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1803

~~~~text
    The `array_amend` operation marks an array value as having been changed via
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1804

~~~~text
    a reference obtained by an `array_access`. It acts as a logical transaction
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1805

~~~~text
    log that is used to merge the final result back with an `array_merge_store`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1806

~~~~text
    operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1807

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1808

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1809

~~~~text
      // fetch the value of one of the array value's elements
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1810

~~~~text
      %1 = fir.array_access %v, %i, %j : (!fir.array<?x?xT>, index, index) -> !fir.ref<T>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1811

~~~~text
      // modify the element by storing data using %1 as a reference
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1812

~~~~text
      %2 = ... %1 ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1813

~~~~text
      // mark the array value
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1814

~~~~text
      %new_v = fir.array_amend %v, %2 : (!fir.array<?x?xT>, !fir.ref<T>) -> !fir.array<?x?xT>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1815

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1816

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1817

~~~~text
    More information about `array_amend` and other array operations can be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1818

~~~~text
    found in flang/docs/FIRArrayOperations.md.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1819

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1820

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1821

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1822

~~~~text
    fir_SequenceType:$sequence,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1823

~~~~text
    fir_ReferenceType:$memref
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1824

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1825

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1826

~~~~text
  let results = (outs fir_SequenceType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1827

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1828

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1829

~~~~text
    $sequence `,` $memref attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1830

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1831

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1832

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1833

~~~~text
def fir_ArrayMergeStoreOp : fir_Op<"array_merge_store",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1834

~~~~text
    [AttrSizedOperandSegments, DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1835

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1836

~~~~text
  let summary = "Store merged array value to memory.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1837

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1838

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1839

~~~~text
    Store a merged array value to memory.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1840

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1841

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1842

~~~~text
      real :: a(n,m)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1843

~~~~text
      ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1844

~~~~text
      a = ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1845

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1846

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1847

~~~~text
    One can use `fir.array_merge_store` to merge/copy the value of `a` in an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1848

~~~~text
    array expression as shown above.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1849

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1850

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1851

~~~~text
      %v = fir.array_load %a(%shape) : ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1852

~~~~text
      %r = fir.array_update %v, %f, %i, %j : (!fir.array<?x?xf32>, f32, index, index) -> !fir.array<?x?xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1853

~~~~text
      fir.array_merge_store %v, %r to %a : !fir.ref<!fir.array<?x?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1854

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1855

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1856

~~~~text
    This operation merges the original loaded array value, `%v`, with the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1857

~~~~text
    chained updates, `%r`, and stores the result to the array at address, `%a`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1858

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1859

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1860

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1861

~~~~text
    fir_SequenceType:$original,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1862

~~~~text
    fir_SequenceType:$sequence,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1863

~~~~text
    AnyRefOrBox:$memref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1864

~~~~text
    Optional<fir_SliceType>:$slice,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1865

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1866

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1867

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1868

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1869

~~~~text
    $original `,` $sequence `to` $memref (`[` $slice^ `]`)? (`typeparams`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1870

~~~~text
      $typeparams^)? attr-dict `:` type(operands)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1871

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1872

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1873

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1874

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1875

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1876

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 1877

~~~~text
// Record and array type operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1878

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 1879

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1880

~~~~text
def fir_ArrayCoorOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1881

~~~~text
    : fir_Op<"array_coor", [NoMemoryEffect, AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1882

~~~~text
                            fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1883

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1884

~~~~text
  let summary = "Find the coordinate of an element of an array";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1885

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1886

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1887

~~~~text
    Compute the location of an element in an array when the shape of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1888

~~~~text
    array is only known at runtime.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1889

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1890

~~~~text
    This operation is intended to capture all the runtime values needed to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1891

~~~~text
    compute the address of an array reference in a single high-level op. Given
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1892

~~~~text
    the following Fortran input:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1893

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1894

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1895

~~~~text
      real :: a(n,m)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1896

~~~~text
      ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1897

~~~~text
      ... a(i,j) ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1898

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1899

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1900

~~~~text
    One can use `fir.array_coor` to determine the address of `a(i,j)`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1901

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1902

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1903

~~~~text
      %s = fir.shape %n, %m : (index, index) -> !fir.shape<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1904

~~~~text
      %1 = fir.array_coor %a(%s) %i, %j : (!fir.ref<!fir.array<?x?xf32>>, !fir.shape<2>, index, index) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1905

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1906

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1907

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1908

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1909

~~~~text
    AnyRefOrBox:$memref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1910

~~~~text
    Optional<AnyShapeOrShiftType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1911

~~~~text
    Optional<fir_SliceType>:$slice,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1912

~~~~text
    Variadic<AnyCoordinateType>:$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1913

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1914

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1915

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1916

~~~~text
  let results = (outs fir_ReferenceType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1917

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1918

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1919

~~~~text
    $memref (`(`$shape^`)`)? (`[`$slice^`]`)? $indices (`typeparams`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1920

~~~~text
      $typeparams^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1921

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1922

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1923

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1924

~~~~text
  let hasCanonicalizer = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1925

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1926

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1927

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getMemref(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1928

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1929

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1930

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1931

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1932

~~~~text
def fir_CoordinateOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1933

~~~~text
    : fir_Op<"coordinate_of", [NoMemoryEffect, ConditionallySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1934

~~~~text
                               fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1935

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1936

~~~~text
  let summary = "Finds the coordinate (location) of a value in memory";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1937

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1938

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1939

~~~~text
    Compute the internal coordinate address starting from a boxed value or
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1940

~~~~text
    unboxed memory reference. Returns a memory reference. When computing the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1941

~~~~text
    coordinate of an array element, the rank of the array must be known and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1942

~~~~text
    the number of indexing expressions must not exceed the rank of the array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1943

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1944

~~~~text
    This operation will apply the access map from a boxed value implicitly.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1945

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1946

~~~~text
    Unlike LLVM's GEP instruction, one cannot stride over the outermost
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1947

~~~~text
    reference; therefore, the leading 0 index must be omitted.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1948

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1949

~~~~text
    This operation can be used to index derived type fields, in which case
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1950

~~~~text
    the operand is the name of the index field.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1951

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1952

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1953

~~~~text
      %i = ... : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1954

~~~~text
      %h = ... : !fir.heap<!fir.array<100 x f32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1955

~~~~text
      %p = fir.coordinate_of %h, %i : (!fir.heap<!fir.array<100 x f32>>, index) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1956

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1957

~~~~text
      %d = ... : !fir.ref<!fir.type<t{field1:i32, field2:f32}>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1958

~~~~text
      %f = fir.coordinate_of %d, field2 : (!fir.ref<!fir.type<t{field1:i32, field2:f32}>>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1959

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1960

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1961

~~~~text
    In the example, `%p` will be a pointer to the `%i`-th f32 value in the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1962

~~~~text
    array `%h`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1963

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1964

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1965

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1966

~~~~text
    AnyRefOrBox:$ref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1967

~~~~text
    Variadic<AnyCoordinateType>:$coor,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1968

~~~~text
    TypeAttr:$baseType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1969

~~~~text
    OptionalAttr<DenseI32ArrayAttr>:$field_indices
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1970

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1971

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1972

~~~~text
  let results = (outs RefOrLLVMPtr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1973

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1974

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1975

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1976

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1977

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1978

~~~~text
    OpBuilder<(ins "mlir::Type":$resultType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1979

~~~~text
      "mlir::Value":$ref, "mlir::ValueRange":$coor)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1980

~~~~text
    OpBuilder<(ins "mlir::Type":$resultType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1981

~~~~text
      "mlir::Value":$ref, "llvm::ArrayRef<fir::IntOrValue>":$coor)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1982

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1983

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1984

~~~~text
    constexpr static int32_t kDynamicIndex = std::numeric_limits<int32_t>::min();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1985

~~~~text
    CoordinateIndicesAdaptor getIndices();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1986

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1987

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1988

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getRef(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1989

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1990

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1991

~~~~text
    // Interface method for ConditionallySpeculatable.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1992

~~~~text
    mlir::Speculation::Speculatability getSpeculatability();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1993

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1994

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1995

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1996

~~~~text
def fir_ExtractValueOp : fir_OneResultOp<"extract_value", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1997

~~~~text
  let summary = "Extract a value from an aggregate SSA-value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1998

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1999

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2000

~~~~text
    Extract a value from an entity with a type composed of tuples, arrays,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2001

~~~~text
    and/or derived types. Returns the value from entity with the type of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2002

~~~~text
    specified component. Cannot be used on values of `!fir.box` type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2003

~~~~text
    It can also be used to access complex parts and elements of a character
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2004

~~~~text
    string.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2005

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2006

~~~~text
    Note that the entity ssa-value must be of compile-time known size in order
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2007

~~~~text
    to use this operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2008

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2009

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2010

~~~~text
      %f = fir.field_index field, !fir.type<X{field:i32}>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2011

~~~~text
      %s = ... : !fir.type<X>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2012

~~~~text
      %v = fir.extract_value %s, %f : (!fir.type<X>, !fir.field) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2013

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2014

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2015

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2016

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2017

~~~~text
    AnyCompositeLike:$adt,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2018

~~~~text
    ArrayAttr:$coor
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2019

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2020

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2021

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2022

~~~~text
    $adt `,` $coor attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2023

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2024

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2025

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2026

~~~~text
def fir_FieldIndexOp : fir_OneResultOp<"field_index", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2027

~~~~text
  let summary = "create a field index value from a field identifier";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2028

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2029

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2030

~~~~text
    Generate a field (offset) value from an identifier.  Field values may be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2031

~~~~text
    lowered into exact offsets when the layout of a Fortran derived type is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2032

~~~~text
    known at compile-time. The type of a field value is `!fir.field` and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2033

~~~~text
    these values can be used with the `fir.coordinate_of`, `fir.extract_value`,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2034

~~~~text
    or `fir.insert_value` instructions to compute (abstract) addresses of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2035

~~~~text
    subobjects.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2036

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2037

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2038

~~~~text
      %f = fir.field_index field, !fir.type<X{field:i32}>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2039

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2040

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2041

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2042

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2043

~~~~text
    StrAttr:$field_id,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2044

~~~~text
    TypeAttr:$on_type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2045

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2046

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2047

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2048

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2049

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2050

~~~~text
  let builders = [OpBuilder<(ins "llvm::StringRef":$fieldName,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2051

~~~~text
      "mlir::Type":$recTy, CArg<"mlir::ValueRange","{}">:$operands)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2052

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2053

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2054

~~~~text
    static constexpr llvm::StringRef getFieldAttrName() { return "field_id"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2055

~~~~text
    static constexpr llvm::StringRef getTypeAttrName() { return "on_type"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2056

~~~~text
    llvm::StringRef getFieldName() { return getFieldId(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2057

~~~~text
    llvm::SmallVector<mlir::Attribute> getAttributes();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2058

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2059

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2060

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2061

~~~~text
def fir_ShapeOp : fir_Op<"shape", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2062

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2063

~~~~text
  let summary = "generate an abstract shape vector of type `!fir.shape`";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2064

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2065

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2066

~~~~text
    The arguments are an ordered list of integral type values that define the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2067

~~~~text
    runtime extent of each dimension of an array. The shape information is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2068

~~~~text
    given in the same row-to-column order as Fortran. This abstract shape value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2069

~~~~text
    must be applied to a reified object, so all shape information must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2070

~~~~text
    specified.  The extent must be nonnegative.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2071

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2072

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2073

~~~~text
      %d = fir.shape %row_sz, %col_sz : (index, index) -> !fir.shape<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2074

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2075

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2076

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2077

~~~~text
  let arguments = (ins Variadic<AnyIntegerType>:$extents);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2078

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2079

~~~~text
  let results = (outs fir_ShapeType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2080

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2081

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2082

~~~~text
    operands attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2083

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2084

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2085

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2086

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2087

~~~~text
  let builders = [OpBuilder<(ins "mlir::ValueRange":$extents)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2088

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2089

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2090

~~~~text
def fir_ShapeShiftOp : fir_Op<"shape_shift", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2091

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2092

~~~~text
  let summary = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2093

~~~~text
    generate an abstract shape and shift vector of type `!fir.shapeshift`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2094

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2095

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2096

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2097

~~~~text
    The arguments are an ordered list of integral type values that is a multiple
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2098

~~~~text
    of 2 in length. Each such pair is defined as: the lower bound and the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2099

~~~~text
    extent for that dimension. The shifted shape information is given in the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2100

~~~~text
    same row-to-column order as Fortran. This abstract shifted shape value must
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2101

~~~~text
    be applied to a reified object, so all shifted shape information must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2102

~~~~text
    specified.  The extent must be nonnegative.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2103

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2104

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2105

~~~~text
      %d = fir.shape_shift %lo, %extent : (index, index) -> !fir.shapeshift<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2106

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2107

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2108

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2109

~~~~text
  let arguments = (ins Variadic<AnyIntegerType>:$pairs);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2110

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2111

~~~~text
  let results = (outs fir_ShapeShiftType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2112

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2113

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2114

~~~~text
    operands attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2115

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2116

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2117

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2118

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2119

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2120

~~~~text
    // Logically unzip the origins from the extent values.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2121

~~~~text
    std::vector<mlir::Value> getOrigins() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2122

~~~~text
      std::vector<mlir::Value> result;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2123

~~~~text
      for (auto i : llvm::enumerate(getPairs()))
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 2124

~~~~text
        if (!(i.index() & 1))
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 2125

~~~~text
          result.push_back(i.value());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2126

~~~~text
      return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2127

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2128

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2129

~~~~text
    // Logically unzip the extents from the origin values.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2130

~~~~text
    std::vector<mlir::Value> getExtents() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2131

~~~~text
      std::vector<mlir::Value> result;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2132

~~~~text
      for (auto i : llvm::enumerate(getPairs()))
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 2133

~~~~text
        if (i.index() & 1)
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 2134

~~~~text
          result.push_back(i.value());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2135

~~~~text
      return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2136

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2137

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2138

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2139

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2140

~~~~text
def fir_ShiftOp : fir_Op<"shift", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2141

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2142

~~~~text
  let summary = "generate an abstract shift vector of type `!fir.shift`";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2143

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2144

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2145

~~~~text
    The arguments are an ordered list of integral type values that define the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2146

~~~~text
    runtime lower bound of each dimension of an array. The shape information is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2147

~~~~text
    given in the same row-to-column order as Fortran. This abstract shift value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2148

~~~~text
    must be applied to a reified object, so all shift information must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2149

~~~~text
    specified.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2150

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2151

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2152

~~~~text
      %d = fir.shift %row_lb, %col_lb : (index, index) -> !fir.shift<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2153

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2154

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2155

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2156

~~~~text
  let arguments = (ins Variadic<AnyIntegerType>:$origins);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2157

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2158

~~~~text
  let results = (outs fir_ShiftType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2159

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2160

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2161

~~~~text
    operands attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2162

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2163

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2164

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2165

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2166

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2167

~~~~text
def fir_SliceOp : fir_Op<"slice", [Pure, AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2168

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2169

~~~~text
  let summary = "generate an abstract slice vector of type `!fir.slice`";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2170

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2171

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2172

~~~~text
    The array slicing arguments are an ordered list of integral type values
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2173

~~~~text
    that must be a multiple of 3 in length.  Each such triple is defined as:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2174

~~~~text
    the lower bound, the upper bound, and the stride for that dimension, as in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2175

~~~~text
    Fortran syntax. Both bounds are inclusive. The array slice information is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2176

~~~~text
    given in the same row-to-column order as Fortran. This abstract slice value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2177

~~~~text
    must be applied to a reified object, so all slice information must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2178

~~~~text
    specified.  The extent must be nonnegative and the stride must not be zero.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2179

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2180

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2181

~~~~text
      %d = fir.slice %lo, %hi, %step : (index, index, index) -> !fir.slice<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2182

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2183

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2184

~~~~text
    To support generalized slicing of Fortran's dynamic derived types, a slice
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2185

~~~~text
    op can be given a component path (narrowing from the product type of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2186

~~~~text
    original array to the specific elemental type of the sliced projection).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2187

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2188

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2189

~~~~text
      %fld = fir.field_index component, !fir.type<t{...component:ct...}>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2190

~~~~text
      %d = fir.slice %lo, %hi, %step path %fld :
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2191

~~~~text
          (index, index, index, !fir.field) -> !fir.slice<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2192

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2193

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2194

~~~~text
    Projections of `!fir.char` type can be further narrowed to invariant
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2195

~~~~text
    substrings.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2196

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2197

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2198

~~~~text
      %d = fir.slice %lo, %hi, %step substr %offset, %width :
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2199

~~~~text
          (index, index, index, index, index) -> !fir.slice<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2200

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2201

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2202

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2203

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2204

~~~~text
    Variadic<AnyIntegerType>:$triples,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2205

~~~~text
    Variadic<AnyComponentType>:$fields,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2206

~~~~text
    Variadic<AnyIntegerType>:$substr
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2207

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2208

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2209

~~~~text
  let results = (outs fir_SliceType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2210

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2211

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2212

~~~~text
    $triples (`path` $fields^)? (`substr` $substr^)? attr-dict `:`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2213

~~~~text
      functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2214

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2215

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2216

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2217

~~~~text
    OpBuilder<(ins "mlir::ValueRange":$triples,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2218

~~~~text
      CArg<"mlir::ValueRange", "{}">:$fields,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2219

~~~~text
      CArg<"mlir::ValueRange", "{}">:$substr)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2220

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2221

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2222

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2223

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2224

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2225

~~~~text
    unsigned getOutRank() { return getOutputRank(getTriples()); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2226

~~~~text
    static unsigned getOutputRank(mlir::ValueRange triples);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2227

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2228

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2229

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2230

~~~~text
def fir_InsertValueOp : fir_OneResultOp<"insert_value", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2231

~~~~text
  let summary = "insert a new sub-value into a copy of an existing aggregate";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2232

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2233

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2234

~~~~text
    Insert a value into an entity with a type composed of tuples, arrays,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2235

~~~~text
    and/or derived types. Returns a new ssa-value with the same type as the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2236

~~~~text
    original entity. Cannot be used on values of `!fir.box` type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2237

~~~~text
    It can also be used to set complex parts and elements of a character
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2238

~~~~text
    string.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2239

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2240

~~~~text
    Note that the entity ssa-value must be of compile-time known size in order
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2241

~~~~text
    to use this operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2242

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2243

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2244

~~~~text
      %a = ... : !fir.array<10xtuple<i32, f32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2245

~~~~text
      %f = ... : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2246

~~~~text
      %o = ... : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2247

~~~~text
      %c = arith.constant 1 : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2248

~~~~text
      %b = fir.insert_value %a, %f, %o, %c : (!fir.array<10x20xtuple<i32, f32>>, f32, i32, i32) -> !fir.array<10x20xtuple<i32, f32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2249

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2250

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2251

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2252

~~~~text
  let arguments = (ins AnyCompositeLike:$adt, AnyType:$val, ArrayAttr:$coor);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2253

~~~~text
  let results = (outs AnyCompositeLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2254

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2255

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2256

~~~~text
    $adt `,` $val `,` $coor attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2257

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2258

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2259

~~~~text
  let hasCanonicalizer = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2260

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2261

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2262

~~~~text
def fir_InsertOnRangeOp : fir_OneResultOp<"insert_on_range", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2263

~~~~text
  let summary = "insert sub-value into a range on an existing sequence";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2264

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2265

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2266

~~~~text
    Insert copies of a value into an entity with an array type of constant shape
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2267

~~~~text
    and size.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2268

~~~~text
    Returns a new ssa-value with the same type as the original entity.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2269

~~~~text
    The values are inserted at a contiguous range of indices in Fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2270

~~~~text
    row-to-column element order as specified by lower and upper bound
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2271

~~~~text
    coordinates.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2272

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2273

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2274

~~~~text
      %a = fir.undefined !fir.array<10x10xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2275

~~~~text
      %c = arith.constant 3.0 : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2276

~~~~text
      %1 = fir.insert_on_range %a, %c from (0, 0) to (7, 2) : (!fir.array<10x10xf32>, f32) -> !fir.array<10x10xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2277

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2278

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2279

~~~~text
    The first 28 elements of %1, with coordinates from (0,0) to (7,2), have
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2280

~~~~text
    the value 3.0.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2281

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2282

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2283

~~~~text
  let arguments = (ins fir_SequenceType:$seq, AnyType:$val, IndexElementsAttr:$coor);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2284

~~~~text
  let results = (outs fir_SequenceType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2285

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2286

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2287

~~~~text
    $seq `,` $val custom<CustomRangeSubscript>($coor) attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2288

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2289

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2290

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2291

~~~~text
    /// Is this insert_on_range inserting on all the values of the result type?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2292

~~~~text
    bool isFullRange();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2293

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2294

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2295

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2296

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2297

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2298

~~~~text
def fir_LenParamIndexOp : fir_OneResultOp<"len_param_index", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2299

~~~~text
  let summary =
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2300

~~~~text
    "create a field index value from a LEN type parameter identifier";
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2301

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2302

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2303

~~~~text
    Generate a LEN parameter (offset) value from a LEN parameter identifier.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2304

~~~~text
    The type of a LEN parameter value is `!fir.len` and these values can be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2305

~~~~text
    used with the `fir.coordinate_of` instructions to compute (abstract)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2306

~~~~text
    addresses of LEN parameters.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2307

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2308

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2309

~~~~text
      %e = fir.len_param_index len1, !fir.type<X(len1:i32)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2310

~~~~text
      %p = ... : !fir.box<!fir.type<X>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2311

~~~~text
      %q = fir.coordinate_of %p, %e : (!fir.box<!fir.type<X>>, !fir.len) -> !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2312

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2313

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2314

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2315

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2316

~~~~text
    StrAttr:$field_id,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2317

~~~~text
    TypeAttr:$on_type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2318

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2319

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2320

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2321

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2322

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2323

~~~~text
  let builders = [OpBuilder<(ins "llvm::StringRef":$fieldName,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2324

~~~~text
      "mlir::Type":$recTy, CArg<"mlir::ValueRange","{}">:$operands)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2325

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2326

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2327

~~~~text
    static constexpr llvm::StringRef getFieldAttrName() { return "field_id"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2328

~~~~text
    static constexpr llvm::StringRef getTypeAttrName() { return "on_type"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2329

~~~~text
    llvm::StringRef getParamName() { return getFieldId(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2330

~~~~text
    llvm::SmallVector<mlir::Attribute> getAttributes();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2331

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2332

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2333

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2334

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 2335

~~~~text
// Fortran loops
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2336

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 2337

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2338

~~~~text
def fir_ResultOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2339

~~~~text
    : fir_Op<"result", [Pure, ReturnLike, Terminator,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2340

~~~~text
                        ParentOneOf<["IfOp", "DoLoopOp", "IterWhileOp"]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2341

~~~~text
  let summary = "special terminator for use in fir region operations";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2342

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2343

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2344

~~~~text
    Result takes a list of ssa-values produced in the block and forwards them
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2345

~~~~text
    as a result to the operation that owns the region of the block. The
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2346

~~~~text
    operation can retain the values or return them to its parent block
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2347

~~~~text
    depending upon its semantics.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2348

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2349

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2350

~~~~text
  let arguments = (ins Variadic<AnyType>:$results);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2351

~~~~text
  let builders = [OpBuilder<(ins), [{ /* do nothing */ }]>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2352

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2353

~~~~text
  let assemblyFormat = "($results^ `:` type($results))? attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2354

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2355

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2356

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2357

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2358

~~~~text
def FirRegionTerminator : SingleBlockImplicitTerminator<"ResultOp">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2359

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2360

~~~~text
class region_Op<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 2361

~~~~text
    fir_Op<mnemonic,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2362

~~~~text
    !listconcat(traits, [FirRegionTerminator, RecursivelySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2363

~~~~text
        RecursiveMemoryEffects])> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2364

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2365

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2366

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2367

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2368

~~~~text
def fir_DoLoopOp : region_Op<"do_loop", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2369

~~~~text
    DeclareOpInterfaceMethods<LoopLikeOpInterface,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2370

~~~~text
        ["getYieldedValuesMutable"]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2371

~~~~text
  let summary = "generalized loop operation";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2372

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2373

~~~~text
    Generalized high-level looping construct. This operation is similar to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2374

~~~~text
    MLIR's `scf.for`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2375

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2376

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2377

~~~~text
      %l = arith.constant 0 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2378

~~~~text
      %u = arith.constant 9 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2379

~~~~text
      %s = arith.constant 1 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2380

~~~~text
      fir.do_loop %i = %l to %u step %s unordered {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2381

~~~~text
        %x = fir.convert %i : (index) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2382

~~~~text
        %v = fir.call @compute(%x) : (i32) -> f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2383

~~~~text
        %p = fir.coordinate_of %A, %i : (!fir.ref<!fir.array<?xf32>>, index) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2384

~~~~text
        fir.store %v to %p : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2385

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2386

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2387

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2388

~~~~text
    The above example iterates over the interval `[%l, %u]`. The unordered
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2389

~~~~text
    keyword indicates that the iterations can be executed in any order.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2390

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2391

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2392

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2393

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2394

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2395

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2396

~~~~text
    Index:$lowerBound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2397

~~~~text
    Index:$upperBound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2398

~~~~text
    Index:$step,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2399

~~~~text
    Variadic<AnyType>:$reduceOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2400

~~~~text
    Variadic<AnyType>:$initArgs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2401

~~~~text
    OptionalAttr<UnitAttr>:$unordered,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2402

~~~~text
    OptionalAttr<UnitAttr>:$finalValue,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2403

~~~~text
    OptionalAttr<ArrayAttr>:$reduceAttrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2404

~~~~text
    OptionalAttr<LoopAnnotationAttr>:$loopAnnotation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2405

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2406

~~~~text
  let results = (outs Variadic<AnyType>:$results);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2407

~~~~text
  let regions = (region SizedRegion<1>:$region);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2408

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2409

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2410

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2411

~~~~text
    OpBuilder<(ins "mlir::Value":$lowerBound, "mlir::Value":$upperBound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2412

~~~~text
      "mlir::Value":$step, CArg<"bool", "false">:$unordered,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2413

~~~~text
      CArg<"bool", "false">:$finalCountValue,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2414

~~~~text
      CArg<"mlir::ValueRange", "{}">:$iterArgs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2415

~~~~text
      CArg<"mlir::ValueRange", "{}">:$reduceOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2416

~~~~text
      CArg<"llvm::ArrayRef<mlir::Attribute>", "{}">:$reduceAttrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2417

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2418

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2419

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2420

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2421

~~~~text
    mlir::Value getInductionVar() { return getBody()->getArgument(0); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2422

~~~~text
    mlir::OpBuilder getBodyBuilder() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2423

~~~~text
      return mlir::OpBuilder(getBody(), std::prev(getBody()->end()));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2424

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2425

~~~~text
    mlir::Block::BlockArgListType getRegionIterArgs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2426

~~~~text
      return getBody()->getArguments().drop_front();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2427

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2428

~~~~text
    mlir::Operation::operand_range getIterOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2429

~~~~text
      return getOperands()
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2430

~~~~text
          .drop_front(getNumControlOperands() + getNumReduceOperands());
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2431

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2432

~~~~text
    llvm::MutableArrayRef<mlir::OpOperand> getInitsMutable() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2433

~~~~text
      return getOperation()->getOpOperands()
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2434

~~~~text
          .drop_front(getNumControlOperands() + getNumReduceOperands());
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2435

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2436

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2437

~~~~text
    void setLowerBound(mlir::Value bound) { (*this)->setOperand(0, bound); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2438

~~~~text
    void setUpperBound(mlir::Value bound) { (*this)->setOperand(1, bound); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2439

~~~~text
    void setStep(mlir::Value step) { (*this)->setOperand(2, step); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2440

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2441

~~~~text
    /// Number of region arguments for loop-carried values
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2442

~~~~text
    unsigned getNumRegionIterArgs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2443

~~~~text
      return getBody()->getNumArguments() - 1;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2444

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2445

~~~~text
    /// Number of operands controlling the loop: lb, ub, step
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2446

~~~~text
    unsigned getNumControlOperands() { return 3; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2447

~~~~text
    /// Does the operation hold operands for loop-carried values
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2448

~~~~text
    bool hasIterOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2449

~~~~text
      return getNumIterOperands() > 0;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2450

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2451

~~~~text
    /// Does the operation hold operands for reduction variables
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2452

~~~~text
    bool hasReduceOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2453

~~~~text
      return getNumReduceOperands() > 0;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2454

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2455

~~~~text
    /// Get Number of variadic operands
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2456

~~~~text
    unsigned getNumOperands(unsigned idx) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2457

~~~~text
      auto segments = (*this)->getAttrOfType<mlir::DenseI32ArrayAttr>(
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 2458

~~~~text
        getOperandSegmentSizeAttr());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2459

~~~~text
      return static_cast<unsigned>(segments[idx]);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2460

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2461

~~~~text
    // Get Number of reduction operands
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2462

~~~~text
    unsigned getNumReduceOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2463

~~~~text
      return getNumOperands(3);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2464

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2465

~~~~text
    /// Get Number of loop-carried values
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2466

~~~~text
    unsigned getNumIterOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2467

~~~~text
      return getNumOperands(4);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2468

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2469

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2470

~~~~text
    /// Get the body of the loop
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2471

~~~~text
    mlir::Block *getBody() { return &getRegion().front(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2472

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2473

~~~~text
    void setUnordered() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2474

~~~~text
      setUnorderedAttr(mlir::UnitAttr::get(getContext()));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2475

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2476

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2477

~~~~text
    mlir::BlockArgument iterArgToBlockArg(mlir::Value iterArg);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2478

~~~~text
    void resultToSourceOps(llvm::SmallVectorImpl<mlir::Value> &results,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2479

~~~~text
                           unsigned resultNum);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2480

~~~~text
    mlir::Value blockArgToSourceOp(unsigned blockArgNum);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2481

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2482

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2483

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2484

~~~~text
def fir_IfOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2485

~~~~text
    : region_Op<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2486

~~~~text
          "if", [DeclareOpInterfaceMethods<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2487

~~~~text
                     RegionBranchOpInterface, ["getRegionInvocationBounds",
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2488

~~~~text
                                               "getEntrySuccessorRegions",
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2489

~~~~text
                                               "getSuccessorInputs"]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2490

~~~~text
                 RecursiveMemoryEffects, NoRegionArguments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2491

~~~~text
                 WeightedRegionBranchOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2492

~~~~text
  let summary = "if-then-else conditional operation";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2493

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2494

~~~~text
    Used to conditionally execute operations. This operation is the FIR
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2495

~~~~text
    dialect's version of `loop.if`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2496

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2497

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2498

~~~~text
      %56 = ... : i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2499

~~~~text
      %78 = ... : !fir.ref<!T>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2500

~~~~text
      fir.if %56 {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2501

~~~~text
        fir.store %76 to %78 : !fir.ref<!T>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2502

~~~~text
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2503

~~~~text
        fir.store %77 to %78 : !fir.ref<!T>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2504

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2505

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2506

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2507

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2508

~~~~text
  let arguments = (ins I1:$condition,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2509

~~~~text
      OptionalAttr<DenseI32ArrayAttr>:$region_weights);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2510

~~~~text
  let results = (outs Variadic<AnyType>:$results);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2511

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2512

~~~~text
  let regions = (region
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2513

~~~~text
    SizedRegion<1>:$thenRegion,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2514

~~~~text
    MaxSizedRegion<1>:$elseRegion
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2515

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2516

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2517

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2518

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2519

~~~~text
    OpBuilder<(ins "mlir::Value":$cond, "bool":$withElseRegion)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2520

~~~~text
    OpBuilder<(ins "mlir::TypeRange":$resultTypes, "mlir::Value":$cond,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2521

~~~~text
        "bool":$withElseRegion)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2522

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2523

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2524

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2525

~~~~text
    mlir::OpBuilder getThenBodyBuilder() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2526

~~~~text
      assert(!getThenRegion().empty() && "Unexpected empty 'where' region.");
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2527

~~~~text
      mlir::Block &body = getThenRegion().front();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2528

~~~~text
      return mlir::OpBuilder(&body, std::prev(body.end()));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2529

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2530

~~~~text
    mlir::OpBuilder getElseBodyBuilder() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2531

~~~~text
      assert(!getElseRegion().empty() && "Unexpected empty 'other' region.");
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2532

~~~~text
      mlir::Block &body = getElseRegion().front();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2533

~~~~text
      return mlir::OpBuilder(&body, std::prev(body.end()));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2534

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2535

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2536

~~~~text
    void resultToSourceOps(llvm::SmallVectorImpl<mlir::Value> &results,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2537

~~~~text
                           unsigned resultNum);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2538

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2539

~~~~text
    /// Returns the display name string for the region_weights attribute.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2540

~~~~text
    static constexpr llvm::StringRef getWeightsAttrAssemblyName() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2541

~~~~text
      return "weights";
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2542

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2543

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2544

~~~~text
    /// Sets WeightedRegionBranchOpInterface weights to indicate
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2545

~~~~text
    /// that either THEN or ELSE branch is unlikely.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2546

~~~~text
    /// By default, THEN branch is set to be unlikely.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2547

~~~~text
    void setUnlikelyIfWeights(bool unlikelyElse = false) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2548

~~~~text
      if (unlikelyElse)
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 2549

~~~~text
        setWeights({1, 0});
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2550

~~~~text
      else
~~~~
- EN: Provides the fallback branch for the preceding condition.
- CN: 为前面的条件提供兜底分支。

### Line 2551

~~~~text
        setWeights({0, 1});
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2552

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2553

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2554

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2555

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2556

~~~~text
def fir_IterWhileOp : region_Op<"iterate_while",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2557

~~~~text
    [DeclareOpInterfaceMethods<LoopLikeOpInterface,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2558

~~~~text
        ["getYieldedValuesMutable"]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2559

~~~~text
  let summary = "DO loop with early exit condition";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2560

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2561

~~~~text
    This single-entry, single-exit looping construct is useful for lowering
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2562

~~~~text
    counted loops that can exit early such as, for instance, implied-DO loops.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2563

~~~~text
    It is very similar to `fir::DoLoopOp` with the addition that it requires
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2564

~~~~text
    a single loop-carried bool value that signals an early exit condition to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2565

~~~~text
    the operation. A `true` disposition means the next loop iteration should
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2566

~~~~text
    proceed. A `false` indicates that the `fir.iterate_while` operation should
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2567

~~~~text
    terminate and return its iteration arguments. This is a degenerate counted
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2568

~~~~text
    loop in that the loop is not guaranteed to execute all iterations.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2569

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2570

~~~~text
    An example iterate_while that returns the counter value, the early
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2571

~~~~text
    termination condition, and an extra loop-carried value is shown here. This
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2572

~~~~text
    loop counts from %lo to %up (inclusive), stepping by %c1, so long as the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2573

~~~~text
    early exit (%ok) is true. The iter_args %sh value is also carried by the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2574

~~~~text
    loop. The result triple is the values of %i=phi(%lo,%i+%c1),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2575

~~~~text
    %ok=phi(%okIn,%okNew), and %sh=phi(%shIn,%shNew) from the last executed
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2576

~~~~text
    iteration.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2577

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2578

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2579

~~~~text
      %v:3 = fir.iterate_while (%i = %lo to %up step %c1) and (%ok = %okIn) iter_args(%sh = %shIn) -> (index, i1, i16) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2580

~~~~text
        %shNew = fir.call @bar(%sh) : (i16) -> i16
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2581

~~~~text
        %okNew = fir.call @foo(%sh) : (i16) -> i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2582

~~~~text
        fir.result %i, %okNew, %shNew : index, i1, i16
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2583

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2584

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2585

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2586

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2587

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2588

~~~~text
    Index:$lowerBound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2589

~~~~text
    Index:$upperBound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2590

~~~~text
    Index:$step,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2591

~~~~text
    I1:$iterateIn,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2592

~~~~text
    Variadic<AnyType>:$initArgs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2593

~~~~text
    OptionalAttr<UnitAttr>:$finalValue
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2594

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2595

~~~~text
  let results = (outs Variadic<AnyType>:$results);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2596

~~~~text
  let regions = (region SizedRegion<1>:$region);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2597

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2598

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2599

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2600

~~~~text
    OpBuilder<(ins "mlir::Value":$lowerBound, "mlir::Value":$upperBound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2601

~~~~text
      "mlir::Value":$step, "mlir::Value":$iterate,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2602

~~~~text
      CArg<"bool", "false">:$finalCountValue,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2603

~~~~text
      CArg<"mlir::ValueRange", "{}">:$iterArgs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2604

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2605

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2606

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2607

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2608

~~~~text
    static constexpr llvm::StringRef getFinalValueAttrNameStr() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2609

~~~~text
      return "finalValue";
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2610

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2611

~~~~text
    mlir::Block *getBody() { return &getRegion().front(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2612

~~~~text
    mlir::Value getIterateVar() { return getBody()->getArgument(1); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2613

~~~~text
    mlir::Value getInductionVar() { return getBody()->getArgument(0); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2614

~~~~text
    mlir::OpBuilder getBodyBuilder() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2615

~~~~text
      return mlir::OpBuilder(getBody(), std::prev(getBody()->end()));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2616

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2617

~~~~text
    mlir::Block::BlockArgListType getRegionIterArgs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2618

~~~~text
      return getBody()->getArguments().drop_front();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2619

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2620

~~~~text
    mlir::Operation::operand_range getIterOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2621

~~~~text
      return getOperands().drop_front(getNumControlOperands());
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2622

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2623

~~~~text
    llvm::MutableArrayRef<mlir::OpOperand> getInitsMutable() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2624

~~~~text
      return
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2625

~~~~text
          getOperation()->getOpOperands().drop_front(getNumControlOperands());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2626

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2627

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2628

~~~~text
    void setLowerBound(mlir::Value bound) { (*this)->setOperand(0, bound); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2629

~~~~text
    void setUpperBound(mlir::Value bound) { (*this)->setOperand(1, bound); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2630

~~~~text
    void setStep(mlir::Value step) { (*this)->setOperand(2, step); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2631

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2632

~~~~text
    /// Number of region arguments for loop-carried values
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2633

~~~~text
    unsigned getNumRegionIterArgs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2634

~~~~text
      return getBody()->getNumArguments() - 1;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2635

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2636

~~~~text
    /// Number of operands controlling the loop
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2637

~~~~text
    unsigned getNumControlOperands() { return 3; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2638

~~~~text
    /// Does the operation hold operands for loop-carried values
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2639

~~~~text
    bool hasIterOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2640

~~~~text
      return (*this)->getNumOperands() > getNumControlOperands();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2641

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2642

~~~~text
    /// Get Number of loop-carried values
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2643

~~~~text
    unsigned getNumIterOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2644

~~~~text
      return (*this)->getNumOperands() - getNumControlOperands();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2645

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2646

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2647

~~~~text
    mlir::BlockArgument iterArgToBlockArg(mlir::Value iterArg);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2648

~~~~text
    void resultToSourceOps(llvm::SmallVectorImpl<mlir::Value> &results,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2649

~~~~text
                           unsigned resultNum);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2650

~~~~text
    mlir::Value blockArgToSourceOp(unsigned blockArgNum);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2651

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2652

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2653

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2654

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 2655

~~~~text
// Procedure call operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2656

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 2657

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2658

~~~~text
def fir_CallOp : fir_Op<"call",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2659

~~~~text
    [CallOpInterface, DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2660

~~~~text
  let summary = "call a procedure";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2661

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2662

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2663

~~~~text
    Call the specified function or function reference.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2664

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2665

~~~~text
    Provides a custom parser and pretty printer to allow a more readable syntax
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2666

~~~~text
    in the FIR dialect, e.g. `fir.call @sub(%12)` or `fir.call %20(%22,%23)`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2667

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2668

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2669

~~~~text
      %a = fir.call %funcref(%arg0) : (!fir.ref<f32>) -> f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2670

~~~~text
      %b = fir.call @function(%arg1, %arg2) : (!fir.ref<f32>, !fir.ref<f32>) -> f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2671

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2672

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2673

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2674

~~~~text
  let arguments = (ins OptionalAttr<SymbolRefAttr>:$callee,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2675

~~~~text
      Variadic<AnyType>:$args, OptionalAttr<DictArrayAttr>:$arg_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2676

~~~~text
      OptionalAttr<DictArrayAttr>:$res_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2677

~~~~text
      OptionalAttr<fir_FortranProcedureFlagsAttr>:$procedure_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2678

~~~~text
      OptionalAttr<fir_FortranInlineAttr>:$inline_attr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2679

~~~~text
      OptionalAttr<LLVM_AccessGroupArrayAttr>:$accessGroups,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2680

~~~~text
      DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2681

~~~~text
                        "::mlir::arith::FastMathFlags::none">:$fastmath);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2682

~~~~text
  let results = (outs Variadic<AnyType>);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2683

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2684

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2685

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2686

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2687

~~~~text
    OpBuilder<(ins "mlir::func::FuncOp":$callee,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2688

~~~~text
        CArg<"mlir::ValueRange", "{}">:$operands)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2689

~~~~text
    OpBuilder<(ins "mlir::SymbolRefAttr":$callee,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2690

~~~~text
        "llvm::ArrayRef<mlir::Type>":$results,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2691

~~~~text
        CArg<"mlir::ValueRange", "{}">:$operands)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2692

~~~~text
    OpBuilder<(ins "llvm::StringRef":$callee,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2693

~~~~text
        "llvm::ArrayRef<mlir::Type>":$results,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2694

~~~~text
        CArg<"mlir::ValueRange", "{}">:$operands),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2695

~~~~text
    [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2696

~~~~text
      build($_builder, $_state,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2697

~~~~text
          mlir::SymbolRefAttr::get($_builder.getContext(), callee), results,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2698

~~~~text
          operands);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2699

~~~~text
    }]>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2700

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2701

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2702

~~~~text
    static constexpr llvm::StringRef getCalleeAttrNameStr() { return "callee"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2703

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2704

~~~~text
    mlir::FunctionType getFunctionType();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2705

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2706

~~~~text
    /// Get the argument operands to the called function.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2707

~~~~text
    operand_range getArgOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2708

~~~~text
      if ((*this)->getAttrOfType<mlir::SymbolRefAttr>(getCalleeAttrName()))
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 2709

~~~~text
        return {arg_operand_begin(), arg_operand_end()};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2710

~~~~text
      return {arg_operand_begin() + 1, arg_operand_end()};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2711

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2712

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2713

~~~~text
    mlir::MutableOperandRange getArgOperandsMutable() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2714

~~~~text
      if ((*this)->getAttrOfType<mlir::SymbolRefAttr>(getCalleeAttrName()))
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 2715

~~~~text
        return getArgsMutable();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2716

~~~~text
      return mlir::MutableOperandRange(*this, 1, getArgs().size() - 1);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2717

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2718

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2719

~~~~text
    operand_iterator arg_operand_begin() { return operand_begin(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2720

~~~~text
    operand_iterator arg_operand_end() { return operand_end(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2721

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2722

~~~~text
    /// Return the callee of this operation.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2723

~~~~text
    mlir::CallInterfaceCallable getCallableForCallee() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2724

~~~~text
      if (auto calling =
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 2725

~~~~text
          (*this)->getAttrOfType<mlir::SymbolRefAttr>(getCalleeAttrName()))
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2726

~~~~text
        return calling;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2727

~~~~text
      return getOperand(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2728

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2729

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2730

~~~~text
    /// Set the callee for this operation.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2731

~~~~text
    void setCalleeFromCallable(mlir::CallInterfaceCallable callee);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2732

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2733

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2734

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2735

~~~~text
def fir_DispatchOp : fir_Op<"dispatch", []> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2736

~~~~text
  let summary = "call a type-bound procedure";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2737

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2738

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2739

~~~~text
    Perform a dynamic dispatch on the method name via the dispatch table
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2740

~~~~text
    associated with the first operand.  The attribute `pass_arg_pos` can be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2741

~~~~text
    used to select a dispatch operand other than the first one.  The absence of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2742

~~~~text
    `pass_arg_pos` attribute means nopass.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2743

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2744

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2745

~~~~text
      // fir.dispatch with no attribute.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2746

~~~~text
      %r = fir.dispatch "methodA"(%o) : (!fir.class<T>) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2747

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2748

~~~~text
      // fir.dispatch with the `pass_arg_pos` attribute.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2749

~~~~text
      %r = fir.dispatch "methodA"(%o : !fir.class<T>) (%o : !fir.class<T>) -> i32 {pass_arg_pos = 0 : i32}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2750

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2751

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2752

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2753

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2754

~~~~text
    StrAttr:$method,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2755

~~~~text
    fir_ClassType:$object,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2756

~~~~text
    Variadic<AnyType>:$args,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2757

~~~~text
    OptionalAttr<I32Attr>:$pass_arg_pos,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2758

~~~~text
    OptionalAttr<DictArrayAttr>:$arg_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2759

~~~~text
    OptionalAttr<DictArrayAttr>:$res_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2760

~~~~text
    OptionalAttr<fir_FortranProcedureFlagsAttr>:$procedure_attrs
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2761

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2762

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2763

~~~~text
  let results = (outs Variadic<AnyType>:$results);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2764

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2765

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2766

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2767

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2768

~~~~text
    $method `(` $object `:` qualified(type($object)) `)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2769

~~~~text
    ( `(` $args^ `:` type($args) `)` )? (`->` type($results)^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2770

~~~~text
    (`proc_attrs` $procedure_attrs^)? attr-dict
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2771

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2772

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2773

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2774

~~~~text
    mlir::FunctionType getFunctionType();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2775

~~~~text
    operand_range getArgOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2776

~~~~text
      return {arg_operand_begin(), arg_operand_end()};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2777

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2778

~~~~text
    // operand[0] is the object (of class type)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2779

~~~~text
    operand_iterator arg_operand_begin() { return operand_begin() + 1; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2780

~~~~text
    operand_iterator arg_operand_end() { return operand_end(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2781

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2782

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2783

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2784

~~~~text
// Constant operations that support Fortran
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2785

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2786

~~~~text
def fir_StringLitOp : fir_Op<"string_lit", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2787

~~~~text
  let summary = "create a string literal constant";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2788

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2789

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2790

~~~~text
    An FIR constant that represents a sequence of characters that correspond
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2791

~~~~text
    to Fortran's CHARACTER type, including a LEN.  We support CHARACTER values
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2792

~~~~text
    of different KINDs (different constant sizes).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2793

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2794

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2795

~~~~text
      %1 = fir.string_lit "Hello, World!"(13) : !fir.char<1> // ASCII
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2796

~~~~text
      %2 = fir.string_lit [158, 2345](2) : !fir.char<2>      // Wide chars
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2797

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2798

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2799

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2800

~~~~text
  let results = (outs fir_CharacterType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2801

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2802

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2803

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2804

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2805

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2806

~~~~text
    OpBuilder<(ins "fir::CharacterType":$inType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2807

~~~~text
      "llvm::StringRef":$value,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2808

~~~~text
      CArg<"std::optional<int64_t>", "{}">:$len)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2809

~~~~text
    OpBuilder<(ins "fir::CharacterType":$inType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2810

~~~~text
      "llvm::ArrayRef<char>":$xlist,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2811

~~~~text
      CArg<"std::optional<int64_t>", "{}">:$len)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2812

~~~~text
    OpBuilder<(ins "fir::CharacterType":$inType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2813

~~~~text
      "llvm::ArrayRef<char16_t>":$xlist,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2814

~~~~text
      CArg<"std::optional<int64_t>", "{}">:$len)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2815

~~~~text
    OpBuilder<(ins "fir::CharacterType":$inType,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2816

~~~~text
      "llvm::ArrayRef<char32_t>":$xlist,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2817

~~~~text
      CArg<"std::optional<int64_t>", "{}">:$len)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2818

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2819

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2820

~~~~text
    static constexpr const char *size() { return "size"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2821

~~~~text
    static constexpr const char *value() { return "value"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2822

~~~~text
    static constexpr const char *xlist() { return "xlist"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2823

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2824

~~~~text
    // Get the LEN attribute of this character constant
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2825

~~~~text
    mlir::Attribute getSize() { return (*this)->getAttr(size()); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2826

~~~~text
    // Get the string value of this character constant
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2827

~~~~text
    mlir::Attribute getValue() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2828

~~~~text
      if (auto attr = (*this)->getAttr(value()))
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 2829

~~~~text
        return attr;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2830

~~~~text
      return (*this)->getAttr(xlist());
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2831

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2832

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2833

~~~~text
    /// Is this a wide character literal (1 character > 8 bits)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2834

~~~~text
    bool isWideValue();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2835

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2836

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2837

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2838

~~~~text
// Complex operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2839

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2840

~~~~text
class fir_ArithmeticOp<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 2841

~~~~text
    fir_Op<mnemonic,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2842

~~~~text
           !listconcat(traits, [NoMemoryEffect, SameOperandsAndResultType])>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2843

~~~~text
    Results<(outs AnyType:$result)> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2844

~~~~text
  let assemblyFormat = "operands attr-dict `:` type($result)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2845

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2846

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2847

~~~~text
class fir_UnaryArithmeticOp<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 2848

~~~~text
      fir_Op<mnemonic,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2849

~~~~text
             !listconcat(traits, [NoMemoryEffect, SameOperandsAndResultType])>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2850

~~~~text
      Results<(outs AnyType:$result)> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2851

~~~~text
  let assemblyFormat = "operands attr-dict `:` type($result)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2852

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2853

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2854

~~~~text
class ComplexUnaryArithmeticOp<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 2855

~~~~text
      fir_UnaryArithmeticOp<mnemonic, traits>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2856

~~~~text
      Arguments<(ins AnyFirComplex:$operand)>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2857

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2858

~~~~text
def fir_NegcOp : ComplexUnaryArithmeticOp<"negc">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2859

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2860

~~~~text
class ComplexArithmeticOp<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 2861

~~~~text
      fir_ArithmeticOp<mnemonic, traits>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2862

~~~~text
      Arguments<(ins AnyFirComplex:$lhs, AnyFirComplex:$rhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2863

~~~~text
          DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2864

~~~~text
                            "::mlir::arith::FastMathFlags::none">:$fastmath)>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2865

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2866

~~~~text
def fir_AddcOp : ComplexArithmeticOp<"addc",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2867

~~~~text
    [Commutative, DeclareOpInterfaceMethods<ArithFastMathInterface>]>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2868

~~~~text
def fir_SubcOp : ComplexArithmeticOp<"subc",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2869

~~~~text
    [DeclareOpInterfaceMethods<ArithFastMathInterface>]>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2870

~~~~text
def fir_MulcOp : ComplexArithmeticOp<"mulc",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2871

~~~~text
    [Commutative, DeclareOpInterfaceMethods<ArithFastMathInterface>]>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2872

~~~~text
def fir_DivcOp : ComplexArithmeticOp<"divc",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2873

~~~~text
    [DeclareOpInterfaceMethods<ArithFastMathInterface>]>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2874

~~~~text
// Pow is a builtin call and not a primitive
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2875

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2876

~~~~text
// Logical operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2877

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2878

~~~~text
class LogicalBinaryOp<string mnemonic, list<Trait> traits = []> :
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 2879

~~~~text
      fir_ArithmeticOp<mnemonic, !listconcat(traits, [Pure])>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2880

~~~~text
      Arguments<(ins AnyLogicalOrIntegerLike:$lhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2881

~~~~text
                     AnyLogicalOrIntegerLike:$rhs)>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2882

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2883

~~~~text
def fir_LogicalAndOp : LogicalBinaryOp<"logical_and", [Commutative]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2884

~~~~text
  let summary = "logical AND";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2885

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2886

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2887

~~~~text
    Logical AND of two values of fir.logical or integer type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2888

~~~~text
    Both operands and the result must have the same type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2889

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2890

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2891

~~~~text
      %r = fir.logical_and %a, %b : !fir.logical<4>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2892

~~~~text
      %s = fir.logical_and %x, %y : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2893

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2894

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2895

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2896

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2897

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2898

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2899

~~~~text
def fir_LogicalOrOp : LogicalBinaryOp<"logical_or", [Commutative]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2900

~~~~text
  let summary = "logical OR";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2901

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2902

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2903

~~~~text
    Logical OR of two values of fir.logical or integer type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2904

~~~~text
    Both operands and the result must have the same type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2905

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2906

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2907

~~~~text
      %r = fir.logical_or %a, %b : !fir.logical<4>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2908

~~~~text
      %s = fir.logical_or %x, %y : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2909

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2910

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2911

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2912

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2913

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2914

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2915

~~~~text
def fir_EqvOp : LogicalBinaryOp<"eqv", [Commutative]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2916

~~~~text
  let summary = "logical equivalence";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2917

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2918

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2919

~~~~text
    Logical equivalence (EQV) of two values of fir.logical or integer type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2920

~~~~text
    Both operands and the result must have the same type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2921

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2922

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2923

~~~~text
      %r = fir.eqv %a, %b : !fir.logical<4>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2924

~~~~text
      %s = fir.eqv %x, %y : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2925

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2926

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2927

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2928

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2929

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2930

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2931

~~~~text
def fir_NeqvOp : LogicalBinaryOp<"neqv", [Commutative]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2932

~~~~text
  let summary = "logical non-equivalence";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2933

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2934

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2935

~~~~text
    Logical non-equivalence (NEQV / XOR) of two values of fir.logical or
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2936

~~~~text
    integer type. Both operands and the result must have the same type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2937

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2938

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2939

~~~~text
      %r = fir.neqv %a, %b : !fir.logical<4>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2940

~~~~text
      %s = fir.neqv %x, %y : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2941

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2942

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2943

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2944

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2945

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2946

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2947

~~~~text
def fir_CmpcOp : fir_Op<"cmpc",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2948

~~~~text
    [NoMemoryEffect, SameTypeOperands, SameOperandsAndResultShape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2949

~~~~text
    DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2950

~~~~text
  let summary = "complex floating-point comparison operator";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2951

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2952

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2953

~~~~text
    A complex comparison to handle complex types found in FIR.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2954

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2955

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2956

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2957

~~~~text
      AnyFirComplex:$lhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2958

~~~~text
      AnyFirComplex:$rhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2959

~~~~text
      DefaultValuedAttr<Arith_FastMathAttr, "::mlir::arith::FastMathFlags::none">:$fastmath);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2960

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2961

~~~~text
  let results = (outs AnyLogicalLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2962

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2963

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2964

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2965

~~~~text
  let builders = [OpBuilder<(ins "mlir::arith::CmpFPredicate":$predicate,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2966

~~~~text
    "mlir::Value":$lhs, "mlir::Value":$rhs), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 2967

~~~~text
      buildCmpCOp($_builder, $_state, predicate, lhs, rhs);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2968

~~~~text
  }]>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2969

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2970

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2971

~~~~text
    static constexpr llvm::StringRef getPredicateAttrName() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2972

~~~~text
      return "predicate";
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2973

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2974

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2975

~~~~text
    mlir::arith::CmpFPredicate getPredicate() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 2976

~~~~text
      return (mlir::arith::CmpFPredicate)(*this)->getAttrOfType<mlir::IntegerAttr>(
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 2977

~~~~text
          getPredicateAttrName()).getInt();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2978

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2979

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2980

~~~~text
    static mlir::arith::CmpFPredicate getPredicateByName(llvm::StringRef name);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 2981

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2982

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 2983

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2984

~~~~text
// Other misc. operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 2985

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2986

~~~~text
def fir_AddrOfOp : fir_OneResultOp<"address_of", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 2987

~~~~text
  let summary = "convert a symbol to an SSA value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2988

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2989

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 2990

~~~~text
    Convert a symbol (a function or global reference) to an SSA-value to be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2991

~~~~text
    used in other operations. References to Fortran symbols are distinguished
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2992

~~~~text
    via this operation from other arbitrary constant values.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2993

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2994

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2995

~~~~text
      %p = fir.address_of(@symbol) : !fir.ref<f64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2996

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 2997

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 2998

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 2999

~~~~text
  let arguments = (ins SymbolRefAttr:$symbol);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3000

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3001

~~~~text
  let results = (outs AnyAddressableLike:$resTy);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3002

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3003

~~~~text
  let assemblyFormat = "`(` $symbol `)` attr-dict `:` type($resTy)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3004

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3005

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3006

~~~~text
def fir_VolatileCastOp : fir_SimpleOneResultOp<"volatile_cast", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3007

~~~~text
  let summary = "cast between volatile and non-volatile types";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3008

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3009

~~~~text
    Cast between volatile and non-volatile types. The types must be otherwise
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3010

~~~~text
    identical. A value's volatility cannot be changed by a fir.convert operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3011

~~~~text
    Reinterpreting a value as volatile must be done explicitly using this operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3012

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3013

~~~~text
  let arguments = (ins AnyRefOrBox:$value);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3014

~~~~text
  let results = (outs AnyRefOrBox:$res);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3015

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3016

~~~~text
    $value attr-dict `:` functional-type($value, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3017

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3018

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3019

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3020

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3021

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3022

~~~~text
def fir_ConvertOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3023

~~~~text
    : fir_SimpleOneResultOp<"convert", [NoMemoryEffect, ViewLikeOpInterface,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3024

~~~~text
                                        ConditionallySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3025

~~~~text
                                        fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3026

~~~~text
  let summary = "encapsulates all Fortran entity type conversions";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3027

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3028

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3029

~~~~text
    Generalized type conversion. Convert the ssa-value from type T to type U.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3030

~~~~text
    Not all pairs of types have conversions. When types T and U are the same
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3031

~~~~text
    type, this instruction is a NOP and may be folded away. This also supports
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3032

~~~~text
    integer to pointer conversion and pointer to integer conversion.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3033

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3034

~~~~text
    This operation also allows limited interaction between FIR and LLVM
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3035

~~~~text
    dialects by allowing conversion between FIR pointer types and llvm.ptr type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3036

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3037

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3038

~~~~text
      %v = ... : i64
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3039

~~~~text
      %w = fir.convert %v : (i64) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3040

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3041

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3042

~~~~text
    The example truncates the value `%v` from an i64 to an i32.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3043

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3044

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3045

~~~~text
  let arguments = (ins AnyType:$value);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3046

~~~~text
  let results = (outs AnyType:$res);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3047

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3048

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3049

~~~~text
    $value attr-dict `:` functional-type($value, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3050

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3051

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3052

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3053

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3054

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3055

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3056

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3057

~~~~text
    static bool isInteger(mlir::Type ty);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3058

~~~~text
    static bool isIntegerCompatible(mlir::Type ty);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3059

~~~~text
    static bool isFloatCompatible(mlir::Type ty);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3060

~~~~text
    static bool isPointerCompatible(mlir::Type ty);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3061

~~~~text
    static bool canBeConverted(mlir::Type inType, mlir::Type outType);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3062

~~~~text
    static bool areVectorsCompatible(mlir::Type inTy, mlir::Type outTy);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3063

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3064

~~~~text
    // ViewLikeOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3065

~~~~text
    mlir::Value getViewSource() { return getValue(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3066

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3067

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3068

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getValue(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3069

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult) { return 0; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3070

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3071

~~~~text
    // Interface method for ConditionallySpeculatable.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3072

~~~~text
    mlir::Speculation::Speculatability getSpeculatability();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3073

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3074

~~~~text
  let hasCanonicalizer = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3075

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3076

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3077

~~~~text
def fir_BitcastOp : fir_SimpleOneResultOp<"bitcast", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3078

~~~~text
  let summary = "bitcast between Fortran scalar types of the same bit size";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3079

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3080

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3081

~~~~text
    Reinterpret the bits of a value from one type to another. Both types must
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3082

~~~~text
    have the same bit size. This is a pure bitwise reinterpretation: no value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3083

~~~~text
    normalization or conversion takes place, unlike `fir.convert`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3084

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3085

~~~~text
    The supported types are integer, floating-point, `fir.logical`, and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3086

~~~~text
    singleton `fir.char` (i.e. `fir.char<K>` with length 1 for any kind K).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3087

~~~~text
    Pointer types are not allowed.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3088

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3089

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3090

~~~~text
      %0 = fir.bitcast %v : (i32) -> !fir.logical<4>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3091

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3092

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3093

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3094

~~~~text
  let arguments = (ins AnyType:$value);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3095

~~~~text
  let results = (outs AnyType:$res);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3096

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3097

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3098

~~~~text
    $value attr-dict `:` functional-type($value, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3099

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3100

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3101

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3102

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3103

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3104

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3105

~~~~text
def FortranTypeAttr : Attr<And<[CPred<"mlir::isa<mlir::TypeAttr>($_self)">,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3106

~~~~text
    Or<[CPred<"mlir::isa<fir::CharacterType, fir::IntegerType, "
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3107

~~~~text
              "fir::UnsignedType, fir::LogicalType, mlir::FloatType, "
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3108

~~~~text
              "mlir::ComplexType, "
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3109

~~~~text
              "fir::RecordType>(mlir::cast<mlir::TypeAttr>($_self).getValue())"
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3110

~~~~text
    >]>]>, "Fortran surface type"> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3111

~~~~text
  let storageType = [{ ::mlir::TypeAttr }];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3112

~~~~text
  let returnType = "mlir::Type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3113

~~~~text
  let convertFromStorage = "mlir::cast<mlir::Type>($_self.getValue())";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3114

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3115

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3116

~~~~text
def fir_TypeDescOp : fir_OneResultOp<"type_desc", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3117

~~~~text
  let summary = "get type descriptor for a given type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3118

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3119

~~~~text
    Generates a constant object that is an abstract type descriptor of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3120

~~~~text
    specified type.  The meta-type of a type descriptor for the type `T`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3121

~~~~text
    is `!fir.tdesc<T>`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3122

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3123

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3124

~~~~text
      %t = fir.type_desc !fir.type<> // returns value of !fir.tdesc<!T>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3125

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3126

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3127

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3128

~~~~text
  let arguments = (ins FortranTypeAttr:$in_type);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3129

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3130

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3131

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3132

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3133

~~~~text
  let builders = [OpBuilder<(ins "mlir::TypeAttr":$inty)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3134

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3135

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3136

~~~~text
def fir_NoReassocOp : fir_OneResultOp<"no_reassoc",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3137

~~~~text
    [Pure, SameOperandsAndResultType]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3138

~~~~text
  let summary = "synthetic op to prevent reassociation";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3139

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3140

~~~~text
    Primitive operation meant to intrusively prevent operator reassociation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3141

~~~~text
    The operation is otherwise a nop and the value returned is the same as the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3142

~~~~text
    argument.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3143

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3144

~~~~text
    The presence of this operation prevents any local optimizations. In the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3145

~~~~text
    example below, this would prevent possibly replacing the multiply and add
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3146

~~~~text
    operations with a single FMA operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3147

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3148

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3149

~~~~text
      %98 = arith.mulf %96, %97 : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3150

~~~~text
      %99 = fir.no_reassoc %98 : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3151

~~~~text
      %a0 = arith.addf %99, %95 : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3152

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3153

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3154

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3155

~~~~text
  let arguments = (ins AnyType:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3156

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3157

~~~~text
  let assemblyFormat = "$val attr-dict `:` type($val)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3158

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3159

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3160

~~~~text
class AtMostRegion<int numBlocks> : Region<
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 3161

~~~~text
  CPred<"$_self.getBlocks().size() <= " # numBlocks>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3162

~~~~text
  "region with " # numBlocks # " blocks">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3163

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3164

~~~~text
def fir_GlobalOp : fir_Op<"global", [IsolatedFromAbove, Symbol]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3165

~~~~text
  let summary = "Global data";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3166

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3167

~~~~text
    A global variable or constant with initial values.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3168

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3169

~~~~text
    The example creates a global variable (writable) named
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3170

~~~~text
    `@_QV_Mquark_Vvarble` with some initial values. The initializer should
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3171

~~~~text
    conform to the variable's type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3172

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3173

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3174

~~~~text
      fir.global @_QV_Mquark_Vvarble : tuple<i32, f32> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3175

~~~~text
        %1 = arith.constant 1 : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3176

~~~~text
        %2 = arith.constant 2.0 : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3177

~~~~text
        %3 = fir.undefined tuple<i32, f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3178

~~~~text
        %z = arith.constant 0 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3179

~~~~text
        %o = arith.constant 1 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3180

~~~~text
        %4 = fir.insert_value %3, %1, %z : (tuple<i32, f32>, i32, index) -> tuple<i32, f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3181

~~~~text
        %5 = fir.insert_value %4, %2, %o : (tuple<i32, f32>, f32, index) -> tuple<i32, f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3182

~~~~text
        fir.has_value %5 : tuple<i32, f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3183

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3184

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3185

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3186

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3187

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3188

~~~~text
    StrAttr:$sym_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3189

~~~~text
    SymbolRefAttr:$symref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3190

~~~~text
    TypeAttr:$type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3191

~~~~text
    OptionalAttr<AnyAttr>:$initVal,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3192

~~~~text
    OptionalAttr<UnitAttr>:$constant,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3193

~~~~text
    OptionalAttr<UnitAttr>:$target,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3194

~~~~text
    OptionalAttr<StrAttr>:$linkName,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3195

~~~~text
    OptionalAttr<cuf_DataAttributeAttr>:$data_attr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3196

~~~~text
    OptionalAttr<I64Attr>:$alignment
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3197

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3198

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3199

~~~~text
  let regions = (region AtMostRegion<1>:$region);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3200

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3201

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3202

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3203

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3204

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3205

~~~~text
    OpBuilder<(ins "llvm::StringRef":$name, "mlir::Type":$type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3206

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attrs)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3207

~~~~text
    OpBuilder<(ins "llvm::StringRef":$name, "bool":$isConstant,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3208

~~~~text
      "bool":$isTarget, "mlir::Type":$type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3209

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attrs)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3210

~~~~text
    OpBuilder<(ins "llvm::StringRef":$name, "mlir::Type":$type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3211

~~~~text
      CArg<"mlir::StringAttr", "{}">:$linkage,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3212

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attrs)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3213

~~~~text
    OpBuilder<(ins "llvm::StringRef":$name, "bool":$isConstant,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3214

~~~~text
      "bool":$isTarget,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3215

~~~~text
      "mlir::Type":$type, CArg<"mlir::StringAttr", "{}">:$linkage,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3216

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attrs)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3217

~~~~text
    OpBuilder<(ins "llvm::StringRef":$name, "mlir::Type":$type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3218

~~~~text
      "mlir::Attribute":$initVal, CArg<"mlir::StringAttr", "{}">:$linkage,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3219

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attrs)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3220

~~~~text
    OpBuilder<(ins "llvm::StringRef":$name, "bool":$isConstant,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3221

~~~~text
      "bool":$isTarget, "mlir::Type":$type, "mlir::Attribute":$initVal,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3222

~~~~text
      CArg<"mlir::StringAttr", "{}">:$linkage,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3223

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attrs)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3224

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3225

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3226

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3227

~~~~text
    /// The semantic type of the global
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3228

~~~~text
    mlir::Type resultType();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3229

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3230

~~~~text
    /// Return the initializer attribute if it exists, or a null attribute.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3231

~~~~text
    mlir::Attribute getValueOrNull() { return getInitVal().value_or(mlir::Attribute()); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3232

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3233

~~~~text
    /// Append the next initializer value to the `GlobalOp` to construct
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3234

~~~~text
    /// the variable's initial value.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3235

~~~~text
    void appendInitialValue(mlir::Operation *op);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3236

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3237

~~~~text
    /// A GlobalOp has one block.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3238

~~~~text
    mlir::Block &getBlock() { return getRegion().front(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3239

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3240

~~~~text
    /// Determine if `linkage` is a supported keyword
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3241

~~~~text
    static mlir::ParseResult verifyValidLinkage(llvm::StringRef linkage);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3242

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3243

~~~~text
    bool hasInitializationBody() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3244

~~~~text
      return ((*this)->getNumRegions() == 1) && !getRegion().empty() &&
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3245

~~~~text
        !mlir::isa<fir::FirEndOp>(getBlock().front());
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3246

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3247

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3248

~~~~text
    mlir::FlatSymbolRefAttr getSymbol() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3249

~~~~text
      return mlir::FlatSymbolRefAttr::get(getContext(),
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3250

~~~~text
          (*this)->getAttrOfType<mlir::StringAttr>(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3251

~~~~text
              mlir::SymbolTable::getSymbolAttrName()).getValue());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 3252

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3253

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3254

~~~~text
    bool isInitialized() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3255

~~~~text
      return getInitVal() || hasInitializationBody();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3256

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3257

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3258

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3259

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3260

~~~~text
def fir_GlobalLenOp : fir_Op<"global_len", []> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3261

~~~~text
  let summary = "map a LEN parameter to a global";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3262

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3263

~~~~text
    A global entity (that is not an automatic data object) can have extra LEN
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3264

~~~~text
    parameter (compile-time) constants associated with the instance's type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3265

~~~~text
    These values can be bound to the global instance used `fir.global_len`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3266

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3267

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3268

~~~~text
      global @g : !fir.type<t(len1:i32)> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3269

~~~~text
        fir.global_len len1, 10 : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3270

~~~~text
        %1 = fir.undefined !fir.type<t(len1:i32)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3271

~~~~text
        fir.has_value %1 : !fir.type<t(len1:i32)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3272

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3273

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3274

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3275

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3276

~~~~text
  let arguments = (ins StrAttr:$lenparam, APIntAttr:$intval);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3277

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3278

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3279

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3280

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3281

~~~~text
    static constexpr llvm::StringRef getLenParamAttrName() { return "lenparam"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3282

~~~~text
    static constexpr llvm::StringRef getIntAttrName() { return "intval"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3283

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3284

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3285

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3286

~~~~text
def fir_UseStmtOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3287

~~~~text
    : fir_Op<"use_stmt", [MemoryEffects<[MemWrite<DebuggingResource>]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3288

~~~~text
  let summary = "Represents a Fortran USE statement";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3289

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3290

~~~~text
    This operation records a Fortran USE statement with its associated only/rename
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3291

~~~~text
    information. It has no runtime effect but preserves semantic information for
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3292

~~~~text
    debug information generation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3293

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3294

~~~~text
    The operation captures:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3295

~~~~text
    - The module being used (via module_name string)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3296

~~~~text
    - Symbol references to symbols imported via the ONLY clause (if present)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3297

~~~~text
    - Symbol renames (local_name and symbol reference)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3298

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3299

~~~~text
    Examples:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3300

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3301

~~~~text
      // USE mod1
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3302

~~~~text
      fir.use_stmt "mod1"
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3303

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3304

~~~~text
      // USE mod1, ONLY: var2
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3305

~~~~text
      fir.use_stmt "mod1" only_symbols [@_QMmod1Evar2]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3306

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3307

~~~~text
      // USE mod2, var4 => var3
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3308

~~~~text
      fir.use_stmt "mod2" renames [#fir.use_rename<"var4", @_QMmod2Evar3>]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3309

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3310

~~~~text
      // USE mod2, ONLY: var1, renamed => original
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3311

~~~~text
      fir.use_stmt "mod2" only_symbols [@_QMmod2Evar1]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3312

~~~~text
                   renames [#fir.use_rename<"renamed", @_QMmod2Eoriginal>]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3313

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3314

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3315

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3316

~~~~text
  let arguments = (ins StrAttr:$module_name,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3317

~~~~text
      OptionalAttr<ArrayAttr>:$only_symbols, OptionalAttr<ArrayAttr>:$renames,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3318

~~~~text
      DefaultValuedAttr<BoolAttr, "false">:$hasOnlyWithRenames);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3319

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3320

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3321

~~~~text
    $module_name
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3322

~~~~text
    (`only_symbols` `[` $only_symbols^ `]`)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3323

~~~~text
    (`renames` `[` $renames^ `]`)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3324

~~~~text
    (`hasOnlyWithRenames` $hasOnlyWithRenames^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3325

~~~~text
    attr-dict
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3326

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3327

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3328

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3329

~~~~text
    /// Returns true if this is a USE with ONLY clause
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3330

~~~~text
    bool hasOnlyClause() { return getOnlySymbols().has_value(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3331

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3332

~~~~text
    /// Returns true if this has any renames
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3333

~~~~text
    bool hasRenames() { return getRenames().has_value(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3334

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3335

~~~~text
    /// Returns true if this imports the entire module (no ONLY clause)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3336

~~~~text
    bool importsAll() { return !(hasOnlyClause() || getHasOnlyWithRenames()); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3337

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3338

~~~~text
    /// Returns true if this USE statement has both ONLY clause and renames
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3339

~~~~text
    bool hasOnlyWithRenames() { return getHasOnlyWithRenames(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3340

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3341

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3342

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3343

~~~~text
def ImplicitFirTerminator : SingleBlockImplicitTerminator<"FirEndOp">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3344

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3345

~~~~text
def fir_ModuleDebugImportsOp : fir_Op<"module_debug_imports", [
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3346

~~~~text
    MemoryEffects<[MemWrite<DebuggingResource>]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3347

~~~~text
    IsolatedFromAbove,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3348

~~~~text
    ImplicitFirTerminator
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3349

~~~~text
  ]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3350

~~~~text
  let summary = "USE statements inside a Fortran module";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3351

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3352

~~~~text
    Records the Fortran `USE` statements that appear in a module's specification
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3353

~~~~text
    part for debug metadata.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3354

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3355

~~~~text
    The region holds one `fir.use_stmt` per `USE`, in program order. This
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3356

~~~~text
    operation has no runtime effect. It is emitted only when full debug
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3357

~~~~text
    information is requested.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3358

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3359

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3360

~~~~text
  let arguments = (ins StrAttr:$module_name);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3361

~~~~text
  let regions = (region SizedRegion<1>:$uses);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3362

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3363

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3364

~~~~text
    $module_name attr-dict-with-keyword $uses
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3365

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3366

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3367

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3368

~~~~text
def fir_TypeInfoOp : fir_Op<"type_info",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3369

~~~~text
    [IsolatedFromAbove, Symbol, ImplicitFirTerminator]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3370

~~~~text
  let summary = "Derived type information";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3371

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3372

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3373

~~~~text
    Define extra information about a !fir.type<> that represents
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3374

~~~~text
    a Fortran derived type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3375

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3376

~~~~text
    The optional dispatch table region defines a dispatch table with the derived
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3377

~~~~text
    type type-bound procedures. It contains a list of associations
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3378

~~~~text
    between method identifiers and corresponding `FuncOp` symbols.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3379

~~~~text
    The ordering of associations in the map is determined by the front end.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3380

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3381

~~~~text
    The "abstract" flag indicates that this type is an ABSTRACT derived type and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3382

~~~~text
    that it cannot be instantiated.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3383

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3384

~~~~text
    The "no_init" flag indicates that this type has no components requiring
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3385

~~~~text
    default initialization (including setting allocatable component to a clean
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3386

~~~~text
    deallocated state).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3387

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3388

~~~~text
    The "no_destroy" flag indicates that there are no allocatable components
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3389

~~~~text
    that require deallocation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3390

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3391

~~~~text
    The "no_final" flag indicates that there are no final methods for this type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3392

~~~~text
    for its parents ,or for components.
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 3393

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3394

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3395

~~~~text
      fir.type_info @_QMquuzTfoo abstract noinit nofinal
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3396

~~~~text
        : !fir.type<_QMquuzTfoo{i:i32}> dispatch_table {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3397

~~~~text
        fir.dt_entry method1, @_QFNMquuzTfooPmethod1AfooR
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3398

~~~~text
        fir.dt_entry method2, @_QFNMquuzTfooPmethod2AfooII
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3399

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3400

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3401

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3402

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3403

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3404

~~~~text
    SymbolNameAttr:$sym_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3405

~~~~text
    TypeAttr:$type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3406

~~~~text
    OptionalAttr<TypeAttr>:$parent_type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3407

~~~~text
    UnitAttr:$abstract,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3408

~~~~text
    UnitAttr:$no_init,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3409

~~~~text
    UnitAttr:$no_destroy,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3410

~~~~text
    UnitAttr:$no_final
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3411

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3412

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3413

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3414

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3415

~~~~text
  let regions = (region
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3416

~~~~text
    MaxSizedRegion<1>:$dispatch_table,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3417

~~~~text
    MaxSizedRegion<1>:$component_info
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3418

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3419

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3420

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3421

~~~~text
    OpBuilder<(ins "fir::RecordType":$type, "fir::RecordType":$parent_type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3422

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attrs)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3423

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3424

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3425

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3426

~~~~text
    $sym_name (`abstract` $abstract^)? (`noinit` $no_init^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3427

~~~~text
    (`nodestroy` $no_destroy^)? (`nofinal` $no_final^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3428

~~~~text
    (`extends` $parent_type^)? attr-dict `:` $type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3429

~~~~text
    (`dispatch_table` $dispatch_table^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3430

~~~~text
    (`component_info` $component_info^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3431

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3432

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3433

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3434

~~~~text
    fir::RecordType getRecordType() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3435

~~~~text
      return mlir::cast<fir::RecordType>(getType());
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3436

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3437

~~~~text
    fir::RecordType getIfParentType() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3438

~~~~text
      if (auto parentType = getParentType())
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 3439

~~~~text
        return mlir::cast<fir::RecordType>(*parentType);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3440

~~~~text
      return {};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3441

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3442

~~~~text
    std::optional<llvm::StringRef> getIfParentName() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3443

~~~~text
      if (auto parentType = getIfParentType())
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 3444

~~~~text
        return parentType.getName();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3445

~~~~text
      return std::nullopt;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3446

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3447

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3448

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3449

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3450

~~~~text
def fir_DTEntryOp : fir_Op<"dt_entry", [HasParent<"TypeInfoOp">]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3451

~~~~text
  let summary = "map entry in a dispatch table";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3452

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3453

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3454

~~~~text
    An entry in a dispatch table. Allows a function symbol to be bound
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3455

~~~~text
    to a specifier method identifier. A dispatch operation uses the dynamic
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3456

~~~~text
    type of a distinguished argument to determine an exact dispatch table
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3457

~~~~text
    and uses the method identifier to select the type-bound procedure to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3458

~~~~text
    be called.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3459

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3460

~~~~text
    The optional "deferred" flag indicates that the binding is a DEFERRED
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3461

~~~~text
    type-bound procedure (declared but without an implementation at this
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3462

~~~~text
    type level).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3463

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3464

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3465

~~~~text
      // Non-deferred binding
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3466

~~~~text
      fir.dt_entry method_name, @uniquedProcedure
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3467

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3468

~~~~text
      // Deferred binding
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3469

~~~~text
      fir.dt_entry method_name, @uniquedProcedure deferred
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3470

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3471

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3472

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3473

~~~~text
  let arguments = (ins StrAttr:$method, SymbolRefAttr:$proc, UnitAttr:$deferred);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3474

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3475

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3476

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3477

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3478

~~~~text
    static constexpr llvm::StringRef getProcAttrNameStr() { return "proc"; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3479

~~~~text
    static constexpr llvm::StringRef getDeferredAttrNameStr() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3480

~~~~text
      return "deferred";
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3481

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3482

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3483

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3484

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3485

~~~~text
def fir_DTComponentOp : fir_Op<"dt_component", [HasParent<"TypeInfoOp">]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3486

~~~~text
  let summary = "define extra information about a component inside fir.type_info";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3487

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3488

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3489

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3490

~~~~text
      fir.dt_component i lbs [-1,2] init @init_val
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3491

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3492

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3493

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3494

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3495

~~~~text
    StrAttr:$name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3496

~~~~text
    OptionalAttr<DenseI64ArrayAttr>:$lower_bounds,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3497

~~~~text
    OptionalAttr<FlatSymbolRefAttr>:$init_val
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3498

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3499

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3500

~~~~text
  let assemblyFormat = "$name (`lbs` $lower_bounds^)? (`init` $init_val^)? attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3501

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3502

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3503

~~~~text
def fir_AbsentOp : fir_OneResultOp<"absent", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3504

~~~~text
  let summary = "create value to be passed for absent optional function argument";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3505

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3506

~~~~text
    Given the type of a function argument, create a value that will signal that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3507

~~~~text
    an optional argument is absent in the call. On the caller side, fir.is_present
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3508

~~~~text
    can be used to query if the value of an optional argument was created with
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3509

~~~~text
    a fir.absent operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3510

~~~~text
    It is undefined to use a value that was created by a fir.absent op in any other
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3511

~~~~text
    operation than fir.call and fir.is_present.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3512

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3513

~~~~text
      %1 = fir.absent fir.box<fir.array<?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3514

~~~~text
      fir.call @_QPfoo(%1) : (fir.box<fir.array<?xf32>>) -> ()
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3515

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3516

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3517

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3518

~~~~text
  let results = (outs AnyRefOrBoxLike:$intype);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3519

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3520

~~~~text
  let assemblyFormat = "type($intype) attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3521

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3522

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3523

~~~~text
def fir_IsPresentOp : fir_SimpleOp<"is_present", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3524

~~~~text
  let summary = "is this optional function argument present?";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3525

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3526

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3527

~~~~text
    Determine if an optional function argument is PRESENT (i.e. that it was not
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3528

~~~~text
    created by a fir.absent op on the caller side).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3529

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3530

~~~~text
      func @_QPfoo(%arg0: !fir.box<!fir.array<?xf32>>) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3531

~~~~text
        %0 = fir.is_present %arg0 : (!fir.box<!fir.array<?xf32>>) -> i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3532

~~~~text
        ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3533

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3534

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3535

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3536

~~~~text
  let arguments = (ins AnyRefOrBoxLike:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3537

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3538

~~~~text
  let results = (outs BoolLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3539

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3540

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3541

~~~~text
// fir.declare leads to no codegen so the side effects implementation should be
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3542

~~~~text
// Pure. However, this would allow dead code elimination to remove these
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3543

~~~~text
// operations if the values are unused. fir.declare may be used to generate
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3544

~~~~text
// debug information so we would like to keep this around even if the value
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3545

~~~~text
// is not used.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3546

~~~~text
def fir_DeclareOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3547

~~~~text
    : fir_Op<"declare", [AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3548

~~~~text
                         MemoryEffects<[MemAlloc<DebuggingResource>]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3549

~~~~text
                         DeclareOpInterfaceMethods<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3550

~~~~text
                             fir_FortranVariableStorageOpInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3551

~~~~text
                         DeclareOpInterfaceMethods<PromotableOpInterface,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3552

~~~~text
                                                   ["requiresReplacedValues",
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3553

~~~~text
                                                    "visitReplacedValues"]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3554

~~~~text
                         fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3555

~~~~text
  let summary = "declare a variable";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3556

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3557

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3558

~~~~text
    Tie the properties of a Fortran variable to an address. The properties
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3559

~~~~text
    include bounds, length parameters, and Fortran attributes.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3560

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3561

~~~~text
    The memref argument describes the storage of the variable. It may be a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3562

~~~~text
    raw address (fir.ref<T>), or a box or class value or address (fir.box<T>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3563

~~~~text
    fir.ref<fir.box<T>>, fir.class<T>, fir.ref<fir.class<T>>).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3564

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3565

~~~~text
    The shape argument encodes explicit extents and lower bounds. It must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3566

~~~~text
    provided if the memref is the raw address of an array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3567

~~~~text
    The shape argument must not be provided if memref operand is a box or
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3568

~~~~text
    class value or address, unless the shape is a shift (encodes lower bounds)
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 3569

~~~~text
    and the memref if a box value (this covers assumed shapes with local lower
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3570

~~~~text
    bounds).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3571

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3572

~~~~text
    The typeparams values are meant to carry the non-deferred length parameters
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3573

~~~~text
    (this includes both Fortran assumed and explicit length parameters).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3574

~~~~text
    It must always be provided for characters and parametrized derived types
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3575

~~~~text
    when memref is not a box value or address.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3576

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3577

~~~~text
    The storage and storage_offset operands are optional and are required
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3578

~~~~text
    for FortranVariableStorageOpInterface, where they are documented.
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 3579

~~~~text
    If these operands are absent, then the storage of the declared variable
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3580

~~~~text
    is only known to start where the memref operand points to.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3581

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3582

~~~~text
    Example:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3583

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3584

~~~~text
    CHARACTER(n), OPTIONAL, TARGET :: c(10:, 20:)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3585

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3586

~~~~text
    Can be represented as:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3587

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3588

~~~~text
    func.func @foo(%arg0: !fir.box<!fir.array<?x?x!fir.char<1,?>>>, %arg1: !fir.ref<i64>) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 3589

~~~~text
      %c10 = arith.constant 10 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3590

~~~~text
      %c20 = arith.constant 20 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3591

~~~~text
      %1 = fir.load %ag1 : fir.ref<i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3592

~~~~text
      %2 = fir.shift %c10, %c20 : (index, index) -> !fir.shift<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3593

~~~~text
      %3 = fir.declare %arg0(%2) typeparams %1 {fortran_attrs = #fir.var_attrs<optional, target>, uniq_name = "c"}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3594

~~~~text
      // ... uses %3 as "c"
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3595

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3596

~~~~text
   ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3597

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3598

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3599

~~~~text
  let arguments = (ins AnyRefOrBox:$memref,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3600

~~~~text
      Optional<AnyShapeOrShiftType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3601

~~~~text
      Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3602

~~~~text
      Optional<fir_DummyScopeType>:$dummy_scope,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3603

~~~~text
      Optional<AnyReferenceLike>:$storage,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3604

~~~~text
      DefaultValuedAttr<UI64Attr, "0">:$storage_offset,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3605

~~~~text
      Builtin_StringAttr:$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3606

~~~~text
      OptionalAttr<fir_FortranVariableFlagsAttr>:$fortran_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3607

~~~~text
      OptionalAttr<cuf_DataAttributeAttr>:$data_attr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3608

~~~~text
      OptionalAttr<UI32Attr>:$dummy_arg_no);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3609

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3610

~~~~text
  let results = (outs AnyRefOrBox);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3611

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3612

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3613

~~~~text
    $memref (`(` $shape^ `)`)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3614

~~~~text
    (`dummy_scope` $dummy_scope^ (`arg` $dummy_arg_no^)?)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3615

~~~~text
    (`storage` `(` $storage^ `[` $storage_offset `]` `)`)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3616

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3617

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3618

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3619

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3620

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3621

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getMemref(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3622

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult) { return 0; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3623

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3624

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3625

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3626

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3627

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3628

~~~~text
def fir_DeclareValueOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3629

~~~~text
    : fir_Op<"declare_value", [MemoryEffects<[MemAlloc<DebuggingResource>]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3630

~~~~text
  let summary = "declare a value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3631

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3632

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3633

~~~~text
    Tie the properties of a simple scalar Fortran variable to a value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3634

~~~~text
    The value must be a scalar integer, real, complex, or logical.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3635

~~~~text
    This is the value based version of fir.declare. It is used to keep track
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3636

~~~~text
    of variable properties and debug info after mem2reg promotion.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3637

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3638

~~~~text
    Note that there is currently nothing that allows pinning this operation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3639

~~~~text
    relatively to the operation using the value of the variable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3640

~~~~text
    this means that the when a mem2reg variable was assigned several
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3641

~~~~text
    values, the variable value printed in the debugger at a given code
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3642

~~~~text
    point may not be the actual value the variable had at that code point
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3643

~~~~text
    if instructions using the value were moved above the fir.declare_value.
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 3644

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3645

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3646

~~~~text
  let arguments = (ins AnyType:$value,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3647

~~~~text
      Optional<fir_DummyScopeType>:$dummy_scope,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3648

~~~~text
      Builtin_StringAttr:$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3649

~~~~text
      OptionalAttr<fir_FortranVariableFlagsAttr>:$fortran_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3650

~~~~text
      OptionalAttr<cuf_DataAttributeAttr>:$data_attr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3651

~~~~text
      OptionalAttr<UI32Attr>:$dummy_arg_no);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3652

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3653

~~~~text
  let results = (outs);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3654

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3655

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3656

~~~~text
    $value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3657

~~~~text
    (`dummy_scope` $dummy_scope^ (`arg` $dummy_arg_no^)?)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3658

~~~~text
    attr-dict `:` type($value)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3659

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3660

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3661

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3662

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3663

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3664

~~~~text
def fir_BoxOffsetOp : fir_Op<"box_offset", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3665

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3666

~~~~text
  let summary = "Get the address of a field in a fir.ref<fir.box>";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3667

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3668

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3669

~~~~text
    Given the address of a fir.box, compute the address of a field inside
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3670

~~~~text
    the fir.box.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3671

~~~~text
    This allows keeping the actual runtime descriptor layout abstract in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3672

~~~~text
    FIR while providing access to the pointer addresses in the runtime
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3673

~~~~text
    descriptor for OpenMP/OpenACC target mapping.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3674

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3675

~~~~text
    To avoid requiring too much information about the fields that the runtime
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3676

~~~~text
    descriptor implementation must have, only the base_addr and derived_type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3677

~~~~text
    descriptor fields can be addressed.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3678

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3679

~~~~text
    It also accepts the address of a fir.boxchar and returns
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3680

~~~~text
    address of the data pointer encapsulated by the fir.boxchar.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3681

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3682

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3683

~~~~text
        %addr = fir.box_offset %box base_addr : (!fir.ref<!fir.box<!fir.array<?xi32>>>) -> !fir.llvm_ptr<!fir.ref<!fir.array<?xi32>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3684

~~~~text
        %tdesc = fir.box_offset %box derived_type : (!fir.ref<!fir.box<!fir.type<t>>>) -> !fir.llvm_ptr<!fir.tdesc<!fir.type<t>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3685

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3686

~~~~text
        %addr1 = fir.box_offset %boxchar base_addr : (!fir.ref<!fir.boxchar<1>>) -> !fir.llvm_ptr<!fir.ref<fir.char<1,?>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3687

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3688

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3689

~~~~text
    The derived_type field cannot be used when the input to this op is a reference to a fir.boxchar.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3690

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3691

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3692

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3693

~~~~text
    AnyReferenceLike:$box_ref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3694

~~~~text
    fir_BoxFieldAttr:$field
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3695

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3696

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3697

~~~~text
  let results = (outs RefOrLLVMPtr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3698

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3699

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3700

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3701

~~~~text
    $box_ref $field attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3702

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3703

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3704

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3705

~~~~text
    OpBuilder<(ins "mlir::Value":$boxRef, "fir::BoxFieldAttr":$field)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3706

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3707

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3708

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3709

~~~~text
def fir_DummyScopeOp : fir_Op<"dummy_scope",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3710

~~~~text
    [MemoryEffects<[MemWrite<DebuggingResource>]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3711

~~~~text
  let summary = "Define a scope for dummy arguments";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3712

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3713

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3714

~~~~text
    An abstract handle to be used to associate dummy arguments of the same
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3715

~~~~text
    subroutine between each other. By lowering, all [hl]fir.declare
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3716

~~~~text
    operations representing declarations of dummy arguments of a subroutine
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3717

~~~~text
    use the result of this operation. This allows recognizing the references
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3718

~~~~text
    of these dummy arguments as belonging to the same runtime instance
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3719

~~~~text
    of the subroutine even after MLIR inlining. Thus, the Fortran aliasing
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3720

~~~~text
    rules might be applied to those references based on the original
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3721

~~~~text
    declarations of the dummy arguments.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3722

~~~~text
    For example:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3723

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3724

~~~~text
      subroutine test(x, y)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3725

~~~~text
        real, target :: x, y
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3726

~~~~text
        x = y ! may alias
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3727

~~~~text
        call inner(x, y)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3728

~~~~text
      contains
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3729

~~~~text
        subroutine inner(x, y)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3730

~~~~text
          real :: x, y
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3731

~~~~text
          x = y ! may not alias
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3732

~~~~text
        end subroutine inner
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3733

~~~~text
      end subroutine test
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3734

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3735

~~~~text
    After MLIR inlining this may look like this:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3736

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3737

~~~~text
      func.func @_QPtest(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3738

~~~~text
          %arg0: !fir.ref<f32> {fir.target},
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3739

~~~~text
          %arg1: !fir.ref<f32> {fir.target}) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3740

~~~~text
        %0 = fir.declare %arg0 {fortran_attrs = #fir.var_attrs<target>} :
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3741

~~~~text
            (!fir.ref<f32>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3742

~~~~text
        %1 = fir.declare %arg1 {fortran_attrs = #fir.var_attrs<target>} :
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3743

~~~~text
            (!fir.ref<f32>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3744

~~~~text
        %2 = fir.load %1 : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3745

~~~~text
        fir.store %2 to %0 : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3746

~~~~text
        %3 = fir.declare %0 : (!fir.ref<f32>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3747

~~~~text
        %4 = fir.declare %1 : (!fir.ref<f32>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3748

~~~~text
        %5 = fir.load %4 : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3749

~~~~text
        fir.store %5 to %3 : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3750

~~~~text
        return
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3751

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3752

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3753

~~~~text
    Without marking %3 and %4 as declaring the dummy arguments
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3754

~~~~text
    of the same runtime instance of `inner` subroutine the FIR
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3755

~~~~text
    AliasAnalysis cannot deduce non-aliasing for the second load/store pair.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3756

~~~~text
    This information may be preserved by using fir.dummy_scope operation:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3757

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3758

~~~~text
      func.func @_QPtest(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3759

~~~~text
          %arg0: !fir.ref<f32> {fir.target},
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3760

~~~~text
          %arg1: !fir.ref<f32> {fir.target}) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3761

~~~~text
        %h1 = fir.dummy_scope : i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3762

~~~~text
        %0 = fir.declare %arg0 dummy_scope(%h1)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3763

~~~~text
            {fortran_attrs = #fir.var_attrs<target>} :
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3764

~~~~text
            (!fir.ref<f32>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3765

~~~~text
        %1 = fir.declare %arg1 dummy_scope(%h1)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3766

~~~~text
            {fortran_attrs = #fir.var_attrs<target>} :
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3767

~~~~text
            (!fir.ref<f32>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3768

~~~~text
        %2 = fir.load %1 : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3769

~~~~text
        fir.store %2 to %0 : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3770

~~~~text
        %h2 = fir.dummy_scope : i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3771

~~~~text
        %3 = fir.declare %0 dummy_scope(%h2) : (!fir.ref<f32>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3772

~~~~text
        %4 = fir.declare %1 dummy_scope(%h2) : (!fir.ref<f32>) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3773

~~~~text
        %5 = fir.load %4 : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3774

~~~~text
        fir.store %5 to %3 : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3775

~~~~text
        return
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 3776

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3777

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3778

~~~~text
    Note that even if `inner` is called and inlined twice inside
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3779

~~~~text
    `test`, the two inlined instances of `inner` must use two different
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3780

~~~~text
    fir.dummy_scope operations for their fir.declare ops. This
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3781

~~~~text
    two distinct fir.dummy_scope must remain distinct during the optimizations.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3782

~~~~text
    This is guaranteed by the write memory effect on the DebuggingResource.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3783

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3784

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3785

~~~~text
  let results = (outs fir_DummyScopeType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3786

~~~~text
  let assemblyFormat = "attr-dict `:` type(results)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3787

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3788

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3789

~~~~text
def fir_PackArrayOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3790

~~~~text
    : fir_Op<"pack_array", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3791

~~~~text
                            AllTypesMatch<["array", "result"]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3792

~~~~text
  let summary = "Pack non-contiguous array into a temporary";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3793

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3794

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3795

~~~~text
    The operation creates a new !fir.box/class<!fir.array<>> value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3796

~~~~text
    to represent either the original array or a newly allocated
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3797

~~~~text
    temporary array, maybe identical to the original array by value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3798

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3799

~~~~text
    Arguments:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3800

~~~~text
      - array is the original array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3801

~~~~text
        It must have !fir.box/class<!fir.array<>> type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3802

~~~~text
      - stack/heap attribute indicates where the temporary array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3803

~~~~text
        needs to be allocated.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3804

~~~~text
      - innermost/whole attribute identifies the contiguity mode.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3805

~~~~text
        innermost means that the repacking has to be done iff the original
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3806

~~~~text
        array is not contiguous in the leading dimension.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3807

~~~~text
        whole means that the repacking has to be done iff the original
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3808

~~~~text
        array is not contiguous in any dimension.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3809

~~~~text
        innermost is disallowed for 1D arrays in favor of whole.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3810

~~~~text
      - no_copy attribute indicates that the original array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3811

~~~~text
        is not copied into the temporary.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3812

~~~~text
      - typeparams specify the length parameters of the original array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3813

~~~~text
        Even though the array is fully represented with a box, the explicit
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3814

~~~~text
        length parameters might be specified to simplify computing
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3815

~~~~text
        the size of the array's element in compilation time (e.g. constant
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3816

~~~~text
        length parameters might be propagated after MLIR inlining).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3817

~~~~text
      - optional constraints attributes:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3818

~~~~text
        * max_size is an unsigned integer attribute specifying the maximum
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3819

~~~~text
          byte size of an array that is eligible for repacking.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3820

~~~~text
        * max_element_size is an unsigned integer attribute specifying
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3821

~~~~text
          the maximum byte element-size of an array that is eligible
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3822

~~~~text
          for repacking.
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 3823

~~~~text
        * min_stride is an unsigned integer attribute specifying
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3824

~~~~text
          the minimum byte stride of the innermost dimension of an array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3825

~~~~text
          that is eligible for repacking.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3826

~~~~text
      - heuristics attribute specifies conditions when the array repacking
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3827

~~~~text
        may be optimized.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3828

~~~~text
      - is_safe is an optional non-empty array of SafeTempArrayCopyAttr
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3829

~~~~text
        attributes. Each attribute implements SafeTempArrayCopyAttrInterface
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3830

~~~~text
        that is used to generate a dynamic predicate value identifying
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3831

~~~~text
        whether the creation of the temporary array copy is safe.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3832

~~~~text
        For example, if omp.fortran_safe_temp_array_copy attribute
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3833

~~~~text
        is attached, its implementation may generate special code
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3834

~~~~text
        to check if fir.pack_array may be executed by multiple
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3835

~~~~text
        threads, and disallow repacking in this case.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3836

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3837

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3838

~~~~text
  let arguments = (ins AnyBoxedArray:$array, UnitAttr:$stack,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3839

~~~~text
      UnitAttr:$innermost, UnitAttr:$no_copy, OptionalAttr<UI64Attr>:$max_size,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3840

~~~~text
      OptionalAttr<UI64Attr>:$max_element_size,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3841

~~~~text
      OptionalAttr<UI64Attr>:$min_stride,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3842

~~~~text
      DefaultValuedAttr<fir_PackArrayHeuristicsAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3843

~~~~text
                        "::fir::PackArrayHeuristics::None">:$heuristics,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3844

~~~~text
      Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3845

~~~~text
      OptionalAttr<NonEmptySafeTempArrayCopyArrayAttr>:$is_safe);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3846

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3847

~~~~text
  let results = (outs AnyBoxedArray:$result);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3848

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3849

~~~~text
    $array (`stack` $stack^):(`heap`)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3850

~~~~text
    (`innermost` $innermost^):(`whole`)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3851

~~~~text
    (`no_copy` $no_copy^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3852

~~~~text
    (`constraints` custom<PackArrayConstraints>($max_size, $max_element_size, $min_stride)^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3853

~~~~text
    (`heuristics` $heuristics^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3854

~~~~text
    (`typeparams` $typeparams^)? (`is_safe` $is_safe^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3855

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3856

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3857

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3858

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3859

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3860

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3861

~~~~text
def fir_UnpackArrayOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3862

~~~~text
    : fir_Op<"unpack_array", [SameTypeOperands,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3863

~~~~text
                              DeclareOpInterfaceMethods<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3864

~~~~text
                                  MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3865

~~~~text
  let summary = "Unpack values from temporary array into original array";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3866

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3867

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3868

~~~~text
    The operation is either a no-op or deallocates the temporary array,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3869

~~~~text
    and maybe copies the temporary array into the original array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3870

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3871

~~~~text
    Arguments:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3872

~~~~text
      - temp is a fir.box/fir.class value produced by fir.pack_array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3873

~~~~text
        It describes either the original array or the temporary array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3874

~~~~text
      - original is the original array descriptor.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3875

~~~~text
      - stack/heap attribute indicates where the temporary array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3876

~~~~text
        was allocated.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3877

~~~~text
      - no_copy attribute indicates that the temporary array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3878

~~~~text
        is not copied into the original temporary array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3879

~~~~text
      - is_safe is an optional non-empty array of SafeTempArrayCopyAttr
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3880

~~~~text
        attributes. Each attribute implements SafeTempArrayCopyAttrInterface
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3881

~~~~text
        that is used to generate extra code before any copy-out or
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3882

~~~~text
        deallocation of the temporary happens.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3883

~~~~text
        For example, if acc.fortran_safe_temp_array_copy attribute
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3884

~~~~text
        is attached, its implementation may generate special code
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3885

~~~~text
        to check if the temporary has been transferred to OpenACC device
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3886

~~~~text
        data environment, and issue a runtime error that the temporary
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3887

~~~~text
        is present on the device while it is about to be deallocated
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3888

~~~~text
        on the host.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3889

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3890

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3891

~~~~text
  let arguments = (ins AnyBoxedArray:$temp, AnyBoxedArray:$original,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3892

~~~~text
      UnitAttr:$stack, UnitAttr:$no_copy,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3893

~~~~text
      OptionalAttr<NonEmptySafeTempArrayCopyArrayAttr>:$is_safe);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3894

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3895

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3896

~~~~text
    $temp `to` $original
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3897

~~~~text
    (`stack` $stack^):(`heap`)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3898

~~~~text
    (`no_copy` $no_copy^)? (`is_safe` $is_safe^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3899

~~~~text
    attr-dict `:` type($original)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3900

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3901

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3902

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3903

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3904

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3905

~~~~text
def fir_IsContiguousBoxOp : fir_Op<"is_contiguous_box", [NoMemoryEffect]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3906

~~~~text
  let summary = "Returns true if the boxed entity is contiguous";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3907

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3908

~~~~text
    Returns true iff the boxed entity is contiguous:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3909

~~~~text
      * in the leading dimension (if `innermost` attribute is set),
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3910

~~~~text
      * in all dimensions (if `innermost` attribute is not set).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3911

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3912

~~~~text
    The input box cannot be absent.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3913

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3914

~~~~text
  let arguments = (ins AnyBoxLike:$box, UnitAttr:$innermost);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3915

~~~~text
  let results = (outs I1);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3916

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3917

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3918

~~~~text
    $box (`innermost` $innermost^):(`whole`)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3919

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3920

~~~~text
  let hasCanonicalizer = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3921

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3922

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3923

~~~~text
def fir_BoxTotalElementsOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3924

~~~~text
    : fir_SimpleOneResultOp<"box_total_elements", [NoMemoryEffect]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3925

~~~~text
  let summary = "Returns the boxed entity's total size in elements";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3926

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3927

~~~~text
    Returns the boxed entity's total size in elements.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3928

~~~~text
    The input box cannot be absent.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3929

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3930

~~~~text
  let arguments = (ins AnyBoxLike:$box);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3931

~~~~text
  let results = (outs AnyIntegerLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3932

~~~~text
  let hasCanonicalizer = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3933

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3934

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3935

~~~~text
def YieldOp : fir_Op<"yield",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3936

~~~~text
    [Pure, ReturnLike, Terminator,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3937

~~~~text
     ParentOneOf<["LocalitySpecifierOp", "DeclareReductionOp"]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3938

~~~~text
  let summary = "loop yield and termination operation";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3939

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3940

~~~~text
    "fir.yield" yields SSA values from a fir dialect op region and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3941

~~~~text
    terminates the region. The semantics of how the values are yielded is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3942

~~~~text
    defined by the parent operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3943

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3944

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3945

~~~~text
  let arguments = (ins Variadic<AnyType>:$results);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3946

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3947

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3948

~~~~text
    OpBuilder<(ins), [{ build($_builder, $_state, {}); }]>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3949

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 3950

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3951

~~~~text
  let assemblyFormat = "( `(` $results^ `:` type($results) `)` )? attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3952

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3953

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3954

~~~~text
def fir_LocalitySpecifierOp : fir_Op<"local", [IsolatedFromAbove]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 3955

~~~~text
  let summary = "Provides declaration of local and local_init logic.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3956

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 3957

~~~~text
    This operation provides a declaration of how to implement the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3958

~~~~text
    localization of a variable. The dialect users should provide
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3959

~~~~text
    which type should be allocated for this variable. The allocated (usually by
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3960

~~~~text
    alloca) variable is passed to the initialization region which does everything
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3961

~~~~text
    else (e.g. initialization of Fortran runtime descriptors). Information about
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3962

~~~~text
    how to initialize the copy from the original item should be given in the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3963

~~~~text
    copy region, and if needed, how to deallocate memory (allocated by the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3964

~~~~text
    initialization region) in the dealloc region.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3965

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3966

~~~~text
    Examples:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3967

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3968

~~~~text
    * `local(x)` would not need any regions because no initialization is
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3969

~~~~text
      required by the standard for i32 variables and this is not local_init.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3970

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3971

~~~~text
    fir.local {type = local} @x.localizer : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3972

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3973

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3974

~~~~text
    * `local_init(x)` would be emitted as:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3975

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3976

~~~~text
    fir.local {type = local_init} @x.localizer : i32 copy {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3977

~~~~text
    ^bb0(%arg0: !fir.ref<i32>, %arg1: !fir.ref<i32>):
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3978

~~~~text
    // %arg0 is the original host variable.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3979

~~~~text
    // %arg1 represents the memory allocated for this private variable.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3980

~~~~text
    ... copy from host to the localized clone ....
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3981

~~~~text
    fir.yield(%arg1 : !fir.ref<i32>)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3982

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 3983

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3984

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 3985

~~~~text
    * `local(x)` for "allocatables" would be emitted as:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3986

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3987

~~~~text
    fir.local {type = local} @x.localizer : !some.type init {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3988

~~~~text
    ^bb0(%arg0: !fir.ref<!some.type>, %arg1: !fir.ref<!some.type>):
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3989

~~~~text
    // initialize %arg1, using %arg0 as a mold for allocations.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3990

~~~~text
    // For example if %arg0 is a heap allocated array with a runtime determined
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3991

~~~~text
    // length and !some.type is a runtime type descriptor, the init region
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3992

~~~~text
    // will read the array length from %arg0, and heap allocate an array of the
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3993

~~~~text
    // right length and initialize %arg1 to contain the array allocation and
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3994

~~~~text
    // length.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3995

~~~~text
    fir.yield(%arg1 : !fir.ref<!some.type>)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3996

~~~~text
    } dealloc {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 3997

~~~~text
    ^bb0(%arg0: !fir.ref<!some.type>):
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 3998

~~~~text
    // ... deallocate memory allocated by the init region...
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 3999

~~~~text
    // In the example above, this will free the heap allocated array data.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4000

~~~~text
    fir.yield
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4001

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4002

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4003

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4004

~~~~text
    There are no restrictions on the body except for:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4005

~~~~text
    - The `dealloc` regions has a single argument.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4006

~~~~text
    - The `init` & `copy` regions have 2 arguments.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4007

~~~~text
    - All three regions are terminated by `fir.yield` ops.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4008

~~~~text
    The above restrictions and other obvious restrictions (e.g. verifying the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4009

~~~~text
    type of yielded values) are verified by the custom op verifier. The actual
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4010

~~~~text
    contents of the blocks inside all regions are not verified.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4011

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4012

~~~~text
    Instances of this op would then be used by ops that model directives that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4013

~~~~text
    accept data-sharing attribute clauses.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4014

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4015

~~~~text
    The `sym_name` attribute provides a symbol by which the privatizer op can be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4016

~~~~text
    referenced by other dialect ops.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4017

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4018

~~~~text
    The `type` attribute is the type of the value being localized. This type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4019

~~~~text
    will be implicitly allocated in MLIR->LLVMIR conversion and passed as the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4020

~~~~text
    second argument to the init region. Therefore the type of arguments to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4021

~~~~text
    the regions should be a type which represents a pointer to `type`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4022

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4023

~~~~text
    The `locality_specifier_type` attribute specifies whether the localized
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4024

~~~~text
    corresponds to a `local` or a `local_init` specifier.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4025

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4026

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4027

~~~~text
  let arguments = (ins SymbolNameAttr:$sym_name,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4028

~~~~text
                       TypeAttrOf<AnyType>:$type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4029

~~~~text
                       LocalitySpecifierTypeAttr:$locality_specifier_type);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4030

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4031

~~~~text
  let regions = (region AnyRegion:$init_region,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4032

~~~~text
                        AnyRegion:$copy_region,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4033

~~~~text
                        AnyRegion:$dealloc_region);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4034

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4035

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4036

~~~~text
    $locality_specifier_type $sym_name `:` $type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4037

~~~~text
      (`init` $init_region^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4038

~~~~text
      (`copy` $copy_region^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4039

~~~~text
      (`dealloc` $dealloc_region^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4040

~~~~text
      attr-dict
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4041

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4042

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4043

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4044

~~~~text
    mlir::BlockArgument getInitMoldArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4045

~~~~text
      auto &region = getInitRegion();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4046

~~~~text
      return region.empty() ? nullptr : region.getArgument(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4047

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4048

~~~~text
    mlir::BlockArgument getInitPrivateArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4049

~~~~text
      auto &region = getInitRegion();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4050

~~~~text
      return region.empty() ? nullptr : region.getArgument(1);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4051

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4052

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4053

~~~~text
    /// Returns true if the init region might read from the mold argument
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4054

~~~~text
    bool initReadsFromMold() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4055

~~~~text
      mlir::BlockArgument moldArg = getInitMoldArg();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4056

~~~~text
      return moldArg && !moldArg.use_empty();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4057

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4058

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4059

~~~~text
    /// Get the type for arguments to nested regions. This should
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4060

~~~~text
    /// generally be either the same as getType() or some pointer
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4061

~~~~text
    /// type (pointing to the type allocated by this op).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4062

~~~~text
    /// This method will return Type{nullptr} if there are no nested
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4063

~~~~text
    /// regions.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4064

~~~~text
    mlir::Type getArgType() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4065

~~~~text
      for (mlir::Region *region : getRegions())
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 4066

~~~~text
        for (mlir::Type ty : region->getArgumentTypes())
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 4067

~~~~text
          return ty;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4068

~~~~text
      return nullptr;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4069

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4070

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4071

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4072

~~~~text
  let hasRegionVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4073

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4074

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4075

~~~~text
def fir_DeclareReductionOp : fir_Op<"declare_reduction", [IsolatedFromAbove,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 4076

~~~~text
                                                         Symbol]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 4077

~~~~text
  let summary = "declares a reduction kind";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4078

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4079

~~~~text
    Note: this operation is adapted from omp::DeclareReductionOp. There is a lot
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4080

~~~~text
    duplication at the moment. TODO Combine both ops into one. See:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4081

~~~~text
    https://discourse.llvm.org/t/dialect-for-data-locality-sharing-specifiers-clauses-in-openmp-openacc-and-do-concurrent/86108.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4082

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4083

~~~~text
    Declares a `do concurrent` reduction. This requires two mandatory and four 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4084

~~~~text
    optional regions.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4085

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4086

~~~~text
      1. The optional alloc region specifies how to allocate the thread-local
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4087

~~~~text
         reduction value. This region should not contain control flow and all
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4088

~~~~text
         IR should be suitable for inlining straight into an entry block. In
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4089

~~~~text
         the common case this is expected to contain only allocas. It is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4090

~~~~text
         expected to `fir.yield` the allocated value on all control paths.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4091

~~~~text
         If allocation is conditional (e.g. only allocate if the mold is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4092

~~~~text
         allocated), this should be done in the initilizer region and this
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4093

~~~~text
         region not included. The alloc region is not used for by-value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4094

~~~~text
         reductions (where allocation is implicit).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4095

~~~~text
      2. The initializer region specifies how to initialize the thread-local
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4096

~~~~text
         reduction value. This is usually the neutral element of the reduction.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4097

~~~~text
         For convenience, the region has an argument that contains the value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4098

~~~~text
         of the reduction accumulator at the start of the reduction. If an alloc
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4099

~~~~text
         region is specified, there is a second block argument containing the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4100

~~~~text
         address of the allocated memory. The initializer region is expected to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4101

~~~~text
         `fir.yield` the new value on all control flow paths.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4102

~~~~text
      3. The reduction region specifies how to combine two values into one, i.e.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4103

~~~~text
         the reduction operator. It accepts the two values as arguments and is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4104

~~~~text
         expected to `fir.yield` the combined value on all control flow paths.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4105

~~~~text
      4. The atomic reduction region is optional and specifies how two values
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4106

~~~~text
         can be combined atomically given local accumulator variables. It is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4107

~~~~text
         expected to store the combined value in the first accumulator variable.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4108

~~~~text
      5. The cleanup region is optional and specifies how to clean up any memory
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4109

~~~~text
         allocated by the initializer region. The region has an argument that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4110

~~~~text
         contains the value of the thread-local reduction accumulator. This will
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4111

~~~~text
         be executed after the reduction has completed.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4112

~~~~text
      6. The DataPtrPtr region specifies how to access the base address of a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4113

~~~~text
         boxed-value. This is used, in particular, for GPU reductions in order
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4114

~~~~text
         know where partial reduction results are stored in remote lanes.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4115

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4116

~~~~text
    Note that the MLIR type system does not allow for type-polymorphic
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4117

~~~~text
    reductions. Separate reduction declarations should be created for different
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4118

~~~~text
    element and accumulator types.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4119

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4120

~~~~text
    For initializer and reduction regions, the operand to `fir.yield` must
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4121

~~~~text
    match the parent operation's results.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4122

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4123

~~~~text
    * `$byref_element_type`: For by-ref reductions, we want to keep track of the
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4124

~~~~text
      boxed/allocated type.  For example, for a `real, allocatable` variable, 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4125

~~~~text
      `real` should be stored in this attribute.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4126

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4127

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4128

~~~~text
  let arguments = (ins SymbolNameAttr:$sym_name,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4129

~~~~text
                       TypeAttr:$type,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4130

~~~~text
                       OptionalAttr<TypeAttr>:$byref_element_type);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4131

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4132

~~~~text
  let regions = (region MaxSizedRegion<1>:$allocRegion,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4133

~~~~text
                        AnyRegion:$initializerRegion,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4134

~~~~text
                        AnyRegion:$reductionRegion,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4135

~~~~text
                        AnyRegion:$atomicReductionRegion,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4136

~~~~text
                        AnyRegion:$cleanupRegion,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4137

~~~~text
                        AnyRegion:$dataPtrPtrRegion);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4138

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4139

~~~~text
  let assemblyFormat = "$sym_name `:` $type attr-dict-with-keyword "
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4140

~~~~text
                       "( `alloc` $allocRegion^ )? "
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4141

~~~~text
                       "`init` $initializerRegion "
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4142

~~~~text
                       "`combiner` $reductionRegion "
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4143

~~~~text
                       "( `atomic` $atomicReductionRegion^ )? "
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4144

~~~~text
                       "( `cleanup` $cleanupRegion^ )? "
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4145

~~~~text
                       "( `data_ptr_ptr` $dataPtrPtrRegion^ )? ";
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4146

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4147

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4148

~~~~text
    mlir::BlockArgument getAllocMoldArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4149

~~~~text
      auto &region = getAllocRegion();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4150

~~~~text
      return region.empty() ? nullptr : region.getArgument(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4151

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4152

~~~~text
    mlir::BlockArgument getInitializerMoldArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4153

~~~~text
      return getInitializerRegion().getArgument(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4154

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4155

~~~~text
    mlir::BlockArgument getInitializerAllocArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4156

~~~~text
      return getAllocRegion().empty() ?
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4157

~~~~text
          nullptr : getInitializerRegion().getArgument(1);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4158

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4159

~~~~text
    mlir::BlockArgument getReductionLhsArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4160

~~~~text
      return getReductionRegion().getArgument(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4161

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4162

~~~~text
    mlir::BlockArgument getReductionRhsArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4163

~~~~text
      return getReductionRegion().getArgument(1);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4164

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4165

~~~~text
    mlir::BlockArgument getAtomicReductionLhsArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4166

~~~~text
      auto &region = getAtomicReductionRegion();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4167

~~~~text
      return region.empty() ? nullptr : region.getArgument(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4168

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4169

~~~~text
    mlir::BlockArgument getAtomicReductionRhsArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4170

~~~~text
      auto &region = getAtomicReductionRegion();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4171

~~~~text
      return region.empty() ? nullptr : region.getArgument(1);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4172

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4173

~~~~text
    mlir::BlockArgument getCleanupAllocArg() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4174

~~~~text
      auto &region = getCleanupRegion();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4175

~~~~text
      return region.empty() ? nullptr : region.getArgument(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4176

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4177

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4178

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4179

~~~~text
  let hasRegionVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4180

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4181

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4182

~~~~text
def fir_DoConcurrentOp : fir_Op<"do_concurrent",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 4183

~~~~text
    [SingleBlock, AutomaticAllocationScope]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 4184

~~~~text
  let summary = "do concurrent loop wrapper";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4185

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4186

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4187

~~~~text
    A wrapper operation for the actual op modeling `do concurrent` loops:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4188

~~~~text
    `fir.do_concurrent.loop` (see op declaration below for more info about it).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4189

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4190

~~~~text
    The `fir.do_concurrent` wrapper op consists of one single-block region with
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4191

~~~~text
    the following properties:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4192

~~~~text
    - The first ops in the region are responsible for allocating storage for the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4193

~~~~text
      loop's iteration variables. This is property is **not** enforced by the op
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4194

~~~~text
      verifier, but expected to be respected when building the op.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4195

~~~~text
    - The terminator of the region is an instance of `fir.do_concurrent.loop`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4196

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4197

~~~~text
    For example, a 2D loop nest would be represented as follows:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4198

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4199

~~~~text
    fir.do_concurrent {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 4200

~~~~text
      %i = fir.alloca i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4201

~~~~text
      %j = fir.alloca i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4202

~~~~text
      fir.do_concurrent.loop ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4203

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4204

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4205

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4206

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4207

~~~~text
  let regions = (region SizedRegion<1>:$region);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4208

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4209

~~~~text
  let assemblyFormat = "$region attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4210

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4211

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4212

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4213

~~~~text
def fir_LocalSpecifier {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 4214

~~~~text
  dag arguments = (ins
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 4215

~~~~text
    Variadic<AnyType>:$local_vars,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4216

~~~~text
    OptionalAttr<SymbolRefArrayAttr>:$local_syms
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4217

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4218

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4219

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4220

~~~~text
def fir_ReduceSpecifier {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 4221

~~~~text
  dag arguments = (ins
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 4222

~~~~text
    Variadic<AnyType>:$reduce_vars,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4223

~~~~text
    OptionalAttr<DenseBoolArrayAttr>:$reduce_byref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4224

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4225

~~~~text
    // This introduces redundency in how reductions are modelled. In particular,
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4226

~~~~text
    // a single reduction is represented by 2 attributes:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4227

~~~~text
    //
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 4228

~~~~text
    // 1. `$reduce_syms` which is a list of `DeclareReductionOp`s.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4229

~~~~text
    // 2. `$reduce_attrs` which is an array of `fir::ReduceAttr` values.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4230

~~~~text
    //
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 4231

~~~~text
    // The first makes it easier to map `do concurrent` to parallization models
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4232

~~~~text
    // (e.g. OpenMP and OpenACC) while the second makes it easier to map it to
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4233

~~~~text
    // nests of `fir.do_loop ... unodered` ops.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4234

~~~~text
    OptionalAttr<SymbolRefArrayAttr>:$reduce_syms,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4235

~~~~text
    OptionalAttr<ArrayAttr>:$reduce_attrs
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4236

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4237

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4238

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4239

~~~~text
def fir_DoConcurrentLoopOp : fir_Op<"do_concurrent.loop",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 4240

~~~~text
    [AttrSizedOperandSegments, DeclareOpInterfaceMethods<LoopLikeOpInterface,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4241

~~~~text
                                                         ["getLoopInductionVars"]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4242

~~~~text
     Terminator, NoTerminator, SingleBlock, ParentOneOf<["DoConcurrentOp"]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 4243

~~~~text
  let summary = "do concurrent loop";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4244

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4245

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4246

~~~~text
    An operation that models a Fortran `do concurrent` loop's header and block.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4247

~~~~text
    This is a single-region single-block terminator op that is expected to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4248

~~~~text
    terminate the region of a `fir.do_concurrent` wrapper op.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4249

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4250

~~~~text
    This op borrows from both `scf.parallel` and `fir.do_loop` ops. Similar to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4251

~~~~text
    `scf.parallel`, a loop nest takes 3 groups of SSA values as operands that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4252

~~~~text
    represent the lower bounds, upper bounds, and steps. Similar to `fir.do_loop`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4253

~~~~text
    the op takes one additional group of SSA values to represent reductions.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4254

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4255

~~~~text
    The body region **does not** have a terminator.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4256

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4257

~~~~text
    For example, a 2D loop nest with 2 reductions (sum and max) would be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4258

~~~~text
    represented as follows:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4259

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4260

~~~~text
    // The wrapper of the loop
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4261

~~~~text
    fir.do_concurrent {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 4262

~~~~text
      %i = fir.alloca i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4263

~~~~text
      %j = fir.alloca i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4264

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4265

~~~~text
      // The actual `do concurrent` loop
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4266

~~~~text
      fir.do_concurrent.loop
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4267

~~~~text
        (%i_iv, %j_iv) = (%i_lb, %j_lb) to (%i_ub, %j_ub) step (%i_st, %j_st)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4268

~~~~text
        reduce(#fir.reduce_attr<add> -> %sum : !fir.ref<i32>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4269

~~~~text
               #fir.reduce_attr<max> -> %max : !fir.ref<f32>) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 4270

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4271

~~~~text
        %0 = fir.convert %i_iv : (index) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4272

~~~~text
        fir.store %0 to %i : !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4273

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4274

~~~~text
        %1 = fir.convert %j_iv : (index) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4275

~~~~text
        fir.store %1 to %j : !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4276

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4277

~~~~text
        // ... loop body goes here ...
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4278

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4279

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4280

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4281

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4282

~~~~text
    Description of arguments:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4283

~~~~text
    - `lowerBound`: The group of SSA values for the nest's lower bounds.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4284

~~~~text
    - `upperBound`: The group of SSA values for the nest's upper bounds.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4285

~~~~text
    - `step`: The group of SSA values for the nest's steps.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4286

~~~~text
    - `loopAnnotation`: Loop metadata to be passed down the compiler pipeline to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4287

~~~~text
      LLVM.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4288

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4289

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4290

~~~~text
  defvar opArgs = (ins
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 4291

~~~~text
    Variadic<Index>:$lowerBound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4292

~~~~text
    Variadic<Index>:$upperBound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4293

~~~~text
    Variadic<Index>:$step,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4294

~~~~text
    OptionalAttr<LoopAnnotationAttr>:$loopAnnotation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4295

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4296

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4297

~~~~text
  let arguments = !con(opArgs,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4298

~~~~text
    fir_LocalSpecifier.arguments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4299

~~~~text
    fir_ReduceSpecifier.arguments);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4300

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4301

~~~~text
  let regions = (region SizedRegion<1>:$region);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4302

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4303

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4304

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4305

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4306

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 4307

~~~~text
    unsigned getNumInductionVars() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4308

~~~~text
      return getLowerBound().size();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4309

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4310

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4311

~~~~text
    unsigned getNumLocalOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4312

~~~~text
      return getLocalVars().size();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4313

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4314

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4315

~~~~text
    unsigned getNumReduceOperands() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4316

~~~~text
      return getReduceVars().size();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4317

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4318

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4319

~~~~text
    unsigned getInductionVarsStart() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4320

~~~~text
      return 0;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4321

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4322

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4323

~~~~text
    unsigned getLocalOperandsStart() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4324

~~~~text
      return getNumInductionVars();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4325

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4326

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4327

~~~~text
    unsigned getReduceOperandsStart() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4328

~~~~text
      return getLocalOperandsStart() + getNumLocalOperands();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4329

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4330

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4331

~~~~text
    mlir::Block::BlockArgListType getInductionVars() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4332

~~~~text
      return getBody()->getArguments().slice(0, getNumInductionVars());
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4333

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4334

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4335

~~~~text
    mlir::Block::BlockArgListType getRegionLocalArgs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4336

~~~~text
      return getBody()->getArguments().slice(getNumInductionVars(),
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4337

~~~~text
                                             getNumLocalOperands());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4338

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4339

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4340

~~~~text
    mlir::Block::BlockArgListType getRegionReduceArgs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 4341

~~~~text
      return getBody()->getArguments().slice(getNumInductionVars()
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 4342

~~~~text
                                               + getNumLocalOperands(),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4343

~~~~text
                                             getNumReduceOperands());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 4344

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4345

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4346

~~~~text
    /// Number of operands controlling the loop
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 4347

~~~~text
    unsigned getNumControlOperands() { return getLowerBound().size() * 3; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 4348

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4349

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 4350

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 4351

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 4352

~~~~text
#endif
~~~~
- EN: Controls conditional compilation with `#endif`.
- CN: 使用 `#endif` 控制条件编译。

## Key Concepts / 关键概念

- **Declarative option data / 声明式选项数据**: The file encodes structured data that downstream tooling can consume or generate from. / 该文件编码了结构化数据，供下游工具消费或据此生成输出。
- **Preprocessor coordination / 预处理器协作**: Preprocessor directives shape how the file is configured or expanded. / 预处理指令决定文件的配置方式或展开方式。
- **Type modeling / 类型建模**: Classes, structs, or enums capture domain concepts in a typed form. / 类、结构体或枚举以类型化方式表达领域概念。
- **Flang/LLVM integration / Flang/LLVM 集成**: Direct project headers show which nearby subsystems this file collaborates with. / 直接包含的工程头文件表明该文件与哪些相邻子系统协作。

## Dependencies / 依赖关系

- **Internal project dependencies / 工程内依赖**:
  - `mlir/Dialect/Arith/IR/ArithBase.td` — referenced directly from this file / 该文件直接引用
  - `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td` — referenced directly from this file / 该文件直接引用
  - `mlir/Dialect/LLVMIR/LLVMAttrDefs.td` — referenced directly from this file / 该文件直接引用
  - `mlir/Interfaces/MemorySlotInterfaces.td` — referenced directly from this file / 该文件直接引用
  - `mlir/Interfaces/ViewLikeInterface.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/CUF/Attributes/CUFAttr.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FIRDialect.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FIRTypes.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FIRAttr.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FortranVariableInterface.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FirAliasTagOpInterface.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/SafeTempArrayCopyAttrInterface.td` — referenced directly from this file / 该文件直接引用
  - `mlir/IR/BuiltinAttributes.td` — referenced directly from this file / 该文件直接引用
- **External or standard dependencies / 外部或标准依赖**: none detected directly / 未直接检测到
