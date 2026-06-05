# MIFOps.td — Code Analysis / 代码分析

## Source / 来源

- File: `flang/include/flang/Optimizer/Dialect/MIF/MIFOps.td`
- Repository: `llvm-project` / `llvm-project`
- Purpose (EN): MIF operation definitions \file Definition of the MIF dialect operations.
- Purpose (CN): 声明与 MIFOps 相关的接口、数据结构或工具。

## Line-by-Line Analysis / 逐行分析

### Line 1

~~~~text
//===-- MIFOps.td - MIF operation definitions --------------*- tablegen -*-===//
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
/// Definition of the MIF dialect operations
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
#ifndef FORTRAN_DIALECT_MIF_MIF_OPS
~~~~
- EN: Controls conditional compilation with `#ifndef`.
- CN: 使用 `#ifndef` 控制条件编译。

### Line 15

~~~~text
#define FORTRAN_DIALECT_MIF_MIF_OPS
~~~~
- EN: Defines the preprocessor macro `FORTRAN_DIALECT_MIF_MIF_OPS`.
- CN: 定义预处理宏 `FORTRAN_DIALECT_MIF_MIF_OPS`。

### Line 16

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 17

~~~~text
include "flang/Optimizer/Dialect/MIF/MIFDialect.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/MIF/MIFDialect.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/MIF/MIFDialect.td`，以便在此复用其中的记录。

### Line 18

~~~~text
include "flang/Optimizer/Dialect/FIRTypes.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FIRTypes.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FIRTypes.td`，以便在此复用其中的记录。

### Line 19

~~~~text
include "flang/Optimizer/Dialect/FIRAttr.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FIRAttr.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FIRAttr.td`，以便在此复用其中的记录。

### Line 20

~~~~text
include "mlir/IR/BuiltinAttributes.td"
~~~~
- EN: Includes the TableGen file `mlir/IR/BuiltinAttributes.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/IR/BuiltinAttributes.td`，以便在此复用其中的记录。

### Line 21

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 22

~~~~text
class mif_Op<string mnemonic, list<Trait> traits>
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 23

~~~~text
    : Op<MIFDialect, mnemonic, traits>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 24

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 25

~~~~text
class region_Op<string mnemonic, list<Trait> traits = []>
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 26

~~~~text
    : mif_Op<mnemonic, !listconcat(traits, [RecursivelySpeculatable,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 27

~~~~text
                                            RecursiveMemoryEffects])> {}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 28

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 29

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 30

~~~~text
// Initialization and Finalization
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 31

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 32

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 33

~~~~text
def mif_InitOp : mif_Op<"init", []> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 34

~~~~text
  let summary = "Initialize the parallel environment";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 35

~~~~text
  let description = [{This operation will initialize the parallel environment}];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 36

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 37

~~~~text
  let results = (outs I32:$stat);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 38

~~~~text
  let assemblyFormat = "`->` type($stat) attr-dict";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 39

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 40

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 41

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 42

~~~~text
// Image Queries
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 43

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 44

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 45

~~~~text
def mif_NumImagesOp : mif_Op<"num_images", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 46

~~~~text
  let summary = "Query the number of images in the specified or current team";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 47

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 48

~~~~text
    This operation query the number of images in the specified or current
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 49

~~~~text
    team and can be called with 3 differents way :
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 50

~~~~text
    - `num_images()`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 51

~~~~text
    - `num_images(team)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 52

~~~~text
    - `num_images(team_number)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 53

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 54

~~~~text
    Arguments:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 55

~~~~text
    - `team` : Shall be a scalar of type `team_type` from the `ISO_FORTRAN_ENV`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 56

~~~~text
            module with a value that identifies the current or ancestor team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 57

~~~~text
    - `team_number` :  Shall be an integer scalar. It shall identify the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 58

~~~~text
            initial team or a sibling team of the current team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 59

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 60

~~~~text
    Result Value: The number of images in the specified team, or in the current
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 61

~~~~text
    team if no team is specified.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 62

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 63

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 64

~~~~text
  let arguments = (ins Optional<AnyInteger>:$team_number,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 65

~~~~text
                       Optional<AnyRefOrBoxType>:$team);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 66

~~~~text
  let results = (outs I32:$res);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 67

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 68

~~~~text
  let builders = [OpBuilder<(ins CArg<"mlir::Value", "{}">:$teamArg)>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 69

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 70

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 71

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 72

~~~~text
    ( `team_number` $team_number^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 73

~~~~text
    ( `team` $team^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 74

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 75

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 76

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 77

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 78

~~~~text
def mif_ThisImageOp : mif_Op<"this_image", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 79

~~~~text
  let summary = "Determine the image index of the current image";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 80

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 81

~~~~text
    Arguments:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 82

~~~~text
    - `coarray` :  Shall be a coarray of any type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 83

~~~~text
    - `dim` : Shall be an integer scalar. Its value shall be in the range of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 84

~~~~text
          1 <= DIM <= N, where N is the corank of the coarray.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 85

~~~~text
    - `team`(optional) : Shall be a scalar of type `team_type` from
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 86

~~~~text
          ISO_FORTRAN_ENV. If the `coarray` is present, it shall be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 87

~~~~text
          established in that team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 88

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 89

~~~~text
    Results:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 90

~~~~text
    - Case(1) : The result of `this_image([team])` is a scalar with a value
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 91

~~~~text
          equal to the index of the image in the current or specified team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 92

~~~~text
    - Case(2) : The result of `this_image(coarray [,team])` is the sequence of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 93

~~~~text
          cosubscript values for `coarray`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 94

~~~~text
    - Case(3) : The result of `this_image(coarray, dim [,team])` is the value of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 95

~~~~text
          cosubscript `dim` in the sequence of cosubscript values for `coarray`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 96

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 97

~~~~text
    Example:
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 98

~~~~text
    ```fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 99

~~~~text
      REAL :: A[10, 0:9, 0:*]
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 100

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 101

~~~~text
    If we take a look on the example and we are on image 5, `this_image` has the
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 102

~~~~text
    value 5, `this_image(A)` has the value [5, 0, 0].
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 103

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 104

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 105

~~~~text
  let arguments = (ins Optional<fir_BoxType>:$coarray,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 106

~~~~text
      Optional<AnyInteger>:$dim, Optional<AnyRefOrBoxType>:$team);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 107

~~~~text
  let results = (outs I32:$res);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 108

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 109

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$coarray, "mlir::Value":$team)>,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 110

~~~~text
                  OpBuilder<(ins "mlir::Value":$team)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 111

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 112

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 113

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 114

~~~~text
    ( `coarray` $coarray^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 115

~~~~text
    ( `team` $team^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 116

~~~~text
    ( `dim` $dim^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 117

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 118

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 119

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 120

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 121

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 122

~~~~text
// Synchronization
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 123

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 124

~~~~text
// NOTE: Every operation in this section corresponds to a Fortran 
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 125

~~~~text
//   "image control statement" (F23 11.7.1). This means they end a Fortran 
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 126

~~~~text
//   "segment" and start another (F23 11.7.2), which affects the memory 
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 127

~~~~text
//   consistency semantics of surrounding accesses in the multi-image execution. 
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 128

~~~~text
//   For now we've modeled this by *deliberately* omitting mlir::MemoryEffects
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 129

~~~~text
//   on these operations, to ensure analysis treats these ops conservatively.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 130

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 131

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 132

~~~~text
def mif_SyncAllOp : mif_Op<"sync_all", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 133

~~~~text
  let summary =
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 134

~~~~text
      "Performs a collective synchronization of all images in the current team";
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 135

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 136

~~~~text
  let arguments = (ins Optional<AnyReferenceLike>:$stat,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 137

~~~~text
                       Optional<AnyRefOrBoxType>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 138

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 139

~~~~text
    (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 140

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 141

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 142

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 143

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 144

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 145

~~~~text
def mif_SyncImagesOp : mif_Op<"sync_images", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 146

~~~~text
  let summary = "Performs a synchronization of image with each of the other "
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 147

~~~~text
                "images in the `image_set`";
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 148

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 149

~~~~text
    This operation can take an optional argument `⁣image_set`, wich must be an integer expression
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 150

~~~~text
    and must be scalar or rank one. If `image_set` is omitted from the call, this operation will 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 151

~~~~text
    adopt the behavior of the Fortran statement `SYNC IMAGES(*)`.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 152

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 153

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 154

~~~~text
  let arguments = (ins Optional<AnyRefOrBoxType>:$image_set,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 155

~~~~text
                       Optional<AnyReferenceLike>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 156

~~~~text
                       Optional<AnyRefOrBoxType>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 157

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 158

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 159

~~~~text
    (`image_set` $image_set^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 160

~~~~text
    (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 161

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 162

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 163

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 164

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 165

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 166

~~~~text
def mif_SyncMemoryOp : mif_Op<"sync_memory", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 167

~~~~text
  let summary = "Operation that ends one segment and begins another.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 168

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 169

~~~~text
    Operation that ends one segment and begins another; Those two segments can 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 170

~~~~text
    be ordered by user-defined way with respect to segments on other images.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 171

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 172

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 173

~~~~text
  let arguments = (ins Optional<AnyReferenceLike>:$stat,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 174

~~~~text
                       Optional<AnyRefOrBoxType>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 175

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 176

~~~~text
    (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 177

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 178

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 179

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 180

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 181

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 182

~~~~text
def mif_SyncTeamOp : mif_Op<"sync_team", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 183

~~~~text
  let summary = "Performs a synchronization of the team, identified by `team`";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 184

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 185

~~~~text
  let arguments = (ins AnyRefOrBoxType:$team, Optional<AnyReferenceLike>:$stat,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 186

~~~~text
      Optional<AnyRefOrBoxType>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 187

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 188

~~~~text
    $team (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 189

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 190

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 191

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 192

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 193

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 194

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 195

~~~~text
// Collective Operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 196

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 197

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 198

~~~~text
def mif_CoBroadcastOp : mif_Op<"co_broadcast", [AttrSizedOperandSegments,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 199

~~~~text
                                                MemoryEffects<[MemWrite]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 200

~~~~text
  let summary = "Broadcast value to images.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 201

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 202

~~~~text
    The co_broadcast operation broadcasts a value from one image to the other images.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 203

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 204

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 205

~~~~text
  let arguments = (ins Arg<fir_BoxType, "", [MemRead, MemWrite]>:$a,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 206

~~~~text
      AnyIntegerType:$source_image,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 207

~~~~text
      Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 208

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 209

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 210

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 211

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 212

~~~~text
    $a `source` $source_image
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 213

~~~~text
    (`stat`  $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 214

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 215

~~~~text
    attr-dict `:` `(` type(operands) `)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 216

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 217

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 218

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 219

~~~~text
def mif_CoMaxOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 220

~~~~text
    : mif_Op<"co_max", [AttrSizedOperandSegments, MemoryEffects<[MemWrite]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 221

~~~~text
  let summary = "Compute maximum value across images.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 222

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 223

~~~~text
    The co_max operation performs the computation of the maximum 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 224

~~~~text
    across images.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 225

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 226

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 227

~~~~text
  let arguments = (ins Arg<fir_BoxType, "", [MemRead, MemWrite]>:$a,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 228

~~~~text
      Optional<AnyIntegerType>:$result_image,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 229

~~~~text
      Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 230

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 231

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 232

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 233

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 234

~~~~text
    $a (`result` $result_image^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 235

~~~~text
    (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 236

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 237

~~~~text
    attr-dict `:` `(` type(operands) `)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 238

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 239

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 240

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 241

~~~~text
def mif_CoMinOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 242

~~~~text
    : mif_Op<"co_min", [AttrSizedOperandSegments, MemoryEffects<[MemWrite]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 243

~~~~text
  let summary = "Compute minimum value across images.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 244

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 245

~~~~text
    The co_min operation performs the computation of the minimum
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 246

~~~~text
    across images.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 247

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 248

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 249

~~~~text
  let arguments = (ins Arg<fir_BoxType, "", [MemRead, MemWrite]>:$a,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 250

~~~~text
      Optional<AnyIntegerType>:$result_image,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 251

~~~~text
      Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 252

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 253

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 254

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 255

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 256

~~~~text
    $a (`result` $result_image^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 257

~~~~text
    (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 258

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 259

~~~~text
    attr-dict `:` `(` type(operands) `)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 260

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 261

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 262

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 263

~~~~text
def mif_CoSumOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 264

~~~~text
    : mif_Op<"co_sum", [AttrSizedOperandSegments, MemoryEffects<[MemWrite]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 265

~~~~text
  let summary = "Compute sum across images.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 266

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 267

~~~~text
    The co_sum operation performs the computation of the sum
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 268

~~~~text
    across images.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 269

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 270

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 271

~~~~text
  let arguments = (ins Arg<fir_BoxType, "", [MemRead, MemWrite]>:$a,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 272

~~~~text
      Optional<AnyIntegerType>:$result_image,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 273

~~~~text
      Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 274

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 275

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 276

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 277

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 278

~~~~text
    $a (`result` $result_image^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 279

~~~~text
    (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 280

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 281

~~~~text
    attr-dict `:` `(` type(operands) `)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 282

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 283

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 284

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 285

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 286

~~~~text
// Teams
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 287

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 288

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 289

~~~~text
def mif_FormTeamOp : mif_Op<"form_team", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 290

~~~~text
  let summary =
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 291

~~~~text
      "Create a set of sibling teams whose parent team is the current team.";
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 292

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 293

~~~~text
    Create a new team for each unique `team_number` value specified.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 294

~~~~text
    Each executing image will belong to the team whose `team_number` is equal 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 295

~~~~text
    to the value of team-number on that image, and `team_var` becomes defined
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 296

~~~~text
    with a value that identifies that team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 297

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 298

~~~~text
    If `new_index` is specified, the image index of the executing image will take
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 299

~~~~text
    this index in its new team. Otherwise, the new image index is processor 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 300

~~~~text
    dependent.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 301

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 302

~~~~text
    Arguments: 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 303

~~~~text
      - `team_number`: Shall be a positive integer.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 304

~~~~text
      - `team_var` : Shall be a variable of type TEAM_TYPE from the intrinsic
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 305

~~~~text
        module ISO_FORTRAN_ENV.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 306

~~~~text
      - `new_index`(optional): Shall be an integer that correspond to the index that
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 307

~~~~text
        the calling image will have in the new team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 308

~~~~text
  }];
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
  let arguments = (ins AnyIntegerType:$team_number,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 311

~~~~text
      Arg<fir_BoxType, "", [MemWrite]>:$team_var,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 312

~~~~text
      Optional<AnyIntegerType>:$new_index,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 313

~~~~text
      Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 314

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 315

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 316

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 317

~~~~text
    `team_number` $team_number `team_var` $team_var
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 318

~~~~text
    (`new_index` $new_index^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 319

~~~~text
    (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 320

~~~~text
    (`errmsg` $errmsg^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 321

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 322

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 323

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 324

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 325

~~~~text
def mif_EndTeamOp : mif_Op<"end_team", [AttrSizedOperandSegments, Terminator,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 326

~~~~text
                                        ParentOneOf<["ChangeTeamOp"]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 327

~~~~text
  let summary = "Changes the current team to the parent team.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 328

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 329

~~~~text
    The END TEAM operation completes the CHANGE TEAM construct and 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 330

~~~~text
    restores the current team to the team that was current before 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 331

~~~~text
    the CHANGE TEAM construct. 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 332

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 333

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 334

~~~~text
  let arguments = (ins Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 335

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 336

~~~~text
  let builders = [OpBuilder<(ins), [{ /* do nothing */ }]>];
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 337

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 338

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 339

~~~~text
    (`stat` $stat^ )? (`errmsg` $errmsg^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 340

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 341

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 342

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 343

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 344

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 345

~~~~text
// NOTE: The CHANGE TEAM region will take a coarray association list in
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 346

~~~~text
// argument. However, coarray management and coarray alias creation are not
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 347

~~~~text
// yet supported by the dialect. The argument is therefore not yet supported by
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 348

~~~~text
// this operation and will be added later.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 349

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 350

~~~~text
def mif_ChangeTeamOp : region_Op<"change_team", [AttrSizedOperandSegments]> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 351

~~~~text
  let summary = "Changes the current team.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 352

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 353

~~~~text
    The CHANGE TEAM construct changes the current team to the specified new
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 354

~~~~text
    team, which must be a child team of the current team.  
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 355

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 356

~~~~text
    ```
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 357

~~~~text
      mif.change_team %team {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 358

~~~~text
        %x = fir.convert %i : (index) -> i32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 359

~~~~text
        ...
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 360

~~~~text
        mif.end_team
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 361

~~~~text
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 362

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 363

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 364

~~~~text
  let arguments = (ins AnyRefOrBoxType:$team,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 365

~~~~text
      Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 366

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 367

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 368

~~~~text
  // The region can contain multiple blocks if the `change_team` construct
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 369

~~~~text
  // contains constructs that create multiple blocks.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 370

~~~~text
  let regions = (region MinSizedRegion<1>:$region);
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
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 373

~~~~text
  let builders =
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 374

~~~~text
      [OpBuilder<(ins "mlir::Value":$team,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 375

~~~~text
           CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 376

~~~~text
       OpBuilder<(ins "mlir::Value":$team, "mlir::Value":$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 377

~~~~text
           "mlir::Value":$errmsg,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 378

~~~~text
           CArg<"llvm::ArrayRef<mlir::NamedAttribute>", "{}">:$attributes)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 379

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 380

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 381

~~~~text
    /// Get the body of the CHANGE TEAM construct 
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 382

~~~~text
    mlir::Block *getBody() { return &getRegion().front(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 383

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 384

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 385

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 386

~~~~text
    $team (`stat` $stat^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 387

~~~~text
    (`errmsg` $errmsg^)?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 388

~~~~text
    attr-dict `:` `(` type(operands) `)`
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 389

~~~~text
    custom<ChangeTeamOpBody>($region)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 390

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 391

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 392

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 393

~~~~text
def mif_GetTeamOp : mif_Op<"get_team", []> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 394

~~~~text
  let summary = "Get the team value for the current or ancestor team.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 395

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 396

~~~~text
    This operation gets the team value for the current or an ancestor team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 397

~~~~text
    `level`(optional): If provided, must equal one of the following constants : 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 398

~~~~text
    `INITIAL_TEAM`, `PARENT_TEAM` or `CURRENT_TEAM` from the module ISO_FORTRAN_ENV.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 399

~~~~text
    If `level` isn't present or has the value `CURRENT_TEAM` the returned
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 400

~~~~text
    value is the current team.     
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 401

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 402

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 403

~~~~text
  let arguments = (ins Optional<AnyIntegerType>:$level);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 404

~~~~text
  let results = (outs fir_BoxType:$team);
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
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 407

~~~~text
    (`level` $level^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 408

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 409

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 410

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 411

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 412

~~~~text
def mif_TeamNumberOp : mif_Op<"team_number", []> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 413

~~~~text
  let summary = "Get the team number";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 414

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 415

~~~~text
    Argument: `team` is optional and shall be a scalar of type TEAM_TYPE from 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 416

~~~~text
    module ISO_FORTRAN_ENV and the value identifies the current or an ancestor team. 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 417

~~~~text
    If `team` is absent, the team specified is the current team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 418

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 419

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 420

~~~~text
  let arguments = (ins Optional<AnyRefOrBoxType>:$team);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 421

~~~~text
  let results = (outs I64);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 422

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 423

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 424

~~~~text
    (`team` $team^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 425

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 426

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 427

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 428

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 429

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 430

~~~~text
// Allocation and Deallocation
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 431

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 432

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 433

~~~~text
def mif_AllocCoarrayOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 434

~~~~text
    : mif_Op<"alloc_coarray", [AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 435

~~~~text
                               MemoryEffects<[MemAlloc<DefaultResource>]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 436

~~~~text
  let summary = "Perform the allocation of a coarray and provide a "
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 437

~~~~text
                "corresponding coarray descriptor";
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 438

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 439

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 440

~~~~text
    This operation allocates a coarray and provides the corresponding 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 441

~~~~text
    coarray descriptor. This call is collective over the current team.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 442

~~~~text
    `lcobound` should be a rank-1 array of size equal to `corank`, and 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 443

~~~~text
    `ucobounds` should be a rank-1 array of size equal to `corank - 1`.
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
  let arguments = (ins StrAttr:$uniq_name,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 447

~~~~text
      Arg<fir_ReferenceType, "", [MemRead, MemWrite]>:$box,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 448

~~~~text
      AnyBoxedArray:$lcobounds, AnyBoxedArray:$ucobounds,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 449

~~~~text
      Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 450

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 451

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 452

~~~~text
  let builders = [OpBuilder<(ins "mlir::Value":$box, "llvm::StringRef":$symName,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 453

~~~~text
                      "mlir::Value":$lcobounds, "mlir::Value":$ucobounds,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 454

~~~~text
                      "mlir::Value":$stat, "mlir::Value":$errmsg)>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 455

~~~~text
                  OpBuilder<(ins "mlir::Value":$box, "llvm::StringRef":$symName,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 456

~~~~text
                      "mlir::Value":$lcobounds, "mlir::Value":$ucobounds)>];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 457

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 458

~~~~text
  let hasVerifier = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 459

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 460

~~~~text
    $box 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 461

~~~~text
    `lcobounds` $lcobounds `ucobounds` $ucobounds 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 462

~~~~text
    (`stat` $stat^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 463

~~~~text
    (`errmsg` $errmsg^ )? 
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 464

~~~~text
    attr-dict `:` functional-type(operands, results)
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
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 467

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 468

~~~~text
def mif_DeallocCoarrayOp
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 469

~~~~text
    : mif_Op<"dealloc_coarray", [AttrSizedOperandSegments,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 470

~~~~text
                                 MemoryEffects<[MemFree<DefaultResource>]>]> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 471

~~~~text
  let summary = "Perform the deallocation of a coarray";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 472

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 473

~~~~text
    This call releases memory allocated by `mif_AllocCoarrayOp` for a coarray.
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
  let arguments = (ins Arg<fir_ReferenceType, "", [MemFree]>:$coarray,
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 476

~~~~text
      Arg<Optional<AnyReferenceLike>, "", [MemWrite]>:$stat,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 477

~~~~text
      Arg<Optional<AnyRefOrBoxType>, "", [MemWrite]>:$errmsg);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 478

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 479

~~~~text
  let assemblyFormat = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 480

~~~~text
    $coarray (`stat` $stat^ )?  (`errmsg` $errmsg^ )?
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 481

~~~~text
    attr-dict `:` functional-type(operands, results)
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 482

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 483

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 484

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 485

~~~~text
#endif // FORTRAN_DIALECT_MIF_MIF_OPS
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
  - `flang/Optimizer/Dialect/MIF/MIFDialect.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FIRTypes.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FIRAttr.td` — referenced directly from this file / 该文件直接引用
  - `mlir/IR/BuiltinAttributes.td` — referenced directly from this file / 该文件直接引用
- **External or standard dependencies / 外部或标准依赖**: none detected directly / 未直接检测到
