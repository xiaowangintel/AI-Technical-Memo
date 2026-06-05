# FIRTypes.td — Code Analysis / 代码分析

## Source / 来源

- File: `flang/include/flang/Optimizer/Dialect/FIRTypes.td`
- Repository: `llvm-project` / `llvm-project`
- Purpose (EN): FIR types This file declares the FIR dialect types.
- Purpose (CN): 声明与 FIRTypes 相关的接口、数据结构或工具。

## Line-by-Line Analysis / 逐行分析

### Line 1

~~~~text
//===- FIRTypes.td - FIR types -----------------------------*- tablegen -*-===//
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
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 9

~~~~text
// This file declares the FIR dialect types.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 10

~~~~text
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 11

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 12

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 13

~~~~text
#ifndef FIR_DIALECT_FIR_TYPES
~~~~
- EN: Controls conditional compilation with `#ifndef`.
- CN: 使用 `#ifndef` 控制条件编译。

### Line 14

~~~~text
#define FIR_DIALECT_FIR_TYPES
~~~~
- EN: Defines the preprocessor macro `FIR_DIALECT_FIR_TYPES`.
- CN: 定义预处理宏 `FIR_DIALECT_FIR_TYPES`。

### Line 15

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 16

~~~~text
include "mlir/IR/AttrTypeBase.td"
~~~~
- EN: Includes the TableGen file `mlir/IR/AttrTypeBase.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `mlir/IR/AttrTypeBase.td`，以便在此复用其中的记录。

### Line 17

~~~~text
include "flang/Optimizer/Dialect/FIRDialect.td"
~~~~
- EN: Includes the TableGen file `flang/Optimizer/Dialect/FIRDialect.td` so its records can be reused here.
- CN: 引入 TableGen 文件 `flang/Optimizer/Dialect/FIRDialect.td`，以便在此复用其中的记录。

### Line 18

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 19

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 20

~~~~text
// FIR Types
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 21

~~~~text
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 22

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 23

~~~~text
class FIR_Type<string name, string typeMnemonic, list<Trait> traits = [],
~~~~
- EN: Declares a reusable TableGen class/template.
- CN: 声明可复用的 TableGen 类/模板。

### Line 24

~~~~text
               string baseCppClass = "::mlir::Type">
~~~~
- EN: Declares a string-valued TableGen field.
- CN: 声明一个字符串类型的 TableGen 字段。

### Line 25

~~~~text
    : TypeDef<FIROpsDialect, name, traits, baseCppClass> {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 26

~~~~text
  let mnemonic = typeMnemonic;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 27

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 28

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 29

~~~~text
def fir_Type : Type<CPred<"fir::isa_fir_or_std_type($_self)">,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 30

~~~~text
    "FIR dialect type">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 31

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 32

~~~~text
def fir_BoxCharType : FIR_Type<"BoxChar", "boxchar"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 33

~~~~text
  let summary = "CHARACTER type descriptor.";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 34

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 35

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 36

~~~~text
    The type of a pair that describes a CHARACTER variable. Specifically, a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 37

~~~~text
    CHARACTER consists of a reference to a buffer (the string value) and a LEN
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 38

~~~~text
    type parameter (the runtime length of the buffer).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 39

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 40

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 41

~~~~text
  let parameters = (ins "KindTy":$kind);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 42

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 43

~~~~text
  let genAccessors = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 44

~~~~text
  let hasCustomAssemblyFormat = 1;
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
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 47

~~~~text
    using KindTy = unsigned;
~~~~
- EN: Creates the alias `KindTy` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `KindTy`。

### Line 48

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 49

~~~~text
    // a !fir.boxchar<k> always wraps a !fir.char<k, ?>
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 50

~~~~text
    CharacterType getElementType(mlir::MLIRContext *context) const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 51

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 52

~~~~text
    CharacterType getEleTy() const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 53

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 54

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 55

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 56

~~~~text
def fir_BoxProcType : FIR_Type<"BoxProc", "boxproc"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 57

~~~~text
  let summary = "";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 58

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 59

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 60

~~~~text
    The type of a pair that describes a PROCEDURE reference. Pointers to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 61

~~~~text
    internal procedures must carry an additional reference to the host's
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 62

~~~~text
    variables that are referenced.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 63

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 64

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 65

~~~~text
  let parameters = (ins "mlir::Type":$eleTy);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 66

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 67

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 68

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 69

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 70

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 71

~~~~text
def fir_BoxType : FIR_Type<"Box", "box", [], "BaseBoxType"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 72

~~~~text
  let summary = "The type of a Fortran descriptor";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 73

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 74

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 75

~~~~text
    Descriptors are tuples of information that describe an entity being passed
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 76

~~~~text
    from a calling context. This information might include (but is not limited
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 77

~~~~text
    to) whether the entity is an array, its size, or what type it has.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 78

~~~~text
  }];
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
  let parameters = (ins "mlir::Type":$eleTy, "bool":$isVolatile);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 81

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 82

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 83

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 84

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 85

~~~~text
    TypeBuilderWithInferredContext<(ins
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 86

~~~~text
        "mlir::Type":$eleTy, CArg<"bool", "false">:$isVolatile), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 87

~~~~text
      return Base::get(eleTy.getContext(), eleTy, isVolatile);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 88

~~~~text
    }]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 89

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 90

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 91

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 92

~~~~text
    mlir::Type getElementType() const { return getEleTy(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 93

~~~~text
    bool isVolatile() const { return getIsVolatile(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 94

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 95

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 96

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 97

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 98

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 99

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 100

~~~~text
def fir_CharacterType : FIR_Type<"Character", "char"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 101

~~~~text
  let summary = "FIR character type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 102

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 103

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 104

~~~~text
    Model of the Fortran CHARACTER intrinsic type, including the KIND type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 105

~~~~text
    parameter. The model optionally includes a LEN type parameter. A
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 106

~~~~text
    CharacterType is thus the type of both a single character value and a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 107

~~~~text
    character with a LEN parameter.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 108

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 109

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 110

~~~~text
  let parameters = (ins "KindTy":$FKind, "CharacterType::LenType":$len);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 111

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 112

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 113

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 114

~~~~text
    using KindTy = unsigned;
~~~~
- EN: Creates the alias `KindTy` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `KindTy`。

### Line 115

~~~~text
    using LenType = std::int64_t;
~~~~
- EN: Creates the alias `LenType` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `LenType`。

### Line 116

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 117

~~~~text
    /// Return unknown length Character type. e.g., CHARACTER(LEN=n).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 118

~~~~text
    static CharacterType getUnknownLen(mlir::MLIRContext *ctxt, KindTy kind) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 119

~~~~text
      return get(ctxt, kind, unknownLen());
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 120

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 121

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 122

~~~~text
    /// Return length 1 Character type. e.g., CHARACTER(LEN=1).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 123

~~~~text
    static CharacterType getSingleton(mlir::MLIRContext *ctxt, KindTy kind) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 124

~~~~text
      return get(ctxt, kind, singleton());
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 125

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 126

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 127

~~~~text
    /// Character is a singleton and has a LEN of 1.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 128

~~~~text
    static constexpr LenType singleton() { return 1; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 129

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 130

~~~~text
    /// Character has a LEN value which is not a compile-time known constant.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 131

~~~~text
    static constexpr LenType unknownLen() { return mlir::ShapedType::kDynamic; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 132

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 133

~~~~text
    /// Character LEN is a runtime value.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 134

~~~~text
    bool hasDynamicLen() { return getLen() == unknownLen(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 135

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 136

~~~~text
    /// Character LEN is a compile-time constant value.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 137

~~~~text
    bool hasConstantLen() { return !hasDynamicLen(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 138

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 139

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 140

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 141

~~~~text
def fir_ClassType : FIR_Type<"Class", "class", [], "BaseBoxType"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 142

~~~~text
  let summary = "Class type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 143

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 144

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 145

~~~~text
    Class type is used to model the Fortran CLASS intrinsic type. A class type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 146

~~~~text
    is equivalent to a fir.box type with a dynamic type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 147

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 148

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 149

~~~~text
  let parameters = (ins "mlir::Type":$eleTy, "bool":$isVolatile);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 150

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 151

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 152

~~~~text
    TypeBuilderWithInferredContext<(ins "mlir::Type":$eleTy, CArg<"bool", "false">:$isVolatile), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 153

~~~~text
      return $_get(eleTy.getContext(), eleTy, isVolatile);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 154

~~~~text
    }]>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 155

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 156

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 157

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 158

~~~~text
    bool isVolatile() const { return getIsVolatile(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 159

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 160

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 161

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 162

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 163

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 164

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 165

~~~~text
def fir_FieldType : FIR_Type<"Field", "field"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 166

~~~~text
  let summary = "A field (in a RecordType) argument's type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 167

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 168

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 169

~~~~text
    The type of a field name. Implementations may defer the layout of a Fortran
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 170

~~~~text
    derived type until runtime. This implies that the runtime must be able to
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 171

~~~~text
    determine the offset of fields within the entity.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 172

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 173

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 174

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 175

~~~~text
def fir_HeapType : FIR_Type<"Heap", "heap"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 176

~~~~text
  let summary = "Reference to an ALLOCATABLE attribute type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 177

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 178

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 179

~~~~text
    The type of a heap pointer. Fortran entities with the ALLOCATABLE attribute
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 180

~~~~text
    may be allocated on the heap at runtime. These pointers are explicitly
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 181

~~~~text
    distinguished to disallow the composition of multiple levels of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 182

~~~~text
    indirection. For example, an ALLOCATABLE POINTER is invalid.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 183

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 184

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 185

~~~~text
  let parameters = (ins "mlir::Type":$eleTy);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 186

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 187

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 188

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 189

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 190

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 191

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 192

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 193

~~~~text
    TypeBuilderWithInferredContext<(ins "mlir::Type":$elementType), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 194

~~~~text
      assert(singleIndirectionLevel(elementType) && "invalid element type");
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 195

~~~~text
      return Base::get(elementType.getContext(), elementType);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 196

~~~~text
    }]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 197

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 198

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 199

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 200

~~~~text
    mlir::Type getElementType() const { return getEleTy(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 201

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 202

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 203

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 204

~~~~text
def fir_IntegerType : FIR_Type<"Integer", "int"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 205

~~~~text
  let summary = "FIR integer type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 206

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 207

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 208

~~~~text
    Model of a Fortran INTEGER intrinsic type, including the KIND type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 209

~~~~text
    parameter.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 210

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 211

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 212

~~~~text
  let parameters = (ins "KindTy":$fKind);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 213

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 214

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 215

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 216

~~~~text
    using KindTy = unsigned;
~~~~
- EN: Creates the alias `KindTy` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `KindTy`。

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
def fir_UnsignedType : FIR_Type<"Unsigned", "unsigned"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 221

~~~~text
  let summary = "FIR unsigned type";
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
    Model of a Fortran UNSIGNED extension intrinsic type, including
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 225

~~~~text
    the KIND type parameter.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 226

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 227

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 228

~~~~text
  let parameters = (ins "KindTy":$fKind);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 229

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 230

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 231

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 232

~~~~text
    using KindTy = unsigned;
~~~~
- EN: Creates the alias `KindTy` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `KindTy`。

### Line 233

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 234

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 235

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 236

~~~~text
def fir_LenType : FIR_Type<"Len", "len"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 237

~~~~text
  let summary = "A LEN parameter (in a RecordType) argument's type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 238

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 239

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 240

~~~~text
    The type of a LEN parameter name. Implementations may defer the layout of a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 241

~~~~text
    Fortran derived type until runtime. This implies that the runtime must be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 242

~~~~text
    able to determine the offset of LEN type parameters related to an entity.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 243

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 244

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 245

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 246

~~~~text
def fir_LogicalType : FIR_Type<"Logical", "logical"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 247

~~~~text
  let summary = "FIR logical type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 248

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 249

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 250

~~~~text
    Model of a Fortran LOGICAL intrinsic type, including the KIND type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 251

~~~~text
    parameter.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 252

~~~~text
  }];
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
  let parameters = (ins "KindTy":$fKind);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 255

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 256

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 257

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 258

~~~~text
    using KindTy = unsigned;
~~~~
- EN: Creates the alias `KindTy` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `KindTy`。

### Line 259

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 260

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 261

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 262

~~~~text
def fir_LLVMPointerType : FIR_Type<"LLVMPointer", "llvm_ptr"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 263

~~~~text
  let summary = "Like LLVM pointer type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 264

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 265

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 266

~~~~text
    A pointer type that does not have any of the constraints and semantics
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 267

~~~~text
    of other FIR pointer types and that translates to llvm pointer types.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 268

~~~~text
    It is meant to implement indirection that cannot be expressed directly
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 269

~~~~text
    in Fortran, but are needed to implement some Fortran features (e.g,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 270

~~~~text
    double indirections).
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 271

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 272

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 273

~~~~text
  let parameters = (ins "mlir::Type":$eleTy);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 274

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 275

~~~~text
  let assemblyFormat = "`<` $eleTy `>`";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 276

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 277

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 278

~~~~text
    TypeBuilderWithInferredContext<(ins "mlir::Type":$elementType), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 279

~~~~text
      return Base::get(elementType.getContext(), elementType);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 280

~~~~text
    }]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 281

~~~~text
  ];
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
    mlir::Type getElementType() const { return getEleTy(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 285

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 286

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 287

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 288

~~~~text
def fir_PointerType : FIR_Type<"Pointer", "ptr"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 289

~~~~text
  let summary = "Reference to a POINTER attribute type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 290

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 291

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 292

~~~~text
    The type of entities with the POINTER attribute.  These pointers are
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 293

~~~~text
    explicitly distinguished to disallow the composition of multiple levels of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 294

~~~~text
    indirection. For example, an ALLOCATABLE POINTER is invalid.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 295

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 296

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 297

~~~~text
  let parameters = (ins "mlir::Type":$eleTy);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 298

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 299

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 300

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 301

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 302

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 303

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 304

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 305

~~~~text
    TypeBuilderWithInferredContext<(ins "mlir::Type":$elementType), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 306

~~~~text
      assert(singleIndirectionLevel(elementType) && "invalid element type");
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 307

~~~~text
      return Base::get(elementType.getContext(), elementType);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 308

~~~~text
    }]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 309

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 310

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 311

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 312

~~~~text
    mlir::Type getElementType() const { return getEleTy(); }
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
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 315

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 316

~~~~text
def fir_RecordType : FIR_Type<"Record", "type"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 317

~~~~text
  let summary = "FIR derived type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 318

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 319

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 320

~~~~text
    Model of Fortran's derived type, TYPE. The name of the TYPE includes any
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 321

~~~~text
    KIND type parameters. The record includes runtime slots for LEN type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 322

~~~~text
    parameters and for data components.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 323

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 324

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 325

~~~~text
  let parameters = (ins StringRefParameter<"name">:$name);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 326

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 327

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 328

~~~~text
  let genStorageClass = 0;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 329

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 330

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 331

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 332

~~~~text
    using TypePair = std::pair<std::string, mlir::Type>;
~~~~
- EN: Creates the alias `TypePair` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `TypePair`。

### Line 333

~~~~text
    using TypeList = llvm::ArrayRef<TypePair>;
~~~~
- EN: Creates the alias `TypeList` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `TypeList`。

### Line 334

~~~~text
    using TypeVector = llvm::SmallVector<TypePair>;
~~~~
- EN: Creates the alias `TypeVector` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `TypeVector`。

### Line 335

~~~~text
    TypeList getTypeList() const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 336

~~~~text
    TypeList getLenParamList() const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 337

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 338

~~~~text
    mlir::Type getType(llvm::StringRef ident);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 339

~~~~text
    // Returns the index of the field \p ident in the type list.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 340

~~~~text
    // Returns maximum unsigned if ident is not a field of this RecordType.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 341

~~~~text
    unsigned getFieldIndex(llvm::StringRef ident);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 342

~~~~text
    mlir::Type getType(unsigned index) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 343

~~~~text
      assert(index < getNumFields());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 344

~~~~text
      return getTypeList()[index].second;
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
    unsigned getNumFields() { return getTypeList().size(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 347

~~~~text
    unsigned getNumLenParams() { return getLenParamList().size(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 348

~~~~text
    bool isDependentType() { return getNumLenParams(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 349

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 350

~~~~text
    bool isFinalized() const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 351

~~~~text
    void finalize(llvm::ArrayRef<TypePair> lenPList,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 352

~~~~text
                  llvm::ArrayRef<TypePair> typeList);
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 353

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 354

~~~~text
    // fir.type is unpacked by default. If the flag is set, the packed fir.type
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 355

~~~~text
    // is generated and the alignment is enforced by explicit padding by i8
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 356

~~~~text
    // array fields.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 357

~~~~text
    bool isPacked() const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 358

~~~~text
    void pack(bool);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 359

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 360

~~~~text
    // The presence of SEQUENCE attribute.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 361

~~~~text
    bool isSequence() const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 362

~~~~text
    void setSequence(bool);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 363

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 364

~~~~text
    detail::RecordTypeStorage const *uniqueKey() const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 365

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 366

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 367

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 368

~~~~text
def fir_ReferenceType : FIR_Type<"Reference", "ref"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 369

~~~~text
  let summary = "Reference to an entity type";
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
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 372

~~~~text
    The type of a reference to an entity in memory.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 373

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 374

~~~~text
    References can be volatile. Any ops taking an operand of a volatile
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 375

~~~~text
    reference must set their memory effects appropriately. Accesses of
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 376

~~~~text
    volatile references are currently modeled as read and write effects
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 377

~~~~text
    to a specific memory resource.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 378

~~~~text
  }];
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
  let parameters = (ins "mlir::Type":$eleTy, "bool":$isVolatile);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 381

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 382

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 383

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 384

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 385

~~~~text
    TypeBuilderWithInferredContext<(ins "mlir::Type":$elementType, CArg<"bool", "false">:$isVolatile), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 386

~~~~text
      return Base::get(elementType.getContext(), elementType, isVolatile);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 387

~~~~text
    }]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 388

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 389

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 390

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 391

~~~~text
    mlir::Type getElementType() const { return getEleTy(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 392

~~~~text
    bool isVolatile() const { return getIsVolatile(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 393

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 394

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 395

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 396

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 397

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 398

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 399

~~~~text
def fir_ShapeType : FIR_Type<"Shape", "shape"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 400

~~~~text
  let summary = "shape of a multidimensional array object";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 401

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 402

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 403

~~~~text
    Type of a vector of runtime values that define the shape of a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 404

~~~~text
    multidimensional array object. The vector is the extents of each array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 405

~~~~text
    dimension. The rank of a ShapeType must be at least 1.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 406

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 407

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 408

~~~~text
  let parameters = (ins "unsigned":$rank);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 409

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

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
def fir_ShapeShiftType : FIR_Type<"ShapeShift", "shapeshift"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 413

~~~~text
  let summary = "shape and origin of a multidimensional array object";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 414

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 415

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 416

~~~~text
    Type of a vector of runtime values that define the shape and the origin of a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 417

~~~~text
    multidimensional array object. The vector is of pairs, origin offset and
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 418

~~~~text
    extent, of each array dimension. The rank of a ShapeShiftType must be at
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 419

~~~~text
    least 1.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 420

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 421

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 422

~~~~text
  let parameters = (ins "unsigned":$rank);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 423

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 424

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 425

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 426

~~~~text
def fir_ShiftType : FIR_Type<"Shift", "shift"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 427

~~~~text
  let summary = "lower bounds of a multidimensional array object";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 428

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 429

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 430

~~~~text
    Type of a vector of runtime values that define the lower bounds of a
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 431

~~~~text
    multidimensional array object. The vector is the lower bounds of each array
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 432

~~~~text
    dimension. The rank of a ShiftType must be at least 1.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 433

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 434

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 435

~~~~text
  let parameters = (ins "unsigned":$rank);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 436

~~~~text
  let hasCustomAssemblyFormat = 1;
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
def fir_SequenceType : FIR_Type<"Sequence", "array"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 440

~~~~text
  let summary = "FIR array type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 441

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 442

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 443

~~~~text
    A sequence type is a multi-dimensional array of values. The sequence type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 444

~~~~text
    may have an unknown number of dimensions or the extent of dimensions may be
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 445

~~~~text
    unknown. A sequence type models a Fortran array entity, giving it a type in
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 446

~~~~text
    FIR. A sequence type is assumed to be stored in a column-major order, which
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 447

~~~~text
    differs from LLVM IR and other dialects of MLIR.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 448

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 449

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 450

~~~~text
  let parameters = (ins
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 451

~~~~text
    ArrayRefParameter<"int64_t", "Sequence shape">:$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 452

~~~~text
    "mlir::Type":$eleTy,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 453

~~~~text
    "mlir::AffineMapAttr":$layoutMap
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 454

~~~~text
  );
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 455

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 456

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 457

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 458

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 459

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 460

~~~~text
    TypeBuilderWithInferredContext<(ins
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 461

~~~~text
        "llvm::ArrayRef<int64_t>":$shape,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 462

~~~~text
        "mlir::Type":$eleTy), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 463

~~~~text
      return get(eleTy.getContext(), shape, eleTy, {});
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 464

~~~~text
    }]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 465

~~~~text
    TypeBuilderWithInferredContext<(ins
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 466

~~~~text
        "mlir::Type":$eleTy,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 467

~~~~text
        "size_t":$dimensions), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 468

~~~~text
      llvm::SmallVector<int64_t> shape(dimensions, getUnknownExtent());
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 469

~~~~text
      return get(eleTy.getContext(), shape, eleTy, {});
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 470

~~~~text
    }]>
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 471

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 472

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 473

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 474

~~~~text
    using Extent = int64_t;
~~~~
- EN: Creates the alias `Extent` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `Extent`。

### Line 475

~~~~text
    using Shape = llvm::SmallVector<Extent>;
~~~~
- EN: Creates the alias `Shape` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `Shape`。

### Line 476

~~~~text
    using ShapeRef = llvm::ArrayRef<int64_t>;
~~~~
- EN: Creates the alias `ShapeRef` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `ShapeRef`。

### Line 477

~~~~text
    unsigned getConstantRows() const;
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 478

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 479

~~~~text
    // The number of dimensions of the sequence
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 480

~~~~text
    unsigned getDimension() const { return getShape().size(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 481

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 482

~~~~text
    // Is the shape of the sequence dynamic?
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 483

~~~~text
    bool hasDynamicExtents() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 484

~~~~text
      for(const auto d : getShape())
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 485

~~~~text
        if (d == getUnknownExtent())
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 486

~~~~text
          return true;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 487

~~~~text
      return false;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 488

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 489

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 490

~~~~text
    // Does the sequence have unknown shape? (`array<* x T>`)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 491

~~~~text
    bool hasUnknownShape() const { return getShape().empty(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 492

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 493

~~~~text
    // The value `kDynamic` represents an unknown extent for a dimension
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 494

~~~~text
    static constexpr Extent getUnknownExtent() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 495

~~~~text
      return mlir::ShapedType::kDynamic;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 496

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 497

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 498

~~~~text
    std::uint64_t getConstantArraySize() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 499

~~~~text
      assert(!hasDynamicExtents() && "array type must have constant shape");
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 500

~~~~text
      std::uint64_t size = 1;
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 501

~~~~text
      for (Extent extent : getShape())
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 502

~~~~text
        size = size * static_cast<std::uint64_t>(extent);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 503

~~~~text
      return size;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 504

~~~~text
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 505

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 506

~~~~text
    mlir::Type getElementType() const { return getEleTy(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 507

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 508

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 509

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 510

~~~~text
def fir_SliceType : FIR_Type<"Slice", "slice"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 511

~~~~text
  let summary = "FIR slice";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 512

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 513

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 514

~~~~text
    Type of a vector that represents an array slice operation on an array.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 515

~~~~text
    Fortran slices are triples of lower bound, upper bound, and stride. The rank
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 516

~~~~text
    of a SliceType must be at least 1.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 517

~~~~text
  }];
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
  let parameters = (ins "unsigned":$rank);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 520

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 521

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 522

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 523

~~~~text
def fir_TypeDescType : FIR_Type<"TypeDesc", "tdesc"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 524

~~~~text
  let summary = "FIR Type descriptor type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 525

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 526

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 527

~~~~text
    The type of a type descriptor object. The runtime may generate type
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 528

~~~~text
    descriptor objects to determine the type of an entity at runtime, etc.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 529

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 530

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 531

~~~~text
  let parameters = (ins "mlir::Type":$ofTy);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 532

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 533

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 534

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 535

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 536

~~~~text
  let skipDefaultBuilders = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 537

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 538

~~~~text
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 539

~~~~text
    TypeBuilderWithInferredContext<(ins "mlir::Type":$elementType), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 540

~~~~text
      return Base::get(elementType.getContext(), elementType);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 541

~~~~text
    }]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 542

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 543

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 544

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 545

~~~~text
def fir_VectorType : FIR_Type<"Vector", "vector"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 546

~~~~text
  let summary = "FIR vector type";
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
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 549

~~~~text
    Replacement for the builtin vector type.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 550

~~~~text
    The FIR vector type is always rank one. Its size is always a constant.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 551

~~~~text
    A vector's element type must be real or integer.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 552

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 553

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 554

~~~~text
  let parameters = (ins "uint64_t":$len, "mlir::Type":$eleTy);
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 555

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 556

~~~~text
  let genVerifyDecl = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 557

~~~~text
  let hasCustomAssemblyFormat = 1;
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 558

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 559

~~~~text
  let extraClassDeclaration = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 560

~~~~text
    static bool isValidElementType(mlir::Type t);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 561

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 562

~~~~text
    mlir::Type getElementType() const { return getEleTy(); }
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
  let skipDefaultBuilders = 1;
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
  let builders = [
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 568

~~~~text
    TypeBuilderWithInferredContext<(ins
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 569

~~~~text
        "uint64_t":$len,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 570

~~~~text
        "mlir::Type":$eleTy), [{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 571

~~~~text
      return Base::get(eleTy.getContext(), len, eleTy);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 572

~~~~text
    }]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 573

~~~~text
  ];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 574

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 575

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 576

~~~~text
def fir_VoidType : FIR_Type<"Void", "void"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 577

~~~~text
  let genStorageClass = 0;
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
def fir_DummyScopeType : FIR_Type<"DummyScope", "dscope"> {
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 581

~~~~text
  let summary = "Dummy scope type";
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 582

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 583

~~~~text
  let description = [{
~~~~
- EN: Assigns or overrides a TableGen field value.
- CN: 为 TableGen 字段赋值或覆盖其值。

### Line 584

~~~~text
    `fir.dscope` is a type returned by fir.dummy_scope operation.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 585

~~~~text
    It defines a unique identifier for a runtime instance of a subroutine
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 586

~~~~text
    that is used by the [hl]fir.declare operations representing
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 587

~~~~text
    the dummy arguments' declarations.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 588

~~~~text
  }];
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 589

~~~~text
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 590

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 591

~~~~text
// Whether a type is a BaseBoxType
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 592

~~~~text
def IsBaseBoxTypePred
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 593

~~~~text
        : CPred<"mlir::isa<::fir::BaseBoxType>($_self)">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 594

~~~~text
def fir_BaseBoxType : Type<IsBaseBoxTypePred, "fir.box or fir.class type">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 595

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 596

~~~~text
// Generalized FIR and standard dialect types representing intrinsic types
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 597

~~~~text
def AnyIntegerLike : TypeConstraint<Or<[SignlessIntegerOrIndexLike.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 598

~~~~text
    AnySignedInteger.predicate, AnyUnsignedInteger.predicate,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 599

~~~~text
    fir_IntegerType.predicate, fir_UnsignedType.predicate]>, "any integer">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 600

~~~~text
def AnyLogicalLike : TypeConstraint<Or<[BoolLike.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 601

~~~~text
    fir_LogicalType.predicate]>, "any logical">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 602

~~~~text
def AnyRealLike : TypeConstraint<FloatLike.predicate, "any real">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 603

~~~~text
def AnyIntegerType : Type<AnyIntegerLike.predicate, "any integer">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 604

~~~~text
def AnyLogicalType : Type<AnyLogicalLike.predicate, "any logical">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 605

~~~~text
def AnyLogicalOrIntegerLike : TypeConstraint<Or<[
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 606

~~~~text
    fir_LogicalType.predicate,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 607

~~~~text
    SignlessIntegerLike.predicate]>, "fir.logical or signless integer">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 608

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 609

~~~~text
def AnyFirComplexLike :  TypeConstraint<CPred<"::fir::isa_complex($_self)">,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 610

~~~~text
  "any floating point complex type">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 611

~~~~text
def AnyFirComplex : Type<AnyFirComplexLike.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 612

~~~~text
  "any floating point complex type">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 613

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 614

~~~~text
// Composable types
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 615

~~~~text
def AnyCompositeLike : TypeConstraint<Or<[fir_RecordType.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 616

~~~~text
    fir_SequenceType.predicate, AnyFirComplexLike.predicate,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 617

~~~~text
    fir_VectorType.predicate, IsTupleTypePred, fir_CharacterType.predicate]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 618

~~~~text
    "any composite">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 619

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 620

~~~~text
// Reference types
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 621

~~~~text
def AnyReferenceLike
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 622

~~~~text
    : Type<Or<[fir_ReferenceType.predicate, fir_HeapType.predicate,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 623

~~~~text
               fir_PointerType.predicate, fir_LLVMPointerType.predicate]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 624

~~~~text
           "any reference">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 625

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 626

~~~~text
def FuncType : TypeConstraint<FunctionType.predicate, "function type">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 627

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 628

~~~~text
def AnyCodeOrDataRefLike : TypeConstraint<Or<[AnyReferenceLike.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 629

~~~~text
    FunctionType.predicate]>, "any code or data reference">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 630

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 631

~~~~text
def RefOrLLVMPtr : TypeConstraint<Or<[fir_ReferenceType.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 632

~~~~text
    fir_LLVMPointerType.predicate]>, "fir.ref or fir.llvm_ptr">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 633

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 634

~~~~text
def AnyBoxLike : TypeConstraint<Or<[fir_BoxType.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 635

~~~~text
    fir_BoxCharType.predicate, fir_BoxProcType.predicate,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 636

~~~~text
    fir_ClassType.predicate]>, "any box">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 637

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 638

~~~~text
def BoxOrClassLike
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 639

~~~~text
    : TypeConstraint<Or<[fir_BoxType.predicate, fir_ClassType.predicate]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 640

~~~~text
                     "box or class">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 641

~~~~text
def BoxOrClassType : Type<BoxOrClassLike.predicate, "box or class">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 642

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 643

~~~~text
def AnyRefOrBoxLike : TypeConstraint<Or<[AnyReferenceLike.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 644

~~~~text
    AnyBoxLike.predicate, FunctionType.predicate]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 645

~~~~text
    "any reference or box like">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 646

~~~~text
def AnyRefOrBox : TypeConstraint<Or<[fir_ReferenceType.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 647

~~~~text
    fir_HeapType.predicate, fir_PointerType.predicate,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 648

~~~~text
    IsBaseBoxTypePred]>, "any reference or box">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 649

~~~~text
def AnyRefOrBoxType : Type<AnyRefOrBox.predicate, "any legal ref or box type">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 650

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 651

~~~~text
def AnyShapeLike : TypeConstraint<Or<[fir_ShapeType.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 652

~~~~text
    fir_ShapeShiftType.predicate]>, "any legal shape type">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 653

~~~~text
def AnyShapeType : Type<AnyShapeLike.predicate, "any legal shape type">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 654

~~~~text
def AnyShapeOrShiftLike : TypeConstraint<Or<[fir_ShapeType.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 655

~~~~text
    fir_ShapeShiftType.predicate, fir_ShiftType.predicate]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 656

~~~~text
    "any legal shape or shift type">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 657

~~~~text
def AnyShapeOrShiftType : Type<AnyShapeOrShiftLike.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 658

~~~~text
    "any legal shape or shift type">;
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
def AnyEmboxLike : TypeConstraint<Or<[AnySignlessInteger.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 661

~~~~text
    Index.predicate, fir_IntegerType.predicate]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 662

~~~~text
    "any legal embox argument type">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 663

~~~~text
def AnyEmboxArg : Type<AnyEmboxLike.predicate, "embox argument type">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 664

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 665

~~~~text
def AnyComponentLike : TypeConstraint<Or<[AnySignlessInteger.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 666

~~~~text
    Index.predicate, fir_IntegerType.predicate, fir_FieldType.predicate]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 667

~~~~text
    "any coordinate index">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 668

~~~~text
def AnyComponentType : Type<AnyComponentLike.predicate, "coordinate type">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 669

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 670

~~~~text
def AnyCoordinateLike : TypeConstraint<Or<[AnySignlessInteger.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 671

~~~~text
    Index.predicate, fir_IntegerType.predicate, fir_FieldType.predicate,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 672

~~~~text
    fir_LenType.predicate]>, "any coordinate index">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 673

~~~~text
def AnyCoordinateType : Type<AnyCoordinateLike.predicate, "coordinate type">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 674

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 675

~~~~text
// The legal types of global symbols
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 676

~~~~text
def AnyAddressableLike : TypeConstraint<Or<[fir_ReferenceType.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 677

~~~~text
    FunctionType.predicate]>, "any addressable">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 678

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 679

~~~~text
def ArrayOrBoxOrRecord : TypeConstraint<Or<[fir_SequenceType.predicate,
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 680

~~~~text
    IsBaseBoxTypePred, fir_RecordType.predicate]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 681

~~~~text
    "fir.box, fir.array or fir.type">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 682

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 683

~~~~text
// Returns true iff the type is an array box or a reference to such type.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 684

~~~~text
def IsArrayBoxPred : CPred<"::fir::getBoxRank($_self) != 0">;
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 685

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 686

~~~~text
// Any boxed array type (not a reference to a boxed array type).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 687

~~~~text
def AnyBoxedArray
~~~~
- EN: Defines a TableGen record that contributes generated data or documentation.
- CN: 定义一个 TableGen 记录，用于生成数据或文档。

### Line 688

~~~~text
    : TypeConstraint<And<[BoxOrClassType.predicate, IsArrayBoxPred]>,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 689

~~~~text
                     "any boxed array">;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 690

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 691

~~~~text
#endif // FIR_DIALECT_FIR_TYPES
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
  - `mlir/IR/AttrTypeBase.td` — referenced directly from this file / 该文件直接引用
  - `flang/Optimizer/Dialect/FIRDialect.td` — referenced directly from this file / 该文件直接引用
- **External or standard dependencies / 外部或标准依赖**: none detected directly / 未直接检测到
