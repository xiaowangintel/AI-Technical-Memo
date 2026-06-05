# ComplexOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Complex/IR/ComplexOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the Complex dialect, focused on dialect IR declarations such as operations, attributes, types, enums, and registration hooks and `ComplexOps`.
  - **CN**: 为 Complex 方言定义聚焦 `ComplexOps` 的声明式 TableGen 规格，覆盖方言 IR 声明，例如操作、属性、类型、枚举与注册钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```tablegen
//===- ComplexOps.td - Complex op definitions ----------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 9-18
```tablegen
#ifndef COMPLEX_OPS
#define COMPLEX_OPS

include "mlir/Dialect/Arith/IR/ArithBase.td"
include "mlir/Dialect/Arith/IR/ArithOpsInterfaces.td"
include "mlir/Dialect/Complex/IR/ComplexBase.td"
include "mlir/IR/OpAsmInterface.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/Interfaces/SideEffectInterfaces.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 19-33
```tablegen
class Complex_Op<string mnemonic, list<Trait> traits = []>
    : Op<Complex_Dialect, mnemonic, traits>;

// Base class for standard arithmetic operations on complex numbers with a
// floating-point element type. These operations take two operands and return
// one result, all of which must be complex numbers of the same type.
class ComplexArithmeticOp<string mnemonic, list<Trait> traits = []> :
    Complex_Op<mnemonic, traits # [Pure, SameOperandsAndResultType,
    Elementwise, DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
  let arguments = (ins Complex<AnyFloat>:$lhs, Complex<AnyFloat>:$rhs, DefaultValuedAttr<
        Arith_FastMathAttr, "::mlir::arith::FastMathFlags::none">:$fastmath);
  let results = (outs Complex<AnyFloat>:$result);
  let assemblyFormat = "$lhs `,` $rhs (`fastmath` `` $fastmath^)? attr-dict `:` type($result)";
}

```
- **EN**: Introduces declarations for `Complex_Op`, `for`, `ComplexArithmeticOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Complex_Op`, `for`, `ComplexArithmeticOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 34-43
```tablegen
// Base class for standard unary operations on complex numbers with a
// floating-point element type. These operations take one operand and return
// one result; the operand must be a complex number.
class ComplexUnaryOp<string mnemonic, list<Trait> traits = []> :
    Complex_Op<mnemonic, traits # [Pure, Elementwise, DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
  let arguments = (ins Complex<AnyFloat>:$complex, DefaultValuedAttr<
        Arith_FastMathAttr, "::mlir::arith::FastMathFlags::none">:$fastmath);
  let assemblyFormat = "$complex (`fastmath` `` $fastmath^)? attr-dict `:` type($complex)";
}

```
- **EN**: Introduces declarations for `for`, `ComplexUnaryOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `for`, `ComplexUnaryOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 44-55
```tablegen
//===----------------------------------------------------------------------===//
// AbsOp
//===----------------------------------------------------------------------===//

def AbsOp : ComplexUnaryOp<"abs",
    [TypesMatchWith<"complex element type matches result type",
                    "complex", "result",
                    "::llvm::cast<ComplexType>($_self).getElementType()">]> {
  let summary = "computes absolute value of a complex number";
  let description = [{
    The `abs` op takes a single complex number and computes its absolute value.

```
- **EN**: Introduces declarations for `AbsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AbsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 56-64
```tablegen
    Example:

    ```mlir
    %a = complex.abs %b : complex<f32>
    ```
  }];
  let results = (outs AnyFloat:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 65-73
```tablegen
//===----------------------------------------------------------------------===//
// AddOp
//===----------------------------------------------------------------------===//

def AddOp : ComplexArithmeticOp<"add"> {
  let summary = "complex addition";
  let description = [{
    The `add` operation takes two complex numbers and returns their sum.

```
- **EN**: Introduces declarations for `AddOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AddOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 74-83
```tablegen
    Example:

    ```mlir
    %a = complex.add %b, %c : complex<f32>
    ```
  }];

  let hasFolder = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 84-93
```tablegen
//===----------------------------------------------------------------------===//
// Atan2
//===----------------------------------------------------------------------===//

def Atan2Op : ComplexArithmeticOp<"atan2"> {
  let summary = "complex 2-argument arctangent";
  let description = [{
    For complex numbers it is expressed using complex logarithm
    atan2(y, x) = -i * log((x + i * y) / sqrt(x**2 + y**2))

```
- **EN**: Introduces declarations for `Atan2Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Atan2Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 94-101
```tablegen
    Example:

    ```mlir
    %a = complex.atan2 %b, %c : complex<f32>
    ```
  }];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 102-111
```tablegen

//===----------------------------------------------------------------------===//
// Bitcast
//===----------------------------------------------------------------------===//

def BitcastOp : Complex_Op<"bitcast", [Pure]> {

  let summary = "computes bitcast between complex and equal arith types";
  let description = [{

```
- **EN**: Introduces declarations for `BitcastOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `BitcastOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 112-121
```tablegen
    Example:

    ```mlir
         %a = complex.bitcast %b : complex<f32> -> i64
    ```
  }];
  let assemblyFormat = "$operand attr-dict `:` type($operand) `to` type($result)";
  let arguments = (ins AnyType:$operand);
  let results = (outs AnyType:$result);

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 122-129
```tablegen
  let hasCanonicalizer = 1;
  let hasFolder = 1;
  let hasVerifier = 1;
}

//===----------------------------------------------------------------------===//
// ConstantOp
//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 130-139
```tablegen

def ConstantOp : Complex_Op<"constant", [
    ConstantLike, Pure,
    DeclareOpInterfaceMethods<OpAsmOpInterface, ["getAsmResultNames"]>
  ]> {
  let summary = "complex number constant operation";
  let description = [{
    The `complex.constant` operation creates a constant complex number from an
    attribute containing the real and imaginary parts.

```
- **EN**: Introduces declarations for `ConstantOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ConstantOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 140-149
```tablegen
    Example:

    ```mlir
    %a = complex.constant [0.1, -1.0] : complex<f64>
    ```
  }];

  let arguments = (ins ArrayAttr:$value);
  let results = (outs AnyComplex:$complex);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 150-160
```tablegen
  let assemblyFormat = "$value attr-dict `:` type($complex)";
  let hasFolder = 1;
  let hasVerifier = 1;

  let extraClassDeclaration = [{
    /// Returns true if a constant operation can be built with the given value
    /// and result type.
    static bool isBuildableWith(Attribute value, Type type);
  }];
}

```
- **EN**: Implements logic around `type`, `isBuildableWith`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type`, `isBuildableWith` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 161-170
```tablegen
//===----------------------------------------------------------------------===//
// CosOp
//===----------------------------------------------------------------------===//

def CosOp : ComplexUnaryOp<"cos", [SameOperandsAndResultType]> {
  let summary = "computes cosine of a complex number";
  let description = [{
    The `cos` op takes a single complex number and computes the cosine of
    it, i.e. `cos(x)`, where `x` is the input value.

```
- **EN**: Introduces declarations for `CosOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CosOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 171-180
```tablegen
    Example:

    ```mlir
    %a = complex.cos %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 181-194
```tablegen
//===----------------------------------------------------------------------===//
// CreateOp
//===----------------------------------------------------------------------===//

def CreateOp : Complex_Op<"create",
    [Pure,
     AllTypesMatch<["real", "imaginary"]>,
     TypesMatchWith<"complex element type matches real operand type",
                    "complex", "real",
                    "::llvm::cast<ComplexType>($_self).getElementType()">,
     TypesMatchWith<"complex element type matches imaginary operand type",
                    "complex", "imaginary",
                    "::llvm::cast<ComplexType>($_self).getElementType()">]> {

```
- **EN**: Introduces declarations for `CreateOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CreateOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 195-206
```tablegen
  let summary = "complex number creation operation";
  let description = [{
    The `complex.create` operation creates a complex number from two
    floating-point operands, the real and the imaginary part.

    Example:

    ```mlir
    %a = complex.create %b, %c : complex<f32>
    ```
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 207-214
```tablegen
  let arguments = (ins AnyFloat:$real, AnyFloat:$imaginary);
  let results = (outs Complex<AnyFloat>:$complex);

  let assemblyFormat = "$real `,` $imaginary attr-dict `:` type($complex)";
  let hasFolder = 1;
}

//===----------------------------------------------------------------------===//
```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 215-223
```tablegen
// DivOp
//===----------------------------------------------------------------------===//

def DivOp : ComplexArithmeticOp<"div"> {
  let summary = "complex division";
  let description = [{
    The `div` operation takes two complex numbers and returns result of their
    division:

```
- **EN**: Introduces declarations for `DivOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DivOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 224-231
```tablegen
    ```mlir
    %a = complex.div %b, %c : complex<f32>
    ```
  }];

  let hasFolder = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 232-241
```tablegen
//===----------------------------------------------------------------------===//
// EqualOp
//===----------------------------------------------------------------------===//

def EqualOp : Complex_Op<"eq",
    [Pure, AllTypesMatch<["lhs", "rhs"]>, Elementwise]> {
  let summary = "computes whether two complex values are equal";
  let description = [{
    The `eq` op takes two complex numbers and returns whether they are equal.

```
- **EN**: Introduces declarations for `EqualOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `EqualOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 242-251
```tablegen
    Example:

    ```mlir
    %a = complex.eq %b, %c : complex<f32>
    ```
  }];

  let arguments = (ins Complex<AnyFloat>:$lhs, Complex<AnyFloat>:$rhs);
  let results = (outs I1:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 252-259
```tablegen
  let assemblyFormat = "$lhs `,` $rhs  attr-dict `:` type($lhs)";
}

//===----------------------------------------------------------------------===//
// ExpOp
//===----------------------------------------------------------------------===//

def ExpOp : ComplexUnaryOp<"exp", [SameOperandsAndResultType]> {
```
- **EN**: Introduces declarations for `ExpOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ExpOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 260-267
```tablegen
  let summary = "computes exponential of a complex number";
  let description = [{
    The `exp` op takes a single complex number and computes the exponential of
    it, i.e. `exp(x)` or `e^(x)`, where `x` is the input value.
    `e` denotes Euler's number and is approximately equal to 2.718281.

    Example:

```
- **EN**: Implements logic around `exp`.
- **CN**: 围绕 `exp` 实现具体逻辑。

### Lines 268-277
```tablegen
    ```mlir
    %a = complex.exp %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);

  let hasFolder = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 278-286
```tablegen
//===----------------------------------------------------------------------===//
// Expm1Op
//===----------------------------------------------------------------------===//

def Expm1Op : ComplexUnaryOp<"expm1", [SameOperandsAndResultType]> {
  let summary = "computes exponential of a complex number minus 1";
  let description = [{
    complex.expm1(x) := complex.exp(x) - 1

```
- **EN**: Introduces declarations for `Expm1Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Expm1Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 287-296
```tablegen
    Example:

    ```mlir
    %a = complex.expm1 %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 297-308
```tablegen
//===----------------------------------------------------------------------===//
// ImOp
//===----------------------------------------------------------------------===//

def ImOp : ComplexUnaryOp<"im",
    [TypesMatchWith<"complex element type matches result type",
                    "complex", "imaginary",
                    "::llvm::cast<ComplexType>($_self).getElementType()">]> {
  let summary = "extracts the imaginary part of a complex number";
  let description = [{
    The `im` op takes a single complex number and extracts the imaginary part.

```
- **EN**: Introduces declarations for `ImOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ImOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 309-320
```tablegen
    Example:

    ```mlir
    %a = complex.im %b : complex<f32>
    ```
  }];

  let results = (outs AnyFloat:$imaginary);
  let hasFolder = 1;
  let hasCanonicalizer = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 321-331
```tablegen
//===----------------------------------------------------------------------===//
// LogOp
//===----------------------------------------------------------------------===//

def LogOp : ComplexUnaryOp<"log", [SameOperandsAndResultType]> {
  let summary = "computes natural logarithm of a complex number";
  let description = [{
    The `log` op takes a single complex number and computes the natural
    logarithm of it, i.e. `log(x)` or `log_e(x)`, where `x` is the input value.
    `e` denotes Euler's number and is approximately equal to 2.718281.

```
- **EN**: Introduces declarations for `LogOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LogOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 332-340
```tablegen
    Example:

    ```mlir
    %a = complex.log %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 341-348
```tablegen
  let hasFolder = 1;
}

//===----------------------------------------------------------------------===//
// Log1pOp
//===----------------------------------------------------------------------===//

def Log1pOp : ComplexUnaryOp<"log1p", [SameOperandsAndResultType]> {
```
- **EN**: Introduces declarations for `Log1pOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Log1pOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 349-357
```tablegen
  let summary = "computes natural logarithm of a complex number";
  let description = [{
    The `log` op takes a single complex number and computes the natural
    logarithm of one plus the given value, i.e. `log(1 + x)` or `log_e(1 + x)`,
    where `x` is the input value. `e` denotes Euler's number and is
    approximately equal to 2.718281.

    Example:

```
- **EN**: Implements logic around `log`.
- **CN**: 围绕 `log` 实现具体逻辑。

### Lines 358-365
```tablegen
    ```mlir
    %a = complex.log1p %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 366-374
```tablegen
//===----------------------------------------------------------------------===//
// MulOp
//===----------------------------------------------------------------------===//

def MulOp : ComplexArithmeticOp<"mul"> {
  let summary = "complex multiplication";
  let description = [{
    The `mul` operation takes two complex numbers and returns their product:

```
- **EN**: Introduces declarations for `MulOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MulOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 375-382
```tablegen
    ```mlir
    %a = complex.mul %b, %c : complex<f32>
    ```
  }];

  let hasFolder = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 383-391
```tablegen
//===----------------------------------------------------------------------===//
// NegOp
//===----------------------------------------------------------------------===//

def NegOp : ComplexUnaryOp<"neg", [SameOperandsAndResultType]> {
  let summary = "Negation operator";
  let description = [{
    The `neg` op takes a single complex number `complex` and returns `-complex`.

```
- **EN**: Introduces declarations for `NegOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NegOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 392-400
```tablegen
    Example:

    ```mlir
    %a = complex.neg %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 401-408
```tablegen
  let hasFolder = 1;
}

//===----------------------------------------------------------------------===//
// NotEqualOp
//===----------------------------------------------------------------------===//

def NotEqualOp : Complex_Op<"neq",
```
- **EN**: Introduces declarations for `NotEqualOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NotEqualOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 409-416
```tablegen
    [Pure, AllTypesMatch<["lhs", "rhs"]>, Elementwise]> {
  let summary = "computes whether two complex values are not equal";
  let description = [{
    The `neq` op takes two complex numbers and returns whether they are not
    equal.

    Example:

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 417-424
```tablegen
    ```mlir
    %a = complex.neq %b, %c : complex<f32>
    ```
  }];

  let arguments = (ins Complex<AnyFloat>:$lhs, Complex<AnyFloat>:$rhs);
  let results = (outs I1:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 425-432
```tablegen
  let assemblyFormat = "$lhs `,` $rhs  attr-dict `:` type($lhs)";
}

//===----------------------------------------------------------------------===//
// PowOp
//===----------------------------------------------------------------------===//

def PowOp : ComplexArithmeticOp<"pow"> {
```
- **EN**: Introduces declarations for `PowOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `PowOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 433-445
```tablegen
  let summary = "complex power function";
  let description = [{
    The `pow` operation takes a complex number raises it to the given complex
    exponent.

    Example:

    ```mlir
    %a = complex.pow %b, %c : complex<f32>
    ```
  }];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 446-459
```tablegen
//===----------------------------------------------------------------------===//
// PowiOp
//===----------------------------------------------------------------------===//

def PowiOp : Complex_Op<"powi",
    [Pure, Elementwise, SameOperandsAndResultShape,
     AllTypesMatch<["lhs", "result"]>,
     DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
  let summary = "complex number raised to signed integer power";
  let description = [{
    The `powi` operation takes a `base` operand of complex type and a `power`
    operand of signed integer type and returns one result of the same type
    as `base`. The result is `base` raised to the power of `power`.

```
- **EN**: Introduces declarations for `PowiOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `PowiOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 460-471
```tablegen
    Example:

    ```mlir
    %a = complex.powi %b, %c : complex<f32>, i32
    ```
  }];

  let arguments = (ins Complex<AnyFloat>:$lhs,
                       AnySignlessInteger:$rhs,
                       OptionalAttr<Arith_FastMathAttr>:$fastmath);
  let results = (outs Complex<AnyFloat>:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 472-479
```tablegen
  let assemblyFormat =
      "$lhs `,` $rhs (`fastmath` `` $fastmath^)? attr-dict `:` type($result) `,` type($rhs)";
}

//===----------------------------------------------------------------------===//
// ReOp
//===----------------------------------------------------------------------===//

```
- **EN**: Declares APIs or declarative rules around `rhs`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `rhs` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 480-487
```tablegen
def ReOp : ComplexUnaryOp<"re",
    [TypesMatchWith<"complex element type matches result type",
                    "complex", "real",
                    "::llvm::cast<ComplexType>($_self).getElementType()">]> {
  let summary = "extracts the real part of a complex number";
  let description = [{
    The `re` op takes a single complex number and extracts the real part.

```
- **EN**: Introduces declarations for `ReOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ReOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 488-499
```tablegen
    Example:

    ```mlir
    %a = complex.re %b : complex<f32>
    ```
  }];

  let results = (outs AnyFloat:$real);
  let hasFolder = 1;
  let hasCanonicalizer = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 500-508
```tablegen
//===----------------------------------------------------------------------===//
// RsqrtOp
//===----------------------------------------------------------------------===//

def RsqrtOp : ComplexUnaryOp<"rsqrt", [SameOperandsAndResultType]> {
  let summary = "complex reciprocal of square root";
  let description = [{
    The `rsqrt` operation computes reciprocal of square root.

```
- **EN**: Introduces declarations for `RsqrtOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `RsqrtOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 509-518
```tablegen
    Example:

    ```mlir
    %a = complex.rsqrt %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 519-528
```tablegen
//===----------------------------------------------------------------------===//
// SignOp
//===----------------------------------------------------------------------===//

def SignOp : ComplexUnaryOp<"sign", [SameOperandsAndResultType]> {
  let summary = "computes sign of a complex number";
  let description = [{
    The `sign` op takes a single complex number and computes the sign of
    it, i.e. `y = sign(x) = x / |x|` if `x != 0`, otherwise `y = 0`.

```
- **EN**: Introduces declarations for `SignOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SignOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 529-538
```tablegen
    Example:

    ```mlir
    %a = complex.sign %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 539-548
```tablegen
//===----------------------------------------------------------------------===//
// SinOp
//===----------------------------------------------------------------------===//

def SinOp : ComplexUnaryOp<"sin", [SameOperandsAndResultType]> {
  let summary = "computes sine of a complex number";
  let description = [{
    The `sin` op takes a single complex number and computes the sine of
    it, i.e. `sin(x)`, where `x` is the input value.

```
- **EN**: Introduces declarations for `SinOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SinOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 549-558
```tablegen
    Example:

    ```mlir
    %a = complex.sin %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 559-567
```tablegen
//===----------------------------------------------------------------------===//
// SqrtOp
//===----------------------------------------------------------------------===//

def SqrtOp : ComplexUnaryOp<"sqrt", [SameOperandsAndResultType]> {
  let summary = "complex square root";
  let description = [{
    The `sqrt` operation takes a complex number and returns its square root.

```
- **EN**: Introduces declarations for `SqrtOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SqrtOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 568-577
```tablegen
    Example:

    ```mlir
    %a = complex.sqrt %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 578-586
```tablegen
//===----------------------------------------------------------------------===//
// SubOp
//===----------------------------------------------------------------------===//

def SubOp : ComplexArithmeticOp<"sub"> {
  let summary = "complex subtraction";
  let description = [{
    The `sub` operation takes two complex numbers and returns their difference.

```
- **EN**: Introduces declarations for `SubOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SubOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 587-596
```tablegen
    Example:

    ```mlir
    %a = complex.sub %b, %c : complex<f32>
    ```
  }];

  let hasFolder = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 597-606
```tablegen
//===----------------------------------------------------------------------===//
// TanhOp
//===----------------------------------------------------------------------===//

def TanhOp : ComplexUnaryOp<"tanh", [SameOperandsAndResultType]> {
  let summary = "complex hyperbolic tangent";
  let description = [{
    The `tanh` operation takes a complex number and returns its hyperbolic
    tangent.

```
- **EN**: Introduces declarations for `TanhOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `TanhOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 607-616
```tablegen
    Example:

    ```mlir
    %a = complex.tanh %b : complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 617-626
```tablegen
//===----------------------------------------------------------------------===//
// TanOp
//===----------------------------------------------------------------------===//

def TanOp : ComplexUnaryOp<"tan", [SameOperandsAndResultType]> {
  let summary = "computes tangent of a complex number";
  let description = [{
    The `tan` op takes a single complex number and computes the tangent of
    it, i.e. `tan(x)`, where `x` is the input value.

```
- **EN**: Introduces declarations for `TanOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `TanOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 627-635
```tablegen
    Example:

    ```mlir
    %a = complex.tan %b : complex<f32>
    ```
  }];
  let results = (outs Complex<AnyFloat>:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 636-645
```tablegen
//===----------------------------------------------------------------------===//
// Conj
//===----------------------------------------------------------------------===//

def ConjOp : ComplexUnaryOp<"conj", [SameOperandsAndResultType]> {
  let summary = "Calculate the complex conjugate";
  let description = [{
    The `conj` op takes a single complex number and computes the
    complex conjugate.

```
- **EN**: Introduces declarations for `ConjOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ConjOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 646-656
```tablegen
    Example:

    ```mlir
    %a = complex.conj %b: complex<f32>
    ```
  }];

  let results = (outs Complex<AnyFloat>:$result);
  let hasFolder = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 657-668
```tablegen
//===----------------------------------------------------------------------===//
// AngleOp
//===----------------------------------------------------------------------===//

def AngleOp : ComplexUnaryOp<"angle",
                           [TypesMatchWith<"complex element type matches result type",
                                           "complex", "result",
                                           "::llvm::cast<ComplexType>($_self).getElementType()">]> {
  let summary = "computes argument value of a complex number";
  let description = [{
    The `angle` op takes a single complex number and computes its argument value with a branch cut along the negative real axis.

```
- **EN**: Introduces declarations for `AngleOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AngleOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 669-677
```tablegen
    Example:

    ```mlir
         %a = complex.angle %b : complex<f32>
    ```
  }];
  let results = (outs AnyFloat:$result);
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 678-678
```tablegen
#endif // COMPLEX_OPS
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

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Dialect/Arith/IR/ArithBase.td`, `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td`, `mlir/Dialect/Complex/IR/ComplexBase.td`, `mlir/IR/OpAsmInterface.td`, `mlir/Interfaces/InferTypeOpInterface.td`, `mlir/Interfaces/SideEffectInterfaces.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (3), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
