# HLFIROps.td — Code Analysis / 代码分析

## Source / 来源

- File: `flang/include/flang/Optimizer/HLFIR/HLFIROps.td`
- Repository: `llvm-project` / `llvm-project`
- Purpose (EN): HLFIR operation definitions \file Definition of the HLFIR dialect operations.
- Purpose (CN): 声明与 HLFIROps 相关的接口、数据结构或工具。

## Line-by-Line Analysis / 逐行分析

### Line 1

~~~~text
//===-- HLFIROps.td - HLFIR operation definitions ----------*- tablegen -*-===//
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
/// Definition of the HLFIR dialect operations
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
#ifndef FORTRAN_DIALECT_HLFIR_OPS
~~~~
- EN: Controls conditional compilation with `#ifndef`.
- CN: 使用 `#ifndef` 控制条件编译。

### Line 15

~~~~text
#define FORTRAN_DIALECT_HLFIR_OPS
~~~~
- EN: Defines the preprocessor macro `FORTRAN_DIALECT_HLFIR_OPS`.
- CN: 定义预处理宏 `FORTRAN_DIALECT_HLFIR_OPS`。

### Line 16

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 17

~~~~text
include "flang/Optimizer/HLFIR/HLFIROpBase.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/HLFIR/HLFIROpBase.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/HLFIR/HLFIROpBase.td`，以便在此复用其中的记录。

### Line 18

~~~~text
include "flang/Optimizer/Dialect/CUF/Attributes/CUFAttr.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/CUF/Attributes/CUFAttr.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/CUF/Attributes/CUFAttr.td`，以便在此复用其中的记录。

### Line 19

~~~~text
include "flang/Optimizer/Dialect/FIRTypes.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FIRTypes.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FIRTypes.td`，以便在此复用其中的记录。

### Line 20

~~~~text
include "flang/Optimizer/Dialect/FIRAttr.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FIRAttr.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FIRAttr.td`，以便在此复用其中的记录。

### Line 21

~~~~text
include "flang/Optimizer/Dialect/FortranVariableInterface.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FortranVariableInterface.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FortranVariableInterface.td`，以便在此复用其中的记录。

### Line 22

~~~~text
include "mlir/Dialect/Arith/IR/ArithBase.td"
~~~~
- EN: Includes the TableGen file `mlir/Dialect/Arith/IR/ArithBase.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/Dialect/Arith/IR/ArithBase.td`，以便在此复用其中的记录。

### Line 23

~~~~text
include "mlir/Dialect/Arith/IR/ArithOpsInterfaces.td"
~~~~
- EN: Includes the TableGen file `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td`，以便在此复用其中的记录。

### Line 24

~~~~text
include "mlir/IR/BuiltinAttributes.td"
~~~~
- EN: Includes the TableGen file `mlir/IR/BuiltinAttributes.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/IR/BuiltinAttributes.td`，以便在此复用其中的记录。

### Line 25

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 26

~~~~text
// Base class for FIR operations.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 27

~~~~text
// All operations automatically get a prefix of "hlfir.".
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 28

~~~~text
class hlfir_Op<string mnemonic, list<Trait> traits>
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 29

~~~~text
  : Op<hlfir_Dialect, mnemonic, traits>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 30

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 31

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 32

~~~~text
// DeclareOp can be lowered to EmboxOp, EmboxCharOp, ReboxOp, etc and so could
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 33

~~~~text
// generate code. All of the operations it can generate are modelled with
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 34

~~~~text
// NoMemoryEffect. However, if hlfir.declare is given NoMemoryEffect, it can be
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 35

~~~~text
// removed by dead code elimination if the value result is unused. Information
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 36

~~~~text
// from the declare operation can be used to generate debug information so we
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 37

~~~~text
// don't want to remove it as dead code
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 38

~~~~text
def hlfir_DeclareOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 39

~~~~text
    : hlfir_Op<"declare", [AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 40

~~~~text
                           MemoryEffects<[MemAlloc<DebuggingResource>]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 41

~~~~text
                           DeclareOpInterfaceMethods<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 42

~~~~text
                               fir_FortranVariableStorageOpInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 43

~~~~text
                           fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 44

~~~~text
  let summary = "declare a variable and produce an SSA value that can be used as a variable in HLFIR operations";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 45

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 46

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 47

~~~~text
    Tie the properties of a Fortran variable to an address. The properties
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 48

~~~~text
    include bounds, length parameters, and Fortran attributes.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 49

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 50

~~~~text
    The arguments are the same as for fir.declare.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 51

~~~~text
    The storage and storage_offset operands are optional and are required
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 52

~~~~text
    for FortranVariableStorageOpInterface, where they are documented.
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 53

~~~~text
    If these operands are absent, then the storage of the declared variable
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 54

~~~~text
    is only known to start where the memref operand points to.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 55

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 56

~~~~text
    The main difference with fir.declare is that hlfir.declare returns two
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 57

~~~~text
    values:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 58

~~~~text
      - the first one is an SSA value that allows retrieving the variable
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 59

~~~~text
        address, bounds, and type parameters at any point without requiring
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 60

~~~~text
        access to the defining operation. This may be:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 61

~~~~text
        - for scalar numerical, logical, or derived type without length
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 62

~~~~text
          parameters: a fir.ref<T> (e.g. fir.ref<i32>)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 63

~~~~text
        - for scalar characters: a fir.boxchar<kind> or fir.ref<fir.char<kind,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 64

~~~~text
          cst_len>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 65

~~~~text
        - for arrays of types without length parameters, without lower bounds,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 66

~~~~text
          that are not polymorphic and with a constant shape:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 67

~~~~text
          fir.ref<fir.array<cst_shapexT>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 68

~~~~text
        - for all non pointer/non allocatable entities: fir.box<T>, and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 69

~~~~text
          fir.class<T> for polymorphic entities.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 70

~~~~text
        - for all pointers/allocatables:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 71

~~~~text
          fir.ref<fir.box<fir.ptr<T>>>/fir.ref<fir.box<fir.heap<T>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 72

~~~~text
      - the second value has the same type as the input memref, and is the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 73

~~~~text
        same. If it is a fir.box or fir.class, it may not contain accurate
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 74

~~~~text
        local lower bound values. It is intended to be used when generating FIR
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 75

~~~~text
        from HLFIR in order to avoid descriptor creation for simple entities.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 76

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 77

~~~~text
    The attribute skip_rebox can be set to indicate that the second and first
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 78

~~~~text
    result are known to be the same descriptors (the input descriptor is known
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 79

~~~~text
    to already have the correct attributes and lower bounds).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 80

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 81

~~~~text
    Example:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 82

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 83

~~~~text
    CHARACTER(n) :: c(10:n, 20:n)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 84

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 85

~~~~text
    Can be represented as:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 86

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 87

~~~~text
    func.func @foo(%arg0: !fir.ref<!fir.array<?x?x!fir.char<1,?>>>, %arg1: !fir.ref<i64>) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 88

~~~~text
      %c10 = arith.constant 10 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 89

~~~~text
      %c20 = arith.constant 20 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 90

~~~~text
      %1 = fir.load %ag1 : fir.ref<i64>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 91

~~~~text
      %2 = fir.shape_shift %c10, %1, %c20, %1 : (index, index, index, index) -> !fir.shapeshift<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 92

~~~~text
      %3 = hfir.declare %arg0(%2) typeparams %1 {uniq_name = "c"} (fir.ref<!fir.array<?x?x!fir.char<1,?>>>, fir.shapeshift<2>, index) -> (fir.box<!fir.array<?x?x!fir.char<1,?>>>, fir.ref<!fir.array<?x?x!fir.char<1,?>>>)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 93

~~~~text
      // ... uses %3#0 as "c"
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 94

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 95

~~~~text
   ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 96

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 97

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 98

~~~~text
  let arguments = (ins AnyRefOrBox:$memref,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 99

~~~~text
      Optional<AnyShapeOrShiftType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 100

~~~~text
      Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 101

~~~~text
      Optional<fir_DummyScopeType>:$dummy_scope,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 102

~~~~text
      Optional<AnyReferenceLike>:$storage,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 103

~~~~text
      DefaultValuedAttr<UI64Attr, "0">:$storage_offset,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 104

~~~~text
      Builtin_StringAttr:$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 105

~~~~text
      OptionalAttr<fir_FortranVariableFlagsAttr>:$fortran_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 106

~~~~text
      OptionalAttr<cuf_DataAttributeAttr>:$data_attr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 107

~~~~text
      OptionalAttr<UnitAttr>:$skip_rebox, OptionalAttr<UI32Attr>:$dummy_arg_no);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 108

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 109

~~~~text
  let results = (outs AnyFortranVariable, AnyRefOrBoxLike);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 110

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 111

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 112

~~~~text
    $memref (`(` $shape^ `)`)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 113

~~~~text
    (`dummy_scope` $dummy_scope^ (`arg` $dummy_arg_no^)?)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 114

~~~~text
    (`storage` `(` $storage^ `[` $storage_offset `]` `)`)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 115

~~~~text
    (`skip_rebox` $skip_rebox^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 116

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 117

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 118

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 119

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$memref,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 120

~~~~text
      "llvm::StringRef":$uniq_name, CArg<"mlir::Value", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 121

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 122

~~~~text
      CArg<"mlir::Value", "{}">:$dummy_scope,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 123

~~~~text
      CArg<"mlir::Value", "{}">:$storage,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 124

~~~~text
      CArg<"std::uint64_t", "0">:$storage_offset,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 125

~~~~text
      CArg<"fir::FortranVariableFlagsAttr", "{}">:$fortran_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 126

~~~~text
      CArg<"cuf::DataAttributeAttr", "{}">:$data_attr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 127

~~~~text
      CArg<"unsigned", "0">:$dummy_arg_no)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 128

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 129

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 130

~~~~text
    /// Get the variable original base (same as input). It lacks
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 131

~~~~text
    /// any explicit lower bounds and the extents might not be retrievable
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 132

~~~~text
    /// from it. This matches what is used as a "base" in FIR.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 133

~~~~text
    mlir::Value getOriginalBase() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 134

~~~~text
      return getResult(1);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 135

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 136

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 137

~~~~text
    /// Override FortranVariableInterface default implementation
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 138

~~~~text
    mlir::Value getBase() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 139

~~~~text
      return getResult(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 140

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 141

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 142

~~~~text
    /// Given a FIR memory type, and information about non default lower
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 143

~~~~text
    /// bounds, get the related HLFIR variable type.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 144

~~~~text
    static mlir::Type getHLFIRVariableType(mlir::Type type, bool hasLowerBounds);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 145

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 146

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 147

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getMemref(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 148

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult) { return 0; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 149

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 150

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 151

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 152

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 153

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 154

~~~~text
def fir_AssignOp : hlfir_Op<"assign", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 155

~~~~text
  let summary = "Assign an expression or variable value to a Fortran variable";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 156

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 157

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 158

~~~~text
    Assign rhs to lhs following Fortran intrinsic assignments rules.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 159

~~~~text
    The operation deals with inserting a temporary if the lhs and rhs
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 160

~~~~text
    may overlap.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 161

~~~~text
    The optional "realloc" flag allows indicating that this assignment
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 162

~~~~text
    has the Fortran 95 semantics for assignments to a whole allocatable.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 163

~~~~text
    In such case, the left hand side must be an allocatable that may be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 164

~~~~text
    unallocated or allocated with a different type and shape than the right
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 165

~~~~text
    hand side. It will be allocated or re-allocated as needed during the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 166

~~~~text
    assignment.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 167

~~~~text
    When "realloc" is set and this is a character assignment, the optional
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 168

~~~~text
    flag "keep_lhs_length_if_realloc" indicates that the character
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 169

~~~~text
    left hand side should retain its length after the assignment. If the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 170

~~~~text
    right hand side has a different length, truncation and padding will
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 171

~~~~text
    occur. This covers the case of explicit and assumed length character
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 172

~~~~text
    allocatables.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 173

~~~~text
    Otherwise, the left hand side will be allocated or reallocated to match the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 174

~~~~text
    right hand side length if they differ. This covers the case of deferred
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 175

~~~~text
    length character allocatables.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 176

~~~~text
    The optional "temporary_lhs" flag indicates that the LHS is a compiler
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 177

~~~~text
    generated temporary. In this case the temporary is initialized if needed
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 178

~~~~text
    (e.g. the LHS is of derived type with allocatable/pointer components),
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 179

~~~~text
    and the assignment is done without LHS (or its subobjects) finalization
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 180

~~~~text
    and with automatic allocation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 181

~~~~text
    If "temporary_lhs" and "keep_lhs_length_if_realloc" are both set,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 182

~~~~text
    this assign operation denotes special case of character allocatable
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 183

~~~~text
    LHS with explicit length. The LHS that must preserve its length
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 184

~~~~text
    during the assignment regardless of the the RHS's length or/and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 185

~~~~text
    allocation status. This assign operation will be lowered into a call
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 186

~~~~text
    to AssignExplicitLengthCharacter().
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 187

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 188

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 189

~~~~text
  let arguments = (ins AnyFortranEntity:$rhs,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 190

~~~~text
                   AnyFortranVariable:$lhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 191

~~~~text
                   UnitAttr:$realloc,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 192

~~~~text
                   UnitAttr:$keep_lhs_length_if_realloc,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 193

~~~~text
                   UnitAttr:$temporary_lhs);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 194

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 195

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 196

~~~~text
    $rhs `to` $lhs (`realloc` $realloc^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 197

~~~~text
    (`keep_lhs_len` $keep_lhs_length_if_realloc^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 198

~~~~text
    (`temporary_lhs` $temporary_lhs^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 199

~~~~text
    attr-dict `:` type(operands)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 200

~~~~text
  }];
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
    /// Does this assignment have the Fortran 95 semantics of assignments
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 204

~~~~text
    /// to a whole allocatable?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 205

~~~~text
    bool isAllocatableAssignment() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 206

~~~~text
      return getRealloc();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 207

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 208

~~~~text
    /// Is the assignment left hand side a whole allocatable character
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 209

~~~~text
    /// that should retain its length after the assignment?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 210

~~~~text
    bool mustKeepLhsLengthInAllocatableAssignment() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 211

~~~~text
      return getKeepLhsLengthIfRealloc();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 212

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 213

~~~~text
    /// Is the assignment's left hand side a compiler generated temporary?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 214

~~~~text
    bool isTemporaryLHS() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 215

~~~~text
      return getTemporaryLhs();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 216

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 217

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 218

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 219

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 220

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 221

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 222

~~~~text
def hlfir_DesignateOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 223

~~~~text
    : hlfir_Op<"designate",
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 224

~~~~text
               [AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 225

~~~~text
                DeclareOpInterfaceMethods<fir_FortranVariableOpInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 226

~~~~text
                NoMemoryEffect, fir_FortranObjectViewOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 227

~~~~text
  let summary = "Designate a Fortran variable";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 228

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 229

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 230

~~~~text
    This operation represents a Fortran "part-ref", except that it can embed a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 231

~~~~text
    substring or or complex part directly, and that vector subscripts cannot be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 232

~~~~text
    used. It returns a Fortran variable that is a part of the input variable.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 233

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 234

~~~~text
    The operands are as follow:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 235

~~~~text
      - memref is the variable being designated.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 236

~~~~text
      - component may be provided if the memref is a derived type to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 237

~~~~text
        represent a reference to a component. It must be the name of a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 238

~~~~text
        component of memref derived type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 239

~~~~text
      - component_shape represents the shape of the component and must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 240

~~~~text
        provided if and only if both component and indices appear.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 241

~~~~text
      - indices can be provided to index arrays. The indices may be simple
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 242

~~~~text
        indices or triplets.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 243

~~~~text
        If indices are provided and there is a component, the component must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 244

~~~~text
        an array component and the indices index the array component.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 245

~~~~text
        If memref is an array, and component is provided and is an array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 246

~~~~text
        component, indices must be provided and must not be triplets. This
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 247

~~~~text
        ensures hlfir.designate does not create arrays of arrays (which is not
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 248

~~~~text
        possible in Fortran).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 249

~~~~text
      - substring may contain two values to represent a substring lower and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 250

~~~~text
        upper bounds.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 251

~~~~text
      - complex_part may be provided to represent a complex part (true
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 252

~~~~text
        represents the imaginary part, and false the real part).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 253

~~~~text
      - shape represents the shape of the result and must be provided if the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 254

~~~~text
        result is an array that is not a box address.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 255

~~~~text
      - typeparams represents the length parameters of the result and must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 256

~~~~text
        provided if the result type has length parameters and is not a box
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 257

~~~~text
        address.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 258

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 259

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 260

~~~~text
  let arguments = (ins AnyFortranVariable:$memref,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 261

~~~~text
                   OptionalAttr<Builtin_StringAttr>:$component,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 262

~~~~text
                   Optional<AnyShapeOrShiftType>:$component_shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 263

~~~~text
                   Variadic<AnyIntegerType>:$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 264

~~~~text
                   DenseBoolArrayAttr:$is_triplet,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 265

~~~~text
                   Variadic<AnyIntegerType>:$substring,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 266

~~~~text
                   OptionalAttr<BoolAttr>:$complex_part,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 267

~~~~text
                   Optional<AnyShapeOrShiftType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 268

~~~~text
                   Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 269

~~~~text
                   OptionalAttr<fir_FortranVariableFlagsAttr>:$fortran_attrs
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 270

~~~~text
                );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 271

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 272

~~~~text
  let results = (outs AnyFortranVariable);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 273

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 274

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 275

~~~~text
    $memref (`{` $component^ `}`)? (`<` $component_shape^ `>`)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 276

~~~~text
    custom<DesignatorIndices>($indices, $is_triplet)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 277

~~~~text
    (`substr` $substring^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 278

~~~~text
    custom<DesignatorComplexPart>($complex_part)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 279

~~~~text
    (`shape` $shape^)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 280

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 281

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 282

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 283

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 284

~~~~text
    using Triplet = std::tuple<mlir::Value, mlir::Value, mlir::Value>;
~~~~
- EN: Creates the alias `Triplet` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `Triplet`。

### Line 285

~~~~text
    using Subscript = std::variant<mlir::Value, Triplet>;
~~~~
- EN: Creates the alias `Subscript` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `Subscript`。

### Line 286

~~~~text
    using Subscripts = llvm::SmallVector<Subscript, 8>;
~~~~
- EN: Creates the alias `Subscripts` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `Subscripts`。

### Line 287

~~~~text
    void setFortranAttrs(fir::FortranVariableFlagsEnum flags) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 288

~~~~text
      this->setFortranAttrs(std::optional<fir::FortranVariableFlagsEnum>(flags));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 289

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 290

~~~~text
    // FortranObjectViewOpInterface methods:
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 291

~~~~text
    mlir::Value getViewSource(mlir::OpResult) { return getMemref(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 292

~~~~text
    std::optional<std::int64_t> getViewOffset(mlir::OpResult);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 293

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 294

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 295

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 296

~~~~text
    OpBuilder<(ins "mlir::Type":$result_type, "mlir::Value":$memref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 297

~~~~text
      "llvm::StringRef":$component, "mlir::Value":$component_shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 298

~~~~text
      "llvm::ArrayRef<std::variant<mlir::Value, std::tuple<mlir::Value, mlir::Value, mlir::Value>>>":$subscripts,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 299

~~~~text
      CArg<"mlir::ValueRange", "{}">:$substring,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 300

~~~~text
      CArg<"std::optional<bool>", "{}">:$complex_part,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 301

~~~~text
      CArg<"mlir::Value", "{}">:$shape, CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 302

~~~~text
      CArg<"fir::FortranVariableFlagsAttr", "{}">:$fortran_attrs)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 303

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 304

~~~~text
    OpBuilder<(ins "mlir::Type":$result_type, "mlir::Value":$memref,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 305

~~~~text
      "mlir::ValueRange":$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 306

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 307

~~~~text
      CArg<"fir::FortranVariableFlagsAttr", "{}">:$fortran_attrs)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 308

~~~~text
    ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 309

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 310

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 311

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 312

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 313

~~~~text
def hlfir_ParentComponentOp : hlfir_Op<"parent_comp", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 314

~~~~text
    DeclareOpInterfaceMethods<fir_FortranVariableOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 315

~~~~text
  let summary = "Designate the parent component of a variable";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 316

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 317

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 318

~~~~text
    This operation represents a Fortran component reference where the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 319

~~~~text
    component name is a parent type of the variable's derived type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 320

~~~~text
    These component references cannot be represented with an hlfir.designate
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 321

~~~~text
    because the parent type names are not embedded in fir.type<> types
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 322

~~~~text
    as opposed to the actual component names.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 323

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 324

~~~~text
    The operands are as follow:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 325

~~~~text
      - memref is a derived type variable whose parent component is being
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 326

~~~~text
        designated.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 327

~~~~text
      - shape is the shape of memref and the result and must be provided if
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 328

~~~~text
        memref is an array. Parent component reference lower bounds are ones,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 329

~~~~text
        so the provided shape must be a fir.shape.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 330

~~~~text
      - typeparams are the type parameters of the parent component type if any.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 331

~~~~text
        It is a subset of memref type parameters.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 332

~~~~text
    The parent component type and name is reflected in the result type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 333

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 334

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 335

~~~~text
  let arguments = (ins AnyFortranVariable:$memref,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 336

~~~~text
                   Optional<AnyShapeType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 337

~~~~text
                   Variadic<AnyIntegerType>:$typeparams);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 338

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 339

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 340

~~~~text
    // Implement FortranVariableInterface interface. Parent components have
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 341

~~~~text
    // no attributes (pointer, allocatable or contiguous can only be added
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 342

~~~~text
    // to regular components).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 343

~~~~text
    std::optional<fir::FortranVariableFlagsEnum> getFortranAttrs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 344

~~~~text
      return std::nullopt;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 345

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 346

~~~~text
    void setFortranAttrs(fir::FortranVariableFlagsEnum flags) {}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 347

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 348

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 349

~~~~text
  let results = (outs AnyFortranVariable);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 350

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 351

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 352

~~~~text
    $memref (`shape` $shape^)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 353

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 354

~~~~text
  }];
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
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 357

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 358

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 359

~~~~text
def hlfir_ConcatOp : hlfir_Op<"concat",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 360

~~~~text
    [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 361

~~~~text
  let summary = "concatenate characters";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 362

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 363

~~~~text
    Concatenate two or more character strings of a same character kind.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 364

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 365

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 366

~~~~text
  let arguments = (ins Variadic<AnyScalarCharacterEntity>:$strings,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 367

~~~~text
                   AnyIntegerType:$length);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 368

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 369

~~~~text
  let results = (outs AnyScalarCharacterExpr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 370

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 371

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 372

~~~~text
    $strings `len` $length
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 373

~~~~text
     attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 374

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 375

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 376

~~~~text
  let builders = [OpBuilder<(ins "mlir::ValueRange":$strings,"mlir::Value":$len)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 377

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 378

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 379

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 380

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 381

~~~~text
def hlfir_CmpCharOp : hlfir_Op<"cmpchar",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 382

~~~~text
    [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 383

~~~~text
  let summary = "compare two characters";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 384

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 385

~~~~text
    Compare two character strings of a same character kind.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 386

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 387

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 388

~~~~text
  let arguments = (ins Arith_CmpIPredicateAttr:$predicate,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 389

~~~~text
                AnyScalarCharacterEntity:$lchr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 390

~~~~text
                AnyScalarCharacterEntity:$rchr);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 391

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 392

~~~~text
  let results = (outs I1);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 393

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 394

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 395

~~~~text
      $predicate $lchr $rchr attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 396

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 397

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 398

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 399

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 400

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 401

~~~~text
def hlfir_CharTrimOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 402

~~~~text
    : hlfir_Op<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 403

~~~~text
          "char_trim", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 404

~~~~text
  let summary = "trim character";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 405

~~~~text
  let description = [{ Trim a character string. }];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 406

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 407

~~~~text
  let arguments = (ins AnyScalarCharacterEntity:$chr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 408

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 409

~~~~text
  let results = (outs AnyScalarCharacterExpr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 410

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 411

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 412

~~~~text
      $chr attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 413

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 414

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 415

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$chr)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 416

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 417

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 418

~~~~text
def hlfir_IndexOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 419

~~~~text
    : hlfir_Op<"index", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 420

~~~~text
  let summary = "index transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 421

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 422

~~~~text
    Search for a substring position within a string, optionally backward
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 423

~~~~text
    if back is set to true.
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 424

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 425

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 426

~~~~text
  let arguments = (ins AnyScalarCharacterEntity:$substr,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 427

~~~~text
      AnyScalarCharacterEntity:$str,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 428

~~~~text
      Optional<Type<AnyLogicalLike.predicate>>:$back);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 429

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 430

~~~~text
  let results = (outs AnyIntegerType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

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
      $substr `in` $str  (`back` $back^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 434

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 435

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 436

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 437

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 438

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 439

~~~~text
def hlfir_AllOp : hlfir_Op<"all", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 440

~~~~text
  let summary = "ALL transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 441

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 442

~~~~text
    Takes a logical array MASK as argument, optionally along a particular dimension,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 443

~~~~text
    and returns true if all elements of MASK are true.
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
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 447

~~~~text
    AnyFortranLogicalArrayObject:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 448

~~~~text
    Optional<AnyIntegerType>:$dim
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 449

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 450

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 451

~~~~text
  let results = (outs AnyFortranValue);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 452

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 453

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 454

~~~~text
    $mask  (`dim` $dim^)?  attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 455

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 456

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 457

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 458

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 459

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 460

~~~~text
def hlfir_AnyOp : hlfir_Op<"any", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 461

~~~~text
  let summary = "ANY transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 462

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 463

~~~~text
    Takes a logical array MASK as argument, optionally along a particular dimension,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 464

~~~~text
    and returns true if any element of MASK is true.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 465

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 466

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 467

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 468

~~~~text
    AnyFortranLogicalArrayObject:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 469

~~~~text
    Optional<AnyIntegerType>:$dim
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 470

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 471

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 472

~~~~text
  let results = (outs AnyFortranValue);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 473

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 474

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 475

~~~~text
    $mask  (`dim` $dim^)?  attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 476

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 477

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 478

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 479

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 480

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 481

~~~~text
def hlfir_CountOp : hlfir_Op<"count", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 482

~~~~text
  let summary = "COUNT transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 483

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 484

~~~~text
    Takes a logical and counts the number of true values.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 485

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 486

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 487

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 488

~~~~text
    AnyFortranLogicalArrayObject:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 489

~~~~text
    Optional<AnyIntegerType>:$dim
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 490

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 491

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 492

~~~~text
  let results = (outs AnyFortranValue);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 493

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 494

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 495

~~~~text
    $mask  (`dim` $dim^)?  attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 496

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 497

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 498

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 499

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 500

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 501

~~~~text
def hlfir_MaxvalOp : hlfir_Op<"maxval", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 502

~~~~text
    DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 503

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 504

~~~~text
  let summary = "MAXVAL transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 505

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 506

~~~~text
    Maximum value(s) of an array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 507

~~~~text
    If DIM is absent, the result is a scalar.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 508

~~~~text
    If DIM is present, the result is an array of rank n-1, where n is the rank of ARRAY.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 509

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 510

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 511

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 512

~~~~text
    AnyFortranArrayObject:$array,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 513

~~~~text
    Optional<AnyIntegerType>:$dim,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 514

~~~~text
    Optional<AnyFortranLogicalOrI1ArrayObject>:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 515

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 516

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 517

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 518

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 519

~~~~text
  let results = (outs AnyFortranValue);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 520

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 521

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 522

~~~~text
    $array (`dim` $dim^)? (`mask` $mask^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 523

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 524

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 525

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 526

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 527

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 528

~~~~text
def hlfir_MinvalOp : hlfir_Op<"minval", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 529

~~~~text
    DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 530

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 531

~~~~text
  let summary = "MINVAL transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 532

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 533

~~~~text
    Minimum value(s) of an array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 534

~~~~text
    If DIM is absent, the result is a scalar.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 535

~~~~text
    If DIM is present, the result is an array of rank n-1, where n is the rank of ARRAY.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 536

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 537

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 538

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 539

~~~~text
    AnyFortranArrayObject:$array,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 540

~~~~text
    Optional<AnyIntegerType>:$dim,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 541

~~~~text
    Optional<AnyFortranLogicalOrI1ArrayObject>:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 542

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 543

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 544

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 545

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 546

~~~~text
  let results = (outs AnyFortranValue);
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
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 549

~~~~text
    $array (`dim` $dim^)? (`mask` $mask^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 550

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 551

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 552

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 553

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 554

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 555

~~~~text
def hlfir_MinlocOp : hlfir_Op<"minloc", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 556

~~~~text
    DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 557

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 558

~~~~text
  let summary = "MINLOC transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 559

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 560

~~~~text
    Minlocs of an array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 561

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 562

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 563

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 564

~~~~text
    AnyFortranArrayObject:$array,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 565

~~~~text
    Optional<AnyIntegerType>:$dim,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 566

~~~~text
    Optional<AnyFortranLogicalOrI1ArrayObject>:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 567

~~~~text
    Optional<Type<AnyLogicalLike.predicate>>:$back,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 568

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 569

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 570

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 571

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 572

~~~~text
  let results = (outs AnyFortranValue);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 573

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 574

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 575

~~~~text
    $array (`dim` $dim^)? (`mask` $mask^)? (`back` $back^)?  attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 576

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 577

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 578

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 579

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 580

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 581

~~~~text
def hlfir_MaxlocOp : hlfir_Op<"maxloc", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 582

~~~~text
    DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 583

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 584

~~~~text
  let summary = "MAXLOC transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 585

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 586

~~~~text
    Maxlocs of an array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 587

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 588

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 589

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 590

~~~~text
    AnyFortranArrayObject:$array,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 591

~~~~text
    Optional<AnyIntegerType>:$dim,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 592

~~~~text
    Optional<AnyFortranLogicalOrI1ArrayObject>:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 593

~~~~text
    Optional<Type<AnyLogicalLike.predicate>>:$back,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 594

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 595

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 596

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 597

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 598

~~~~text
  let results = (outs AnyFortranValue);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 599

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 600

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 601

~~~~text
    $array (`dim` $dim^)? (`mask` $mask^)? (`back` $back^)?  attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 602

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 603

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 604

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 605

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 606

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 607

~~~~text
def hlfir_ProductOp : hlfir_Op<"product", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 608

~~~~text
    DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 609

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 610

~~~~text
  let summary = "PRODUCT transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 611

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 612

~~~~text
    Multiplies the elements of an array, optionally along a particular dimension,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 613

~~~~text
    optionally if a mask is true.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 614

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 615

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 616

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 617

~~~~text
    AnyFortranNumericalArrayObject:$array,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 618

~~~~text
    Optional<AnyIntegerType>:$dim,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 619

~~~~text
    Optional<AnyFortranLogicalOrI1ArrayObject>:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 620

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 621

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 622

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 623

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 624

~~~~text
  let results = (outs AnyFortranValue);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 625

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 626

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 627

~~~~text
    $array (`dim` $dim^)? (`mask` $mask^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 628

~~~~text
  }];
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
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 631

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 632

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 633

~~~~text
def hlfir_SetLengthOp : hlfir_Op<"set_length",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 634

~~~~text
  [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 635

~~~~text
  let summary = "change the length of a character entity";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 636

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 637

~~~~text
    Change the length of character entity. This trims or pads the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 638

~~~~text
    character argument according to the new length.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 639

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 640

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 641

~~~~text
  let arguments = (ins AnyScalarCharacterEntity:$string,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 642

~~~~text
                   AnyIntegerType:$length);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 643

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 644

~~~~text
  let results = (outs AnyScalarCharacterExpr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 645

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 646

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 647

~~~~text
    $string `len` $length
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 648

~~~~text
     attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 649

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 650

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 651

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$string,"mlir::Value":$len)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 652

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 653

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 654

~~~~text
def hlfir_GetLengthOp : hlfir_Op<"get_length", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 655

~~~~text
  let summary = "get the length of a character entity";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 656

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 657

~~~~text
    Get the length of character entity represented as hlfir.expr.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 658

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 659

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 660

~~~~text
  let arguments = (ins AnyScalarOrArrayCharacterExpr:$expr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 661

~~~~text
  let results = (outs Index);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 662

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 663

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 664

~~~~text
    $expr attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 665

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 666

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 667

~~~~text
  // If character length is know via the type, then the operation
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 668

~~~~text
  // may be immediately canonicalized into arith::ConstantOp.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 669

~~~~text
  let hasCanonicalizeMethod = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 670

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 671

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 672

~~~~text
def hlfir_SumOp : hlfir_Op<"sum", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 673

~~~~text
    DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 674

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 675

~~~~text
  let summary = "SUM transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 676

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 677

~~~~text
    Sums the elements of an array, optionally along a particular dimension,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 678

~~~~text
    optionally if a mask is true.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 679

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 680

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 681

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 682

~~~~text
    AnyFortranNumericalArrayObject:$array,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 683

~~~~text
    Optional<AnyIntegerType>:$dim,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 684

~~~~text
    Optional<AnyFortranLogicalOrI1ArrayObject>:$mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 685

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 686

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 687

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 688

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 689

~~~~text
  let results = (outs AnyFortranValue);
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
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 692

~~~~text
    $array (`dim` $dim^)? (`mask` $mask^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 693

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 694

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 695

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 696

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 697

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 698

~~~~text
def hlfir_DotProductOp : hlfir_Op<"dot_product",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 699

~~~~text
    [DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 700

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 701

~~~~text
  let summary = "DOT_PRODUCT transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 702

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 703

~~~~text
    Dot product of two vectors
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 704

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 705

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 706

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 707

~~~~text
    AnyFortranNumericalOrLogicalArrayObject:$lhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 708

~~~~text
    AnyFortranNumericalOrLogicalArrayObject:$rhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 709

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 710

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 711

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 712

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 713

~~~~text
  let results = (outs AnyFortranValue);
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
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 716

~~~~text
    $lhs $rhs attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 717

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 718

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 719

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 720

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 721

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 722

~~~~text
def hlfir_MatmulOp : hlfir_Op<"matmul",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 723

~~~~text
    [DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 724

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 725

~~~~text
  let summary = "MATMUL transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 726

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 727

~~~~text
    Matrix multiplication
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 728

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 729

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 730

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 731

~~~~text
    AnyFortranNumericalOrLogicalArrayObject:$lhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 732

~~~~text
    AnyFortranNumericalOrLogicalArrayObject:$rhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 733

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 734

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 735

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 736

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 737

~~~~text
  let results = (outs hlfir_ExprType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 738

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 739

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 740

~~~~text
    $lhs $rhs attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 741

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 742

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 743

~~~~text
  // MATMUL(TRANSPOSE(...), ...) => hlfir.matmul_transpose
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 744

~~~~text
  let hasCanonicalizeMethod = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 745

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 746

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 747

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 748

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 749

~~~~text
def hlfir_TransposeOp : hlfir_Op<"transpose",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 750

~~~~text
    [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 751

~~~~text
  let summary = "TRANSPOSE transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 752

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 753

~~~~text
    Transpose a rank 2 array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 754

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 755

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 756

~~~~text
  let arguments = (ins AnyFortranArrayObject:$array);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 757

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 758

~~~~text
  let results = (outs hlfir_ExprType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 759

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 760

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 761

~~~~text
    $array attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 762

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 763

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 764

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 765

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 766

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 767

~~~~text
def hlfir_MatmulTransposeOp : hlfir_Op<"matmul_transpose",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 768

~~~~text
    [DeclareOpInterfaceMethods<ArithFastMathInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 769

~~~~text
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 770

~~~~text
  let summary = "Optimized MATMUL(TRANSPOSE(...), ...)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 771

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 772

~~~~text
    Matrix multiplication where the left hand side is transposed
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 773

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 774

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 775

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 776

~~~~text
    AnyFortranNumericalOrLogicalArrayObject:$lhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 777

~~~~text
    AnyFortranNumericalOrLogicalArrayObject:$rhs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 778

~~~~text
    DefaultValuedAttr<Arith_FastMathAttr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 779

~~~~text
                      "::mlir::arith::FastMathFlags::none">:$fastmath
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 780

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 781

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 782

~~~~text
  let results = (outs hlfir_ExprType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 783

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 784

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 785

~~~~text
    $lhs $rhs attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 786

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 787

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 788

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 789

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 790

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 791

~~~~text
def hlfir_CShiftOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 792

~~~~text
    : hlfir_Op<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 793

~~~~text
          "cshift", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 794

~~~~text
  let summary = "CSHIFT transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 795

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 796

~~~~text
    Circular shift of an array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 797

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 798

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 799

~~~~text
  let arguments = (ins AnyFortranArrayObject:$array,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 800

~~~~text
      AnyFortranIntegerScalarOrArrayObject:$shift,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 801

~~~~text
      Optional<AnyIntegerType>:$dim);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 802

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 803

~~~~text
  let results = (outs hlfir_ExprType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 804

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 805

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 806

~~~~text
    $array $shift (`dim` $dim^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 807

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 808

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 809

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 810

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 811

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 812

~~~~text
def hlfir_EOShiftOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 813

~~~~text
    : hlfir_Op<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 814

~~~~text
          "eoshift", [AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 815

~~~~text
                      DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 816

~~~~text
  let summary = "EOSHIFT transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 817

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 818

~~~~text
    End-off shift of an array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 819

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 820

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 821

~~~~text
  let arguments = (ins AnyFortranArrayObject:$array,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 822

~~~~text
      AnyFortranIntegerScalarOrArrayObject:$shift,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 823

~~~~text
      Optional<AnyFortranEntity>:$boundary, Optional<AnyIntegerType>:$dim);
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
  let results = (outs hlfir_ExprType);
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
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 828

~~~~text
    $array $shift (`boundary` $boundary^)? (`dim` $dim^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 829

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 830

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 831

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 832

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 833

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 834

~~~~text
def hlfir_ReshapeOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 835

~~~~text
    : hlfir_Op<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 836

~~~~text
          "reshape", [AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 837

~~~~text
                      DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 838

~~~~text
  let summary = "RESHAPE transformational intrinsic";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 839

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 840

~~~~text
    Reshapes an ARRAY to correspond to the given SHAPE.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 841

~~~~text
    If PAD is specified the new array may be padded with elements
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 842

~~~~text
    from PAD array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 843

~~~~text
    If ORDER is specified the new array may be permuted accordingly.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 844

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 845

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 846

~~~~text
  let arguments = (ins AnyFortranArrayEntity:$array,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 847

~~~~text
      AnyFortranNumericalArrayEntity:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 848

~~~~text
      Optional<AnyFortranArrayEntity>:$pad,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 849

~~~~text
      Optional<AnyFortranNumericalArrayEntity>:$order);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 850

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 851

~~~~text
  let results = (outs hlfir_ExprType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 852

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 853

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 854

~~~~text
    $array $shape (`pad` $pad^)? (`order` $order^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 855

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 856

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 857

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 858

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 859

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 860

~~~~text
// An allocation effect is needed because the value produced by the associate
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 861

~~~~text
// is "deallocated" by hlfir.end_associate (the end_associate must not be
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 862

~~~~text
// removed, and there must be only one hlfir.end_associate).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 863

~~~~text
def hlfir_AssociateOp : hlfir_Op<"associate", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 864

~~~~text
    DeclareOpInterfaceMethods<fir_FortranVariableOpInterface>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 865

~~~~text
    MemoryEffects<[MemAlloc]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 866

~~~~text
  let summary = "Create a variable from an expression value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 867

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 868

~~~~text
    Create a variable from an expression value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 869

~~~~text
    For expressions, this operation is an incentive to re-use the expression
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 870

~~~~text
    storage, if any, after the bufferization pass when possible (if the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 871

~~~~text
    expression is not used afterwards).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 872

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 873

~~~~text
    For aliasing purposes, hlfir.associate with the source being
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 874

~~~~text
    a trivial FIR value is considered to be a unique allocation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 875

~~~~text
    that does not alias with anything else. For non-trivial cases
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 876

~~~~text
    it may be a unique allocation or an alias for the source expression
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 877

~~~~text
    storage, so FIR alias analysis will look through it for finding
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 878

~~~~text
    the source.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 879

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 880

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 881

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 882

~~~~text
    AnyFortranValue:$source,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 883

~~~~text
    Optional<AnyShapeOrShiftType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 884

~~~~text
    Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 885

~~~~text
    OptionalAttr<Builtin_StringAttr>:$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 886

~~~~text
    OptionalAttr<fir_FortranVariableFlagsAttr>:$fortran_attrs
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 887

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 888

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 889

~~~~text
  let results = (outs AnyFortranVariable, AnyRefOrBoxLike, I1);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 890

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 891

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 892

~~~~text
    $source (`(` $shape^ `)`)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 893

~~~~text
     attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 894

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 895

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 896

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 897

~~~~text
    OpBuilder<(ins "mlir::Value":$source, CArg<"llvm::StringRef", "{}">:$uniq_name,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 898

~~~~text
      CArg<"mlir::Value", "{}">:$shape, CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 899

~~~~text
      CArg<"fir::FortranVariableFlagsAttr", "{}">:$fortran_attrs)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 900

~~~~text
    OpBuilder<(ins "mlir::Value":$memref, CArg<"mlir::Value", "{}">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 901

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 902

~~~~text
      CArg<"fir::FortranVariableFlagsAttr", "{}">:$fortran_attrs,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 903

~~~~text
      CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 904

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 905

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 906

~~~~text
    void setFortranAttrs(fir::FortranVariableFlagsEnum flags) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 907

~~~~text
      this->setFortranAttrs(std::optional<fir::FortranVariableFlagsEnum>(flags));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 908

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 909

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 910

~~~~text
    /// Override FortranVariableInterface default implementation
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 911

~~~~text
    mlir::Value getBase() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 912

~~~~text
      return getResult(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 913

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 914

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 915

~~~~text
    /// Get the variable FIR base (same as input). It lacks
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 916

~~~~text
    /// any explicit lower bounds and the extents might not be retrievable
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 917

~~~~text
    /// from it. This matches what is used as a "base" in FIR. All non
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 918

~~~~text
    /// polymorphic expressions FIR base is a simple raw address (they are
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 919

~~~~text
    /// contiguous in memory).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 920

~~~~text
    mlir::Value getFirBase() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 921

~~~~text
      return getResult(1);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 922

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 923

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 924

~~~~text
    /// Return the result value that indicates if the variable storage
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 925

~~~~text
    /// was allocated on the heap. At the HLFIR level, this may not be
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 926

~~~~text
    /// known yet, and lowering will need to conditionally free the storage.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 927

~~~~text
    mlir::Value getMustFreeStrorageFlag() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 928

~~~~text
      return getResult(2);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 929

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 930

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 931

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 932

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 933

~~~~text
def hlfir_EndAssociateOp : hlfir_Op<"end_associate", [MemoryEffects<[MemFree]>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 934

~~~~text
  let summary = "Mark the end of life of a variable associated to an expression";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 935

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 936

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 937

~~~~text
    Mark the end of life of a variable associated to an expression.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 938

~~~~text
    If the expression has a derived type that may contain allocatable
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 939

~~~~text
    components, the variable operand must be a Fortran entity.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 940

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 941

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 942

~~~~text
  let arguments = (ins AnyRefOrBoxLike:$var,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 943

~~~~text
                   I1:$must_free);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 944

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 945

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 946

~~~~text
    $var `,` $must_free attr-dict `:` type(operands)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 947

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 948

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 949

~~~~text
  let builders = [OpBuilder<(ins "hlfir::AssociateOp":$associate)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 950

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 951

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 952

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 953

~~~~text
def hlfir_AsExprOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 954

~~~~text
    : hlfir_Op<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 955

~~~~text
          "as_expr", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 956

~~~~text
  let summary = "Take the value of an array, character or derived variable";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 957

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 958

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 959

~~~~text
    Take the value of an array, character or derived variable.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 960

~~~~text
    In general, this operation will lead to a copy of the variable
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 961

~~~~text
    in the bufferization pass if it was not transformed.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 962

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 963

~~~~text
    However, if it is known that the variable storage will not be used anymore
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 964

~~~~text
    afterwards, the variable storage ownership can be passed to the hlfir.expr
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 965

~~~~text
    by providing the $must_free argument that is a boolean that indicates if
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 966

~~~~text
    the storage must be freed (when it was allocated on the heap).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 967

~~~~text
    This allows Fortran lowering to build some expression value in memory when
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 968

~~~~text
    there is no adequate hlfir operation, and to promote the result to an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 969

~~~~text
    hlfir.expr value without paying the price of introducing a copy.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 970

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 971

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 972

~~~~text
  let arguments = (ins AnyFortranVariable:$var,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 973

~~~~text
                       Optional<I1>:$must_free);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 974

~~~~text
  let results = (outs hlfir_ExprType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 975

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 976

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 977

~~~~text
    // Is this a "move" ?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 978

~~~~text
    bool isMove() { return getMustFree() != mlir::Value{}; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 979

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 980

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 981

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 982

~~~~text
    $var (`move` $must_free^)? attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 983

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 984

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 985

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 986

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$var, CArg<"mlir::Value", "{}">:$must_free)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 987

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 988

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 989

~~~~text
def hlfir_NoReassocOp : hlfir_Op<"no_reassoc", [NoMemoryEffect, SameOperandsAndResultType]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 990

~~~~text
  let summary = "synthetic op to prevent reassociation";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 991

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 992

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 993

~~~~text
    Same as fir.reassoc, except it accepts hlfir.expr arguments.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 994

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 995

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 996

~~~~text
  let arguments = (ins AnyFortranEntity:$val);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 997

~~~~text
  let results = (outs AnyFortranEntity);
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
  let assemblyFormat = "$val attr-dict `:` type($val)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1000

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1001

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1002

~~~~text
def hlfir_ElementalOpInterface : OpInterface<"ElementalOpInterface"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1003

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1004

~~~~text
    Interface for the operation holding a region with elemental computation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1005

~~~~text
    It is used as a common interface bewteen hlfir.elemental and hlfir.elemental_addr.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1006

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1007

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1008

~~~~text
  let methods = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1009

~~~~text
    InterfaceMethod<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1010

~~~~text
      /*desc=*/"Return the one based elemental indices.",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1011

~~~~text
      /*retTy=*/"mlir::Block::BlockArgListType",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1012

~~~~text
      /*methodName=*/"getIndices",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1013

~~~~text
      /*args=*/(ins),
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1014

~~~~text
      /*methodBody=*/[{}]
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1015

~~~~text
    >,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1016

~~~~text
    InterfaceMethod<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1017

~~~~text
      /*desc=*/"Return the element entity being computed",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1018

~~~~text
      /*retTy=*/"mlir::Value",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1019

~~~~text
      /*methodName=*/"getElementEntity",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1020

~~~~text
      /*args=*/(ins),
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1021

~~~~text
      /*methodBody=*/[{}]
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1022

~~~~text
    >,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1023

~~~~text
    InterfaceMethod<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1024

~~~~text
      /*desc=*/"Get element cleanup region, if any.",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1025

~~~~text
      /*retTy=*/"mlir::Region*",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1026

~~~~text
      /*methodName=*/"getElementCleanup",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1027

~~~~text
      /*args=*/(ins),
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1028

~~~~text
      /*methodBody=*/[{}]
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1029

~~~~text
    >,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1030

~~~~text
    InterfaceMethod<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1031

~~~~text
      /*desc=*/"Get elemental region.",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1032

~~~~text
      /*retTy=*/"mlir::Region&",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1033

~~~~text
      /*methodName=*/"getElementalRegion",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1034

~~~~text
      /*args=*/(ins),
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1035

~~~~text
      /*methodBody=*/[{}]
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1036

~~~~text
    >,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1037

~~~~text
    InterfaceMethod<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1038

~~~~text
      /*desc=*/"Must this elemental operation be evaluated in order?",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1039

~~~~text
      /*retTy=*/"bool",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1040

~~~~text
      /*methodName=*/"isOrdered",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1041

~~~~text
      /*args=*/(ins),
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1042

~~~~text
      /*methodBody=*/[{}]
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1043

~~~~text
    >,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1044

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1045

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1046

~~~~text
  let cppNamespace = "hlfir";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1047

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1048

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1049

~~~~text
def hlfir_ElementalOp : hlfir_Op<"elemental",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1050

~~~~text
    // The ElementalOp, in general, causes an allocation of a temporary,
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1051

~~~~text
    // so to guarantee proper behavior of MLIR optimization passes
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1052

~~~~text
    // we explicitly set MemAlloc effect for it. On top of this,
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1053

~~~~text
    // the recursive memory effects also apply.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1054

~~~~text
    [RecursiveMemoryEffects, MemoryEffects<[MemAlloc]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1055

~~~~text
     hlfir_ElementalOpInterface, AttrSizedOperandSegments]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1056

~~~~text
  let summary = "elemental expression";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1057

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1058

~~~~text
    Represent an elemental expression as a function of the indices.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1059

~~~~text
    This operation contain a region whose block arguments are one
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1060

~~~~text
    based indices iterating over the elemental expression shape.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1061

~~~~text
    Given these indices, the element value for the given iteration
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1062

~~~~text
    can be computed in the region and yielded with the hlfir.yield_element
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1063

~~~~text
    operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1064

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1065

~~~~text
    The shape and typeparams operands represent the extents and type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1066

~~~~text
    parameters of the resulting array value.
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
    The optional mold is an entity carrying the information about
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1069

~~~~text
    the dynamic type of the polymorphic result. Note that the shape
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1070

~~~~text
    of the mold does not necessarily match the shape of the result,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1071

~~~~text
    for example, the result of `merge(poly_scalar1, poly_scalar2, mask_array)`
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 1072

~~~~text
    will have the shape of `mask_array` and the dynamic type of `poly_scalar*`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1073

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1074

~~~~text
    The unordered attribute can be set to allow out of order processing
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1075

~~~~text
    of the indices. This is safe only if the operations in the body
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1076

~~~~text
    of the elemental do not have side effects.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1077

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1078

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1079

~~~~text
    Example: Y + X,  with Integer :: X(10, 20), Y(10,20)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1080

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1081

~~~~text
      %0 = fir.shape %c10, %c20 : (index, index) -> !fir.shape<2>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1082

~~~~text
      %5 = hlfir.elemental %0 : (!fir.shape<2>) -> !hlfir.expr<10x20xi32> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1083

~~~~text
      ^bb0(%i: index, %j: index):
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1084

~~~~text
        %6 = hlfir.designate %x (%i, %j)  : (!fir.ref<!fir.array<10x20xi32>>, index, index) -> !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1085

~~~~text
        %7 = hlfir.designate %y (%i, %j)  : (!fir.ref<!fir.array<10x20xi32>>, index, index) -> !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1086

~~~~text
        %8 = fir.load %6 : !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1087

~~~~text
        %9 = fir.load %7 : !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1088

~~~~text
        %10 = arith.addi %8, %9 : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1089

~~~~text
        hlfir.yield_element %10 : i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1090

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1091

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1092

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1093

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1094

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1095

~~~~text
    AnyShapeType:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1096

~~~~text
    Optional<AnyPolymorphicObject>:$mold,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1097

~~~~text
    Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1098

~~~~text
    OptionalAttr<UnitAttr>:$unordered
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1099

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1100

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1101

~~~~text
  let results = (outs hlfir_ExprType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1102

~~~~text
  let regions = (region SizedRegion<1>:$region);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1103

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1104

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1105

~~~~text
    $shape (`mold` $mold^)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1106

~~~~text
    (`unordered` $unordered^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1107

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1108

~~~~text
    $region
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1109

~~~~text
    }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1110

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1111

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1112

~~~~text
      mlir::Block *getBody() { return &getRegion().front(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1113

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1114

~~~~text
      /// Get the indices iterating over the shape.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1115

~~~~text
      mlir::Block::BlockArgListType getIndices() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1116

~~~~text
       return getBody()->getArguments();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1117

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1118

~~~~text
      /// ElementalOpInterface implementation.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1119

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1120

~~~~text
      mlir::Region& getElementalRegion() { return getRegion(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1121

~~~~text
      mlir::Value getElementEntity();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1122

~~~~text
      mlir::Region* getElementCleanup() { return nullptr; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1123

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1124

~~~~text
      /// Must this elemental be evaluated in order?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1125

~~~~text
      bool isOrdered() { return !getUnordered(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1126

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1127

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1128

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1129

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1130

~~~~text
    OpBuilder<(ins "mlir::Type":$result_type, "mlir::Value":$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1131

~~~~text
      CArg<"mlir::Value", "{}">:$mold,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1132

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1133

~~~~text
      CArg<"bool", "false">:$isUnordered)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1134

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1135

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1136

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1137

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1138

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1139

~~~~text
def hlfir_YieldElementOp : hlfir_Op<"yield_element", [Terminator, HasParent<"ElementalOp">, Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1140

~~~~text
  let summary = "Yield the elemental value in an ElementalOp";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1141

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1142

~~~~text
    Yield the element value of the current elemental expression iteration
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1143

~~~~text
    in an hlfir.elemental region. See hlfir.elemental description for an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1144

~~~~text
    example.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1145

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1146

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1147

~~~~text
  let arguments = (ins AnyType:$element_value);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1148

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1149

~~~~text
  let assemblyFormat = "$element_value attr-dict `:` type($element_value)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1150

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1151

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1152

~~~~text
def hlfir_ApplyOp : hlfir_Op<"apply", [NoMemoryEffect, AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1153

~~~~text
  let summary = "get the element value of an expression";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1154

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1155

~~~~text
    Given an hlfir.expr array value, hlfir.apply allow retrieving
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1156

~~~~text
    the value for an element given one based indices.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1157

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1158

~~~~text
    When hlfir.apply is used on an hlfir.elemental, and if the hlfir.elemental
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1159

~~~~text
    operation evaluation can be moved to the location of the hlfir.apply, it is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1160

~~~~text
    as if the hlfir.elemental body was evaluated given the hlfir.apply indices.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1161

~~~~text
    Therefore, apply operations on hlfir.elemental expressions should be located
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1162

~~~~text
    such that evaluating the hlfir.elemental at the position of the hlfir.apply
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1163

~~~~text
    operation produces the same result as evaluating the hlfir.elemental at its
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1164

~~~~text
    location in the instruction stream. Attention should be paid to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1165

~~~~text
    hlfir.elemental memory side effects (in practice these are unlikely).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1166

~~~~text
    "10.1.4 Evaluation of operations" says that expression evaluation shall not
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1167

~~~~text
    impact/be impacted by other expression evaluation in the statement.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1168

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1169

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1170

~~~~text
  let arguments = (ins hlfir_ExprType:$expr,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1171

~~~~text
                   Variadic<Index>:$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1172

~~~~text
                   Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1173

~~~~text
                  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1174

~~~~text
  let results = (outs AnyFortranValue:$element_value);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1175

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1176

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1177

~~~~text
    $expr `,` $indices (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1178

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1179

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1180

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1181

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1182

~~~~text
    OpBuilder<(ins "mlir::Value":$expr, "mlir::ValueRange":$indices,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1183

~~~~text
      "mlir::ValueRange":$typeparams)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1184

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1185

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1186

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1187

~~~~text
def hlfir_NullOp : hlfir_Op<"null", [Pure, fir_FortranVariableOpInterface]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1188

~~~~text
  let summary = "create a NULL() address";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1189

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1190

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1191

~~~~text
    Create a NULL() address.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1192

~~~~text
    So far is not intended to represent NULL(MOLD).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1193

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1194

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1195

~~~~text
  let results = (outs AnyFortranVariable);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1196

~~~~text
  let builders = [OpBuilder<(ins)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1197

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1198

~~~~text
  let assemblyFormat = "type(results) attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1199

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1200

~~~~text
    // Implement FortranVariableInterface interface.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1201

~~~~text
    std::optional<fir::FortranVariableFlagsEnum> getFortranAttrs() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1202

~~~~text
      return std::nullopt;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1203

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1204

~~~~text
    void setFortranAttrs(fir::FortranVariableFlagsEnum flags) {}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1205

~~~~text
    mlir::Value getShape() const {return mlir::Value{};}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1206

~~~~text
    mlir::OperandRange getExplicitTypeParams() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1207

~~~~text
      // Return an empty range.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1208

~~~~text
      return {(*this)->getOperands().begin(), (*this)->getOperands().begin()};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1209

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1210

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1211

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1212

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1213

~~~~text
def hlfir_DestroyOp : hlfir_Op<"destroy", [MemoryEffects<[MemFree]>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1214

~~~~text
  let summary = "Mark the last use of an hlfir.expr";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1215

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1216

~~~~text
    Mark the last use of an hlfir.expr. This will be the point at which the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1217

~~~~text
    buffer of an hlfir.expr, if any, will be deallocated if it was heap
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1218

~~~~text
    allocated.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1219

~~~~text
    If "finalize" attribute is set, the hlfir.expr value will be finalized
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1220

~~~~text
    before the deallocation. Note that this implies that the hlfir.expr
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1221

~~~~text
    is placed into a memory buffer, so that the library runtime
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1222

~~~~text
    can be called on it. The element type of the hlfir.expr must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1223

~~~~text
    derived type in this case.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1224

~~~~text
    It is not required to create an hlfir.destroy operation for and hlfir.expr
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1225

~~~~text
    created inside an hlfir.elemental and returned in the hlfir.yield_element.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1226

~~~~text
    The last use of such expression is implicit and an hlfir.destroy could
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1227

~~~~text
    not be emitted after the hlfir.yield_element since it is a terminator.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1228

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1229

~~~~text
    Note that hlfir.destroy are currently generated by Fortran lowering that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1230

~~~~text
    has a good view of the expression use contexts, but this will need to be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1231

~~~~text
    revisited if any motion of hlfir.expr is done (like CSE) since
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1232

~~~~text
    transformations should not introduce any hlfir.expr usages after an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1233

~~~~text
    hlfir.destroy.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1234

~~~~text
    The future will probably be to identify the last use points automatically
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1235

~~~~text
    in bufferization instead.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1236

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1237

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1238

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1239

~~~~text
    hlfir_ExprType:$expr,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1240

~~~~text
    UnitAttr:$finalize
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1241

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1242

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1243

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1244

~~~~text
    $expr (`finalize` $finalize^)? attr-dict `:` qualified(type($expr))
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1245

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1246

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1247

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1248

~~~~text
    bool mustFinalizeExpr() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1249

~~~~text
      return getFinalize();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1250

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1251

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1252

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1253

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1254

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1255

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1256

~~~~text
def hlfir_CopyInOp : hlfir_Op<"copy_in", [MemoryEffects<[MemAlloc]>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1257

~~~~text
  let summary = "copy a variable into a contiguous temporary if it is not contiguous";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1258

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1259

~~~~text
    Copy a variable into a contiguous temporary if the variable is not
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1260

~~~~text
    an absent optional and is not contiguous at runtime. When a copy is made this
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1261

~~~~text
    operation returns the temporary as first result, otherwise, it returns the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1262

~~~~text
    potentially absent variable storage. The second result indicates if a copy
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1263

~~~~text
    was made.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1264

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1265

~~~~text
    A descriptor address must be provided for the temporary. This descriptor will
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1266

~~~~text
    be set if a temporary copy was made.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1267

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1268

~~~~text
    This operation is meant to be used in combination with the hlfir.copy_out
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1269

~~~~text
    operation that takes the address of the descriptor for the temporary, deletes
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1270

~~~~text
    the temporary if it was created, and copies the data back if needed.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1271

~~~~text
    This operation allows passing non contiguous arrays to contiguous dummy
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1272

~~~~text
    arguments, which is possible in Fortran procedure references.
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
    To deal with the optional case, an extra boolean value can be pass to the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1275

~~~~text
    operation. In such cases, the copy-in will only be done if "var_is_present"
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1276

~~~~text
    is true and, when it is false, the original value will be returned instead.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1277

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1278

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1279

~~~~text
  let arguments = (ins Arg<fir_BaseBoxType, "", [MemRead]>:$var,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1280

~~~~text
                    Arg<AnyReferenceLike, "", [MemWrite]>:$tempBox,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1281

~~~~text
                    Optional<I1>:$var_is_present);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1282

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1283

~~~~text
  let results = (outs fir_BaseBoxType, I1);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1284

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1285

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1286

~~~~text
    $var `to` $tempBox (`handle_optional` $var_is_present^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1287

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1288

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1289

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1290

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1291

~~~~text
    OpBuilder<(ins "mlir::Value":$var, "mlir::Value":$temp_box,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1292

~~~~text
          "mlir::Value":$var_is_present)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1293

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1294

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1295

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1296

~~~~text
    /// Get the resulting copied-in fir.box or fir.class.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1297

~~~~text
    mlir::Value getCopiedIn() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1298

~~~~text
      return getResult(0);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1299

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1300

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1301

~~~~text
    /// Get the result indicating if a copy was made.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1302

~~~~text
    mlir::Value getWasCopied() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1303

~~~~text
      return getResult(1);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1304

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1305

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1306

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1307

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1308

~~~~text
def hlfir_CopyOutOp : hlfir_Op<"copy_out", [MemoryEffects<[MemFree]>]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1309

~~~~text
  let summary = "copy out a variable after a copy in";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1310

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1311

~~~~text
    If the variable was copied in a temporary in the related hlfir.copy_in,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1312

~~~~text
    optionally copy back the temporary value to it (that may have been
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1313

~~~~text
    modified between the hlfir.copy_in and hlfir.copy_out). Then deallocate
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1314

~~~~text
    the temporary.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1315

~~~~text
    The copy back is done if $var is provided and $was_copied is true.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1316

~~~~text
    The deallocation of $temp is done if $was_copied is true.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1317

~~~~text
    $temp must be the descriptor address that was provided to hlfir.copy_in.
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
  let arguments = (ins Arg<AnyReferenceLike, "", [MemRead]>:$temp,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1321

~~~~text
                       I1:$was_copied,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1322

~~~~text
                       Arg<Optional<fir_BaseBoxType>, "", [MemWrite]>:$var);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1323

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1324

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1325

~~~~text
    $temp `,` $was_copied (`to` $var^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1326

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1327

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1328

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1329

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1330

~~~~text
def hlfir_ShapeOfOp : hlfir_Op<"shape_of", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1331

~~~~text
  let summary = "Get the shape of a hlfir.expr";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1332

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1333

~~~~text
    Gets the runtime shape of a hlfir.expr. In lowering to FIR, the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1334

~~~~text
    hlfir.shape_of operation will be replaced by an fir.shape.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1335

~~~~text
    It is not valid to request the shape of a hlfir.expr which has no shape.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1336

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1337

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1338

~~~~text
  let arguments = (ins hlfir_ExprType:$expr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1339

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1340

~~~~text
  let results = (outs fir_ShapeType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1341

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1342

~~~~text
  let hasVerifier = 1;
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
  // If all extents are known at compile time, the hlfir.shape_of can be
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1345

~~~~text
  // immediately folded into a fir.shape operation. This makes information
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1346

~~~~text
  // available sooner to inform bufferization decisions
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1347

~~~~text
  let hasCanonicalizeMethod = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1348

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1349

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1350

~~~~text
    std::size_t getRank();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1351

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1352

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1353

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1354

~~~~text
    $expr attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1355

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1356

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1357

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$expr)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1358

~~~~text
  let hasFolder = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1359

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1360

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1361

~~~~text
def hlfir_GetExtentOp : hlfir_Op<"get_extent", [Pure]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1362

~~~~text
  let summary = "Get an extent value from a fir.shape";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1363

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1364

~~~~text
    Gets an extent value from a fir.shape. The dimension argument uses C style
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1365

~~~~text
    indexing and so should be between 0 and 1 less than the rank of the shape
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1366

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1367

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1368

~~~~text
  let arguments = (ins fir_ShapeType:$shape,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1369

~~~~text
                       IndexAttr:$dim);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1370

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1371

~~~~text
  let results = (outs Index);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1372

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1373

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1374

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1375

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1376

~~~~text
    $shape attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1377

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1378

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1379

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$shape, "unsigned":$dim)>];
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
def hlfir_OrderedAssignmentTreeOpInterface : OpInterface<"OrderedAssignmentTreeOpInterface"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1383

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1384

~~~~text
    Interface for the operations representing Forall and Where constructs and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1385

~~~~text
    statements as an mlir::Region tree.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1386

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1387

~~~~text
    These operations all have in common that they have "leaf" regions that contains
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1388

~~~~text
    some code that should be evaluated for "all active combinations of Forall
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1389

~~~~text
    index-name values" before the next OrderedAssignmentTreeOpInterface is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1390

~~~~text
    evaluated.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1391

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1392

~~~~text
    These operations are ordered in a tree fashion: Some operations, like
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1393

~~~~text
    hlfir.forall or hlfir.where, contain a list of OrderedAssignmentTreeOpInterface
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1394

~~~~text
    that should be evaluated after the "Leaf" regions, and before the next
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1395

~~~~text
    OrderedAssignmentTreeOpInterface.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1396

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1397

~~~~text
    Nested OrderedAssignmentTreeOpInterface operations are affected by the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1398

~~~~text
    OrderedAssignmentTreeOpInterface operations that contain them (e.g:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1399

~~~~text
    hlfir.region_assign may be masked by the value of the mask region of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1400

~~~~text
    an hlfir.where that contains it).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1401

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1402

~~~~text
    OrderedAssignmentTreeOpInterface operations that contain nested operation
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1403

~~~~text
    must return a "sub-tree" region that contains the list of nested
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1404

~~~~text
    OrderedAssignmentTreeOpInterface operations.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1405

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1406

~~~~text
    There is no constraints over what IR a leaf region may contain. There is also
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1407

~~~~text
    no restriction regarding how many leaf regions an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1408

~~~~text
    OrderedAssignmentTreeOpInterface operation may contain.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1409

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1410

~~~~text
    A "sub-tree" region, if any, must contain only OrderedAssignmentTreeOpInterface
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1411

~~~~text
    operations and, maybe, a fir.end terminator.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1412

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1413

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1414

~~~~text
  let methods = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1415

~~~~text
    InterfaceMethod<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1416

~~~~text
      /*desc=*/"Get the OrderedAssignmentTreeOpInterface leaf regions that contain evaluation code",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1417

~~~~text
      /*retTy=*/"void",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1418

~~~~text
      /*methodName=*/"getLeafRegions",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1419

~~~~text
      /*args=*/(ins "llvm::SmallVectorImpl<mlir::Region*>&":$regions),
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1420

~~~~text
      /*methodBody=*/[{}]
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1421

~~~~text
    >,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1422

~~~~text
    InterfaceMethod<
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1423

~~~~text
      /*desc=*/"Get the region, if any, containing the list of sub-tree OrderedAssignmentTreeOpInterface nodes",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1424

~~~~text
      /*retTy=*/"mlir::Region*",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1425

~~~~text
      /*methodName=*/"getSubTreeRegion",
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1426

~~~~text
      /*args=*/(ins),
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1427

~~~~text
      /*methodBody=*/[{}]
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1428

~~~~text
    >,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1429

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1430

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1431

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1432

~~~~text
    /// Interface verifier imlementation.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1433

~~~~text
    llvm::LogicalResult verifyImpl();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1434

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1435

~~~~text
    mlir::Block* getSubTreeBlock() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1436

~~~~text
      mlir::Region* region = getSubTreeRegion();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1437

~~~~text
      return region && !region->empty()? &region->front() : nullptr;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1438

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1439

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1440

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1441

~~~~text
  let verify = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1442

~~~~text
    return ::mlir::cast<::hlfir::OrderedAssignmentTreeOpInterface>($_op).verifyImpl();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1443

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1444

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1445

~~~~text
  let cppNamespace = "hlfir";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1446

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1447

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1448

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1449

~~~~text
def hlfir_RegionAssignOp : hlfir_Op<"region_assign", [hlfir_OrderedAssignmentTreeOpInterface]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1450

~~~~text
  let summary = "represent a Fortran assignment using regions for the LHS and RHS evaluation";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1451

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1452

~~~~text
    This operation can represent Forall and Where assignment when inside an
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1453

~~~~text
    hlfir.forall or hlfir.where "ordered assignment tree". It can
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1454

~~~~text
    also represent user defined assignments and assignment to vector
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1455

~~~~text
    subscripted entities without requiring the materialization of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1456

~~~~text
    right-hand side temporary copy that may be needed to implement Fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1457

~~~~text
    assignment semantic.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1458

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1459

~~~~text
    The right-hand side and left-hand side evaluations are held in their
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1460

~~~~text
    own regions terminated with hlfir.yield operations (or hlfir.elemental_addr
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1461

~~~~text
    for a left-hand side with vector subscript).
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 1462

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1463

~~~~text
    An optional region may be added to implement user defined assignment.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1464

~~~~text
    This region provides two block arguments with the same type as the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1465

~~~~text
    yielded rhs and lhs entities (in that order), or the element type if this
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1466

~~~~text
    is an elemental user defined assignment.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1467

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1468

~~~~text
    If this optional region is not provided, intrinsic assignment is performed.
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
    Example: "X = Y",  where "=" is a user defined elemental assignment "foo"
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1471

~~~~text
    taking Y by value.
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
    hlfir.region_assign {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1474

~~~~text
      hlfir.yield %y : !fir.box<!fir.array<?x!f32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1475

~~~~text
    } to {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1476

~~~~text
      hlfir.yield %x : !fir.box<!fir.array<?x!fir.type<t>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1477

~~~~text
    } user_defined_assignment (%rhs_elt: !fir.ref<f32>) to (%lhs_elt: !fir.ref<!fir.type<t>>) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1478

~~~~text
      %0 = fir.load %rhs_elt : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1479

~~~~text
      fir.call @foo(%lhs_elt, %0) : (!fir.ref<!fir.type<t>>, f32) -> ()
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1480

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1481

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1482

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1483

~~~~text
    TODO: add optional "realloc" semantics like for hlfir.assign.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1484

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1485

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1486

~~~~text
  let regions = (region  SizedRegion<1>:$rhs_region,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1487

~~~~text
                         SizedRegion<1>:$lhs_region,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1488

~~~~text
                         MaxSizedRegion<1>:$user_defined_assignment);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1489

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1490

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1491

~~~~text
    mlir::Value getUserAssignmentRhs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1492

~~~~text
      return getUserDefinedAssignment().getArguments()[0];
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1493

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1494

~~~~text
    mlir::Value getUserAssignmentLhs() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1495

~~~~text
      return getUserDefinedAssignment().getArguments()[1];
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1496

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1497

~~~~text
    void getLeafRegions(llvm::SmallVectorImpl<mlir::Region*>& regions) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1498

~~~~text
      regions.push_back(&getRhsRegion());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1499

~~~~text
      regions.push_back(&getLhsRegion());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1500

~~~~text
      if (!getUserDefinedAssignment().empty())
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 1501

~~~~text
        regions.push_back(&getUserDefinedAssignment());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1502

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1503

~~~~text
    mlir::Region* getSubTreeRegion() { return nullptr; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1504

~~~~text
    bool isPointerAssignment();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1505

~~~~text
    bool isPointerObjectAssignment();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1506

~~~~text
    bool isProcedurePointerAssignment();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1507

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1508

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1509

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1510

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1511

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1512

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1513

~~~~text
def hlfir_YieldOp : hlfir_Op<"yield", [Terminator, ParentOneOf<["RegionAssignOp",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1514

~~~~text
    "ElementalAddrOp", "ForallOp", "ForallMaskOp", "WhereOp", "ElseWhereOp",
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1515

~~~~text
    "ExactlyOnceOp"]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1516

~~~~text
    SingleBlockImplicitTerminator<"fir::FirEndOp">, RecursivelySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1517

~~~~text
        RecursiveMemoryEffects]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1518

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1519

~~~~text
  let summary = "Yield a value or variable inside a forall, where or region assignment";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1520

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1521

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1522

~~~~text
    Terminator operation that yields an HLFIR value or variable that was computed in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1523

~~~~text
    a region and hold the yielded entity cleanup, if any, into its own region.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1524

~~~~text
    This allows representing any Fortran expression evaluation in its own region so
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1525

~~~~text
    that the evaluation can easily be scheduled/moved around in a pass.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1526

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1527

~~~~text
    Example: "foo(x)" where foo returns an allocatable array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1528

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1529

~~~~text
    {
~~~~
- EN: Opens a new scope block.
- CN: 打开一个新的作用域块。

### Line 1530

~~~~text
      // In some region.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1531

~~~~text
      %0 = fir.call @foo(x) (!fir.ref<f32>) -> !fir.box<fir.heap<!fir.array<?xf32>>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1532

~~~~text
      hlfir.yield %0 : !fir.box<!fir.heap<!fir.array<?xf32>>> cleanup {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1533

~~~~text
        %1 = fir.box_addr %0 : !fir.box<!fir.heap<!fir.array<?xf32>>> -> !fir.heap<!fir.array<?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1534

~~~~text
        %fir.freemem %1 : !fir.heap<!fir.array<?xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1535

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1536

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1537

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1538

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1539

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1540

~~~~text
  let arguments = (ins AnyFortranEntity:$entity);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1541

~~~~text
  let regions = (region  MaxSizedRegion<1>:$cleanup);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1542

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1543

~~~~text
  let assemblyFormat = "$entity attr-dict `:` type($entity) custom<YieldOpCleanup>($cleanup)";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1544

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1545

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1546

~~~~text
def hlfir_ElementalAddrOp : hlfir_Op<"elemental_addr", [Terminator, HasParent<"RegionAssignOp">,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1547

~~~~text
    RecursiveMemoryEffects, RecursivelySpeculatable, hlfir_ElementalOpInterface,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1548

~~~~text
    AttrSizedOperandSegments]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1549

~~~~text
  let summary = "Yield the address of a vector subscripted variable inside an hlfir.region_assign";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1550

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1551

~~~~text
    Special terminator node for the left-hand side region of an hlfir.region_assign
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1552

~~~~text
    to a vector subscripted entity.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1553

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1554

~~~~text
    It represents how the address of an element of such entity is computed given
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1555

~~~~text
    one based indices.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1556

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1557

~~~~text
    It is very similar to hlfir.elemental, except that it does not produce an SSA
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1558

~~~~text
    value because there is no hlfir type to describe a vector subscripted entity
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1559

~~~~text
    (the codegen of such type would be problematic). Hence, it is tightly linked
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1560

~~~~text
    to an hlfir.region_assign by its terminator property.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1561

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1562

~~~~text
    An optional cleanup region may be provided if any of the subscript expressions
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1563

~~~~text
    of the designator require a cleanup.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1564

~~~~text
    This allows documenting cleanups that cannot be generated after the vector
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1565

~~~~text
    subscripted designator usage (that has not been materizaled yet). The cleanups
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1566

~~~~text
    will be evaluated after the assignment once the related
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1567

~~~~text
    hlfir.region_assign is lowered.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1568

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1569

~~~~text
    Example: "X(VECTOR) = Y"
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1570

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1571

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1572

~~~~text
    hlfir.region_assign {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1573

~~~~text
      hlfir.yield %y : !fir.ref<!fir.array<20xf32>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1574

~~~~text
    } to {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1575

~~~~text
      hlfir.elemental_addr %vector_shape  : !fir.shape<1> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1576

~~~~text
        ^bb0(%i: index):
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1577

~~~~text
        %0 = hlfir.designate %vector (%i)  : (!fir.ref<!fir.array<20xi32>>, index) -> !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1578

~~~~text
        %1 = fir.load %0 : !fir.ref<i32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1579

~~~~text
        %x_element_addr = hlfir.designate %x (%1)  : (!fir.ref<!fir.array<100xf32>>, i32) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1580

~~~~text
        hlfir.yield %x_element_addr : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1581

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1582

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1583

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1584

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1585

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1586

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1587

~~~~text
    fir_ShapeType:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1588

~~~~text
    Optional<AnyPolymorphicObject>:$mold,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1589

~~~~text
    Variadic<AnyIntegerType>:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1590

~~~~text
    OptionalAttr<UnitAttr>:$unordered
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1591

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1592

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1593

~~~~text
  let regions = (region  SizedRegion<1>:$body,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1594

~~~~text
                         MaxSizedRegion<1>:$cleanup);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1595

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1596

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1597

~~~~text
    OpBuilder<(ins "mlir::Value":$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1598

~~~~text
          CArg<"mlir::Value", "{}">:$mold,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1599

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1600

~~~~text
      CArg<"bool", "false">:$isUnordered)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1601

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1602

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1603

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1604

~~~~text
    $shape (`mold` $mold^)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1605

~~~~text
    (`unordered` $unordered^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1606

~~~~text
    attr-dict `:` type(operands) $body
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1607

~~~~text
    custom<YieldOpCleanup>($cleanup)}];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1608

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1609

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1610

~~~~text
    mlir::Region::BlockArgListType getIndices() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1611

~~~~text
      return getBody().getArguments();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1612

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1613

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1614

~~~~text
    /// Return the hlfir::YieldOp terminator of the operation
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1615

~~~~text
    /// body. It yields the variable element address.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1616

~~~~text
    /// This should only be called once the ElementalAddrOp has been built.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1617

~~~~text
    hlfir::YieldOp getYieldOp();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1618

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1619

~~~~text
    /// ElementalOpInterface implementation.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1620

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1621

~~~~text
    mlir::Region& getElementalRegion() { return getBody(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1622

~~~~text
    mlir::Value getElementEntity();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1623

~~~~text
    mlir::Region* getElementCleanup();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1624

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1625

~~~~text
    /// Must this elemental be evaluated in order?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1626

~~~~text
    bool isOrdered() { return !getUnordered(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1627

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1628

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1629

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1630

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1631

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1632

~~~~text
/// Define ODS constraints to verify that a region ends with a yield of a
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1633

~~~~text
/// certain type.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1634

~~~~text
def YieldIntegerOrEmpty : CPred<"yieldsIntegerOrEmpty($_self)">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1635

~~~~text
def YieldIntegerRegion : RegionConstraint<
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1636

~~~~text
  And<[SizedRegion<1>.predicate, YieldIntegerOrEmpty]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1637

~~~~text
  "single block region that yields an integer scalar value">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1638

~~~~text
def MaybeYieldIntegerRegion : RegionConstraint<
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1639

~~~~text
  And<[MaxSizedRegion<1>.predicate, YieldIntegerOrEmpty]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1640

~~~~text
  "optional single block region that yields an integer scalar value">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1641

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1642

~~~~text
def hlfir_ForallOp : hlfir_Op<"forall", [hlfir_OrderedAssignmentTreeOpInterface]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1643

~~~~text
  let summary = "represent a Fortran forall";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1644

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1645

~~~~text
    This operation allows representing Fortran forall. It computes
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1646

~~~~text
    a set of "index-name" values based on lower bound, upper bound,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1647

~~~~text
    and step values whose evaluations are represented in their own
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1648

~~~~text
    regions.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1649

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1650

~~~~text
    Operations nested in its body region are evaluated in order.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1651

~~~~text
    As opposed to a regular loop, each nested operation is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1652

~~~~text
    fully evaluated for all the values in the "active set of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1653

~~~~text
    index-name" before the next nested operation. In practice, the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1654

~~~~text
    nested operation evaluation may be fused if it is proven that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1655

~~~~text
    they do not have data dependency.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1656

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1657

~~~~text
    The "index-name" value is represented as the argument of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1658

~~~~text
    body region.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1659

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1660

~~~~text
    The lower, upper, and step region (if provided), must be terminated
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1661

~~~~text
    by hlfir.yield that yields scalar integers.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1662

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1663

~~~~text
    The body region must only contain other OrderedAssignmentTreeOpInterface
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1664

~~~~text
    operations (like hlfir.region_assign, or other hlfir.forall).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1665

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1666

~~~~text
    A Fortran forall with several indices is represented as a nest
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1667

~~~~text
    of hlfir.forall.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1668

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1669

~~~~text
    All the regions contained in the hlfir.forall must only contain
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1670

~~~~text
    code that is pure from a Fortran point of view, except for the
~~~~
- EN: Defines a literal code/documentation fragment stored in a TableGen field.
- CN: 定义存入 TableGen 字段的代码/文档片段。

### Line 1671

~~~~text
    assignment effect of the hlfir.region_assign.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1672

~~~~text
    This matches Fortran constraint C1037, but requires the outer
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1673

~~~~text
    controls to be evaluated outside of the hlfir.forall (these
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1674

~~~~text
    controls may have side effects as per Fortran 2018 10.1.4 section).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1675

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1676

~~~~text
    Example: FORALL(I=1:10) X(I) = FOO(I)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1677

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1678

~~~~text
      hlfir.forall lb {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1679

~~~~text
        hlfir.yield %c1 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1680

~~~~text
      } ub {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1681

~~~~text
        hlfir.yield %c10 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1682

~~~~text
      } (%i : index) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1683

~~~~text
        hlfir.region_assign {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1684

~~~~text
          %res = fir.call @foo(%i) : (index) -> f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1685

~~~~text
          hlfir.yield %res : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1686

~~~~text
        } to {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1687

~~~~text
          %xi = hlfir.designate %x(%i) : (!fir.box<!fir.array<?xf32>>, index) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1688

~~~~text
          hlfir.yield %xi : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1689

~~~~text
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1690

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1691

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1692

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1693

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1694

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1695

~~~~text
  let regions = (region  YieldIntegerRegion:$lb_region,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1696

~~~~text
                         YieldIntegerRegion:$ub_region,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1697

~~~~text
                         MaybeYieldIntegerRegion:$step_region,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1698

~~~~text
                         SizedRegion<1>:$body);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1699

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1700

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1701

~~~~text
    mlir::Value getForallIndexValue() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1702

~~~~text
      return getBody().getArguments()[0];
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1703

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1704

~~~~text
    void getLeafRegions(llvm::SmallVectorImpl<mlir::Region*>& regions) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1705

~~~~text
      regions.push_back(&getLbRegion());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1706

~~~~text
      regions.push_back(&getUbRegion());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1707

~~~~text
      if (!getStepRegion().empty())
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 1708

~~~~text
        regions.push_back(&getStepRegion());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1709

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1710

~~~~text
    mlir::Region* getSubTreeRegion() { return &getBody(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1711

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1712

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1713

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1714

~~~~text
    attr-dict `lb` $lb_region
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1715

~~~~text
    `ub` $ub_region
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1716

~~~~text
    (`step` $step_region^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1717

~~~~text
    custom<ForallOpBody>($body)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1718

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1719

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1720

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1721

~~~~text
/// Shared definition for hlfir.forall_mask and hlfir.where
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1722

~~~~text
/// that have the same structure and assembly format, but not the same
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1723

~~~~text
/// constraints.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1724

~~~~text
class hlfir_AssignmentMaskOp<string mnemonic> : hlfir_Op<mnemonic,
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 1725

~~~~text
    [hlfir_OrderedAssignmentTreeOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1726

~~~~text
  let regions = (region  SizedRegion<1>:$mask_region,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1727

~~~~text
                         SizedRegion<1>:$body);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1728

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1729

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1730

~~~~text
    void getLeafRegions(llvm::SmallVectorImpl<mlir::Region*>& regions) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1731

~~~~text
      regions.push_back(&getMaskRegion());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1732

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1733

~~~~text
    mlir::Region* getSubTreeRegion() { return &getBody(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1734

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1735

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1736

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1737

~~~~text
    $mask_region
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1738

~~~~text
    attr-dict `do`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1739

~~~~text
    custom<AssignmentMaskOpBody>($body)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1740

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1741

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1742

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1743

~~~~text
def hlfir_ForallMaskOp : hlfir_AssignmentMaskOp<"forall_mask"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1744

~~~~text
  let summary = "Represent a Fortran forall mask";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1745

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1746

~~~~text
    Fortran Forall can have a scalar mask expression that depends on the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1747

~~~~text
    Forall index-name value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1748

~~~~text
    hlfir.forall_mask allows representing this mask. The expression
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1749

~~~~text
    evaluation is held in the mask region that must yield an i1 scalar
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1750

~~~~text
    value.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1751

~~~~text
    An hlfir.forall_mask must be directly nested in the body region of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1752

~~~~text
    an hlfir.forall. It is a separate operation so that it can use the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1753

~~~~text
    index SSA value defined by the hlfir.forall body region.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1754

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1755

~~~~text
    Example: "FORALL(I=1:10, SOME_CONDITION(I)) X(I) = FOO(I)"
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
    hlfir.forall lb {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1758

~~~~text
      hlfir.yield %c1 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1759

~~~~text
    } ub {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1760

~~~~text
      hlfir.yield %c10 : index
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1761

~~~~text
    } (%i : index) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1762

~~~~text
      hlfir.forall_mask {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1763

~~~~text
        %mask = fir.call @some_condition(%i) : (index) -> i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1764

~~~~text
        hlfir.yield %mask : i1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1765

~~~~text
      } do {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1766

~~~~text
        hlfir.region_assign {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1767

~~~~text
          %res = fir.call @foo(%i) : (index) -> f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1768

~~~~text
          hlfir.yield %res : f32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1769

~~~~text
        } to {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1770

~~~~text
          %xi = hlfir.designate %x(%i) : (!fir.box<!fir.array<?xf32>>, index) -> !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1771

~~~~text
          hlfir.yield %xi : !fir.ref<f32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1772

~~~~text
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1773

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1774

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1775

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1776

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1777

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1778

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1779

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1780

~~~~text
def hlfir_ExactlyOnceOp : hlfir_Op<"exactly_once", [RecursiveMemoryEffects]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1781

~~~~text
  let summary = "Execute exactly once its region in a WhereOp";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1782

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1783

~~~~text
    Inside a Where assignment, Fortran requires a non elemental call and its
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1784

~~~~text
    arguments to be executed exactly once, regardless of the mask values.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1785

~~~~text
    This operation allows holding these evaluations that cannot be hoisted
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1786

~~~~text
    until potential parent Forall loops have been created.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1787

~~~~text
    It also allows inlining the calls without losing the information that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1788

~~~~text
    these calls must be hoisted.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1789

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1790

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1791

~~~~text
  let regions = (region SizedRegion<1>:$body);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1792

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1793

~~~~text
  let results = (outs AnyFortranEntity:$result);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1794

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1795

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1796

~~~~text
    attr-dict `:` type($result)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1797

~~~~text
    $body
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1798

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1799

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1800

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1801

~~~~text
def hlfir_WhereOp : hlfir_AssignmentMaskOp<"where"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1802

~~~~text
  let summary = "Represent a Fortran where construct or statement";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1803

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1804

~~~~text
    Represent Fortran "where" construct or statement. The mask
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1805

~~~~text
    expression evaluation is held in the mask region that must yield
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1806

~~~~text
    logical array that has the same shape as all the nested
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1807

~~~~text
    hlfir.region_assign left-hand sides, and all the nested hlfir.where
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1808

~~~~text
    or hlfir.elsewhere masks.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1809

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1810

~~~~text
    The values of the where and elsewhere masks form a control mask that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1811

~~~~text
    controls all the nested hlfir.region_assign: only the array element for
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1812

~~~~text
    which the related control mask value is true are assigned. Any right-hand
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1813

~~~~text
    side elemental expression is only evaluated for elements where the control
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1814

~~~~text
    mask is true. See Fortran standard 2018 section 10.2.3 for more detailed
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1815

~~~~text
    about the control mask semantic.
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
    An hlfir.where must not contain any hlfir.forall but it may be contained
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1818

~~~~text
    in such operation. This matches Fortran rules.
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
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1821

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1822

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1823

~~~~text
def hlfir_ElseWhereOp : hlfir_Op<"elsewhere", [Terminator,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1824

~~~~text
    ParentOneOf<["WhereOp", "ElseWhereOp"]>, hlfir_OrderedAssignmentTreeOpInterface]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1825

~~~~text
  let summary = "Represent a Fortran elsewhere statement";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1826

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1827

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1828

~~~~text
    Represent Fortran "elsewhere" construct or statement.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1829

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1830

~~~~text
    It has an optional mask region to hold the evaluation of Fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1831

~~~~text
    optional elsewhere mask expressions. If this region is provided,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1832

~~~~text
    it must satisfy the same constraints as hlfir.where mask region.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1833

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1834

~~~~text
    An hlfir.elsewhere must be the last operation of an hlfir.where or,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1835

~~~~text
    hlfir.elsewhere body, which is enforced by its terminator property.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1836

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1837

~~~~text
    Like in Fortran, an hlfir.elsewhere negate the current control mask,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1838

~~~~text
    and if provided, adds the mask the resulting control mask (with a logical
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1839

~~~~text
    AND).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1840

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1841

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1842

~~~~text
  let regions = (region  MaxSizedRegion<1>:$mask_region,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1843

~~~~text
                         SizedRegion<1>:$body);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1844

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1845

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1846

~~~~text
    void getLeafRegions(llvm::SmallVectorImpl<mlir::Region*>& regions) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1847

~~~~text
      if (!getMaskRegion().empty())
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 1848

~~~~text
        regions.push_back(&getMaskRegion());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1849

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1850

~~~~text
    mlir::Region* getSubTreeRegion() { return &getBody(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1851

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1852

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1853

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1854

~~~~text
    (`mask` $mask_region^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1855

~~~~text
    attr-dict `do`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1856

~~~~text
    custom<AssignmentMaskOpBody>($body)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1857

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1858

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1859

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1860

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1861

~~~~text
def hlfir_ForallIndexOp : hlfir_Op<"forall_index", [fir_FortranVariableOpInterface,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1862

~~~~text
    hlfir_OrderedAssignmentTreeOpInterface, Pure]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1863

~~~~text
  let summary = "represent a Fortran forall index declaration";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1864

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1865

~~~~text
    This operation allows placing an hlfir.forall index in memory with
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1866

~~~~text
    the related Fortran index-value name and type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1867

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1868

~~~~text
    So far, lowering needs to manipulate symbols as memory entities.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1869

~~~~text
    This operation allows fulfilling this requirements without allowing
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1870

~~~~text
    bare alloca/declare/store inside the body of hlfir.forall, which would
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1871

~~~~text
    make their analysis more complex.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1872

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1873

~~~~text
    Given Forall index-value cannot be modified it also allows defining
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1874

~~~~text
    a canonicalization of all its loads into a fir.convert of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1875

~~~~text
    hlfir.forall index, which helps simplifying the data dependency analysis
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1876

~~~~text
    of hlfir.forall.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1877

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1878

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1879

~~~~text
  let arguments = (ins AnyIntegerType:$index,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1880

~~~~text
                       Builtin_StringAttr:$name);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1881

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1882

~~~~text
  let results = (outs AnyFortranVariable);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1883

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1884

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1885

~~~~text
    $name $index attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1886

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1887

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1888

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1889

~~~~text
    /// Implement FortranVariableInterface interface.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1890

~~~~text
    std::optional<fir::FortranVariableFlagsEnum> getFortranAttrs() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1891

~~~~text
      return std::nullopt;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1892

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1893

~~~~text
    void setFortranAttrs(fir::FortranVariableFlagsEnum flags) {}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1894

~~~~text
    mlir::Value getShape() const {return mlir::Value{};}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1895

~~~~text
    mlir::OperandRange getExplicitTypeParams() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1896

~~~~text
      // Return an empty range.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1897

~~~~text
      return {(*this)->getOperands().begin(), (*this)->getOperands().begin()};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1898

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1899

~~~~text
    /// Implement OrderedAssignmentTreeOpInterface interface.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1900

~~~~text
    void getLeafRegions(llvm::SmallVectorImpl<mlir::Region*>& regions) {}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1901

~~~~text
    mlir::Region* getSubTreeRegion() { return nullptr; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1902

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1903

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1904

~~~~text
  let hasCanonicalizeMethod = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1905

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1906

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1907

~~~~text
def hlfir_CharExtremumOp : hlfir_Op<"char_extremum",
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1908

~~~~text
    [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1909

~~~~text
  let summary = "Find max/min from given character strings";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1910

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1911

~~~~text
    Find the lexicographical minimum or maximum of two or more character
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1912

~~~~text
    strings of the same character kind and return the string with the lexicographical
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1913

~~~~text
    minimum or maximum number of characters. Example:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1914

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1915

~~~~text
    %0 = hlfir.char_extremum min, %arg0, %arg1 : (!fir.ref<!fir.char<1,10>>, !fir.ref<!fir.char<1,20>>) -> !hlfir.expr<!fir.char<1,10>>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1916

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1917

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1918

~~~~text
  let arguments = (ins hlfir_CharExtremumPredicateAttr:$predicate,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1919

~~~~text
                  Variadic<AnyScalarCharacterEntity>:$strings
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1920

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1921

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1922

~~~~text
  let results = (outs AnyScalarCharacterExpr);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1923

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1924

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1925

~~~~text
    $predicate `,` $strings attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1926

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1927

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1928

~~~~text
  let builders = [OpBuilder<(ins "hlfir::CharExtremumPredicate":$predicate, "mlir::ValueRange":$strings)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1929

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1930

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1931

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1932

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1933

~~~~text
def hlfir_EvaluateInMemoryOp : hlfir_Op<"eval_in_mem", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 1934

~~~~text
    RecursiveMemoryEffects, RecursivelySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1935

~~~~text
    SingleBlockImplicitTerminator<"fir::FirEndOp">]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1936

~~~~text
  let summary = "Wrap an in-memory implementation that computes expression value";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1937

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1938

~~~~text
    Returns a Fortran expression value for which the computation is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1939

~~~~text
    implemented inside the region operating on the block argument which
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1940

~~~~text
    is a raw memory reference corresponding to the expression type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1941

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1942

~~~~text
    The shape and type parameters of the expressions are operands of the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1943

~~~~text
    operations.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1944

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1945

~~~~text
    The memory cannot escape the region, and it is not described how it is
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1946

~~~~text
    allocated. This facilitates later elision of the temporary storage for the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1947

~~~~text
    expression evaluation if it can be evaluated in some other storage (like a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1948

~~~~text
    left-hand side variable).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1949

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1950

~~~~text
    Example:
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
    A function returning an array can be represented as:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1953

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1954

~~~~text
      %1 = fir.shape %c10 : (index) -> !fir.shape<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1955

~~~~text
      %2 = hlfir.eval_in_mem shape %1 : (!fir.shape<1>) -> !hlfir.expr<10xf32> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1956

~~~~text
      ^bb0(%arg0: !fir.ref<!fir.array<10xf32>>):
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1957

~~~~text
        %3 = fir.call @_QParray_func() fastmath<contract> : () -> !fir.array<10xf32>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1958

~~~~text
        fir.save_result %3 to %arg0(%1) : !fir.array<10xf32>, !fir.ref<!fir.array<10xf32>>, !fir.shape<1>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1959

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1960

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1961

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1962

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1963

~~~~text
  let arguments = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1964

~~~~text
    Optional<fir_ShapeType>:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1965

~~~~text
    Variadic<AnyIntegerType>:$typeparams
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1966

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1967

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1968

~~~~text
  let results = (outs hlfir_ExprType);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1969

~~~~text
  let regions = (region  SizedRegion<1>:$body);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1970

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1971

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1972

~~~~text
    (`shape` $shape^)? (`typeparams` $typeparams^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1973

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1974

~~~~text
    $body}];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1975

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1976

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1977

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1978

~~~~text
    OpBuilder<(ins "mlir::Type":$result_type, "mlir::Value":$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1979

~~~~text
      CArg<"mlir::ValueRange", "{}">:$typeparams)>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1980

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1981

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1982

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1983

~~~~text
      // Return block argument representing the memory where the expression
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1984

~~~~text
      // is evaluated.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1985

~~~~text
      mlir::Value getMemory() {return getBody().getArgument(0);}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1986

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1987

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1988

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 1989

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1990

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1991

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1992

~~~~text
#endif // FORTRAN_DIALECT_HLFIR_OPS
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
  - `flang/Optimizer/HLFIR/HLFIROpBase.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/CUF/Attributes/CUFAttr.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FIRTypes.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FIRAttr.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FortranVariableInterface.td` — referenced directly from this file / 该文件直接引用
  - `mlir/Dialect/Arith/IR/ArithBase.td` — referenced directly from this file / 该文件直接引用
  - `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td` — referenced directly from this file / 该文件直接引用
  - `mlir/IR/BuiltinAttributes.td` — referenced directly from this file / 该文件直接引用
- **External or standard dependencies / 外部或标准依赖**: none detected directly / 未直接检测到
