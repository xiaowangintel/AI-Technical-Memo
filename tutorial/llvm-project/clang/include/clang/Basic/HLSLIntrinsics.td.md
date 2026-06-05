# HLSLIntrinsics.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/HLSLIntrinsics.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: This file defines HLSL intrinsic functions in tablegen. The HLSLEmitter backend processes these definitions and generates two .inc files: hlsl_alias_intrinsics_gen.inc: builtin alias declarations using _HLSL_BUILTIN_ALIAS, included by hlsl_alias_intrinsics.h. hlsl_inline_intrinsics_gen.inc: inline function definitions (detail helper calls and literal bodies), included by hlsl_intrinsics.h.
- **Purpose (CN)**: 声明与 `HLSLIntrinsics` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1879

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines HLSL intrinsic functions in tablegen. The HLSLEmitter
// backend processes these definitions and generates two .inc files:
//   - hlsl_alias_intrinsics_gen.inc: builtin alias declarations using
//     _HLSL_BUILTIN_ALIAS, included by hlsl_alias_intrinsics.h.
//   - hlsl_inline_intrinsics_gen.inc: inline function definitions (detail
//     helper calls and literal bodies), included by hlsl_intrinsics.h.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Argument and return type base classes
//===----------------------------------------------------------------------===//

// Base class for argument and return type positions.
class HLSLArgType;

````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines HLSL intrinsic functions in tablegen. The HLSLEmitter`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines HLSL intrinsic functions in tablegen. The HLSLEmitter`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `backend processes these definitions and generates two .inc files:`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`backend processes these definitions and generates two .inc files:`。
- **L11 EN**: Comment explains nearby logic, constraints, or intent: `hlsl_alias_intrinsics_gen.inc: builtin alias declarations using`.
  **L11 CN**: 注释解释附近代码的逻辑、约束或设计意图：`hlsl_alias_intrinsics_gen.inc: builtin alias declarations using`。
- **L12 EN**: Comment explains nearby logic, constraints, or intent: `_HLSL_BUILTIN_ALIAS, included by hlsl_alias_intrinsics.h.`.
  **L12 CN**: 注释解释附近代码的逻辑、约束或设计意图：`_HLSL_BUILTIN_ALIAS, included by hlsl_alias_intrinsics.h.`。
- **L13 EN**: Comment explains nearby logic, constraints, or intent: `hlsl_inline_intrinsics_gen.inc: inline function definitions (detail`.
  **L13 CN**: 注释解释附近代码的逻辑、约束或设计意图：`hlsl_inline_intrinsics_gen.inc: inline function definitions (detail`。
- **L14 EN**: Comment explains nearby logic, constraints, or intent: `helper calls and literal bodies), included by hlsl_intrinsics.h.`.
  **L14 CN**: 注释解释附近代码的逻辑、约束或设计意图：`helper calls and literal bodies), included by hlsl_intrinsics.h.`。
- **L15 EN**: Separator comment used for visual grouping.
  **L15 CN**: 用于视觉分组的分隔注释。
- **L16 EN**: Banner comment marking a file or section boundary.
  **L16 CN**: 横幅注释，用于标记文件或章节边界。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L18 EN**: Banner comment marking a file or section boundary.
  **L18 CN**: 横幅注释，用于标记文件或章节边界。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Argument and return type base classes`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Argument and return type base classes`。
- **L20 EN**: Banner comment marking a file or section boundary.
  **L20 CN**: 横幅注释，用于标记文件或章节边界。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L22 EN**: Comment explains nearby logic, constraints, or intent: `Base class for argument and return type positions.`.
  **L22 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Base class for argument and return type positions.`。
- **L23 EN**: Declares TableGen class record `HLSLArgType;`.
  **L23 CN**: 声明 TableGen class 记录 `HLSLArgType;`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 25-48

````tablegen
// Base class for return type positions.
class HLSLReturnType;

// Void return type.
def VoidTy : HLSLReturnType;

//===----------------------------------------------------------------------===//
// HLSL element types
//===----------------------------------------------------------------------===//

// Represents a concrete HLSL scalar element type.
// Can be used directly as an argument or return type for a fixed scalar
// (e.g., FloatTy in Args produces a 'float' argument).
class HLSLType<string name> : HLSLArgType, HLSLReturnType {
  string Name = name;
  string TypeName = name;

  // When set, overloads using this type are guarded by
  // #ifdef __HLSL_ENABLE_16_BIT and emitted with
  // _HLSL_AVAILABILITY(shadermodel, 6.2), or the intrinsic's Availability
  // if it is greater.
  bit Is16Bit = 0;

  // When set, overloads using this type are emitted with
````
- **L25 EN**: Comment explains nearby logic, constraints, or intent: `Base class for return type positions.`.
  **L25 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Base class for return type positions.`。
- **L26 EN**: Declares TableGen class record `HLSLReturnType;`.
  **L26 CN**: 声明 TableGen class 记录 `HLSLReturnType;`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `Void return type.`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Void return type.`。
- **L29 EN**: Declares TableGen def record `VoidTy`.
  **L29 CN**: 声明 TableGen def 记录 `VoidTy`。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Banner comment marking a file or section boundary.
  **L31 CN**: 横幅注释，用于标记文件或章节边界。
- **L32 EN**: Comment explains nearby logic, constraints, or intent: `HLSL element types`.
  **L32 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HLSL element types`。
- **L33 EN**: Banner comment marking a file or section boundary.
  **L33 CN**: 横幅注释，用于标记文件或章节边界。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Comment explains nearby logic, constraints, or intent: `Represents a concrete HLSL scalar element type.`.
  **L35 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Represents a concrete HLSL scalar element type.`。
- **L36 EN**: Comment explains nearby logic, constraints, or intent: `Can be used directly as an argument or return type for a fixed scalar`.
  **L36 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Can be used directly as an argument or return type for a fixed scalar`。
- **L37 EN**: Comment explains nearby logic, constraints, or intent: `(e.g., FloatTy in Args produces a 'float' argument).`.
  **L37 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(e.g., FloatTy in Args produces a 'float' argument).`。
- **L38 EN**: Declares TableGen class record `HLSLType`.
  **L38 CN**: 声明 TableGen class 记录 `HLSLType`。
- **L39 EN**: Initializes variable `Name` from the expression on the right-hand side.
  **L39 CN**: 使用右侧表达式初始化变量 `Name`。
- **L40 EN**: Initializes variable `TypeName` from the expression on the right-hand side.
  **L40 CN**: 使用右侧表达式初始化变量 `TypeName`。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Comment explains nearby logic, constraints, or intent: `When set, overloads using this type are guarded by`.
  **L42 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When set, overloads using this type are guarded by`。
- **L43 EN**: Comment explains nearby logic, constraints, or intent: `#ifdef __HLSL_ENABLE_16_BIT and emitted with`.
  **L43 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#ifdef __HLSL_ENABLE_16_BIT and emitted with`。
- **L44 EN**: Comment explains nearby logic, constraints, or intent: `_HLSL_AVAILABILITY(shadermodel, 6.2), or the intrinsic's Availability`.
  **L44 CN**: 注释解释附近代码的逻辑、约束或设计意图：`_HLSL_AVAILABILITY(shadermodel, 6.2), or the intrinsic's Availability`。
- **L45 EN**: Comment explains nearby logic, constraints, or intent: `if it is greater.`.
  **L45 CN**: 注释解释附近代码的逻辑、约束或设计意图：`if it is greater.`。
- **L46 EN**: Initializes variable `Is16Bit` from the expression on the right-hand side.
  **L46 CN**: 使用右侧表达式初始化变量 `Is16Bit`。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L48 EN**: Comment explains nearby logic, constraints, or intent: `When set, overloads using this type are emitted with`.
  **L48 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When set, overloads using this type are emitted with`。

### Lines 49-72

````tablegen
  // _HLSL_16BIT_AVAILABILITY(shadermodel, 6.2) instead of _HLSL_AVAILABILITY.
  // This macro expands to an availability attribute only when
  // __HLSL_ENABLE_16_BIT is defined (i.e. half is a true 16-bit float);
  // otherwise it expands to nothing since half is an alias for float.
  // If the intrinsic's Availability is >= SM6.2, _HLSL_AVAILABILITY is used
  // instead because 16-bit support is already implied.
  bit IsConditionally16Bit = 0;
}

def BoolTy   : HLSLType<"bool">;
def HalfTy   : HLSLType<"half">   { let IsConditionally16Bit = 1; }
def FloatTy  : HLSLType<"float">;
def DoubleTy : HLSLType<"double">;
def Int16Ty  : HLSLType<"int16_t">  { let Is16Bit = 1; }
def UInt16Ty : HLSLType<"uint16_t"> { let Is16Bit = 1; }
def IntTy    : HLSLType<"int">;
def UIntTy   : HLSLType<"uint">;
def Int64Ty  : HLSLType<"int64_t">;
def UInt64Ty : HLSLType<"uint64_t">;

//===----------------------------------------------------------------------===//
// Element type groups
//===----------------------------------------------------------------------===//

````
- **L49 EN**: Comment explains nearby logic, constraints, or intent: `_HLSL_16BIT_AVAILABILITY(shadermodel, 6.2) instead of _HLSL_AVAILABILITY.`.
  **L49 CN**: 注释解释附近代码的逻辑、约束或设计意图：`_HLSL_16BIT_AVAILABILITY(shadermodel, 6.2) instead of _HLSL_AVAILABILITY.`。
- **L50 EN**: Comment explains nearby logic, constraints, or intent: `This macro expands to an availability attribute only when`.
  **L50 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This macro expands to an availability attribute only when`。
- **L51 EN**: Comment explains nearby logic, constraints, or intent: `__HLSL_ENABLE_16_BIT is defined (i.e. half is a true 16-bit float);`.
  **L51 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__HLSL_ENABLE_16_BIT is defined (i.e. half is a true 16-bit float);`。
- **L52 EN**: Comment explains nearby logic, constraints, or intent: `otherwise it expands to nothing since half is an alias for float.`.
  **L52 CN**: 注释解释附近代码的逻辑、约束或设计意图：`otherwise it expands to nothing since half is an alias for float.`。
- **L53 EN**: Comment explains nearby logic, constraints, or intent: `If the intrinsic's Availability is > SM6.2, _HLSL_AVAILABILITY is used`.
  **L53 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If the intrinsic's Availability is > SM6.2, _HLSL_AVAILABILITY is used`。
- **L54 EN**: Comment explains nearby logic, constraints, or intent: `instead because 16-bit support is already implied.`.
  **L54 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead because 16-bit support is already implied.`。
- **L55 EN**: Initializes variable `IsConditionally16Bit` from the expression on the right-hand side.
  **L55 CN**: 使用右侧表达式初始化变量 `IsConditionally16Bit`。
- **L56 EN**: Closes the current lexical scope or compound statement.
  **L56 CN**: 结束当前词法作用域或复合语句块。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L58 EN**: Declares TableGen def record `BoolTy`.
  **L58 CN**: 声明 TableGen def 记录 `BoolTy`。
- **L59 EN**: Declares TableGen def record `HalfTy`.
  **L59 CN**: 声明 TableGen def 记录 `HalfTy`。
- **L60 EN**: Declares TableGen def record `FloatTy`.
  **L60 CN**: 声明 TableGen def 记录 `FloatTy`。
- **L61 EN**: Declares TableGen def record `DoubleTy`.
  **L61 CN**: 声明 TableGen def 记录 `DoubleTy`。
- **L62 EN**: Declares TableGen def record `Int16Ty`.
  **L62 CN**: 声明 TableGen def 记录 `Int16Ty`。
- **L63 EN**: Declares TableGen def record `UInt16Ty`.
  **L63 CN**: 声明 TableGen def 记录 `UInt16Ty`。
- **L64 EN**: Declares TableGen def record `IntTy`.
  **L64 CN**: 声明 TableGen def 记录 `IntTy`。
- **L65 EN**: Declares TableGen def record `UIntTy`.
  **L65 CN**: 声明 TableGen def 记录 `UIntTy`。
- **L66 EN**: Declares TableGen def record `Int64Ty`.
  **L66 CN**: 声明 TableGen def 记录 `Int64Ty`。
- **L67 EN**: Declares TableGen def record `UInt64Ty`.
  **L67 CN**: 声明 TableGen def 记录 `UInt64Ty`。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L69 EN**: Banner comment marking a file or section boundary.
  **L69 CN**: 横幅注释，用于标记文件或章节边界。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `Element type groups`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Element type groups`。
- **L71 EN**: Banner comment marking a file or section boundary.
  **L71 CN**: 横幅注释，用于标记文件或章节边界。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 73-96

````tablegen
defvar AllFloatTypes = [HalfTy, FloatTy, DoubleTy];
defvar SignedIntTypes = [Int16Ty, IntTy, Int64Ty];
defvar UnsignedIntTypes = [UInt16Ty, UIntTy, UInt64Ty];
defvar AllIntTypes = [Int16Ty, UInt16Ty, IntTy, UIntTy,
                      Int64Ty, UInt64Ty];
defvar SignedTypes = [Int16Ty, HalfTy, IntTy, FloatTy,
                      Int64Ty, DoubleTy];
defvar AllNumericTypes = [Int16Ty, UInt16Ty, HalfTy, IntTy, UIntTy,
                          FloatTy, Int64Ty, UInt64Ty, DoubleTy];
defvar AllTypesWithBool = [BoolTy, Int16Ty, UInt16Ty, HalfTy,
                           IntTy, UIntTy, FloatTy, Int64Ty,
                           UInt64Ty, DoubleTy];
defvar NumericTypesNoDbl = [Int16Ty, UInt16Ty, HalfTy, IntTy, UIntTy,
                            FloatTy, Int64Ty, UInt64Ty];

//===----------------------------------------------------------------------===//
// Argument/return types
//
// These classes are usable in both argument and return type positions.
//===----------------------------------------------------------------------===//

// The varying type - expanded per VaryingTypes.
// As an argument: the arg type varies with each overload.
// As a return type: returns the same type as the varying arg.
````
- **L73 EN**: Declares TableGen defvar record `AllFloatTypes = [HalfTy, FloatTy, DoubleTy];`.
  **L73 CN**: 声明 TableGen defvar 记录 `AllFloatTypes = [HalfTy, FloatTy, DoubleTy];`。
- **L74 EN**: Declares TableGen defvar record `SignedIntTypes = [Int16Ty, IntTy, Int64Ty];`.
  **L74 CN**: 声明 TableGen defvar 记录 `SignedIntTypes = [Int16Ty, IntTy, Int64Ty];`。
- **L75 EN**: Declares TableGen defvar record `UnsignedIntTypes = [UInt16Ty, UIntTy, UInt64Ty];`.
  **L75 CN**: 声明 TableGen defvar 记录 `UnsignedIntTypes = [UInt16Ty, UIntTy, UInt64Ty];`。
- **L76 EN**: Declares TableGen defvar record `AllIntTypes = [Int16Ty, UInt16Ty, IntTy, UIntTy,`.
  **L76 CN**: 声明 TableGen defvar 记录 `AllIntTypes = [Int16Ty, UInt16Ty, IntTy, UIntTy,`。
- **L77 EN**: Adds a standalone statement or declaration: `Int64Ty, UInt64Ty];`.
  **L77 CN**: 添加一条独立语句或声明：`Int64Ty, UInt64Ty];`。
- **L78 EN**: Declares TableGen defvar record `SignedTypes = [Int16Ty, HalfTy, IntTy, FloatTy,`.
  **L78 CN**: 声明 TableGen defvar 记录 `SignedTypes = [Int16Ty, HalfTy, IntTy, FloatTy,`。
- **L79 EN**: Adds a standalone statement or declaration: `Int64Ty, DoubleTy];`.
  **L79 CN**: 添加一条独立语句或声明：`Int64Ty, DoubleTy];`。
- **L80 EN**: Declares TableGen defvar record `AllNumericTypes = [Int16Ty, UInt16Ty, HalfTy, IntTy, UIntTy,`.
  **L80 CN**: 声明 TableGen defvar 记录 `AllNumericTypes = [Int16Ty, UInt16Ty, HalfTy, IntTy, UIntTy,`。
- **L81 EN**: Adds a standalone statement or declaration: `FloatTy, Int64Ty, UInt64Ty, DoubleTy];`.
  **L81 CN**: 添加一条独立语句或声明：`FloatTy, Int64Ty, UInt64Ty, DoubleTy];`。
- **L82 EN**: Declares TableGen defvar record `AllTypesWithBool = [BoolTy, Int16Ty, UInt16Ty, HalfTy,`.
  **L82 CN**: 声明 TableGen defvar 记录 `AllTypesWithBool = [BoolTy, Int16Ty, UInt16Ty, HalfTy,`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntTy, UIntTy, FloatTy, Int64Ty,`.
  **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntTy, UIntTy, FloatTy, Int64Ty,`。
- **L84 EN**: Adds a standalone statement or declaration: `UInt64Ty, DoubleTy];`.
  **L84 CN**: 添加一条独立语句或声明：`UInt64Ty, DoubleTy];`。
- **L85 EN**: Declares TableGen defvar record `NumericTypesNoDbl = [Int16Ty, UInt16Ty, HalfTy, IntTy, UIntTy,`.
  **L85 CN**: 声明 TableGen defvar 记录 `NumericTypesNoDbl = [Int16Ty, UInt16Ty, HalfTy, IntTy, UIntTy,`。
- **L86 EN**: Adds a standalone statement or declaration: `FloatTy, Int64Ty, UInt64Ty];`.
  **L86 CN**: 添加一条独立语句或声明：`FloatTy, Int64Ty, UInt64Ty];`。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L88 EN**: Banner comment marking a file or section boundary.
  **L88 CN**: 横幅注释，用于标记文件或章节边界。
- **L89 EN**: Comment explains nearby logic, constraints, or intent: `Argument/return types`.
  **L89 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Argument/return types`。
- **L90 EN**: Separator comment used for visual grouping.
  **L90 CN**: 用于视觉分组的分隔注释。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `These classes are usable in both argument and return type positions.`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These classes are usable in both argument and return type positions.`。
- **L92 EN**: Banner comment marking a file or section boundary.
  **L92 CN**: 横幅注释，用于标记文件或章节边界。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L94 EN**: Comment explains nearby logic, constraints, or intent: `The varying type - expanded per VaryingTypes.`.
  **L94 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The varying type - expanded per VaryingTypes.`。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `As an argument: the arg type varies with each overload.`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`As an argument: the arg type varies with each overload.`。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `As a return type: returns the same type as the varying arg.`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`As a return type: returns the same type as the varying arg.`。

### Lines 97-120

````tablegen
def Varying : HLSLArgType, HLSLReturnType;

// The scalar element of the varying type.
// As an argument: always the scalar element type regardless of overload shape.
// As a return type: returns the scalar element type (e.g., float dot(float3, float3)).
def VaryingElemType : HLSLArgType, HLSLReturnType;

// The varying shape with a fixed element type.
// As an argument: same shape as Varying but with the given element type.
// As a return type: same shape as the varying arg but with the given element type.
// For example, VaryingShape<UIntTy> with a float3 overload produces uint3.
class VaryingShape<HLSLType ty> : HLSLArgType, HLSLReturnType {
  HLSLType ElementType = ty;
}

// A concrete vector type (e.g., VectorType<UIntTy, 4> -> uint4).
// As an argument: the arg is always this vector type.
// As a return type: always returns this vector type, ignoring argument shape.
class VectorType<HLSLType ty, int size> : HLSLArgType, HLSLReturnType {
  HLSLType ElementType = ty;
  int Size = size;
}

//===----------------------------------------------------------------------===//
````
- **L97 EN**: Declares TableGen def record `Varying`.
  **L97 CN**: 声明 TableGen def 记录 `Varying`。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L99 EN**: Comment explains nearby logic, constraints, or intent: `The scalar element of the varying type.`.
  **L99 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The scalar element of the varying type.`。
- **L100 EN**: Comment explains nearby logic, constraints, or intent: `As an argument: always the scalar element type regardless of overload shape.`.
  **L100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`As an argument: always the scalar element type regardless of overload shape.`。
- **L101 EN**: Comment explains nearby logic, constraints, or intent: `As a return type: returns the scalar element type (e.g., float dot(float3, float3)).`.
  **L101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`As a return type: returns the scalar element type (e.g., float dot(float3, float3)).`。
- **L102 EN**: Declares TableGen def record `VaryingElemType`.
  **L102 CN**: 声明 TableGen def 记录 `VaryingElemType`。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L104 EN**: Comment explains nearby logic, constraints, or intent: `The varying shape with a fixed element type.`.
  **L104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The varying shape with a fixed element type.`。
- **L105 EN**: Comment explains nearby logic, constraints, or intent: `As an argument: same shape as Varying but with the given element type.`.
  **L105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`As an argument: same shape as Varying but with the given element type.`。
- **L106 EN**: Comment explains nearby logic, constraints, or intent: `As a return type: same shape as the varying arg but with the given element type.`.
  **L106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`As a return type: same shape as the varying arg but with the given element type.`。
- **L107 EN**: Comment explains nearby logic, constraints, or intent: `For example, VaryingShape<UIntTy> with a float3 overload produces uint3.`.
  **L107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For example, VaryingShape<UIntTy> with a float3 overload produces uint3.`。
- **L108 EN**: Declares TableGen class record `VaryingShape`.
  **L108 CN**: 声明 TableGen class 记录 `VaryingShape`。
- **L109 EN**: Initializes variable `ElementType` from the expression on the right-hand side.
  **L109 CN**: 使用右侧表达式初始化变量 `ElementType`。
- **L110 EN**: Closes the current lexical scope or compound statement.
  **L110 CN**: 结束当前词法作用域或复合语句块。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L112 EN**: Comment explains nearby logic, constraints, or intent: `A concrete vector type (e.g., VectorType<UIntTy, 4> -> uint4).`.
  **L112 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A concrete vector type (e.g., VectorType<UIntTy, 4> -> uint4).`。
- **L113 EN**: Comment explains nearby logic, constraints, or intent: `As an argument: the arg is always this vector type.`.
  **L113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`As an argument: the arg is always this vector type.`。
- **L114 EN**: Comment explains nearby logic, constraints, or intent: `As a return type: always returns this vector type, ignoring argument shape.`.
  **L114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`As a return type: always returns this vector type, ignoring argument shape.`。
- **L115 EN**: Declares TableGen class record `VectorType`.
  **L115 CN**: 声明 TableGen class 记录 `VectorType`。
- **L116 EN**: Initializes variable `ElementType` from the expression on the right-hand side.
  **L116 CN**: 使用右侧表达式初始化变量 `ElementType`。
- **L117 EN**: Initializes variable `Size` from the expression on the right-hand side.
  **L117 CN**: 使用右侧表达式初始化变量 `Size`。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L120 EN**: Banner comment marking a file or section boundary.
  **L120 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 121-144

````tablegen
// Shader model versions
//===----------------------------------------------------------------------===//

// Represents a shader model version
class ShaderModel<int major, int minor> {
  int Major = major;
  int Minor = minor;
}

// Sentinel: no shader model requirement.
def NoSM  : ShaderModel<0, 0>;

// Valid Shader Model records
foreach i = 0...9 in {
  def SM6_ #i : ShaderModel<6, i>;
}

//===----------------------------------------------------------------------===//
// Matrix dimension records
//===----------------------------------------------------------------------===//

class MatDim<int rows, int cols> {
  int Rows = rows;
  int Cols = cols;
````
- **L121 EN**: Comment explains nearby logic, constraints, or intent: `Shader model versions`.
  **L121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shader model versions`。
- **L122 EN**: Banner comment marking a file or section boundary.
  **L122 CN**: 横幅注释，用于标记文件或章节边界。
- **L123 EN**: Blank line separating nearby declarations or logic blocks.
  **L123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L124 EN**: Comment explains nearby logic, constraints, or intent: `Represents a shader model version`.
  **L124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Represents a shader model version`。
- **L125 EN**: Declares TableGen class record `ShaderModel`.
  **L125 CN**: 声明 TableGen class 记录 `ShaderModel`。
- **L126 EN**: Initializes variable `Major` from the expression on the right-hand side.
  **L126 CN**: 使用右侧表达式初始化变量 `Major`。
- **L127 EN**: Initializes variable `Minor` from the expression on the right-hand side.
  **L127 CN**: 使用右侧表达式初始化变量 `Minor`。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L130 EN**: Comment explains nearby logic, constraints, or intent: `Sentinel: no shader model requirement.`.
  **L130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sentinel: no shader model requirement.`。
- **L131 EN**: Declares TableGen def record `NoSM`.
  **L131 CN**: 声明 TableGen def 记录 `NoSM`。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L133 EN**: Comment explains nearby logic, constraints, or intent: `Valid Shader Model records`.
  **L133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Valid Shader Model records`。
- **L134 EN**: Starts a TableGen iteration used to generate repeated records: `foreach i = 0...9 in {`.
  **L134 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach i = 0...9 in {`。
- **L135 EN**: Declares TableGen def record `SM6_ #i`.
  **L135 CN**: 声明 TableGen def 记录 `SM6_ #i`。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Banner comment marking a file or section boundary.
  **L138 CN**: 横幅注释，用于标记文件或章节边界。
- **L139 EN**: Comment explains nearby logic, constraints, or intent: `Matrix dimension records`.
  **L139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Matrix dimension records`。
- **L140 EN**: Banner comment marking a file or section boundary.
  **L140 CN**: 横幅注释，用于标记文件或章节边界。
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L142 EN**: Declares TableGen class record `MatDim`.
  **L142 CN**: 声明 TableGen class 记录 `MatDim`。
- **L143 EN**: Initializes variable `Rows` from the expression on the right-hand side.
  **L143 CN**: 使用右侧表达式初始化变量 `Rows`。
- **L144 EN**: Initializes variable `Cols` from the expression on the right-hand side.
  **L144 CN**: 使用右侧表达式初始化变量 `Cols`。

### Lines 145-168

````tablegen
}

foreach r = 1...4 in
  foreach c = 1...4 in
    def Mat#r#"x"#c : MatDim<r, c>;

// All non-1x1 matrix dimensions (1x2 through 4x4).
defvar AllMatDims = [Mat1x2, Mat1x3, Mat1x4,
                     Mat2x1, Mat2x2, Mat2x3, Mat2x4,
                     Mat3x1, Mat3x2, Mat3x3, Mat3x4,
                     Mat4x1, Mat4x2, Mat4x3, Mat4x4];

//===----------------------------------------------------------------------===//
// HLSLBuiltin class
//===----------------------------------------------------------------------===//

class HLSLBuiltin<string name, string builtin = ""> {
  string Name = name;

  // When set, generates a _HLSL_BUILTIN_ALIAS(Builtin) declaration that
  // aliases the named Clang builtin. Mutually exclusive with DetailFunc
  // and Body.
  string Builtin = builtin;

````
- **L145 EN**: Closes the current lexical scope or compound statement.
  **L145 CN**: 结束当前词法作用域或复合语句块。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L147 EN**: Starts a TableGen iteration used to generate repeated records: `foreach r = 1...4 in`.
  **L147 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach r = 1...4 in`。
- **L148 EN**: Starts a TableGen iteration used to generate repeated records: `foreach c = 1...4 in`.
  **L148 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach c = 1...4 in`。
- **L149 EN**: Declares TableGen def record `Mat#r#"x"#c`.
  **L149 CN**: 声明 TableGen def 记录 `Mat#r#"x"#c`。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L151 EN**: Comment explains nearby logic, constraints, or intent: `All non-1x1 matrix dimensions (1x2 through 4x4).`.
  **L151 CN**: 注释解释附近代码的逻辑、约束或设计意图：`All non-1x1 matrix dimensions (1x2 through 4x4).`。
- **L152 EN**: Declares TableGen defvar record `AllMatDims = [Mat1x2, Mat1x3, Mat1x4,`.
  **L152 CN**: 声明 TableGen defvar 记录 `AllMatDims = [Mat1x2, Mat1x3, Mat1x4,`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Mat2x1, Mat2x2, Mat2x3, Mat2x4,`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`Mat2x1, Mat2x2, Mat2x3, Mat2x4,`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Mat3x1, Mat3x2, Mat3x3, Mat3x4,`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`Mat3x1, Mat3x2, Mat3x3, Mat3x4,`。
- **L155 EN**: Adds a standalone statement or declaration: `Mat4x1, Mat4x2, Mat4x3, Mat4x4];`.
  **L155 CN**: 添加一条独立语句或声明：`Mat4x1, Mat4x2, Mat4x3, Mat4x4];`。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L157 EN**: Banner comment marking a file or section boundary.
  **L157 CN**: 横幅注释，用于标记文件或章节边界。
- **L158 EN**: Comment explains nearby logic, constraints, or intent: `HLSLBuiltin class`.
  **L158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HLSLBuiltin class`。
- **L159 EN**: Banner comment marking a file or section boundary.
  **L159 CN**: 横幅注释，用于标记文件或章节边界。
- **L160 EN**: Blank line separating nearby declarations or logic blocks.
  **L160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L161 EN**: Declares TableGen class record `HLSLBuiltin`.
  **L161 CN**: 声明 TableGen class 记录 `HLSLBuiltin`。
- **L162 EN**: Initializes variable `Name` from the expression on the right-hand side.
  **L162 CN**: 使用右侧表达式初始化变量 `Name`。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L164 EN**: Comment explains nearby logic, constraints, or intent: `When set, generates a _HLSL_BUILTIN_ALIAS(Builtin) declaration that`.
  **L164 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When set, generates a _HLSL_BUILTIN_ALIAS(Builtin) declaration that`。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `aliases the named Clang builtin. Mutually exclusive with DetailFunc`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`aliases the named Clang builtin. Mutually exclusive with DetailFunc`。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `and Body.`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and Body.`。
- **L167 EN**: Initializes variable `Builtin` from the expression on the right-hand side.
  **L167 CN**: 使用右侧表达式初始化变量 `Builtin`。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 169-192

````tablegen
  // Doxygen documentation comment emitted before overloads in generated code.
  string Doc = "";

  // When set, generates an inline function body calling
  // __detail::DetailFunc(args...) instead of _HLSL_BUILTIN_ALIAS(Builtin).
  // Parameters are named p0, p1, p2, ... by default, or use ParamNames to
  // specify custom names. Mutually exclusive with Body and Builtin.
  string DetailFunc = "";

  // When set, generates an inline function with this literal body text.
  // Intended for single-statement functions. Multi-line functions should
  // instead be defined as a helper and called with DetailFunc.
  // Parameters are named p0, p1, p2, ... by default, or use ParamNames to
  // specify custom names. Mutually exclusive with DetailFunc and Builtin.
  code Body = "";

  // Determines how the return type is derived for each overload.
  HLSLReturnType ReturnType = VoidTy;

  // Argument list. Each entry is either:
  //   Varying     - type varies with VaryingTypes (expanded per type)
  //   HLSLType    - a fixed scalar type at that position (e.g., UIntTy)
  //   VectorType  - a fixed vector type at that position
  // The number of arguments is deduced from the length of this list.
````
- **L169 EN**: Comment explains nearby logic, constraints, or intent: `Doxygen documentation comment emitted before overloads in generated code.`.
  **L169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Doxygen documentation comment emitted before overloads in generated code.`。
- **L170 EN**: Initializes variable `Doc` from the expression on the right-hand side.
  **L170 CN**: 使用右侧表达式初始化变量 `Doc`。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L172 EN**: Comment explains nearby logic, constraints, or intent: `When set, generates an inline function body calling`.
  **L172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When set, generates an inline function body calling`。
- **L173 EN**: Comment explains nearby logic, constraints, or intent: `__detail::DetailFunc(args...) instead of _HLSL_BUILTIN_ALIAS(Builtin).`.
  **L173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__detail::DetailFunc(args...) instead of _HLSL_BUILTIN_ALIAS(Builtin).`。
- **L174 EN**: Comment explains nearby logic, constraints, or intent: `Parameters are named p0, p1, p2, ... by default, or use ParamNames to`.
  **L174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Parameters are named p0, p1, p2, ... by default, or use ParamNames to`。
- **L175 EN**: Comment explains nearby logic, constraints, or intent: `specify custom names. Mutually exclusive with Body and Builtin.`.
  **L175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specify custom names. Mutually exclusive with Body and Builtin.`。
- **L176 EN**: Initializes variable `DetailFunc` from the expression on the right-hand side.
  **L176 CN**: 使用右侧表达式初始化变量 `DetailFunc`。
- **L177 EN**: Blank line separating nearby declarations or logic blocks.
  **L177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L178 EN**: Comment explains nearby logic, constraints, or intent: `When set, generates an inline function with this literal body text.`.
  **L178 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When set, generates an inline function with this literal body text.`。
- **L179 EN**: Comment explains nearby logic, constraints, or intent: `Intended for single-statement functions. Multi-line functions should`.
  **L179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intended for single-statement functions. Multi-line functions should`。
- **L180 EN**: Comment explains nearby logic, constraints, or intent: `instead be defined as a helper and called with DetailFunc.`.
  **L180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead be defined as a helper and called with DetailFunc.`。
- **L181 EN**: Comment explains nearby logic, constraints, or intent: `Parameters are named p0, p1, p2, ... by default, or use ParamNames to`.
  **L181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Parameters are named p0, p1, p2, ... by default, or use ParamNames to`。
- **L182 EN**: Comment explains nearby logic, constraints, or intent: `specify custom names. Mutually exclusive with DetailFunc and Builtin.`.
  **L182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specify custom names. Mutually exclusive with DetailFunc and Builtin.`。
- **L183 EN**: Initializes variable `Body` from the expression on the right-hand side.
  **L183 CN**: 使用右侧表达式初始化变量 `Body`。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L185 EN**: Comment explains nearby logic, constraints, or intent: `Determines how the return type is derived for each overload.`.
  **L185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Determines how the return type is derived for each overload.`。
- **L186 EN**: Initializes variable `ReturnType` from the expression on the right-hand side.
  **L186 CN**: 使用右侧表达式初始化变量 `ReturnType`。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L188 EN**: Comment explains nearby logic, constraints, or intent: `Argument list. Each entry is either:`.
  **L188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Argument list. Each entry is either:`。
- **L189 EN**: Comment explains nearby logic, constraints, or intent: `Varying - type varies with VaryingTypes (expanded per type)`.
  **L189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Varying - type varies with VaryingTypes (expanded per type)`。
- **L190 EN**: Comment explains nearby logic, constraints, or intent: `HLSLType - a fixed scalar type at that position (e.g., UIntTy)`.
  **L190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HLSLType - a fixed scalar type at that position (e.g., UIntTy)`。
- **L191 EN**: Comment explains nearby logic, constraints, or intent: `VectorType - a fixed vector type at that position`.
  **L191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VectorType - a fixed vector type at that position`。
- **L192 EN**: Comment explains nearby logic, constraints, or intent: `The number of arguments is deduced from the length of this list.`.
  **L192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The number of arguments is deduced from the length of this list.`。

### Lines 193-216

````tablegen
  // Examples:
  //   [Varying]                              -> func(T)
  //   [Varying, Varying, Varying]            -> func(T, T, T)
  //   [Varying, UIntTy]                      -> func(T, uint)
  //   [UIntTy, UIntTy, IntTy]                -> func(uint, uint, int)
  //   []                                     -> func()
  list<HLSLArgType> Args = [];

  // Custom parameter names for generated functions.
  // When empty, inline functions (Body or DetailFunc) use p0, p1, p2, ...
  // and alias functions omit parameter names.
  list<string> ParamNames = [];

  // When set, emits 'constexpr' instead of 'inline' for inline functions
  // (i.e., functions using Body or DetailFunc).
  bit IsConstexpr = 0;

  // Whether the function has the convergent attribute
  bit IsConvergent = 0;

  // Argument element types - drives overload expansion.
  // One overload set is generated per type (scalar + vectors + matrices).
  // Only used when Args contains Varying entries.
  list<HLSLType> VaryingTypes = [];
````
- **L193 EN**: Comment explains nearby logic, constraints, or intent: `Examples:`.
  **L193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Examples:`。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `[Varying] -> func(T)`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[Varying] -> func(T)`。
- **L195 EN**: Comment explains nearby logic, constraints, or intent: `[Varying, Varying, Varying] -> func(T, T, T)`.
  **L195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[Varying, Varying, Varying] -> func(T, T, T)`。
- **L196 EN**: Comment explains nearby logic, constraints, or intent: `[Varying, UIntTy] -> func(T, uint)`.
  **L196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[Varying, UIntTy] -> func(T, uint)`。
- **L197 EN**: Comment explains nearby logic, constraints, or intent: `[UIntTy, UIntTy, IntTy] -> func(uint, uint, int)`.
  **L197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[UIntTy, UIntTy, IntTy] -> func(uint, uint, int)`。
- **L198 EN**: Comment explains nearby logic, constraints, or intent: `[] -> func()`.
  **L198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[] -> func()`。
- **L199 EN**: Initializes variable `Args` from the expression on the right-hand side.
  **L199 CN**: 使用右侧表达式初始化变量 `Args`。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L201 EN**: Comment explains nearby logic, constraints, or intent: `Custom parameter names for generated functions.`.
  **L201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Custom parameter names for generated functions.`。
- **L202 EN**: Comment explains nearby logic, constraints, or intent: `When empty, inline functions (Body or DetailFunc) use p0, p1, p2, ...`.
  **L202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When empty, inline functions (Body or DetailFunc) use p0, p1, p2, ...`。
- **L203 EN**: Comment explains nearby logic, constraints, or intent: `and alias functions omit parameter names.`.
  **L203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and alias functions omit parameter names.`。
- **L204 EN**: Initializes variable `ParamNames` from the expression on the right-hand side.
  **L204 CN**: 使用右侧表达式初始化变量 `ParamNames`。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Comment explains nearby logic, constraints, or intent: `When set, emits 'constexpr' instead of 'inline' for inline functions`.
  **L206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When set, emits 'constexpr' instead of 'inline' for inline functions`。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `(i.e., functions using Body or DetailFunc).`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(i.e., functions using Body or DetailFunc).`。
- **L208 EN**: Initializes variable `IsConstexpr` from the expression on the right-hand side.
  **L208 CN**: 使用右侧表达式初始化变量 `IsConstexpr`。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L210 EN**: Comment explains nearby logic, constraints, or intent: `Whether the function has the convergent attribute`.
  **L210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Whether the function has the convergent attribute`。
- **L211 EN**: Initializes variable `IsConvergent` from the expression on the right-hand side.
  **L211 CN**: 使用右侧表达式初始化变量 `IsConvergent`。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L213 EN**: Comment explains nearby logic, constraints, or intent: `Argument element types - drives overload expansion.`.
  **L213 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Argument element types - drives overload expansion.`。
- **L214 EN**: Comment explains nearby logic, constraints, or intent: `One overload set is generated per type (scalar + vectors + matrices).`.
  **L214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`One overload set is generated per type (scalar + vectors + matrices).`。
- **L215 EN**: Comment explains nearby logic, constraints, or intent: `Only used when Args contains Varying entries.`.
  **L215 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Only used when Args contains Varying entries.`。
- **L216 EN**: Initializes variable `VaryingTypes` from the expression on the right-hand side.
  **L216 CN**: 使用右侧表达式初始化变量 `VaryingTypes`。

### Lines 217-240

````tablegen

  // Whether to generate scalar overloads for Varying typed arguments.
  bit VaryingScalar = 0;

  // Vector sizes to generate for Varying typed arguments (e.g., [2,3,4]).
  list<int> VaryingVecSizes = [];

  // Matrix dimensions to generate for Varying typed arguments.
  list<MatDim> VaryingMatDims = [];

  // Default shader model availability version for all types.
  // Use NoSM for no availability requirement.
  ShaderModel Availability = NoSM;
}

//===----------------------------------------------------------------------===//
// HLSLBuiltin helper subclasses
//===----------------------------------------------------------------------===//

// T func(T) with scalar + vec2/3/4 + matrix overloads.
class HLSLOneArgBuiltin<string name, string builtin>
    : HLSLBuiltin<name, builtin> {
  let Args = [Varying];
  let ReturnType = Varying;
````
- **L217 EN**: Blank line separating nearby declarations or logic blocks.
  **L217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L218 EN**: Comment explains nearby logic, constraints, or intent: `Whether to generate scalar overloads for Varying typed arguments.`.
  **L218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Whether to generate scalar overloads for Varying typed arguments.`。
- **L219 EN**: Initializes variable `VaryingScalar` from the expression on the right-hand side.
  **L219 CN**: 使用右侧表达式初始化变量 `VaryingScalar`。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L221 EN**: Comment explains nearby logic, constraints, or intent: `Vector sizes to generate for Varying typed arguments (e.g., [2,3,4]).`.
  **L221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector sizes to generate for Varying typed arguments (e.g., [2,3,4]).`。
- **L222 EN**: Initializes variable `VaryingVecSizes` from the expression on the right-hand side.
  **L222 CN**: 使用右侧表达式初始化变量 `VaryingVecSizes`。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L224 EN**: Comment explains nearby logic, constraints, or intent: `Matrix dimensions to generate for Varying typed arguments.`.
  **L224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Matrix dimensions to generate for Varying typed arguments.`。
- **L225 EN**: Initializes variable `VaryingMatDims` from the expression on the right-hand side.
  **L225 CN**: 使用右侧表达式初始化变量 `VaryingMatDims`。
- **L226 EN**: Blank line separating nearby declarations or logic blocks.
  **L226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L227 EN**: Comment explains nearby logic, constraints, or intent: `Default shader model availability version for all types.`.
  **L227 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Default shader model availability version for all types.`。
- **L228 EN**: Comment explains nearby logic, constraints, or intent: `Use NoSM for no availability requirement.`.
  **L228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use NoSM for no availability requirement.`。
- **L229 EN**: Initializes variable `Availability` from the expression on the right-hand side.
  **L229 CN**: 使用右侧表达式初始化变量 `Availability`。
- **L230 EN**: Closes the current lexical scope or compound statement.
  **L230 CN**: 结束当前词法作用域或复合语句块。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L232 EN**: Banner comment marking a file or section boundary.
  **L232 CN**: 横幅注释，用于标记文件或章节边界。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `HLSLBuiltin helper subclasses`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HLSLBuiltin helper subclasses`。
- **L234 EN**: Banner comment marking a file or section boundary.
  **L234 CN**: 横幅注释，用于标记文件或章节边界。
- **L235 EN**: Blank line separating nearby declarations or logic blocks.
  **L235 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L236 EN**: Comment explains nearby logic, constraints, or intent: `T func(T) with scalar + vec2/3/4 + matrix overloads.`.
  **L236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`T func(T) with scalar + vec2/3/4 + matrix overloads.`。
- **L237 EN**: Declares TableGen class record `HLSLOneArgBuiltin`.
  **L237 CN**: 声明 TableGen class 记录 `HLSLOneArgBuiltin`。
- **L238 EN**: Continues the surrounding expression or declaration: `: HLSLBuiltin<name, builtin> {`.
  **L238 CN**: 继续构造周围的表达式或声明：`: HLSLBuiltin<name, builtin> {`。
- **L239 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L239 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L240 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L240 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。

### Lines 241-264

````tablegen
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = AllMatDims;
}

// T func(T, T) with scalar + vec2/3/4 + matrix overloads.
class HLSLTwoArgBuiltin<string name, string builtin>
    : HLSLBuiltin<name, builtin> {
  let Args = [Varying, Varying];
  let ReturnType = Varying;
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = AllMatDims;
}

// T func(T, T, T) with scalar + vec2/3/4 + matrix overloads.
class HLSLThreeArgBuiltin<string name, string builtin>
    : HLSLBuiltin<name, builtin> {
  let Args = [Varying, Varying, Varying];
  let ReturnType = Varying;
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = AllMatDims;
}
````
- **L241 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L241 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L242 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L242 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L243 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = AllMatDims;`.
  **L243 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = AllMatDims;`。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L246 EN**: Comment explains nearby logic, constraints, or intent: `T func(T, T) with scalar + vec2/3/4 + matrix overloads.`.
  **L246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`T func(T, T) with scalar + vec2/3/4 + matrix overloads.`。
- **L247 EN**: Declares TableGen class record `HLSLTwoArgBuiltin`.
  **L247 CN**: 声明 TableGen class 记录 `HLSLTwoArgBuiltin`。
- **L248 EN**: Continues the surrounding expression or declaration: `: HLSLBuiltin<name, builtin> {`.
  **L248 CN**: 继续构造周围的表达式或声明：`: HLSLBuiltin<name, builtin> {`。
- **L249 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L249 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L250 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L250 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L251 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L251 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L252 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L252 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L253 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = AllMatDims;`.
  **L253 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = AllMatDims;`。
- **L254 EN**: Closes the current lexical scope or compound statement.
  **L254 CN**: 结束当前词法作用域或复合语句块。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L256 EN**: Comment explains nearby logic, constraints, or intent: `T func(T, T, T) with scalar + vec2/3/4 + matrix overloads.`.
  **L256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`T func(T, T, T) with scalar + vec2/3/4 + matrix overloads.`。
- **L257 EN**: Declares TableGen class record `HLSLThreeArgBuiltin`.
  **L257 CN**: 声明 TableGen class 记录 `HLSLThreeArgBuiltin`。
- **L258 EN**: Continues the surrounding expression or declaration: `: HLSLBuiltin<name, builtin> {`.
  **L258 CN**: 继续构造周围的表达式或声明：`: HLSLBuiltin<name, builtin> {`。
- **L259 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying, Varying];`.
  **L259 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying, Varying];`。
- **L260 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L260 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L261 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L261 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L262 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L262 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L263 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = AllMatDims;`.
  **L263 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = AllMatDims;`。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。

### Lines 265-288

````tablegen

// Detail function base: generates inline function bodies calling
// __detail::DetailFunc(args...) instead of _HLSL_BUILTIN_ALIAS.
class HLSLDetail<string name, string detail> : HLSLBuiltin<name> {
  let DetailFunc = detail;
}

// T func(T) with scalar + vec2/3/4 + matrix overloads.
class HLSLOneArgDetail<string name, string detail>
    : HLSLDetail<name, detail> {
  let Args = [Varying];
  let ReturnType = Varying;
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = AllMatDims;
}

// T func(T, T) with scalar + vec2/3/4 + matrix overloads.
class HLSLTwoArgDetail<string name, string detail>
    : HLSLDetail<name, detail> {
  let Args = [Varying, Varying];
  let ReturnType = Varying;
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
````
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L266 EN**: Comment explains nearby logic, constraints, or intent: `Detail function base: generates inline function bodies calling`.
  **L266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Detail function base: generates inline function bodies calling`。
- **L267 EN**: Comment explains nearby logic, constraints, or intent: `__detail::DetailFunc(args...) instead of _HLSL_BUILTIN_ALIAS.`.
  **L267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__detail::DetailFunc(args...) instead of _HLSL_BUILTIN_ALIAS.`。
- **L268 EN**: Declares TableGen class record `HLSLDetail`.
  **L268 CN**: 声明 TableGen class 记录 `HLSLDetail`。
- **L269 EN**: Assigns a TableGen property that affects following records or inherited fields: `let DetailFunc = detail;`.
  **L269 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let DetailFunc = detail;`。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Blank line separating nearby declarations or logic blocks.
  **L271 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L272 EN**: Comment explains nearby logic, constraints, or intent: `T func(T) with scalar + vec2/3/4 + matrix overloads.`.
  **L272 CN**: 注释解释附近代码的逻辑、约束或设计意图：`T func(T) with scalar + vec2/3/4 + matrix overloads.`。
- **L273 EN**: Declares TableGen class record `HLSLOneArgDetail`.
  **L273 CN**: 声明 TableGen class 记录 `HLSLOneArgDetail`。
- **L274 EN**: Continues the surrounding expression or declaration: `: HLSLDetail<name, detail> {`.
  **L274 CN**: 继续构造周围的表达式或声明：`: HLSLDetail<name, detail> {`。
- **L275 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L275 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L276 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L276 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L277 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L277 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L278 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L278 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L279 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = AllMatDims;`.
  **L279 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = AllMatDims;`。
- **L280 EN**: Closes the current lexical scope or compound statement.
  **L280 CN**: 结束当前词法作用域或复合语句块。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L282 EN**: Comment explains nearby logic, constraints, or intent: `T func(T, T) with scalar + vec2/3/4 + matrix overloads.`.
  **L282 CN**: 注释解释附近代码的逻辑、约束或设计意图：`T func(T, T) with scalar + vec2/3/4 + matrix overloads.`。
- **L283 EN**: Declares TableGen class record `HLSLTwoArgDetail`.
  **L283 CN**: 声明 TableGen class 记录 `HLSLTwoArgDetail`。
- **L284 EN**: Continues the surrounding expression or declaration: `: HLSLDetail<name, detail> {`.
  **L284 CN**: 继续构造周围的表达式或声明：`: HLSLDetail<name, detail> {`。
- **L285 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L285 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L286 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L286 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L287 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L287 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L288 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L288 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。

### Lines 289-312

````tablegen
  let VaryingMatDims = AllMatDims;
}

// T func(T, T, T) with scalar + vec2/3/4 + matrix overloads.
class HLSLThreeArgDetail<string name, string detail>
    : HLSLDetail<name, detail> {
  let Args = [Varying, Varying, Varying];
  let ReturnType = Varying;
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = AllMatDims;
}

// Inline body variant: T func(T) with a literal inline body (no builtin alias).
// Body must be specified (e.g., let Body = "return p0;").
class HLSLOneArgInlineBuiltin<string name> : HLSLBuiltin<name> {
  let Args = [Varying];
  let ReturnType = Varying;
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = AllMatDims;
}

//===----------------------------------------------------------------------===//
````
- **L289 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = AllMatDims;`.
  **L289 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = AllMatDims;`。
- **L290 EN**: Closes the current lexical scope or compound statement.
  **L290 CN**: 结束当前词法作用域或复合语句块。
- **L291 EN**: Blank line separating nearby declarations or logic blocks.
  **L291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L292 EN**: Comment explains nearby logic, constraints, or intent: `T func(T, T, T) with scalar + vec2/3/4 + matrix overloads.`.
  **L292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`T func(T, T, T) with scalar + vec2/3/4 + matrix overloads.`。
- **L293 EN**: Declares TableGen class record `HLSLThreeArgDetail`.
  **L293 CN**: 声明 TableGen class 记录 `HLSLThreeArgDetail`。
- **L294 EN**: Continues the surrounding expression or declaration: `: HLSLDetail<name, detail> {`.
  **L294 CN**: 继续构造周围的表达式或声明：`: HLSLDetail<name, detail> {`。
- **L295 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying, Varying];`.
  **L295 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying, Varying];`。
- **L296 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L296 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L297 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L297 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L298 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L298 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L299 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = AllMatDims;`.
  **L299 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = AllMatDims;`。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L302 EN**: Comment explains nearby logic, constraints, or intent: `Inline body variant: T func(T) with a literal inline body (no builtin alias).`.
  **L302 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Inline body variant: T func(T) with a literal inline body (no builtin alias).`。
- **L303 EN**: Comment explains nearby logic, constraints, or intent: `Body must be specified (e.g., let Body "return p0;").`.
  **L303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Body must be specified (e.g., let Body "return p0;").`。
- **L304 EN**: Declares TableGen class record `HLSLOneArgInlineBuiltin`.
  **L304 CN**: 声明 TableGen class 记录 `HLSLOneArgInlineBuiltin`。
- **L305 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L305 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L306 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L306 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L307 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L307 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L308 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L308 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L309 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = AllMatDims;`.
  **L309 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = AllMatDims;`。
- **L310 EN**: Closes the current lexical scope or compound statement.
  **L310 CN**: 结束当前词法作用域或复合语句块。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L312 EN**: Banner comment marking a file or section boundary.
  **L312 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 313-336

````tablegen
// Intrinsic definitions (sorted alphabetically by function name)
//===----------------------------------------------------------------------===//

// Returns the absolute value of the input value, Val.
def hlsl_abs : HLSLOneArgBuiltin<"abs", "__builtin_elementwise_abs"> {
  let Doc = [{
\fn T abs(T Val)
\brief Returns the absolute value of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = SignedTypes;
  let VaryingMatDims = [];
}

// Unsigned abs is a constexpr identity - unsigned values are already non-negative.
def hlsl_abs_unsigned : HLSLOneArgInlineBuiltin<"abs"> {
  let Doc = [{
\fn T abs(T Val)
\brief Returns the absolute value of the input value, \a Val.
\param Val The input value.

Unsigned overload - unsigned values are already non-negative, so this
function returns its input unchanged.
}];
````
- **L313 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsic definitions (sorted alphabetically by function name)`.
  **L313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsic definitions (sorted alphabetically by function name)`。
- **L314 EN**: Banner comment marking a file or section boundary.
  **L314 CN**: 横幅注释，用于标记文件或章节边界。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L316 EN**: Comment explains nearby logic, constraints, or intent: `Returns the absolute value of the input value, Val.`.
  **L316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the absolute value of the input value, Val.`。
- **L317 EN**: Declares TableGen def record `hlsl_abs`.
  **L317 CN**: 声明 TableGen def 记录 `hlsl_abs`。
- **L318 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L318 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L319 EN**: Continues logic associated with callable symbol `abs`.
  **L319 CN**: 继续与可调用符号 `abs` 相关的逻辑。
- **L320 EN**: Continues the surrounding expression or declaration: `\brief Returns the absolute value of the input value, \a Val.`.
  **L320 CN**: 继续构造周围的表达式或声明：`\brief Returns the absolute value of the input value, \a Val.`。
- **L321 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L321 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L322 EN**: Adds a standalone statement or declaration: `}];`.
  **L322 CN**: 添加一条独立语句或声明：`}];`。
- **L323 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = SignedTypes;`.
  **L323 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = SignedTypes;`。
- **L324 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L324 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L325 EN**: Closes the current lexical scope or compound statement.
  **L325 CN**: 结束当前词法作用域或复合语句块。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Comment explains nearby logic, constraints, or intent: `Unsigned abs is a constexpr identity - unsigned values are already non-negative.`.
  **L327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unsigned abs is a constexpr identity - unsigned values are already non-negative.`。
- **L328 EN**: Declares TableGen def record `hlsl_abs_unsigned`.
  **L328 CN**: 声明 TableGen def 记录 `hlsl_abs_unsigned`。
- **L329 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L329 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L330 EN**: Continues logic associated with callable symbol `abs`.
  **L330 CN**: 继续与可调用符号 `abs` 相关的逻辑。
- **L331 EN**: Continues the surrounding expression or declaration: `\brief Returns the absolute value of the input value, \a Val.`.
  **L331 CN**: 继续构造周围的表达式或声明：`\brief Returns the absolute value of the input value, \a Val.`。
- **L332 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L332 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L334 EN**: Continues the surrounding expression or declaration: `Unsigned overload - unsigned values are already non-negative, so this`.
  **L334 CN**: 继续构造周围的表达式或声明：`Unsigned overload - unsigned values are already non-negative, so this`。
- **L335 EN**: Continues the surrounding expression or declaration: `function returns its input unchanged.`.
  **L335 CN**: 继续构造周围的表达式或声明：`function returns its input unchanged.`。
- **L336 EN**: Adds a standalone statement or declaration: `}];`.
  **L336 CN**: 添加一条独立语句或声明：`}];`。

### Lines 337-360

````tablegen
  let ParamNames = ["V"];
  let Body = "return V;";
  let IsConstexpr = 1;
  let VaryingTypes = UnsignedIntTypes;
  let VaryingMatDims = [];
}

// Returns the arccosine of the input value, Val.
def hlsl_acos : HLSLOneArgBuiltin<"acos", "__builtin_elementwise_acos"> {
  let Doc = [{
\fn T acos(T Val)
\brief Returns the arccosine of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Implements unsigned 64-bit integer addition using pairs of unsigned 32-bit
// integers.
def hlsl_adduint64 : HLSLBuiltin<"AddUint64", "__builtin_hlsl_adduint64"> {
  let Doc = [{
\fn T AddUint64(T a, T b)
\brief Implements unsigned 64-bit integer addition using pairs of unsigned
````
- **L337 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["V"];`.
  **L337 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["V"];`。
- **L338 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Body = "return V;";`.
  **L338 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Body = "return V;";`。
- **L339 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConstexpr = 1;`.
  **L339 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConstexpr = 1;`。
- **L340 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = UnsignedIntTypes;`.
  **L340 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = UnsignedIntTypes;`。
- **L341 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L341 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L344 EN**: Comment explains nearby logic, constraints, or intent: `Returns the arccosine of the input value, Val.`.
  **L344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the arccosine of the input value, Val.`。
- **L345 EN**: Declares TableGen def record `hlsl_acos`.
  **L345 CN**: 声明 TableGen def 记录 `hlsl_acos`。
- **L346 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L346 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L347 EN**: Continues logic associated with callable symbol `acos`.
  **L347 CN**: 继续与可调用符号 `acos` 相关的逻辑。
- **L348 EN**: Continues the surrounding expression or declaration: `\brief Returns the arccosine of the input value, \a Val.`.
  **L348 CN**: 继续构造周围的表达式或声明：`\brief Returns the arccosine of the input value, \a Val.`。
- **L349 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L349 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L350 EN**: Adds a standalone statement or declaration: `}];`.
  **L350 CN**: 添加一条独立语句或声明：`}];`。
- **L351 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L351 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L352 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L352 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L353 EN**: Closes the current lexical scope or compound statement.
  **L353 CN**: 结束当前词法作用域或复合语句块。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L355 EN**: Comment explains nearby logic, constraints, or intent: `Implements unsigned 64-bit integer addition using pairs of unsigned 32-bit`.
  **L355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Implements unsigned 64-bit integer addition using pairs of unsigned 32-bit`。
- **L356 EN**: Comment explains nearby logic, constraints, or intent: `integers.`.
  **L356 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integers.`。
- **L357 EN**: Declares TableGen def record `hlsl_adduint64`.
  **L357 CN**: 声明 TableGen def 记录 `hlsl_adduint64`。
- **L358 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L358 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L359 EN**: Continues logic associated with callable symbol `AddUint64`.
  **L359 CN**: 继续与可调用符号 `AddUint64` 相关的逻辑。
- **L360 EN**: Continues the surrounding expression or declaration: `\brief Implements unsigned 64-bit integer addition using pairs of unsigned`.
  **L360 CN**: 继续构造周围的表达式或声明：`\brief Implements unsigned 64-bit integer addition using pairs of unsigned`。

### Lines 361-384

````tablegen
32-bit integers.
\param x [in] The first unsigned 32-bit integer pair(s)
\param y [in] The second unsigned 32-bit integer pair(s)

This function takes one or two pairs (low, high) of unsigned 32-bit integer
values and returns pairs (low, high) of unsigned 32-bit integer
values representing the result of unsigned 64-bit integer addition.
}];
  let Args = [Varying, Varying];
  let ReturnType = Varying;
  let VaryingTypes = [UIntTy];
  let VaryingVecSizes = [2, 4];
  let Availability = SM6_0;
}

// Returns True if all components of the x parameter are non-zero;
// otherwise, false.
def hlsl_all : HLSLOneArgBuiltin<"all", "__builtin_hlsl_all"> {
  let Doc = [{
\fn bool all(T x)
\brief Returns True if all components of the \a x parameter are non-zero;
otherwise, false.
\param x The input value.
}];
````
- **L361 EN**: Continues the surrounding expression or declaration: `32-bit integers.`.
  **L361 CN**: 继续构造周围的表达式或声明：`32-bit integers.`。
- **L362 EN**: Continues logic associated with callable symbol `pair`.
  **L362 CN**: 继续与可调用符号 `pair` 相关的逻辑。
- **L363 EN**: Continues logic associated with callable symbol `pair`.
  **L363 CN**: 继续与可调用符号 `pair` 相关的逻辑。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L365 EN**: Continues logic associated with callable symbol `pairs`.
  **L365 CN**: 继续与可调用符号 `pairs` 相关的逻辑。
- **L366 EN**: Continues logic associated with callable symbol `pairs`.
  **L366 CN**: 继续与可调用符号 `pairs` 相关的逻辑。
- **L367 EN**: Continues the surrounding expression or declaration: `values representing the result of unsigned 64-bit integer addition.`.
  **L367 CN**: 继续构造周围的表达式或声明：`values representing the result of unsigned 64-bit integer addition.`。
- **L368 EN**: Adds a standalone statement or declaration: `}];`.
  **L368 CN**: 添加一条独立语句或声明：`}];`。
- **L369 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L369 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L370 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L370 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L371 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [UIntTy];`.
  **L371 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [UIntTy];`。
- **L372 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 4];`.
  **L372 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 4];`。
- **L373 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L373 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L374 EN**: Closes the current lexical scope or compound statement.
  **L374 CN**: 结束当前词法作用域或复合语句块。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L376 EN**: Comment explains nearby logic, constraints, or intent: `Returns True if all components of the x parameter are non-zero;`.
  **L376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns True if all components of the x parameter are non-zero;`。
- **L377 EN**: Comment explains nearby logic, constraints, or intent: `otherwise, false.`.
  **L377 CN**: 注释解释附近代码的逻辑、约束或设计意图：`otherwise, false.`。
- **L378 EN**: Declares TableGen def record `hlsl_all`.
  **L378 CN**: 声明 TableGen def 记录 `hlsl_all`。
- **L379 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L379 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L380 EN**: Continues logic associated with callable symbol `all`.
  **L380 CN**: 继续与可调用符号 `all` 相关的逻辑。
- **L381 EN**: Adds a standalone statement or declaration: `\brief Returns True if all components of the \a x parameter are non-zero;`.
  **L381 CN**: 添加一条独立语句或声明：`\brief Returns True if all components of the \a x parameter are non-zero;`。
- **L382 EN**: Continues the surrounding expression or declaration: `otherwise, false.`.
  **L382 CN**: 继续构造周围的表达式或声明：`otherwise, false.`。
- **L383 EN**: Continues the surrounding expression or declaration: `\param x The input value.`.
  **L383 CN**: 继续构造周围的表达式或声明：`\param x The input value.`。
- **L384 EN**: Adds a standalone statement or declaration: `}];`.
  **L384 CN**: 添加一条独立语句或声明：`}];`。

### Lines 385-408

````tablegen
  let ReturnType = BoolTy;
  let VaryingTypes = AllTypesWithBool;
  let VaryingMatDims = [];
}

// Returns the boolean AND of two boolean scalars, vectors, or matrices.
def hlsl_and : HLSLTwoArgBuiltin<"and", "__builtin_hlsl_and"> {
  let Doc = [{
\fn bool and(bool x, bool y)
\brief Logically ands two boolean vectors or matrices elementwise and
produces a bool vector or matrix output.
}];
  let VaryingTypes = [BoolTy];
}

// Returns True if any components of the x parameter are non-zero;
// otherwise, false.
def hlsl_any : HLSLOneArgBuiltin<"any", "__builtin_hlsl_any"> {
  let Doc = [{
\fn bool any(T x)
\brief Returns True if any components of the \a x parameter are non-zero;
otherwise, false.
\param x The input value.
}];
````
- **L385 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = BoolTy;`.
  **L385 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = BoolTy;`。
- **L386 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllTypesWithBool;`.
  **L386 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllTypesWithBool;`。
- **L387 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L387 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L388 EN**: Closes the current lexical scope or compound statement.
  **L388 CN**: 结束当前词法作用域或复合语句块。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L390 EN**: Comment explains nearby logic, constraints, or intent: `Returns the boolean AND of two boolean scalars, vectors, or matrices.`.
  **L390 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the boolean AND of two boolean scalars, vectors, or matrices.`。
- **L391 EN**: Declares TableGen def record `hlsl_and`.
  **L391 CN**: 声明 TableGen def 记录 `hlsl_and`。
- **L392 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L392 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L393 EN**: Continues logic associated with callable symbol `and`.
  **L393 CN**: 继续与可调用符号 `and` 相关的逻辑。
- **L394 EN**: Continues the surrounding expression or declaration: `\brief Logically ands two boolean vectors or matrices elementwise and`.
  **L394 CN**: 继续构造周围的表达式或声明：`\brief Logically ands two boolean vectors or matrices elementwise and`。
- **L395 EN**: Continues the surrounding expression or declaration: `produces a bool vector or matrix output.`.
  **L395 CN**: 继续构造周围的表达式或声明：`produces a bool vector or matrix output.`。
- **L396 EN**: Adds a standalone statement or declaration: `}];`.
  **L396 CN**: 添加一条独立语句或声明：`}];`。
- **L397 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [BoolTy];`.
  **L397 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [BoolTy];`。
- **L398 EN**: Closes the current lexical scope or compound statement.
  **L398 CN**: 结束当前词法作用域或复合语句块。
- **L399 EN**: Blank line separating nearby declarations or logic blocks.
  **L399 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L400 EN**: Comment explains nearby logic, constraints, or intent: `Returns True if any components of the x parameter are non-zero;`.
  **L400 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns True if any components of the x parameter are non-zero;`。
- **L401 EN**: Comment explains nearby logic, constraints, or intent: `otherwise, false.`.
  **L401 CN**: 注释解释附近代码的逻辑、约束或设计意图：`otherwise, false.`。
- **L402 EN**: Declares TableGen def record `hlsl_any`.
  **L402 CN**: 声明 TableGen def 记录 `hlsl_any`。
- **L403 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L403 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L404 EN**: Continues logic associated with callable symbol `any`.
  **L404 CN**: 继续与可调用符号 `any` 相关的逻辑。
- **L405 EN**: Adds a standalone statement or declaration: `\brief Returns True if any components of the \a x parameter are non-zero;`.
  **L405 CN**: 添加一条独立语句或声明：`\brief Returns True if any components of the \a x parameter are non-zero;`。
- **L406 EN**: Continues the surrounding expression or declaration: `otherwise, false.`.
  **L406 CN**: 继续构造周围的表达式或声明：`otherwise, false.`。
- **L407 EN**: Continues the surrounding expression or declaration: `\param x The input value.`.
  **L407 CN**: 继续构造周围的表达式或声明：`\param x The input value.`。
- **L408 EN**: Adds a standalone statement or declaration: `}];`.
  **L408 CN**: 添加一条独立语句或声明：`}];`。

### Lines 409-432

````tablegen
  let ReturnType = BoolTy;
  let VaryingTypes = AllTypesWithBool;
  let VaryingMatDims = [];
}

// Reinterprets a cast value (two 32-bit values) into a double.
def hlsl_asdouble : HLSLTwoArgBuiltin<"asdouble", "__builtin_hlsl_asdouble"> {
  let Doc = [{
\fn double asdouble(uint LowBits, uint HighBits)
\brief Reinterprets a cast value (two 32-bit values) into a double.
\param LowBits The low 32-bit pattern of the input value.
\param HighBits The high 32-bit pattern of the input value.
}];
  let ReturnType = VaryingShape<DoubleTy>;
  let VaryingTypes = [UIntTy];
  let VaryingMatDims = [];
}

// Returns the arcsine of the input value, Val.
def hlsl_asin : HLSLOneArgBuiltin<"asin", "__builtin_elementwise_asin"> {
  let Doc = [{
\fn T asin(T Val)
\brief Returns the arcsine of the input value, \a Val.
\param Val The input value.
````
- **L409 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = BoolTy;`.
  **L409 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = BoolTy;`。
- **L410 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllTypesWithBool;`.
  **L410 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllTypesWithBool;`。
- **L411 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L411 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `Reinterprets a cast value (two 32-bit values) into a double.`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reinterprets a cast value (two 32-bit values) into a double.`。
- **L415 EN**: Declares TableGen def record `hlsl_asdouble`.
  **L415 CN**: 声明 TableGen def 记录 `hlsl_asdouble`。
- **L416 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L416 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L417 EN**: Continues logic associated with callable symbol `asdouble`.
  **L417 CN**: 继续与可调用符号 `asdouble` 相关的逻辑。
- **L418 EN**: Continues logic associated with callable symbol `value`.
  **L418 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L419 EN**: Continues the surrounding expression or declaration: `\param LowBits The low 32-bit pattern of the input value.`.
  **L419 CN**: 继续构造周围的表达式或声明：`\param LowBits The low 32-bit pattern of the input value.`。
- **L420 EN**: Continues the surrounding expression or declaration: `\param HighBits The high 32-bit pattern of the input value.`.
  **L420 CN**: 继续构造周围的表达式或声明：`\param HighBits The high 32-bit pattern of the input value.`。
- **L421 EN**: Adds a standalone statement or declaration: `}];`.
  **L421 CN**: 添加一条独立语句或声明：`}];`。
- **L422 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<DoubleTy>;`.
  **L422 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<DoubleTy>;`。
- **L423 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [UIntTy];`.
  **L423 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [UIntTy];`。
- **L424 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L424 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L425 EN**: Closes the current lexical scope or compound statement.
  **L425 CN**: 结束当前词法作用域或复合语句块。
- **L426 EN**: Blank line separating nearby declarations or logic blocks.
  **L426 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L427 EN**: Comment explains nearby logic, constraints, or intent: `Returns the arcsine of the input value, Val.`.
  **L427 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the arcsine of the input value, Val.`。
- **L428 EN**: Declares TableGen def record `hlsl_asin`.
  **L428 CN**: 声明 TableGen def 记录 `hlsl_asin`。
- **L429 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L429 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L430 EN**: Continues logic associated with callable symbol `asin`.
  **L430 CN**: 继续与可调用符号 `asin` 相关的逻辑。
- **L431 EN**: Continues the surrounding expression or declaration: `\brief Returns the arcsine of the input value, \a Val.`.
  **L431 CN**: 继续构造周围的表达式或声明：`\brief Returns the arcsine of the input value, \a Val.`。
- **L432 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L432 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。

### Lines 433-456

````tablegen
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the arctangent of the input value, Val.
def hlsl_atan : HLSLOneArgBuiltin<"atan", "__builtin_elementwise_atan"> {
  let Doc = [{
\fn T atan(T Val)
\brief Returns the arctangent of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the arctangent of y/x, using the signs of the arguments to determine
// the correct quadrant.
def hlsl_atan2 : HLSLTwoArgBuiltin<"atan2", "__builtin_elementwise_atan2"> {
  let Doc = [{
\fn T atan2(T y, T x)
\brief Returns the arctangent of y/x, using the signs of the arguments to
determine the correct quadrant.
\param y The y-coordinate.
````
- **L433 EN**: Adds a standalone statement or declaration: `}];`.
  **L433 CN**: 添加一条独立语句或声明：`}];`。
- **L434 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L434 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L435 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L435 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L436 EN**: Closes the current lexical scope or compound statement.
  **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `Returns the arctangent of the input value, Val.`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the arctangent of the input value, Val.`。
- **L439 EN**: Declares TableGen def record `hlsl_atan`.
  **L439 CN**: 声明 TableGen def 记录 `hlsl_atan`。
- **L440 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L440 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L441 EN**: Continues logic associated with callable symbol `atan`.
  **L441 CN**: 继续与可调用符号 `atan` 相关的逻辑。
- **L442 EN**: Continues the surrounding expression or declaration: `\brief Returns the arctangent of the input value, \a Val.`.
  **L442 CN**: 继续构造周围的表达式或声明：`\brief Returns the arctangent of the input value, \a Val.`。
- **L443 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L443 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L444 EN**: Adds a standalone statement or declaration: `}];`.
  **L444 CN**: 添加一条独立语句或声明：`}];`。
- **L445 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L445 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L446 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L446 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L447 EN**: Closes the current lexical scope or compound statement.
  **L447 CN**: 结束当前词法作用域或复合语句块。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L449 EN**: Comment explains nearby logic, constraints, or intent: `Returns the arctangent of y/x, using the signs of the arguments to determine`.
  **L449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the arctangent of y/x, using the signs of the arguments to determine`。
- **L450 EN**: Comment explains nearby logic, constraints, or intent: `the correct quadrant.`.
  **L450 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the correct quadrant.`。
- **L451 EN**: Declares TableGen def record `hlsl_atan2`.
  **L451 CN**: 声明 TableGen def 记录 `hlsl_atan2`。
- **L452 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L452 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L453 EN**: Continues logic associated with callable symbol `atan2`.
  **L453 CN**: 继续与可调用符号 `atan2` 相关的逻辑。
- **L454 EN**: Continues the surrounding expression or declaration: `\brief Returns the arctangent of y/x, using the signs of the arguments to`.
  **L454 CN**: 继续构造周围的表达式或声明：`\brief Returns the arctangent of y/x, using the signs of the arguments to`。
- **L455 EN**: Continues the surrounding expression or declaration: `determine the correct quadrant.`.
  **L455 CN**: 继续构造周围的表达式或声明：`determine the correct quadrant.`。
- **L456 EN**: Continues the surrounding expression or declaration: `\param y The y-coordinate.`.
  **L456 CN**: 继续构造周围的表达式或声明：`\param y The y-coordinate.`。

### Lines 457-480

````tablegen
\param x The x-coordinate.
}];
  let VaryingTypes = [HalfTy, FloatTy];
}

// Returns the smallest integer value that is greater than or equal to the
// input value, Val.
def hlsl_ceil : HLSLOneArgBuiltin<"ceil", "__builtin_elementwise_ceil"> {
  let Doc = [{
\fn T ceil(T Val)
\brief Returns the smallest integer value that is greater than or equal to
the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Checks whether the value is fully mapped.
def hlsl_check_access_fully_mapped : HLSLBuiltin<"CheckAccessFullyMapped"> {
  let Doc = [{
\fn bool CheckAccessFullyMapped(uint Status)
\brief Checks whether the value is fully mapped.
\param Status The status value to check.
````
- **L457 EN**: Continues the surrounding expression or declaration: `\param x The x-coordinate.`.
  **L457 CN**: 继续构造周围的表达式或声明：`\param x The x-coordinate.`。
- **L458 EN**: Adds a standalone statement or declaration: `}];`.
  **L458 CN**: 添加一条独立语句或声明：`}];`。
- **L459 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L459 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L460 EN**: Closes the current lexical scope or compound statement.
  **L460 CN**: 结束当前词法作用域或复合语句块。
- **L461 EN**: Blank line separating nearby declarations or logic blocks.
  **L461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L462 EN**: Comment explains nearby logic, constraints, or intent: `Returns the smallest integer value that is greater than or equal to the`.
  **L462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the smallest integer value that is greater than or equal to the`。
- **L463 EN**: Comment explains nearby logic, constraints, or intent: `input value, Val.`.
  **L463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`input value, Val.`。
- **L464 EN**: Declares TableGen def record `hlsl_ceil`.
  **L464 CN**: 声明 TableGen def 记录 `hlsl_ceil`。
- **L465 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L465 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L466 EN**: Continues logic associated with callable symbol `ceil`.
  **L466 CN**: 继续与可调用符号 `ceil` 相关的逻辑。
- **L467 EN**: Continues the surrounding expression or declaration: `\brief Returns the smallest integer value that is greater than or equal to`.
  **L467 CN**: 继续构造周围的表达式或声明：`\brief Returns the smallest integer value that is greater than or equal to`。
- **L468 EN**: Continues the surrounding expression or declaration: `the input value, \a Val.`.
  **L468 CN**: 继续构造周围的表达式或声明：`the input value, \a Val.`。
- **L469 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L469 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L470 EN**: Adds a standalone statement or declaration: `}];`.
  **L470 CN**: 添加一条独立语句或声明：`}];`。
- **L471 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L471 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L472 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L472 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L473 EN**: Closes the current lexical scope or compound statement.
  **L473 CN**: 结束当前词法作用域或复合语句块。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Comment explains nearby logic, constraints, or intent: `Checks whether the value is fully mapped.`.
  **L475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Checks whether the value is fully mapped.`。
- **L476 EN**: Declares TableGen def record `hlsl_check_access_fully_mapped`.
  **L476 CN**: 声明 TableGen def 记录 `hlsl_check_access_fully_mapped`。
- **L477 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L477 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L478 EN**: Continues logic associated with callable symbol `CheckAccessFullyMapped`.
  **L478 CN**: 继续与可调用符号 `CheckAccessFullyMapped` 相关的逻辑。
- **L479 EN**: Continues the surrounding expression or declaration: `\brief Checks whether the value is fully mapped.`.
  **L479 CN**: 继续构造周围的表达式或声明：`\brief Checks whether the value is fully mapped.`。
- **L480 EN**: Continues the surrounding expression or declaration: `\param Status The status value to check.`.
  **L480 CN**: 继续构造周围的表达式或声明：`\param Status The status value to check.`。

### Lines 481-504

````tablegen
}];
  let ParamNames = ["Status"];
  let Body = "return static_cast<bool>(Status);";
  let Args = [UIntTy];
  let ReturnType = BoolTy;
}

// Clamps the specified value X to the specified minimum (Min) and maximum (Max)
// range.
def hlsl_clamp : HLSLThreeArgBuiltin<"clamp", "__builtin_hlsl_elementwise_clamp"> {
  let Doc = [{
\fn T clamp(T X, T Min, T Max)
\brief Clamps the specified value \a X to the specified
minimum ( \a Min) and maximum ( \a Max) range.
\param X A value to clamp.
\param Min The specified minimum range.
\param Max The specified maximum range.

Returns The clamped value for the \a X parameter.
For values of -INF or INF, clamp will behave as expected.
However for values of NaN, the results are undefined.
}];
  let VaryingTypes = AllNumericTypes;
  let VaryingMatDims = [];
````
- **L481 EN**: Adds a standalone statement or declaration: `}];`.
  **L481 CN**: 添加一条独立语句或声明：`}];`。
- **L482 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["Status"];`.
  **L482 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["Status"];`。
- **L483 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Body = "return static_cast<bool>(Status);";`.
  **L483 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Body = "return static_cast<bool>(Status);";`。
- **L484 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [UIntTy];`.
  **L484 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [UIntTy];`。
- **L485 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = BoolTy;`.
  **L485 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = BoolTy;`。
- **L486 EN**: Closes the current lexical scope or compound statement.
  **L486 CN**: 结束当前词法作用域或复合语句块。
- **L487 EN**: Blank line separating nearby declarations or logic blocks.
  **L487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L488 EN**: Comment explains nearby logic, constraints, or intent: `Clamps the specified value X to the specified minimum (Min) and maximum (Max)`.
  **L488 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Clamps the specified value X to the specified minimum (Min) and maximum (Max)`。
- **L489 EN**: Comment explains nearby logic, constraints, or intent: `range.`.
  **L489 CN**: 注释解释附近代码的逻辑、约束或设计意图：`range.`。
- **L490 EN**: Declares TableGen def record `hlsl_clamp`.
  **L490 CN**: 声明 TableGen def 记录 `hlsl_clamp`。
- **L491 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L491 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L492 EN**: Continues logic associated with callable symbol `clamp`.
  **L492 CN**: 继续与可调用符号 `clamp` 相关的逻辑。
- **L493 EN**: Continues the surrounding expression or declaration: `\brief Clamps the specified value \a X to the specified`.
  **L493 CN**: 继续构造周围的表达式或声明：`\brief Clamps the specified value \a X to the specified`。
- **L494 EN**: Continues logic associated with callable symbol `minimum`.
  **L494 CN**: 继续与可调用符号 `minimum` 相关的逻辑。
- **L495 EN**: Continues the surrounding expression or declaration: `\param X A value to clamp.`.
  **L495 CN**: 继续构造周围的表达式或声明：`\param X A value to clamp.`。
- **L496 EN**: Continues the surrounding expression or declaration: `\param Min The specified minimum range.`.
  **L496 CN**: 继续构造周围的表达式或声明：`\param Min The specified minimum range.`。
- **L497 EN**: Continues the surrounding expression or declaration: `\param Max The specified maximum range.`.
  **L497 CN**: 继续构造周围的表达式或声明：`\param Max The specified maximum range.`。
- **L498 EN**: Blank line separating nearby declarations or logic blocks.
  **L498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L499 EN**: Continues the surrounding expression or declaration: `Returns The clamped value for the \a X parameter.`.
  **L499 CN**: 继续构造周围的表达式或声明：`Returns The clamped value for the \a X parameter.`。
- **L500 EN**: Continues the surrounding expression or declaration: `For values of -INF or INF, clamp will behave as expected.`.
  **L500 CN**: 继续构造周围的表达式或声明：`For values of -INF or INF, clamp will behave as expected.`。
- **L501 EN**: Continues the surrounding expression or declaration: `However for values of NaN, the results are undefined.`.
  **L501 CN**: 继续构造周围的表达式或声明：`However for values of NaN, the results are undefined.`。
- **L502 EN**: Adds a standalone statement or declaration: `}];`.
  **L502 CN**: 添加一条独立语句或声明：`}];`。
- **L503 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L503 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L504 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L504 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。

### Lines 505-528

````tablegen
}

// Discards the current pixel if the specified value is less than zero.
def hlsl_clip : HLSLOneArgBuiltin<"clip", "__builtin_hlsl_elementwise_clip"> {
  let Doc = [{
\fn void clip(T Val)
\brief Discards the current pixel if the specified value is less than zero.
\param Val The input value.
}];
  let ReturnType = VoidTy;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the cosine of the input value, Val.
def hlsl_cos : HLSLOneArgBuiltin<"cos", "__builtin_elementwise_cos"> {
  let Doc = [{
\fn T cos(T Val)
\brief Returns the cosine of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}
````
- **L505 EN**: Closes the current lexical scope or compound statement.
  **L505 CN**: 结束当前词法作用域或复合语句块。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L507 EN**: Comment explains nearby logic, constraints, or intent: `Discards the current pixel if the specified value is less than zero.`.
  **L507 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Discards the current pixel if the specified value is less than zero.`。
- **L508 EN**: Declares TableGen def record `hlsl_clip`.
  **L508 CN**: 声明 TableGen def 记录 `hlsl_clip`。
- **L509 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L509 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L510 EN**: Continues logic associated with callable symbol `clip`.
  **L510 CN**: 继续与可调用符号 `clip` 相关的逻辑。
- **L511 EN**: Continues the surrounding expression or declaration: `\brief Discards the current pixel if the specified value is less than zero.`.
  **L511 CN**: 继续构造周围的表达式或声明：`\brief Discards the current pixel if the specified value is less than zero.`。
- **L512 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L512 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L513 EN**: Adds a standalone statement or declaration: `}];`.
  **L513 CN**: 添加一条独立语句或声明：`}];`。
- **L514 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VoidTy;`.
  **L514 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VoidTy;`。
- **L515 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L515 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L516 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L516 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L517 EN**: Closes the current lexical scope or compound statement.
  **L517 CN**: 结束当前词法作用域或复合语句块。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Comment explains nearby logic, constraints, or intent: `Returns the cosine of the input value, Val.`.
  **L519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the cosine of the input value, Val.`。
- **L520 EN**: Declares TableGen def record `hlsl_cos`.
  **L520 CN**: 声明 TableGen def 记录 `hlsl_cos`。
- **L521 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L521 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L522 EN**: Continues logic associated with callable symbol `cos`.
  **L522 CN**: 继续与可调用符号 `cos` 相关的逻辑。
- **L523 EN**: Continues the surrounding expression or declaration: `\brief Returns the cosine of the input value, \a Val.`.
  **L523 CN**: 继续构造周围的表达式或声明：`\brief Returns the cosine of the input value, \a Val.`。
- **L524 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L524 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L525 EN**: Adds a standalone statement or declaration: `}];`.
  **L525 CN**: 添加一条独立语句或声明：`}];`。
- **L526 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L526 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L527 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L527 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L528 EN**: Closes the current lexical scope or compound statement.
  **L528 CN**: 结束当前词法作用域或复合语句块。

### Lines 529-552

````tablegen

// Returns the hyperbolic cosine of the input value, Val.
def hlsl_cosh : HLSLOneArgBuiltin<"cosh", "__builtin_elementwise_cosh"> {
  let Doc = [{
\fn T cosh(T Val)
\brief Returns the hyperbolic cosine of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the number of bits (per component) set in the input integer.
def hlsl_countbits : HLSLBuiltin<"countbits"> {
  let Doc = [{
\fn T countbits(T Val)
\brief Return the number of bits (per component) set in the input integer.
\param Val The input value.
}];
  let ParamNames = ["x"];
  let Body = "return __builtin_elementwise_popcount(x);";
  let Args = [Varying];
  let ReturnType = VaryingShape<UIntTy>;
  let VaryingTypes = AllIntTypes;
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Comment explains nearby logic, constraints, or intent: `Returns the hyperbolic cosine of the input value, Val.`.
  **L530 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the hyperbolic cosine of the input value, Val.`。
- **L531 EN**: Declares TableGen def record `hlsl_cosh`.
  **L531 CN**: 声明 TableGen def 记录 `hlsl_cosh`。
- **L532 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L532 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L533 EN**: Continues logic associated with callable symbol `cosh`.
  **L533 CN**: 继续与可调用符号 `cosh` 相关的逻辑。
- **L534 EN**: Continues the surrounding expression or declaration: `\brief Returns the hyperbolic cosine of the input value, \a Val.`.
  **L534 CN**: 继续构造周围的表达式或声明：`\brief Returns the hyperbolic cosine of the input value, \a Val.`。
- **L535 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L535 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L536 EN**: Adds a standalone statement or declaration: `}];`.
  **L536 CN**: 添加一条独立语句或声明：`}];`。
- **L537 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L537 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L538 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L538 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L539 EN**: Closes the current lexical scope or compound statement.
  **L539 CN**: 结束当前词法作用域或复合语句块。
- **L540 EN**: Blank line separating nearby declarations or logic blocks.
  **L540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L541 EN**: Comment explains nearby logic, constraints, or intent: `Returns the number of bits (per component) set in the input integer.`.
  **L541 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the number of bits (per component) set in the input integer.`。
- **L542 EN**: Declares TableGen def record `hlsl_countbits`.
  **L542 CN**: 声明 TableGen def 记录 `hlsl_countbits`。
- **L543 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L543 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L544 EN**: Continues logic associated with callable symbol `countbits`.
  **L544 CN**: 继续与可调用符号 `countbits` 相关的逻辑。
- **L545 EN**: Continues logic associated with callable symbol `bits`.
  **L545 CN**: 继续与可调用符号 `bits` 相关的逻辑。
- **L546 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L546 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L547 EN**: Adds a standalone statement or declaration: `}];`.
  **L547 CN**: 添加一条独立语句或声明：`}];`。
- **L548 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["x"];`.
  **L548 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["x"];`。
- **L549 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Body = "return __builtin_elementwise_popcount(x);";`.
  **L549 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Body = "return __builtin_elementwise_popcount(x);";`。
- **L550 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L550 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L551 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<UIntTy>;`.
  **L551 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<UIntTy>;`。
- **L552 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllIntTypes;`.
  **L552 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllIntTypes;`。

### Lines 553-576

````tablegen
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
}

// Returns the cross product of two floating-point, 3D vectors.
def hlsl_cross_float : HLSLBuiltin<"cross", "__builtin_hlsl_crossf32"> {
  let Args = [VectorType<FloatTy, 3>, VectorType<FloatTy, 3>];
  let ReturnType = VectorType<FloatTy, 3>;
}

def hlsl_cross_half : HLSLBuiltin<"cross", "__builtin_hlsl_crossf16"> {
  let Doc = [{
\fn T cross(T x, T y)
\brief Returns the cross product of two floating-point, 3D vectors.
\param x [in] The first floating-point, 3D vector.
\param y [in] The second floating-point, 3D vector.

Result is the cross product of x and y, i.e., the resulting
components are, in order :
x[1] * y[2] - y[1] * x[2]
x[2] * y[0] - y[2] * x[0]
x[0] * y[1] - y[0] * x[1]
}];
  let Args = [VectorType<HalfTy, 3>, VectorType<HalfTy, 3>];
````
- **L553 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L553 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L554 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L554 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L555 EN**: Closes the current lexical scope or compound statement.
  **L555 CN**: 结束当前词法作用域或复合语句块。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L557 EN**: Comment explains nearby logic, constraints, or intent: `Returns the cross product of two floating-point, 3D vectors.`.
  **L557 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the cross product of two floating-point, 3D vectors.`。
- **L558 EN**: Declares TableGen def record `hlsl_cross_float`.
  **L558 CN**: 声明 TableGen def 记录 `hlsl_cross_float`。
- **L559 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [VectorType<FloatTy, 3>, VectorType<FloatTy, 3>];`.
  **L559 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [VectorType<FloatTy, 3>, VectorType<FloatTy, 3>];`。
- **L560 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VectorType<FloatTy, 3>;`.
  **L560 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VectorType<FloatTy, 3>;`。
- **L561 EN**: Closes the current lexical scope or compound statement.
  **L561 CN**: 结束当前词法作用域或复合语句块。
- **L562 EN**: Blank line separating nearby declarations or logic blocks.
  **L562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L563 EN**: Declares TableGen def record `hlsl_cross_half`.
  **L563 CN**: 声明 TableGen def 记录 `hlsl_cross_half`。
- **L564 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L564 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L565 EN**: Continues logic associated with callable symbol `cross`.
  **L565 CN**: 继续与可调用符号 `cross` 相关的逻辑。
- **L566 EN**: Continues the surrounding expression or declaration: `\brief Returns the cross product of two floating-point, 3D vectors.`.
  **L566 CN**: 继续构造周围的表达式或声明：`\brief Returns the cross product of two floating-point, 3D vectors.`。
- **L567 EN**: Continues the surrounding expression or declaration: `\param x [in] The first floating-point, 3D vector.`.
  **L567 CN**: 继续构造周围的表达式或声明：`\param x [in] The first floating-point, 3D vector.`。
- **L568 EN**: Continues the surrounding expression or declaration: `\param y [in] The second floating-point, 3D vector.`.
  **L568 CN**: 继续构造周围的表达式或声明：`\param y [in] The second floating-point, 3D vector.`。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L570 EN**: Continues the surrounding expression or declaration: `Result is the cross product of x and y, i.e., the resulting`.
  **L570 CN**: 继续构造周围的表达式或声明：`Result is the cross product of x and y, i.e., the resulting`。
- **L571 EN**: Continues the surrounding expression or declaration: `components are, in order :`.
  **L571 CN**: 继续构造周围的表达式或声明：`components are, in order :`。
- **L572 EN**: Continues the surrounding expression or declaration: `x[1] * y[2] - y[1] * x[2]`.
  **L572 CN**: 继续构造周围的表达式或声明：`x[1] * y[2] - y[1] * x[2]`。
- **L573 EN**: Continues the surrounding expression or declaration: `x[2] * y[0] - y[2] * x[0]`.
  **L573 CN**: 继续构造周围的表达式或声明：`x[2] * y[0] - y[2] * x[0]`。
- **L574 EN**: Continues the surrounding expression or declaration: `x[0] * y[1] - y[0] * x[1]`.
  **L574 CN**: 继续构造周围的表达式或声明：`x[0] * y[1] - y[0] * x[1]`。
- **L575 EN**: Adds a standalone statement or declaration: `}];`.
  **L575 CN**: 添加一条独立语句或声明：`}];`。
- **L576 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [VectorType<HalfTy, 3>, VectorType<HalfTy, 3>];`.
  **L576 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [VectorType<HalfTy, 3>, VectorType<HalfTy, 3>];`。

### Lines 577-600

````tablegen
  let ReturnType = VectorType<HalfTy, 3>;
  let Availability = SM6_2;
}

// Converts a floating-point, 4D vector set by a D3DCOLOR to a UBYTE4.
def hlsl_d3d_color_to_ubyte4 : HLSLBuiltin<"D3DCOLORtoUBYTE4"> {
  let Doc = [{
\fn int4 D3DCOLORtoUBYTE4(float4 x)
\brief Converts a floating-point, 4D vector set by a D3DCOLOR to a UBYTE4.
\param x [in] The floating-point vector4 to convert.

The return value is the UBYTE4 representation of the \a x parameter.

This function swizzles and scales components of the \a x parameter. Use this
function to compensate for the lack of UBYTE4 support in some hardware.
}];
  // Use the scaling factor used by FXC, and DXC for DXIL (i.e., 255.001953).
  // The [DXC implementation] refers to [stack overflow] to justify the scaling
  // factor:
  // > Built-in rounding, necessary because of truncation. 0.001953 * 256 = 0.5
  // 
  // [DXC implementation]: https://github.com/microsoft/DirectXShaderCompiler/blob/070d0d5a2beacef9eeb51037a9b04665716fd6f3/lib/HLSL/HLOperationLower.cpp#L666C1-L697C2
  // [stack overflow]: https://stackoverflow.com/questions/52103720/why-does-d3dcolortoubyte4-multiplies-components-by-255-001953f
  let Body = "return V.zyxw * 255.001953f;";
````
- **L577 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VectorType<HalfTy, 3>;`.
  **L577 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VectorType<HalfTy, 3>;`。
- **L578 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_2;`.
  **L578 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_2;`。
- **L579 EN**: Closes the current lexical scope or compound statement.
  **L579 CN**: 结束当前词法作用域或复合语句块。
- **L580 EN**: Blank line separating nearby declarations or logic blocks.
  **L580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L581 EN**: Comment explains nearby logic, constraints, or intent: `Converts a floating-point, 4D vector set by a D3DCOLOR to a UBYTE4.`.
  **L581 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converts a floating-point, 4D vector set by a D3DCOLOR to a UBYTE4.`。
- **L582 EN**: Declares TableGen def record `hlsl_d3d_color_to_ubyte4`.
  **L582 CN**: 声明 TableGen def 记录 `hlsl_d3d_color_to_ubyte4`。
- **L583 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L583 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L584 EN**: Continues logic associated with callable symbol `D3DCOLORtoUBYTE4`.
  **L584 CN**: 继续与可调用符号 `D3DCOLORtoUBYTE4` 相关的逻辑。
- **L585 EN**: Continues the surrounding expression or declaration: `\brief Converts a floating-point, 4D vector set by a D3DCOLOR to a UBYTE4.`.
  **L585 CN**: 继续构造周围的表达式或声明：`\brief Converts a floating-point, 4D vector set by a D3DCOLOR to a UBYTE4.`。
- **L586 EN**: Continues the surrounding expression or declaration: `\param x [in] The floating-point vector4 to convert.`.
  **L586 CN**: 继续构造周围的表达式或声明：`\param x [in] The floating-point vector4 to convert.`。
- **L587 EN**: Blank line separating nearby declarations or logic blocks.
  **L587 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L588 EN**: Continues the surrounding expression or declaration: `The return value is the UBYTE4 representation of the \a x parameter.`.
  **L588 CN**: 继续构造周围的表达式或声明：`The return value is the UBYTE4 representation of the \a x parameter.`。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L590 EN**: Continues the surrounding expression or declaration: `This function swizzles and scales components of the \a x parameter. Use this`.
  **L590 CN**: 继续构造周围的表达式或声明：`This function swizzles and scales components of the \a x parameter. Use this`。
- **L591 EN**: Continues the surrounding expression or declaration: `function to compensate for the lack of UBYTE4 support in some hardware.`.
  **L591 CN**: 继续构造周围的表达式或声明：`function to compensate for the lack of UBYTE4 support in some hardware.`。
- **L592 EN**: Adds a standalone statement or declaration: `}];`.
  **L592 CN**: 添加一条独立语句或声明：`}];`。
- **L593 EN**: Comment explains nearby logic, constraints, or intent: `Use the scaling factor used by FXC, and DXC for DXIL (i.e., 255.001953).`.
  **L593 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use the scaling factor used by FXC, and DXC for DXIL (i.e., 255.001953).`。
- **L594 EN**: Comment explains nearby logic, constraints, or intent: `The [DXC implementation] refers to [stack overflow] to justify the scaling`.
  **L594 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The [DXC implementation] refers to [stack overflow] to justify the scaling`。
- **L595 EN**: Comment explains nearby logic, constraints, or intent: `factor:`.
  **L595 CN**: 注释解释附近代码的逻辑、约束或设计意图：`factor:`。
- **L596 EN**: Comment explains nearby logic, constraints, or intent: `> Built-in rounding, necessary because of truncation. 0.001953 * 256 0.5`.
  **L596 CN**: 注释解释附近代码的逻辑、约束或设计意图：`> Built-in rounding, necessary because of truncation. 0.001953 * 256 0.5`。
- **L597 EN**: Separator comment used for visual grouping.
  **L597 CN**: 用于视觉分组的分隔注释。
- **L598 EN**: Comment explains nearby logic, constraints, or intent: `[DXC implementation]: https://github.com/microsoft/DirectXShaderCompiler/blob/070d0d5a2beacef9eeb51037a9b04665716fd6f3/lib/HLSL/HLOperationLower.cpp#L666C1-L697C2`.
  **L598 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[DXC implementation]: https://github.com/microsoft/DirectXShaderCompiler/blob/070d0d5a2beacef9eeb51037a9b04665716fd6f3/lib/HLSL/HLOperationLower.cpp#L666C1-L697C2`。
- **L599 EN**: Comment explains nearby logic, constraints, or intent: `[stack overflow]: https://stackoverflow.com/questions/52103720/why-does-d3dcolortoubyte4-multiplies-components-by-255-001953f`.
  **L599 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[stack overflow]: https://stackoverflow.com/questions/52103720/why-does-d3dcolortoubyte4-multiplies-components-by-255-001953f`。
- **L600 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Body = "return V.zyxw * 255.001953f;";`.
  **L600 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Body = "return V.zyxw * 255.001953f;";`。

### Lines 601-624

````tablegen
  let ParamNames = ["V"];
  let Args = [VectorType<FloatTy, 4>];
  let ReturnType = VectorType<IntTy, 4>;
  let IsConstexpr = 1;
}

// Computes the partial derivative with regard to the x screen space coordinate.
def hlsl_ddx : HLSLOneArgDetail<"ddx", "ddx_impl"> {
  let Doc = [{
\fn T ddx(T x)
\brief Computes the partial derivative of the specified value with regard to
the screen-space x-coordinate.
\param x [in] The floating-point scalar or vector to process.
}];
  let ParamNames = ["input"];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Computes a low precision partial derivative with respect to the screen-space
// x-coordinate.
def hlsl_ddx_coarse : HLSLOneArgBuiltin<"ddx_coarse",
                                    "__builtin_hlsl_elementwise_ddx_coarse"> {
  let Doc = [{
````
- **L601 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["V"];`.
  **L601 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["V"];`。
- **L602 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [VectorType<FloatTy, 4>];`.
  **L602 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [VectorType<FloatTy, 4>];`。
- **L603 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VectorType<IntTy, 4>;`.
  **L603 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VectorType<IntTy, 4>;`。
- **L604 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConstexpr = 1;`.
  **L604 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConstexpr = 1;`。
- **L605 EN**: Closes the current lexical scope or compound statement.
  **L605 CN**: 结束当前词法作用域或复合语句块。
- **L606 EN**: Blank line separating nearby declarations or logic blocks.
  **L606 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L607 EN**: Comment explains nearby logic, constraints, or intent: `Computes the partial derivative with regard to the x screen space coordinate.`.
  **L607 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes the partial derivative with regard to the x screen space coordinate.`。
- **L608 EN**: Declares TableGen def record `hlsl_ddx`.
  **L608 CN**: 声明 TableGen def 记录 `hlsl_ddx`。
- **L609 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L609 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L610 EN**: Continues logic associated with callable symbol `ddx`.
  **L610 CN**: 继续与可调用符号 `ddx` 相关的逻辑。
- **L611 EN**: Continues the surrounding expression or declaration: `\brief Computes the partial derivative of the specified value with regard to`.
  **L611 CN**: 继续构造周围的表达式或声明：`\brief Computes the partial derivative of the specified value with regard to`。
- **L612 EN**: Continues the surrounding expression or declaration: `the screen-space x-coordinate.`.
  **L612 CN**: 继续构造周围的表达式或声明：`the screen-space x-coordinate.`。
- **L613 EN**: Continues the surrounding expression or declaration: `\param x [in] The floating-point scalar or vector to process.`.
  **L613 CN**: 继续构造周围的表达式或声明：`\param x [in] The floating-point scalar or vector to process.`。
- **L614 EN**: Adds a standalone statement or declaration: `}];`.
  **L614 CN**: 添加一条独立语句或声明：`}];`。
- **L615 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["input"];`.
  **L615 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["input"];`。
- **L616 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L616 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L617 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L617 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L618 EN**: Closes the current lexical scope or compound statement.
  **L618 CN**: 结束当前词法作用域或复合语句块。
- **L619 EN**: Blank line separating nearby declarations or logic blocks.
  **L619 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L620 EN**: Comment explains nearby logic, constraints, or intent: `Computes a low precision partial derivative with respect to the screen-space`.
  **L620 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a low precision partial derivative with respect to the screen-space`。
- **L621 EN**: Comment explains nearby logic, constraints, or intent: `x-coordinate.`.
  **L621 CN**: 注释解释附近代码的逻辑、约束或设计意图：`x-coordinate.`。
- **L622 EN**: Declares TableGen def record `hlsl_ddx_coarse`.
  **L622 CN**: 声明 TableGen def 记录 `hlsl_ddx_coarse`。
- **L623 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_elementwise_ddx_coarse"> {`.
  **L623 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_elementwise_ddx_coarse"> {`。
- **L624 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L624 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。

### Lines 625-648

````tablegen
\fn T ddx_coarse(T value)
\brief Computes a low precision partial derivative with respect to the
screen-space x-coordinate.
\param value The input value.

The return value is a floating point scalar or vector containing the low
prevision partial derivative of the input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Computes a high precision partial derivative with respect to the screen-space
// x-coordinate.
def hlsl_ddx_fine : HLSLOneArgBuiltin<"ddx_fine",
                                  "__builtin_hlsl_elementwise_ddx_fine"> {
  let Doc = [{
\fn T ddx_fine(T value)
\brief Computes a high precision partial derivative with respect to the
screen-space x-coordinate.
\param value The input value.

The return value is a floating point scalar or vector containing the high
prevision partial derivative of the input value.
````
- **L625 EN**: Continues logic associated with callable symbol `ddx_coarse`.
  **L625 CN**: 继续与可调用符号 `ddx_coarse` 相关的逻辑。
- **L626 EN**: Continues the surrounding expression or declaration: `\brief Computes a low precision partial derivative with respect to the`.
  **L626 CN**: 继续构造周围的表达式或声明：`\brief Computes a low precision partial derivative with respect to the`。
- **L627 EN**: Continues the surrounding expression or declaration: `screen-space x-coordinate.`.
  **L627 CN**: 继续构造周围的表达式或声明：`screen-space x-coordinate.`。
- **L628 EN**: Continues the surrounding expression or declaration: `\param value The input value.`.
  **L628 CN**: 继续构造周围的表达式或声明：`\param value The input value.`。
- **L629 EN**: Blank line separating nearby declarations or logic blocks.
  **L629 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L630 EN**: Continues the surrounding expression or declaration: `The return value is a floating point scalar or vector containing the low`.
  **L630 CN**: 继续构造周围的表达式或声明：`The return value is a floating point scalar or vector containing the low`。
- **L631 EN**: Continues the surrounding expression or declaration: `prevision partial derivative of the input value.`.
  **L631 CN**: 继续构造周围的表达式或声明：`prevision partial derivative of the input value.`。
- **L632 EN**: Adds a standalone statement or declaration: `}];`.
  **L632 CN**: 添加一条独立语句或声明：`}];`。
- **L633 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L633 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L634 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L634 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L635 EN**: Closes the current lexical scope or compound statement.
  **L635 CN**: 结束当前词法作用域或复合语句块。
- **L636 EN**: Blank line separating nearby declarations or logic blocks.
  **L636 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L637 EN**: Comment explains nearby logic, constraints, or intent: `Computes a high precision partial derivative with respect to the screen-space`.
  **L637 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a high precision partial derivative with respect to the screen-space`。
- **L638 EN**: Comment explains nearby logic, constraints, or intent: `x-coordinate.`.
  **L638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`x-coordinate.`。
- **L639 EN**: Declares TableGen def record `hlsl_ddx_fine`.
  **L639 CN**: 声明 TableGen def 记录 `hlsl_ddx_fine`。
- **L640 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_elementwise_ddx_fine"> {`.
  **L640 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_elementwise_ddx_fine"> {`。
- **L641 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L641 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L642 EN**: Continues logic associated with callable symbol `ddx_fine`.
  **L642 CN**: 继续与可调用符号 `ddx_fine` 相关的逻辑。
- **L643 EN**: Continues the surrounding expression or declaration: `\brief Computes a high precision partial derivative with respect to the`.
  **L643 CN**: 继续构造周围的表达式或声明：`\brief Computes a high precision partial derivative with respect to the`。
- **L644 EN**: Continues the surrounding expression or declaration: `screen-space x-coordinate.`.
  **L644 CN**: 继续构造周围的表达式或声明：`screen-space x-coordinate.`。
- **L645 EN**: Continues the surrounding expression or declaration: `\param value The input value.`.
  **L645 CN**: 继续构造周围的表达式或声明：`\param value The input value.`。
- **L646 EN**: Blank line separating nearby declarations or logic blocks.
  **L646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L647 EN**: Continues the surrounding expression or declaration: `The return value is a floating point scalar or vector containing the high`.
  **L647 CN**: 继续构造周围的表达式或声明：`The return value is a floating point scalar or vector containing the high`。
- **L648 EN**: Continues the surrounding expression or declaration: `prevision partial derivative of the input value.`.
  **L648 CN**: 继续构造周围的表达式或声明：`prevision partial derivative of the input value.`。

### Lines 649-672

````tablegen
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Computes the partial derivative with regard to the y screen space coordinate.
def hlsl_ddy : HLSLOneArgDetail<"ddy", "ddy_impl"> {
  let Doc = [{
\fn T ddy(T x)
\brief Computes the partial derivative of the specified value with regard to
the screen-space y-coordinate.
\param x [in] The floating-point scalar or vector to process.
}];
  let ParamNames = ["input"];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Computes a low precision partial derivative with respect to the screen-space
// y-coordinate.
def hlsl_ddy_coarse : HLSLOneArgBuiltin<"ddy_coarse",
                                    "__builtin_hlsl_elementwise_ddy_coarse"> {
  let Doc = [{
\fn T ddy_coarse(T value)
````
- **L649 EN**: Adds a standalone statement or declaration: `}];`.
  **L649 CN**: 添加一条独立语句或声明：`}];`。
- **L650 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L650 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L651 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L651 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L652 EN**: Closes the current lexical scope or compound statement.
  **L652 CN**: 结束当前词法作用域或复合语句块。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Comment explains nearby logic, constraints, or intent: `Computes the partial derivative with regard to the y screen space coordinate.`.
  **L654 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes the partial derivative with regard to the y screen space coordinate.`。
- **L655 EN**: Declares TableGen def record `hlsl_ddy`.
  **L655 CN**: 声明 TableGen def 记录 `hlsl_ddy`。
- **L656 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L656 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L657 EN**: Continues logic associated with callable symbol `ddy`.
  **L657 CN**: 继续与可调用符号 `ddy` 相关的逻辑。
- **L658 EN**: Continues the surrounding expression or declaration: `\brief Computes the partial derivative of the specified value with regard to`.
  **L658 CN**: 继续构造周围的表达式或声明：`\brief Computes the partial derivative of the specified value with regard to`。
- **L659 EN**: Continues the surrounding expression or declaration: `the screen-space y-coordinate.`.
  **L659 CN**: 继续构造周围的表达式或声明：`the screen-space y-coordinate.`。
- **L660 EN**: Continues the surrounding expression or declaration: `\param x [in] The floating-point scalar or vector to process.`.
  **L660 CN**: 继续构造周围的表达式或声明：`\param x [in] The floating-point scalar or vector to process.`。
- **L661 EN**: Adds a standalone statement or declaration: `}];`.
  **L661 CN**: 添加一条独立语句或声明：`}];`。
- **L662 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["input"];`.
  **L662 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["input"];`。
- **L663 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L663 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L664 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L664 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L665 EN**: Closes the current lexical scope or compound statement.
  **L665 CN**: 结束当前词法作用域或复合语句块。
- **L666 EN**: Blank line separating nearby declarations or logic blocks.
  **L666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L667 EN**: Comment explains nearby logic, constraints, or intent: `Computes a low precision partial derivative with respect to the screen-space`.
  **L667 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a low precision partial derivative with respect to the screen-space`。
- **L668 EN**: Comment explains nearby logic, constraints, or intent: `y-coordinate.`.
  **L668 CN**: 注释解释附近代码的逻辑、约束或设计意图：`y-coordinate.`。
- **L669 EN**: Declares TableGen def record `hlsl_ddy_coarse`.
  **L669 CN**: 声明 TableGen def 记录 `hlsl_ddy_coarse`。
- **L670 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_elementwise_ddy_coarse"> {`.
  **L670 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_elementwise_ddy_coarse"> {`。
- **L671 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L671 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L672 EN**: Continues logic associated with callable symbol `ddy_coarse`.
  **L672 CN**: 继续与可调用符号 `ddy_coarse` 相关的逻辑。

### Lines 673-696

````tablegen
\brief Computes a low precision partial derivative with respect to the
screen-space y-coordinate.
\param value The input value.

The return value is a floating point scalar or vector containing the low
prevision partial derivative of the input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Computes a high precision partial derivative with respect to the screen-space
// y-coordinate.
def hlsl_ddy_fine : HLSLOneArgBuiltin<"ddy_fine",
                                  "__builtin_hlsl_elementwise_ddy_fine"> {
  let Doc = [{
\fn T ddy_fine(T value)
\brief Computes a high precision partial derivative with respect to the
screen-space y-coordinate.
\param value The input value.

The return value is a floating point scalar or vector containing the high
prevision partial derivative of the input value.
}];
````
- **L673 EN**: Continues the surrounding expression or declaration: `\brief Computes a low precision partial derivative with respect to the`.
  **L673 CN**: 继续构造周围的表达式或声明：`\brief Computes a low precision partial derivative with respect to the`。
- **L674 EN**: Continues the surrounding expression or declaration: `screen-space y-coordinate.`.
  **L674 CN**: 继续构造周围的表达式或声明：`screen-space y-coordinate.`。
- **L675 EN**: Continues the surrounding expression or declaration: `\param value The input value.`.
  **L675 CN**: 继续构造周围的表达式或声明：`\param value The input value.`。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L677 EN**: Continues the surrounding expression or declaration: `The return value is a floating point scalar or vector containing the low`.
  **L677 CN**: 继续构造周围的表达式或声明：`The return value is a floating point scalar or vector containing the low`。
- **L678 EN**: Continues the surrounding expression or declaration: `prevision partial derivative of the input value.`.
  **L678 CN**: 继续构造周围的表达式或声明：`prevision partial derivative of the input value.`。
- **L679 EN**: Adds a standalone statement or declaration: `}];`.
  **L679 CN**: 添加一条独立语句或声明：`}];`。
- **L680 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L680 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L681 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L681 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L682 EN**: Closes the current lexical scope or compound statement.
  **L682 CN**: 结束当前词法作用域或复合语句块。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L684 EN**: Comment explains nearby logic, constraints, or intent: `Computes a high precision partial derivative with respect to the screen-space`.
  **L684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a high precision partial derivative with respect to the screen-space`。
- **L685 EN**: Comment explains nearby logic, constraints, or intent: `y-coordinate.`.
  **L685 CN**: 注释解释附近代码的逻辑、约束或设计意图：`y-coordinate.`。
- **L686 EN**: Declares TableGen def record `hlsl_ddy_fine`.
  **L686 CN**: 声明 TableGen def 记录 `hlsl_ddy_fine`。
- **L687 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_elementwise_ddy_fine"> {`.
  **L687 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_elementwise_ddy_fine"> {`。
- **L688 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L688 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L689 EN**: Continues logic associated with callable symbol `ddy_fine`.
  **L689 CN**: 继续与可调用符号 `ddy_fine` 相关的逻辑。
- **L690 EN**: Continues the surrounding expression or declaration: `\brief Computes a high precision partial derivative with respect to the`.
  **L690 CN**: 继续构造周围的表达式或声明：`\brief Computes a high precision partial derivative with respect to the`。
- **L691 EN**: Continues the surrounding expression or declaration: `screen-space y-coordinate.`.
  **L691 CN**: 继续构造周围的表达式或声明：`screen-space y-coordinate.`。
- **L692 EN**: Continues the surrounding expression or declaration: `\param value The input value.`.
  **L692 CN**: 继续构造周围的表达式或声明：`\param value The input value.`。
- **L693 EN**: Blank line separating nearby declarations or logic blocks.
  **L693 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L694 EN**: Continues the surrounding expression or declaration: `The return value is a floating point scalar or vector containing the high`.
  **L694 CN**: 继续构造周围的表达式或声明：`The return value is a floating point scalar or vector containing the high`。
- **L695 EN**: Continues the surrounding expression or declaration: `prevision partial derivative of the input value.`.
  **L695 CN**: 继续构造周围的表达式或声明：`prevision partial derivative of the input value.`。
- **L696 EN**: Adds a standalone statement or declaration: `}];`.
  **L696 CN**: 添加一条独立语句或声明：`}];`。

### Lines 697-720

````tablegen
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Converts the specified value from radians to degrees.
def hlsl_degrees : HLSLOneArgBuiltin<"degrees", "__builtin_hlsl_elementwise_degrees"> {
  let Doc = [{
\fn T degrees(T x)
\brief Converts the specified value from radians to degrees.
\param x The specified input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns a distance scalar between X and Y.
// The distance between X and Y is length(X - Y).
def hlsl_distance : HLSLBuiltin<"distance"> {
  let Doc = [{
\fn K distance(T X, T Y)
\brief Returns a distance scalar between \a X and \a Y.
\param X The X input value.
\param Y The Y input value.
}];
````
- **L697 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L697 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L698 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L698 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L699 EN**: Closes the current lexical scope or compound statement.
  **L699 CN**: 结束当前词法作用域或复合语句块。
- **L700 EN**: Blank line separating nearby declarations or logic blocks.
  **L700 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L701 EN**: Comment explains nearby logic, constraints, or intent: `Converts the specified value from radians to degrees.`.
  **L701 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converts the specified value from radians to degrees.`。
- **L702 EN**: Declares TableGen def record `hlsl_degrees`.
  **L702 CN**: 声明 TableGen def 记录 `hlsl_degrees`。
- **L703 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L703 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L704 EN**: Continues logic associated with callable symbol `degrees`.
  **L704 CN**: 继续与可调用符号 `degrees` 相关的逻辑。
- **L705 EN**: Continues the surrounding expression or declaration: `\brief Converts the specified value from radians to degrees.`.
  **L705 CN**: 继续构造周围的表达式或声明：`\brief Converts the specified value from radians to degrees.`。
- **L706 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L706 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L707 EN**: Adds a standalone statement or declaration: `}];`.
  **L707 CN**: 添加一条独立语句或声明：`}];`。
- **L708 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L708 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L709 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L709 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L710 EN**: Closes the current lexical scope or compound statement.
  **L710 CN**: 结束当前词法作用域或复合语句块。
- **L711 EN**: Blank line separating nearby declarations or logic blocks.
  **L711 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L712 EN**: Comment explains nearby logic, constraints, or intent: `Returns a distance scalar between X and Y.`.
  **L712 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a distance scalar between X and Y.`。
- **L713 EN**: Comment explains nearby logic, constraints, or intent: `The distance between X and Y is length(X - Y).`.
  **L713 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The distance between X and Y is length(X - Y).`。
- **L714 EN**: Declares TableGen def record `hlsl_distance`.
  **L714 CN**: 声明 TableGen def 记录 `hlsl_distance`。
- **L715 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L715 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L716 EN**: Continues logic associated with callable symbol `distance`.
  **L716 CN**: 继续与可调用符号 `distance` 相关的逻辑。
- **L717 EN**: Continues the surrounding expression or declaration: `\brief Returns a distance scalar between \a X and \a Y.`.
  **L717 CN**: 继续构造周围的表达式或声明：`\brief Returns a distance scalar between \a X and \a Y.`。
- **L718 EN**: Continues the surrounding expression or declaration: `\param X The X input value.`.
  **L718 CN**: 继续构造周围的表达式或声明：`\param X The X input value.`。
- **L719 EN**: Continues the surrounding expression or declaration: `\param Y The Y input value.`.
  **L719 CN**: 继续构造周围的表达式或声明：`\param Y The Y input value.`。
- **L720 EN**: Adds a standalone statement or declaration: `}];`.
  **L720 CN**: 添加一条独立语句或声明：`}];`。

### Lines 721-744

````tablegen
  let Body = "return __detail::length_impl(X - Y);";
  let ParamNames = ["X", "Y"];
  let Args = [Varying, Varying];
  let ReturnType = VaryingElemType;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = [];
}

// Returns the dot product (a scalar value) of X and Y.
def hlsl_dot : HLSLTwoArgBuiltin<"dot", "__builtin_hlsl_dot"> {
  let Doc = [{
\fn K dot(T X, T Y)
\brief Return the dot product (a scalar value) of \a X and \a Y.
\param X The X input value.
\param Y The Y input value.
}];
  let ReturnType = VaryingElemType;
  let VaryingTypes = NumericTypesNoDbl;
  let VaryingMatDims = [];
}

// double dot only has scalar overload (no vectors).
````
- **L721 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Body = "return __detail::length_impl(X - Y);";`.
  **L721 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Body = "return __detail::length_impl(X - Y);";`。
- **L722 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["X", "Y"];`.
  **L722 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["X", "Y"];`。
- **L723 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L723 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L724 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingElemType;`.
  **L724 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingElemType;`。
- **L725 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L725 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L726 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L726 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L727 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L727 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L728 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L728 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L729 EN**: Closes the current lexical scope or compound statement.
  **L729 CN**: 结束当前词法作用域或复合语句块。
- **L730 EN**: Blank line separating nearby declarations or logic blocks.
  **L730 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L731 EN**: Comment explains nearby logic, constraints, or intent: `Returns the dot product (a scalar value) of X and Y.`.
  **L731 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the dot product (a scalar value) of X and Y.`。
- **L732 EN**: Declares TableGen def record `hlsl_dot`.
  **L732 CN**: 声明 TableGen def 记录 `hlsl_dot`。
- **L733 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L733 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L734 EN**: Continues logic associated with callable symbol `dot`.
  **L734 CN**: 继续与可调用符号 `dot` 相关的逻辑。
- **L735 EN**: Continues logic associated with callable symbol `product`.
  **L735 CN**: 继续与可调用符号 `product` 相关的逻辑。
- **L736 EN**: Continues the surrounding expression or declaration: `\param X The X input value.`.
  **L736 CN**: 继续构造周围的表达式或声明：`\param X The X input value.`。
- **L737 EN**: Continues the surrounding expression or declaration: `\param Y The Y input value.`.
  **L737 CN**: 继续构造周围的表达式或声明：`\param Y The Y input value.`。
- **L738 EN**: Adds a standalone statement or declaration: `}];`.
  **L738 CN**: 添加一条独立语句或声明：`}];`。
- **L739 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingElemType;`.
  **L739 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingElemType;`。
- **L740 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = NumericTypesNoDbl;`.
  **L740 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = NumericTypesNoDbl;`。
- **L741 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L741 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L742 EN**: Closes the current lexical scope or compound statement.
  **L742 CN**: 结束当前词法作用域或复合语句块。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L744 EN**: Comment explains nearby logic, constraints, or intent: `double dot only has scalar overload (no vectors).`.
  **L744 CN**: 注释解释附近代码的逻辑、约束或设计意图：`double dot only has scalar overload (no vectors).`。

### Lines 745-768

````tablegen
def hlsl_dot_double : HLSLBuiltin<"dot", "__builtin_hlsl_dot"> {
  let Args = [Varying, Varying];
  let ReturnType = VaryingElemType;
  let VaryingTypes = [DoubleTy];
  let VaryingScalar = 1;
}

// Dot product of 2 half vectors plus a float scalar.
def hlsl_dot2add : HLSLBuiltin<"dot2add"> {
  let Doc = [{
\fn float dot2add(half2 A, half2 B, float Acc)
\brief Multiplies the elements of the two half-precision float input vectors
together and sums the results into the 32-bit float accumulator.
\param A The first input value to dot product.
\param B The second input value to dot product.
\param Acc The accumulator value added to the dot product.

This instruction operates within a single 32-bit wide SIMD lane.
The inputs are 16-bit quantities packed into the same lane.
}];
  let DetailFunc = "dot2add_impl";
  let ParamNames = ["A", "B", "Acc"];
  let Args = [VectorType<HalfTy, 2>, VectorType<HalfTy, 2>, FloatTy];
  let ReturnType = FloatTy;
````
- **L745 EN**: Declares TableGen def record `hlsl_dot_double`.
  **L745 CN**: 声明 TableGen def 记录 `hlsl_dot_double`。
- **L746 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L746 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L747 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingElemType;`.
  **L747 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingElemType;`。
- **L748 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [DoubleTy];`.
  **L748 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [DoubleTy];`。
- **L749 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L749 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L750 EN**: Closes the current lexical scope or compound statement.
  **L750 CN**: 结束当前词法作用域或复合语句块。
- **L751 EN**: Blank line separating nearby declarations or logic blocks.
  **L751 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L752 EN**: Comment explains nearby logic, constraints, or intent: `Dot product of 2 half vectors plus a float scalar.`.
  **L752 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Dot product of 2 half vectors plus a float scalar.`。
- **L753 EN**: Declares TableGen def record `hlsl_dot2add`.
  **L753 CN**: 声明 TableGen def 记录 `hlsl_dot2add`。
- **L754 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L754 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L755 EN**: Continues logic associated with callable symbol `dot2add`.
  **L755 CN**: 继续与可调用符号 `dot2add` 相关的逻辑。
- **L756 EN**: Continues the surrounding expression or declaration: `\brief Multiplies the elements of the two half-precision float input vectors`.
  **L756 CN**: 继续构造周围的表达式或声明：`\brief Multiplies the elements of the two half-precision float input vectors`。
- **L757 EN**: Continues the surrounding expression or declaration: `together and sums the results into the 32-bit float accumulator.`.
  **L757 CN**: 继续构造周围的表达式或声明：`together and sums the results into the 32-bit float accumulator.`。
- **L758 EN**: Continues the surrounding expression or declaration: `\param A The first input value to dot product.`.
  **L758 CN**: 继续构造周围的表达式或声明：`\param A The first input value to dot product.`。
- **L759 EN**: Continues the surrounding expression or declaration: `\param B The second input value to dot product.`.
  **L759 CN**: 继续构造周围的表达式或声明：`\param B The second input value to dot product.`。
- **L760 EN**: Continues the surrounding expression or declaration: `\param Acc The accumulator value added to the dot product.`.
  **L760 CN**: 继续构造周围的表达式或声明：`\param Acc The accumulator value added to the dot product.`。
- **L761 EN**: Blank line separating nearby declarations or logic blocks.
  **L761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L762 EN**: Continues the surrounding expression or declaration: `This instruction operates within a single 32-bit wide SIMD lane.`.
  **L762 CN**: 继续构造周围的表达式或声明：`This instruction operates within a single 32-bit wide SIMD lane.`。
- **L763 EN**: Continues the surrounding expression or declaration: `The inputs are 16-bit quantities packed into the same lane.`.
  **L763 CN**: 继续构造周围的表达式或声明：`The inputs are 16-bit quantities packed into the same lane.`。
- **L764 EN**: Adds a standalone statement or declaration: `}];`.
  **L764 CN**: 添加一条独立语句或声明：`}];`。
- **L765 EN**: Assigns a TableGen property that affects following records or inherited fields: `let DetailFunc = "dot2add_impl";`.
  **L765 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let DetailFunc = "dot2add_impl";`。
- **L766 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["A", "B", "Acc"];`.
  **L766 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["A", "B", "Acc"];`。
- **L767 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [VectorType<HalfTy, 2>, VectorType<HalfTy, 2>, FloatTy];`.
  **L767 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [VectorType<HalfTy, 2>, VectorType<HalfTy, 2>, FloatTy];`。
- **L768 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = FloatTy;`.
  **L768 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = FloatTy;`。

### Lines 769-792

````tablegen
  let Availability = SM6_4;
}

// Dot product of two packed 8-bit signed integer vectors, accumulated with a
// 32-bit signed integer.
def hlsl_dot4add_i8packed :
    HLSLBuiltin<"dot4add_i8packed", "__builtin_hlsl_dot4add_i8packed"> {
  let Doc = [{
\fn int dot4add_i8packed(uint A, uint B, int C)
\brief Dot product of two packed 8-bit signed integer vectors, accumulated
with a 32-bit signed integer.
}];
  let Args = [UIntTy, UIntTy, IntTy];
  let ReturnType = IntTy;
  let Availability = SM6_4;
}

// Dot product of two packed 8-bit unsigned integer vectors, accumulated with a
// 32-bit unsigned integer.
def hlsl_dot4add_u8packed :
    HLSLBuiltin<"dot4add_u8packed", "__builtin_hlsl_dot4add_u8packed"> {
  let Doc = [{
\fn uint dot4add_u8packed(uint A, uint B, uint C)
\brief Dot product of two packed 8-bit unsigned integer vectors, accumulated
````
- **L769 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_4;`.
  **L769 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_4;`。
- **L770 EN**: Closes the current lexical scope or compound statement.
  **L770 CN**: 结束当前词法作用域或复合语句块。
- **L771 EN**: Blank line separating nearby declarations or logic blocks.
  **L771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L772 EN**: Comment explains nearby logic, constraints, or intent: `Dot product of two packed 8-bit signed integer vectors, accumulated with a`.
  **L772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Dot product of two packed 8-bit signed integer vectors, accumulated with a`。
- **L773 EN**: Comment explains nearby logic, constraints, or intent: `32-bit signed integer.`.
  **L773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit signed integer.`。
- **L774 EN**: Declares TableGen def record `hlsl_dot4add_i8packed`.
  **L774 CN**: 声明 TableGen def 记录 `hlsl_dot4add_i8packed`。
- **L775 EN**: Continues the surrounding expression or declaration: `HLSLBuiltin<"dot4add_i8packed", "__builtin_hlsl_dot4add_i8packed"> {`.
  **L775 CN**: 继续构造周围的表达式或声明：`HLSLBuiltin<"dot4add_i8packed", "__builtin_hlsl_dot4add_i8packed"> {`。
- **L776 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L776 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L777 EN**: Continues logic associated with callable symbol `dot4add_i8packed`.
  **L777 CN**: 继续与可调用符号 `dot4add_i8packed` 相关的逻辑。
- **L778 EN**: Continues the surrounding expression or declaration: `\brief Dot product of two packed 8-bit signed integer vectors, accumulated`.
  **L778 CN**: 继续构造周围的表达式或声明：`\brief Dot product of two packed 8-bit signed integer vectors, accumulated`。
- **L779 EN**: Continues the surrounding expression or declaration: `with a 32-bit signed integer.`.
  **L779 CN**: 继续构造周围的表达式或声明：`with a 32-bit signed integer.`。
- **L780 EN**: Adds a standalone statement or declaration: `}];`.
  **L780 CN**: 添加一条独立语句或声明：`}];`。
- **L781 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [UIntTy, UIntTy, IntTy];`.
  **L781 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [UIntTy, UIntTy, IntTy];`。
- **L782 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = IntTy;`.
  **L782 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = IntTy;`。
- **L783 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_4;`.
  **L783 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_4;`。
- **L784 EN**: Closes the current lexical scope or compound statement.
  **L784 CN**: 结束当前词法作用域或复合语句块。
- **L785 EN**: Blank line separating nearby declarations or logic blocks.
  **L785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L786 EN**: Comment explains nearby logic, constraints, or intent: `Dot product of two packed 8-bit unsigned integer vectors, accumulated with a`.
  **L786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Dot product of two packed 8-bit unsigned integer vectors, accumulated with a`。
- **L787 EN**: Comment explains nearby logic, constraints, or intent: `32-bit unsigned integer.`.
  **L787 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit unsigned integer.`。
- **L788 EN**: Declares TableGen def record `hlsl_dot4add_u8packed`.
  **L788 CN**: 声明 TableGen def 记录 `hlsl_dot4add_u8packed`。
- **L789 EN**: Continues the surrounding expression or declaration: `HLSLBuiltin<"dot4add_u8packed", "__builtin_hlsl_dot4add_u8packed"> {`.
  **L789 CN**: 继续构造周围的表达式或声明：`HLSLBuiltin<"dot4add_u8packed", "__builtin_hlsl_dot4add_u8packed"> {`。
- **L790 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L790 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L791 EN**: Continues logic associated with callable symbol `dot4add_u8packed`.
  **L791 CN**: 继续与可调用符号 `dot4add_u8packed` 相关的逻辑。
- **L792 EN**: Continues the surrounding expression or declaration: `\brief Dot product of two packed 8-bit unsigned integer vectors, accumulated`.
  **L792 CN**: 继续构造周围的表达式或声明：`\brief Dot product of two packed 8-bit unsigned integer vectors, accumulated`。

### Lines 793-816

````tablegen
with a 32-bit unsigned integer.
}];
  let Args = [UIntTy, UIntTy, UIntTy];
  let ReturnType = UIntTy;
  let Availability = SM6_4;
}

// Calculates a distance vector.
def hlsl_dst : HLSLBuiltin<"dst"> {
  let Doc = [{
\fn vector<T, 4> dst(vector<T, 4> Src0, vector<T, 4> Src1)
\brief Calculates a distance vector.
\param Src0 [in] Contains the squared distance.
\param Src1 [in] Contains the reciprocal distance.

Return the computed distance vector.
}];
  let ParamNames = ["Src0", "Src1"];
  let Body = "return {1, Src0[1] * Src1[1], Src0[2], Src1[3]};";
  let Args = [Varying, Varying];
  let ReturnType = Varying;
  let VaryingTypes = AllFloatTypes;
  let VaryingVecSizes = [4];
  let VaryingMatDims = [];
````
- **L793 EN**: Continues the surrounding expression or declaration: `with a 32-bit unsigned integer.`.
  **L793 CN**: 继续构造周围的表达式或声明：`with a 32-bit unsigned integer.`。
- **L794 EN**: Adds a standalone statement or declaration: `}];`.
  **L794 CN**: 添加一条独立语句或声明：`}];`。
- **L795 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [UIntTy, UIntTy, UIntTy];`.
  **L795 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [UIntTy, UIntTy, UIntTy];`。
- **L796 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = UIntTy;`.
  **L796 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = UIntTy;`。
- **L797 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_4;`.
  **L797 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_4;`。
- **L798 EN**: Closes the current lexical scope or compound statement.
  **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L800 EN**: Comment explains nearby logic, constraints, or intent: `Calculates a distance vector.`.
  **L800 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Calculates a distance vector.`。
- **L801 EN**: Declares TableGen def record `hlsl_dst`.
  **L801 CN**: 声明 TableGen def 记录 `hlsl_dst`。
- **L802 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L802 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L803 EN**: Continues logic associated with callable symbol `dst`.
  **L803 CN**: 继续与可调用符号 `dst` 相关的逻辑。
- **L804 EN**: Continues the surrounding expression or declaration: `\brief Calculates a distance vector.`.
  **L804 CN**: 继续构造周围的表达式或声明：`\brief Calculates a distance vector.`。
- **L805 EN**: Continues the surrounding expression or declaration: `\param Src0 [in] Contains the squared distance.`.
  **L805 CN**: 继续构造周围的表达式或声明：`\param Src0 [in] Contains the squared distance.`。
- **L806 EN**: Continues the surrounding expression or declaration: `\param Src1 [in] Contains the reciprocal distance.`.
  **L806 CN**: 继续构造周围的表达式或声明：`\param Src1 [in] Contains the reciprocal distance.`。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L808 EN**: Continues the surrounding expression or declaration: `Return the computed distance vector.`.
  **L808 CN**: 继续构造周围的表达式或声明：`Return the computed distance vector.`。
- **L809 EN**: Adds a standalone statement or declaration: `}];`.
  **L809 CN**: 添加一条独立语句或声明：`}];`。
- **L810 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["Src0", "Src1"];`.
  **L810 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["Src0", "Src1"];`。
- **L811 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Body = "return {1, Src0[1] * Src1[1], Src0[2], Src1[3]};";`.
  **L811 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Body = "return {1, Src0[1] * Src1[1], Src0[2], Src1[3]};";`。
- **L812 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L812 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L813 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L813 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L814 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllFloatTypes;`.
  **L814 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllFloatTypes;`。
- **L815 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [4];`.
  **L815 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [4];`。
- **L816 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L816 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。

### Lines 817-840

````tablegen
}

// Returns the base-e exponential, or e**x, of the specified value.
def hlsl_exp : HLSLOneArgBuiltin<"exp", "__builtin_elementwise_exp"> {
  let Doc = [{
\fn T exp(T x)
\brief Returns the base-e exponential, or \a e**x, of the specified value.
\param x The specified input value.

The return value is the base-e exponential of the \a x parameter.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the base 2 exponential, or 2**x, of the specified value.
def hlsl_exp2 : HLSLOneArgBuiltin<"exp2", "__builtin_elementwise_exp2"> {
  let Doc = [{
\fn T exp2(T x)
\brief Returns the base 2 exponential, or \a 2**x, of the specified value.
\param x The specified input value.

The base 2 exponential of the \a x parameter.
}];
````
- **L817 EN**: Closes the current lexical scope or compound statement.
  **L817 CN**: 结束当前词法作用域或复合语句块。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L819 EN**: Comment explains nearby logic, constraints, or intent: `Returns the base-e exponential, or e**x, of the specified value.`.
  **L819 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the base-e exponential, or e**x, of the specified value.`。
- **L820 EN**: Declares TableGen def record `hlsl_exp`.
  **L820 CN**: 声明 TableGen def 记录 `hlsl_exp`。
- **L821 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L821 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L822 EN**: Continues logic associated with callable symbol `exp`.
  **L822 CN**: 继续与可调用符号 `exp` 相关的逻辑。
- **L823 EN**: Continues the surrounding expression or declaration: `\brief Returns the base-e exponential, or \a e**x, of the specified value.`.
  **L823 CN**: 继续构造周围的表达式或声明：`\brief Returns the base-e exponential, or \a e**x, of the specified value.`。
- **L824 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L824 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Continues the surrounding expression or declaration: `The return value is the base-e exponential of the \a x parameter.`.
  **L826 CN**: 继续构造周围的表达式或声明：`The return value is the base-e exponential of the \a x parameter.`。
- **L827 EN**: Adds a standalone statement or declaration: `}];`.
  **L827 CN**: 添加一条独立语句或声明：`}];`。
- **L828 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L828 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L829 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L829 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L830 EN**: Closes the current lexical scope or compound statement.
  **L830 CN**: 结束当前词法作用域或复合语句块。
- **L831 EN**: Blank line separating nearby declarations or logic blocks.
  **L831 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L832 EN**: Comment explains nearby logic, constraints, or intent: `Returns the base 2 exponential, or 2**x, of the specified value.`.
  **L832 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the base 2 exponential, or 2**x, of the specified value.`。
- **L833 EN**: Declares TableGen def record `hlsl_exp2`.
  **L833 CN**: 声明 TableGen def 记录 `hlsl_exp2`。
- **L834 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L834 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L835 EN**: Continues logic associated with callable symbol `exp2`.
  **L835 CN**: 继续与可调用符号 `exp2` 相关的逻辑。
- **L836 EN**: Continues the surrounding expression or declaration: `\brief Returns the base 2 exponential, or \a 2**x, of the specified value.`.
  **L836 CN**: 继续构造周围的表达式或声明：`\brief Returns the base 2 exponential, or \a 2**x, of the specified value.`。
- **L837 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L837 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L838 EN**: Blank line separating nearby declarations or logic blocks.
  **L838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L839 EN**: Continues the surrounding expression or declaration: `The base 2 exponential of the \a x parameter.`.
  **L839 CN**: 继续构造周围的表达式或声明：`The base 2 exponential of the \a x parameter.`。
- **L840 EN**: Adds a standalone statement or declaration: `}];`.
  **L840 CN**: 添加一条独立语句或声明：`}];`。

### Lines 841-864

````tablegen
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the half value stored in the low 16 bits of the uint arg converted
// to a float.
def hlsl_f16tof32 : HLSLOneArgBuiltin<"f16tof32",
    "__builtin_hlsl_elementwise_f16tof32"> {
  let Doc = [{
\fn float f16tof32(uint x)
\brief Returns the half value stored in the low 16 bits of the uint arg
converted to a float.
\param x The uint containing two half values.

The float value of the half value found in the low 16 bits of the \a x
parameter.
}];
  let ReturnType = VaryingShape<FloatTy>;
  let VaryingTypes = [UIntTy];
  let VaryingMatDims = [];
}

// Returns the float arg value converted to half in the low 16 bits of the uint
// return value.
````
- **L841 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L841 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L842 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L842 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L843 EN**: Closes the current lexical scope or compound statement.
  **L843 CN**: 结束当前词法作用域或复合语句块。
- **L844 EN**: Blank line separating nearby declarations or logic blocks.
  **L844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L845 EN**: Comment explains nearby logic, constraints, or intent: `Returns the half value stored in the low 16 bits of the uint arg converted`.
  **L845 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the half value stored in the low 16 bits of the uint arg converted`。
- **L846 EN**: Comment explains nearby logic, constraints, or intent: `to a float.`.
  **L846 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a float.`。
- **L847 EN**: Declares TableGen def record `hlsl_f16tof32`.
  **L847 CN**: 声明 TableGen def 记录 `hlsl_f16tof32`。
- **L848 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_elementwise_f16tof32"> {`.
  **L848 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_elementwise_f16tof32"> {`。
- **L849 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L849 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L850 EN**: Continues logic associated with callable symbol `f16tof32`.
  **L850 CN**: 继续与可调用符号 `f16tof32` 相关的逻辑。
- **L851 EN**: Continues the surrounding expression or declaration: `\brief Returns the half value stored in the low 16 bits of the uint arg`.
  **L851 CN**: 继续构造周围的表达式或声明：`\brief Returns the half value stored in the low 16 bits of the uint arg`。
- **L852 EN**: Continues the surrounding expression or declaration: `converted to a float.`.
  **L852 CN**: 继续构造周围的表达式或声明：`converted to a float.`。
- **L853 EN**: Continues the surrounding expression or declaration: `\param x The uint containing two half values.`.
  **L853 CN**: 继续构造周围的表达式或声明：`\param x The uint containing two half values.`。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L855 EN**: Continues the surrounding expression or declaration: `The float value of the half value found in the low 16 bits of the \a x`.
  **L855 CN**: 继续构造周围的表达式或声明：`The float value of the half value found in the low 16 bits of the \a x`。
- **L856 EN**: Continues the surrounding expression or declaration: `parameter.`.
  **L856 CN**: 继续构造周围的表达式或声明：`parameter.`。
- **L857 EN**: Adds a standalone statement or declaration: `}];`.
  **L857 CN**: 添加一条独立语句或声明：`}];`。
- **L858 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<FloatTy>;`.
  **L858 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<FloatTy>;`。
- **L859 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [UIntTy];`.
  **L859 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [UIntTy];`。
- **L860 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L860 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L861 EN**: Closes the current lexical scope or compound statement.
  **L861 CN**: 结束当前词法作用域或复合语句块。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L863 EN**: Comment explains nearby logic, constraints, or intent: `Returns the float arg value converted to half in the low 16 bits of the uint`.
  **L863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the float arg value converted to half in the low 16 bits of the uint`。
- **L864 EN**: Comment explains nearby logic, constraints, or intent: `return value.`.
  **L864 CN**: 注释解释附近代码的逻辑、约束或设计意图：`return value.`。

### Lines 865-888

````tablegen
def hlsl_f32tof16 : HLSLOneArgBuiltin<"f32tof16",
    "__builtin_hlsl_elementwise_f32tof16"> {
  let Doc = [{
\fn uint f32tof16(float x)
\brief Returns the float arg value converted to half in the low 16 bits of
the uint return value.
\param x The float to be converted to half.

The return value is a uint containing the converted half value in the low
16 bits.
}];
  let ReturnType = VaryingShape<UIntTy>;
  let VaryingTypes = [FloatTy];
  let VaryingMatDims = [];
}

// Flips the surface-normal to face in a direction opposite to I.
def hlsl_faceforward : HLSLThreeArgDetail<"faceforward", "faceforward_impl"> {
  let Doc = [{
\fn T faceforward(T N, T I, T Ng)
\brief Flips the surface-normal (if needed) to face in a direction opposite
to \a I. Returns the result in terms of \a N.
\param N The resulting floating-point surface-normal vector.
\param I A floating-point, incident vector that points from the view
````
- **L865 EN**: Declares TableGen def record `hlsl_f32tof16`.
  **L865 CN**: 声明 TableGen def 记录 `hlsl_f32tof16`。
- **L866 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_elementwise_f32tof16"> {`.
  **L866 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_elementwise_f32tof16"> {`。
- **L867 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L867 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L868 EN**: Continues logic associated with callable symbol `f32tof16`.
  **L868 CN**: 继续与可调用符号 `f32tof16` 相关的逻辑。
- **L869 EN**: Continues the surrounding expression or declaration: `\brief Returns the float arg value converted to half in the low 16 bits of`.
  **L869 CN**: 继续构造周围的表达式或声明：`\brief Returns the float arg value converted to half in the low 16 bits of`。
- **L870 EN**: Continues the surrounding expression or declaration: `the uint return value.`.
  **L870 CN**: 继续构造周围的表达式或声明：`the uint return value.`。
- **L871 EN**: Continues the surrounding expression or declaration: `\param x The float to be converted to half.`.
  **L871 CN**: 继续构造周围的表达式或声明：`\param x The float to be converted to half.`。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L873 EN**: Continues the surrounding expression or declaration: `The return value is a uint containing the converted half value in the low`.
  **L873 CN**: 继续构造周围的表达式或声明：`The return value is a uint containing the converted half value in the low`。
- **L874 EN**: Continues the surrounding expression or declaration: `16 bits.`.
  **L874 CN**: 继续构造周围的表达式或声明：`16 bits.`。
- **L875 EN**: Adds a standalone statement or declaration: `}];`.
  **L875 CN**: 添加一条独立语句或声明：`}];`。
- **L876 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<UIntTy>;`.
  **L876 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<UIntTy>;`。
- **L877 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [FloatTy];`.
  **L877 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [FloatTy];`。
- **L878 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L878 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L879 EN**: Closes the current lexical scope or compound statement.
  **L879 CN**: 结束当前词法作用域或复合语句块。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L881 EN**: Comment explains nearby logic, constraints, or intent: `Flips the surface-normal to face in a direction opposite to I.`.
  **L881 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Flips the surface-normal to face in a direction opposite to I.`。
- **L882 EN**: Declares TableGen def record `hlsl_faceforward`.
  **L882 CN**: 声明 TableGen def 记录 `hlsl_faceforward`。
- **L883 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L883 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L884 EN**: Continues logic associated with callable symbol `faceforward`.
  **L884 CN**: 继续与可调用符号 `faceforward` 相关的逻辑。
- **L885 EN**: Continues logic associated with callable symbol `normal`.
  **L885 CN**: 继续与可调用符号 `normal` 相关的逻辑。
- **L886 EN**: Continues the surrounding expression or declaration: `to \a I. Returns the result in terms of \a N.`.
  **L886 CN**: 继续构造周围的表达式或声明：`to \a I. Returns the result in terms of \a N.`。
- **L887 EN**: Continues the surrounding expression or declaration: `\param N The resulting floating-point surface-normal vector.`.
  **L887 CN**: 继续构造周围的表达式或声明：`\param N The resulting floating-point surface-normal vector.`。
- **L888 EN**: Continues the surrounding expression or declaration: `\param I A floating-point, incident vector that points from the view`.
  **L888 CN**: 继续构造周围的表达式或声明：`\param I A floating-point, incident vector that points from the view`。

### Lines 889-912

````tablegen
position to the shading position.
\param Ng A floating-point surface-normal vector.

Return a floating-point, surface normal vector that is facing the view
direction.
}];
  let ParamNames = ["N", "I", "Ng"];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Return fused multiply-add of double-precision arguments: a * b + c.
def hlsl_fma : HLSLThreeArgBuiltin<"fma", "__builtin_elementwise_fma"> {
  let Doc = [{
\fn double fma(double a, double b, double c)
\brief Returns the double-precision fused multiply-addition of a * b + c.
\param a The first value in the fused multiply-addition.
\param b The second value in the fused multiply-addition.
\param c The third value in the fused multiply-addition.
}];
  let VaryingTypes = [DoubleTy];
}

// Returns the location of the first set bit starting from the lowest order bit
````
- **L889 EN**: Continues the surrounding expression or declaration: `position to the shading position.`.
  **L889 CN**: 继续构造周围的表达式或声明：`position to the shading position.`。
- **L890 EN**: Continues the surrounding expression or declaration: `\param Ng A floating-point surface-normal vector.`.
  **L890 CN**: 继续构造周围的表达式或声明：`\param Ng A floating-point surface-normal vector.`。
- **L891 EN**: Blank line separating nearby declarations or logic blocks.
  **L891 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L892 EN**: Continues the surrounding expression or declaration: `Return a floating-point, surface normal vector that is facing the view`.
  **L892 CN**: 继续构造周围的表达式或声明：`Return a floating-point, surface normal vector that is facing the view`。
- **L893 EN**: Continues the surrounding expression or declaration: `direction.`.
  **L893 CN**: 继续构造周围的表达式或声明：`direction.`。
- **L894 EN**: Adds a standalone statement or declaration: `}];`.
  **L894 CN**: 添加一条独立语句或声明：`}];`。
- **L895 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["N", "I", "Ng"];`.
  **L895 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["N", "I", "Ng"];`。
- **L896 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L896 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L897 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L897 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L898 EN**: Closes the current lexical scope or compound statement.
  **L898 CN**: 结束当前词法作用域或复合语句块。
- **L899 EN**: Blank line separating nearby declarations or logic blocks.
  **L899 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L900 EN**: Comment explains nearby logic, constraints, or intent: `Return fused multiply-add of double-precision arguments: a * b + c.`.
  **L900 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Return fused multiply-add of double-precision arguments: a * b + c.`。
- **L901 EN**: Declares TableGen def record `hlsl_fma`.
  **L901 CN**: 声明 TableGen def 记录 `hlsl_fma`。
- **L902 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L902 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L903 EN**: Continues logic associated with callable symbol `fma`.
  **L903 CN**: 继续与可调用符号 `fma` 相关的逻辑。
- **L904 EN**: Continues the surrounding expression or declaration: `\brief Returns the double-precision fused multiply-addition of a * b + c.`.
  **L904 CN**: 继续构造周围的表达式或声明：`\brief Returns the double-precision fused multiply-addition of a * b + c.`。
- **L905 EN**: Continues the surrounding expression or declaration: `\param a The first value in the fused multiply-addition.`.
  **L905 CN**: 继续构造周围的表达式或声明：`\param a The first value in the fused multiply-addition.`。
- **L906 EN**: Continues the surrounding expression or declaration: `\param b The second value in the fused multiply-addition.`.
  **L906 CN**: 继续构造周围的表达式或声明：`\param b The second value in the fused multiply-addition.`。
- **L907 EN**: Continues the surrounding expression or declaration: `\param c The third value in the fused multiply-addition.`.
  **L907 CN**: 继续构造周围的表达式或声明：`\param c The third value in the fused multiply-addition.`。
- **L908 EN**: Adds a standalone statement or declaration: `}];`.
  **L908 CN**: 添加一条独立语句或声明：`}];`。
- **L909 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [DoubleTy];`.
  **L909 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [DoubleTy];`。
- **L910 EN**: Closes the current lexical scope or compound statement.
  **L910 CN**: 结束当前词法作用域或复合语句块。
- **L911 EN**: Blank line separating nearby declarations or logic blocks.
  **L911 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L912 EN**: Comment explains nearby logic, constraints, or intent: `Returns the location of the first set bit starting from the lowest order bit`.
  **L912 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the location of the first set bit starting from the lowest order bit`。

### Lines 913-936

````tablegen
// and working upward, per component.
def hlsl_firstbitlow : HLSLOneArgBuiltin<"firstbitlow",
                                    "__builtin_hlsl_elementwise_firstbitlow"> {
  let Doc = [{
\fn T firstbitlow(T Val)
\brief Returns the location of the first set bit starting from the lowest
order bit and working upward, per component.
\param Val the input value.
}];
  let ReturnType = VaryingShape<UIntTy>;
  let VaryingTypes = AllIntTypes;
  let VaryingMatDims = [];
}

// Returns the largest integer that is less than or equal to the input
// value, Val.
def hlsl_floor : HLSLOneArgBuiltin<"floor", "__builtin_elementwise_floor"> {
  let Doc = [{
\fn T floor(T Val)
\brief Returns the largest integer that is less than or equal to the input
value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
````
- **L913 EN**: Comment explains nearby logic, constraints, or intent: `and working upward, per component.`.
  **L913 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and working upward, per component.`。
- **L914 EN**: Declares TableGen def record `hlsl_firstbitlow`.
  **L914 CN**: 声明 TableGen def 记录 `hlsl_firstbitlow`。
- **L915 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_elementwise_firstbitlow"> {`.
  **L915 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_elementwise_firstbitlow"> {`。
- **L916 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L916 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L917 EN**: Continues logic associated with callable symbol `firstbitlow`.
  **L917 CN**: 继续与可调用符号 `firstbitlow` 相关的逻辑。
- **L918 EN**: Continues the surrounding expression or declaration: `\brief Returns the location of the first set bit starting from the lowest`.
  **L918 CN**: 继续构造周围的表达式或声明：`\brief Returns the location of the first set bit starting from the lowest`。
- **L919 EN**: Continues the surrounding expression or declaration: `order bit and working upward, per component.`.
  **L919 CN**: 继续构造周围的表达式或声明：`order bit and working upward, per component.`。
- **L920 EN**: Continues the surrounding expression or declaration: `\param Val the input value.`.
  **L920 CN**: 继续构造周围的表达式或声明：`\param Val the input value.`。
- **L921 EN**: Adds a standalone statement or declaration: `}];`.
  **L921 CN**: 添加一条独立语句或声明：`}];`。
- **L922 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<UIntTy>;`.
  **L922 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<UIntTy>;`。
- **L923 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllIntTypes;`.
  **L923 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllIntTypes;`。
- **L924 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L924 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L925 EN**: Closes the current lexical scope or compound statement.
  **L925 CN**: 结束当前词法作用域或复合语句块。
- **L926 EN**: Blank line separating nearby declarations or logic blocks.
  **L926 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L927 EN**: Comment explains nearby logic, constraints, or intent: `Returns the largest integer that is less than or equal to the input`.
  **L927 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the largest integer that is less than or equal to the input`。
- **L928 EN**: Comment explains nearby logic, constraints, or intent: `value, Val.`.
  **L928 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value, Val.`。
- **L929 EN**: Declares TableGen def record `hlsl_floor`.
  **L929 CN**: 声明 TableGen def 记录 `hlsl_floor`。
- **L930 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L930 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L931 EN**: Continues logic associated with callable symbol `floor`.
  **L931 CN**: 继续与可调用符号 `floor` 相关的逻辑。
- **L932 EN**: Continues the surrounding expression or declaration: `\brief Returns the largest integer that is less than or equal to the input`.
  **L932 CN**: 继续构造周围的表达式或声明：`\brief Returns the largest integer that is less than or equal to the input`。
- **L933 EN**: Continues the surrounding expression or declaration: `value, \a Val.`.
  **L933 CN**: 继续构造周围的表达式或声明：`value, \a Val.`。
- **L934 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L934 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L935 EN**: Adds a standalone statement or declaration: `}];`.
  **L935 CN**: 添加一条独立语句或声明：`}];`。
- **L936 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L936 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。

### Lines 937-960

````tablegen
  let VaryingMatDims = [];
}

// Returns the floating-point remainder of x/y (scalar overloads).
def hlsl_fmod : HLSLBuiltin<"fmod"> {
  let Doc = [{
\fn T fmod(T x, T y)
\brief Returns the floating-point remainder of x/y.
\param x [in] The dividend.
\param y [in] The divisor.

Return the floating-point remainder of the x parameter divided by the y
parameter.
}];
  let DetailFunc = "fmod_impl";
  let ParamNames = ["X", "Y"];
  let Args = [Varying, Varying];
  let ReturnType = Varying;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingScalar = 1;
  let VaryingMatDims = [];
}

// Returns the floating-point remainder of x/y (vector overloads).
````
- **L937 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L937 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L938 EN**: Closes the current lexical scope or compound statement.
  **L938 CN**: 结束当前词法作用域或复合语句块。
- **L939 EN**: Blank line separating nearby declarations or logic blocks.
  **L939 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L940 EN**: Comment explains nearby logic, constraints, or intent: `Returns the floating-point remainder of x/y (scalar overloads).`.
  **L940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the floating-point remainder of x/y (scalar overloads).`。
- **L941 EN**: Declares TableGen def record `hlsl_fmod`.
  **L941 CN**: 声明 TableGen def 记录 `hlsl_fmod`。
- **L942 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L942 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L943 EN**: Continues logic associated with callable symbol `fmod`.
  **L943 CN**: 继续与可调用符号 `fmod` 相关的逻辑。
- **L944 EN**: Continues the surrounding expression or declaration: `\brief Returns the floating-point remainder of x/y.`.
  **L944 CN**: 继续构造周围的表达式或声明：`\brief Returns the floating-point remainder of x/y.`。
- **L945 EN**: Continues the surrounding expression or declaration: `\param x [in] The dividend.`.
  **L945 CN**: 继续构造周围的表达式或声明：`\param x [in] The dividend.`。
- **L946 EN**: Continues the surrounding expression or declaration: `\param y [in] The divisor.`.
  **L946 CN**: 继续构造周围的表达式或声明：`\param y [in] The divisor.`。
- **L947 EN**: Blank line separating nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L948 EN**: Continues the surrounding expression or declaration: `Return the floating-point remainder of the x parameter divided by the y`.
  **L948 CN**: 继续构造周围的表达式或声明：`Return the floating-point remainder of the x parameter divided by the y`。
- **L949 EN**: Continues the surrounding expression or declaration: `parameter.`.
  **L949 CN**: 继续构造周围的表达式或声明：`parameter.`。
- **L950 EN**: Adds a standalone statement or declaration: `}];`.
  **L950 CN**: 添加一条独立语句或声明：`}];`。
- **L951 EN**: Assigns a TableGen property that affects following records or inherited fields: `let DetailFunc = "fmod_impl";`.
  **L951 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let DetailFunc = "fmod_impl";`。
- **L952 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["X", "Y"];`.
  **L952 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["X", "Y"];`。
- **L953 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L953 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L954 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L954 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L955 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L955 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L956 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L956 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L957 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L957 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L960 EN**: Comment explains nearby logic, constraints, or intent: `Returns the floating-point remainder of x/y (vector overloads).`.
  **L960 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the floating-point remainder of x/y (vector overloads).`。

### Lines 961-984

````tablegen
def hlsl_fmod_vec : HLSLBuiltin<"fmod"> {
  let DetailFunc = "fmod_vec_impl";
  let ParamNames = ["X", "Y"];
  let Args = [Varying, Varying];
  let ReturnType = Varying;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = [];
}

// Returns the fractional (or decimal) part of x.
def hlsl_frac : HLSLOneArgBuiltin<"frac", "__builtin_hlsl_elementwise_frac"> {
  let Doc = [{
\fn T frac(T x)
\brief Returns the fractional (or decimal) part of x. \a x parameter.
\param x The specified input value.

If \a the return value is greater than or equal to 0 and less than 1.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Computes the sum of the absolute values of the partial derivatives.
````
- **L961 EN**: Declares TableGen def record `hlsl_fmod_vec`.
  **L961 CN**: 声明 TableGen def 记录 `hlsl_fmod_vec`。
- **L962 EN**: Assigns a TableGen property that affects following records or inherited fields: `let DetailFunc = "fmod_vec_impl";`.
  **L962 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let DetailFunc = "fmod_vec_impl";`。
- **L963 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["X", "Y"];`.
  **L963 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["X", "Y"];`。
- **L964 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L964 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L965 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L965 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L966 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L966 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L967 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L967 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L968 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L968 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L969 EN**: Closes the current lexical scope or compound statement.
  **L969 CN**: 结束当前词法作用域或复合语句块。
- **L970 EN**: Blank line separating nearby declarations or logic blocks.
  **L970 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L971 EN**: Comment explains nearby logic, constraints, or intent: `Returns the fractional (or decimal) part of x.`.
  **L971 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the fractional (or decimal) part of x.`。
- **L972 EN**: Declares TableGen def record `hlsl_frac`.
  **L972 CN**: 声明 TableGen def 记录 `hlsl_frac`。
- **L973 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L973 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L974 EN**: Continues logic associated with callable symbol `frac`.
  **L974 CN**: 继续与可调用符号 `frac` 相关的逻辑。
- **L975 EN**: Continues logic associated with callable symbol `fractional`.
  **L975 CN**: 继续与可调用符号 `fractional` 相关的逻辑。
- **L976 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L976 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L978 EN**: Continues the surrounding expression or declaration: `If \a the return value is greater than or equal to 0 and less than 1.`.
  **L978 CN**: 继续构造周围的表达式或声明：`If \a the return value is greater than or equal to 0 and less than 1.`。
- **L979 EN**: Adds a standalone statement or declaration: `}];`.
  **L979 CN**: 添加一条独立语句或声明：`}];`。
- **L980 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L980 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L981 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L981 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L982 EN**: Closes the current lexical scope or compound statement.
  **L982 CN**: 结束当前词法作用域或复合语句块。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L984 EN**: Comment explains nearby logic, constraints, or intent: `Computes the sum of the absolute values of the partial derivatives.`.
  **L984 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes the sum of the absolute values of the partial derivatives.`。

### Lines 985-1008

````tablegen
def hlsl_fwidth : HLSLOneArgDetail<"fwidth", "fwidth_impl"> {
  let Doc = [{
\fn T fwidth(T x)
\brief Computes the sum of the absolute values of the partial derivatives
with regard to the x and y screen space coordinates.
\param x [in] The floating-point scalar or vector to process.
}];
  let ParamNames = ["input"];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Blocks execution of all threads in a group until all memory accesses
// have been completed.
def hlsl_all_memory_barrier :
    HLSLBuiltin<"AllMemoryBarrier",
                "__builtin_hlsl_all_memory_barrier"> {
  let Doc = [{
\fn void AllMemoryBarrier(void)
\brief Blocks execution of all threads in a group until all memory
accesses have been completed.
}];
  let IsConvergent = 1;
}
````
- **L985 EN**: Declares TableGen def record `hlsl_fwidth`.
  **L985 CN**: 声明 TableGen def 记录 `hlsl_fwidth`。
- **L986 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L986 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L987 EN**: Continues logic associated with callable symbol `fwidth`.
  **L987 CN**: 继续与可调用符号 `fwidth` 相关的逻辑。
- **L988 EN**: Continues the surrounding expression or declaration: `\brief Computes the sum of the absolute values of the partial derivatives`.
  **L988 CN**: 继续构造周围的表达式或声明：`\brief Computes the sum of the absolute values of the partial derivatives`。
- **L989 EN**: Continues the surrounding expression or declaration: `with regard to the x and y screen space coordinates.`.
  **L989 CN**: 继续构造周围的表达式或声明：`with regard to the x and y screen space coordinates.`。
- **L990 EN**: Continues the surrounding expression or declaration: `\param x [in] The floating-point scalar or vector to process.`.
  **L990 CN**: 继续构造周围的表达式或声明：`\param x [in] The floating-point scalar or vector to process.`。
- **L991 EN**: Adds a standalone statement or declaration: `}];`.
  **L991 CN**: 添加一条独立语句或声明：`}];`。
- **L992 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["input"];`.
  **L992 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["input"];`。
- **L993 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L993 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L994 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L994 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L995 EN**: Closes the current lexical scope or compound statement.
  **L995 CN**: 结束当前词法作用域或复合语句块。
- **L996 EN**: Blank line separating nearby declarations or logic blocks.
  **L996 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L997 EN**: Comment explains nearby logic, constraints, or intent: `Blocks execution of all threads in a group until all memory accesses`.
  **L997 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Blocks execution of all threads in a group until all memory accesses`。
- **L998 EN**: Comment explains nearby logic, constraints, or intent: `have been completed.`.
  **L998 CN**: 注释解释附近代码的逻辑、约束或设计意图：`have been completed.`。
- **L999 EN**: Declares TableGen def record `hlsl_all_memory_barrier`.
  **L999 CN**: 声明 TableGen def 记录 `hlsl_all_memory_barrier`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"AllMemoryBarrier",`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"AllMemoryBarrier",`。
- **L1001 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_all_memory_barrier"> {`.
  **L1001 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_all_memory_barrier"> {`。
- **L1002 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1002 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1003 EN**: Continues logic associated with callable symbol `AllMemoryBarrier`.
  **L1003 CN**: 继续与可调用符号 `AllMemoryBarrier` 相关的逻辑。
- **L1004 EN**: Continues the surrounding expression or declaration: `\brief Blocks execution of all threads in a group until all memory`.
  **L1004 CN**: 继续构造周围的表达式或声明：`\brief Blocks execution of all threads in a group until all memory`。
- **L1005 EN**: Continues the surrounding expression or declaration: `accesses have been completed.`.
  **L1005 CN**: 继续构造周围的表达式或声明：`accesses have been completed.`。
- **L1006 EN**: Adds a standalone statement or declaration: `}];`.
  **L1006 CN**: 添加一条独立语句或声明：`}];`。
- **L1007 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1007 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1008 EN**: Closes the current lexical scope or compound statement.
  **L1008 CN**: 结束当前词法作用域或复合语句块。

### Lines 1009-1032

````tablegen

// Blocks execution of all threads in a group until all memory accesses
// have been completed and all threads in the group have reached this call.
def hlsl_all_memory_barrier_with_group_sync :
    HLSLBuiltin<"AllMemoryBarrierWithGroupSync",
                "__builtin_hlsl_all_memory_barrier_with_group_sync"> {
  let Doc = [{
\fn void AllMemoryBarrierWithGroupSync(void)
\brief Blocks execution of all threads in a group until all memory
accesses have been completed and all threads in the group have reached this call.
}];
  let IsConvergent = 1;
}

// Blocks execution of all threads in a group until all device memory accesses
// have been completed.
def hlsl_device_memory_barrier :
    HLSLBuiltin<"DeviceMemoryBarrier",
                "__builtin_hlsl_device_memory_barrier"> {
  let Doc = [{
\fn void DeviceMemoryBarrier(void)
\brief Blocks execution of all threads in a group until all device memory
accesses have been completed.
}];
````
- **L1009 EN**: Blank line separating nearby declarations or logic blocks.
  **L1009 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1010 EN**: Comment explains nearby logic, constraints, or intent: `Blocks execution of all threads in a group until all memory accesses`.
  **L1010 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Blocks execution of all threads in a group until all memory accesses`。
- **L1011 EN**: Comment explains nearby logic, constraints, or intent: `have been completed and all threads in the group have reached this call.`.
  **L1011 CN**: 注释解释附近代码的逻辑、约束或设计意图：`have been completed and all threads in the group have reached this call.`。
- **L1012 EN**: Declares TableGen def record `hlsl_all_memory_barrier_with_group_sync`.
  **L1012 CN**: 声明 TableGen def 记录 `hlsl_all_memory_barrier_with_group_sync`。
- **L1013 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"AllMemoryBarrierWithGroupSync",`.
  **L1013 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"AllMemoryBarrierWithGroupSync",`。
- **L1014 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_all_memory_barrier_with_group_sync"> {`.
  **L1014 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_all_memory_barrier_with_group_sync"> {`。
- **L1015 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1015 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1016 EN**: Continues logic associated with callable symbol `AllMemoryBarrierWithGroupSync`.
  **L1016 CN**: 继续与可调用符号 `AllMemoryBarrierWithGroupSync` 相关的逻辑。
- **L1017 EN**: Continues the surrounding expression or declaration: `\brief Blocks execution of all threads in a group until all memory`.
  **L1017 CN**: 继续构造周围的表达式或声明：`\brief Blocks execution of all threads in a group until all memory`。
- **L1018 EN**: Continues the surrounding expression or declaration: `accesses have been completed and all threads in the group have reached this call.`.
  **L1018 CN**: 继续构造周围的表达式或声明：`accesses have been completed and all threads in the group have reached this call.`。
- **L1019 EN**: Adds a standalone statement or declaration: `}];`.
  **L1019 CN**: 添加一条独立语句或声明：`}];`。
- **L1020 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1020 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1021 EN**: Closes the current lexical scope or compound statement.
  **L1021 CN**: 结束当前词法作用域或复合语句块。
- **L1022 EN**: Blank line separating nearby declarations or logic blocks.
  **L1022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1023 EN**: Comment explains nearby logic, constraints, or intent: `Blocks execution of all threads in a group until all device memory accesses`.
  **L1023 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Blocks execution of all threads in a group until all device memory accesses`。
- **L1024 EN**: Comment explains nearby logic, constraints, or intent: `have been completed.`.
  **L1024 CN**: 注释解释附近代码的逻辑、约束或设计意图：`have been completed.`。
- **L1025 EN**: Declares TableGen def record `hlsl_device_memory_barrier`.
  **L1025 CN**: 声明 TableGen def 记录 `hlsl_device_memory_barrier`。
- **L1026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"DeviceMemoryBarrier",`.
  **L1026 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"DeviceMemoryBarrier",`。
- **L1027 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_device_memory_barrier"> {`.
  **L1027 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_device_memory_barrier"> {`。
- **L1028 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1028 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1029 EN**: Continues logic associated with callable symbol `DeviceMemoryBarrier`.
  **L1029 CN**: 继续与可调用符号 `DeviceMemoryBarrier` 相关的逻辑。
- **L1030 EN**: Continues the surrounding expression or declaration: `\brief Blocks execution of all threads in a group until all device memory`.
  **L1030 CN**: 继续构造周围的表达式或声明：`\brief Blocks execution of all threads in a group until all device memory`。
- **L1031 EN**: Continues the surrounding expression or declaration: `accesses have been completed.`.
  **L1031 CN**: 继续构造周围的表达式或声明：`accesses have been completed.`。
- **L1032 EN**: Adds a standalone statement or declaration: `}];`.
  **L1032 CN**: 添加一条独立语句或声明：`}];`。

### Lines 1033-1056

````tablegen
  let IsConvergent = 1;
}

// Blocks execution of all threads in a group until all device memory accesses
// have been completed and all threads in the group have reached this call.
def hlsl_device_memory_barrier_with_group_sync :
    HLSLBuiltin<"DeviceMemoryBarrierWithGroupSync",
                "__builtin_hlsl_device_memory_barrier_with_group_sync"> {
  let Doc = [{
\fn void DeviceMemoryBarrierWithGroupSync(void)
\brief Blocks execution of all threads in a group until all device memory
accesses have been completed and all threads in the group have reached this call.
}];
  let IsConvergent = 1;
}

// Blocks execution of all threads in a group until all group shared accesses
// have been completed.
def hlsl_group_memory_barrier :
    HLSLBuiltin<"GroupMemoryBarrier",
                "__builtin_hlsl_group_memory_barrier"> {
  let Doc = [{
\fn void GroupMemoryBarrier(void)
\brief Blocks execution of all threads in a group until all group shared
````
- **L1033 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1033 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1034 EN**: Closes the current lexical scope or compound statement.
  **L1034 CN**: 结束当前词法作用域或复合语句块。
- **L1035 EN**: Blank line separating nearby declarations or logic blocks.
  **L1035 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1036 EN**: Comment explains nearby logic, constraints, or intent: `Blocks execution of all threads in a group until all device memory accesses`.
  **L1036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Blocks execution of all threads in a group until all device memory accesses`。
- **L1037 EN**: Comment explains nearby logic, constraints, or intent: `have been completed and all threads in the group have reached this call.`.
  **L1037 CN**: 注释解释附近代码的逻辑、约束或设计意图：`have been completed and all threads in the group have reached this call.`。
- **L1038 EN**: Declares TableGen def record `hlsl_device_memory_barrier_with_group_sync`.
  **L1038 CN**: 声明 TableGen def 记录 `hlsl_device_memory_barrier_with_group_sync`。
- **L1039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"DeviceMemoryBarrierWithGroupSync",`.
  **L1039 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"DeviceMemoryBarrierWithGroupSync",`。
- **L1040 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_device_memory_barrier_with_group_sync"> {`.
  **L1040 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_device_memory_barrier_with_group_sync"> {`。
- **L1041 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1041 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1042 EN**: Continues logic associated with callable symbol `DeviceMemoryBarrierWithGroupSync`.
  **L1042 CN**: 继续与可调用符号 `DeviceMemoryBarrierWithGroupSync` 相关的逻辑。
- **L1043 EN**: Continues the surrounding expression or declaration: `\brief Blocks execution of all threads in a group until all device memory`.
  **L1043 CN**: 继续构造周围的表达式或声明：`\brief Blocks execution of all threads in a group until all device memory`。
- **L1044 EN**: Continues the surrounding expression or declaration: `accesses have been completed and all threads in the group have reached this call.`.
  **L1044 CN**: 继续构造周围的表达式或声明：`accesses have been completed and all threads in the group have reached this call.`。
- **L1045 EN**: Adds a standalone statement or declaration: `}];`.
  **L1045 CN**: 添加一条独立语句或声明：`}];`。
- **L1046 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1046 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1047 EN**: Closes the current lexical scope or compound statement.
  **L1047 CN**: 结束当前词法作用域或复合语句块。
- **L1048 EN**: Blank line separating nearby declarations or logic blocks.
  **L1048 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1049 EN**: Comment explains nearby logic, constraints, or intent: `Blocks execution of all threads in a group until all group shared accesses`.
  **L1049 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Blocks execution of all threads in a group until all group shared accesses`。
- **L1050 EN**: Comment explains nearby logic, constraints, or intent: `have been completed.`.
  **L1050 CN**: 注释解释附近代码的逻辑、约束或设计意图：`have been completed.`。
- **L1051 EN**: Declares TableGen def record `hlsl_group_memory_barrier`.
  **L1051 CN**: 声明 TableGen def 记录 `hlsl_group_memory_barrier`。
- **L1052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"GroupMemoryBarrier",`.
  **L1052 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"GroupMemoryBarrier",`。
- **L1053 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_group_memory_barrier"> {`.
  **L1053 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_group_memory_barrier"> {`。
- **L1054 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1054 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1055 EN**: Continues logic associated with callable symbol `GroupMemoryBarrier`.
  **L1055 CN**: 继续与可调用符号 `GroupMemoryBarrier` 相关的逻辑。
- **L1056 EN**: Continues the surrounding expression or declaration: `\brief Blocks execution of all threads in a group until all group shared`.
  **L1056 CN**: 继续构造周围的表达式或声明：`\brief Blocks execution of all threads in a group until all group shared`。

### Lines 1057-1080

````tablegen
accesses have been completed.
}];
  let IsConvergent = 1;
}

// Blocks execution of all threads in a group until all group shared accesses
// have been completed and all threads in the group have reached this call.
def hlsl_group_memory_barrier_with_group_sync :
    HLSLBuiltin<"GroupMemoryBarrierWithGroupSync",
                "__builtin_hlsl_group_memory_barrier_with_group_sync"> {
  let Doc = [{
\fn void GroupMemoryBarrierWithGroupSync(void)
\brief Blocks execution of all threads in a group until all group shared
accesses have been completed and all threads in the group have reached this
call.
}];
  let IsConvergent = 1;
}

// Determines if the specified value x is infinite.
def hlsl_isinf : HLSLOneArgBuiltin<"isinf", "__builtin_hlsl_elementwise_isinf"> {
  let Doc = [{
\fn T isinf(T x)
\brief Determines if the specified value \a x  is infinite.
````
- **L1057 EN**: Continues the surrounding expression or declaration: `accesses have been completed.`.
  **L1057 CN**: 继续构造周围的表达式或声明：`accesses have been completed.`。
- **L1058 EN**: Adds a standalone statement or declaration: `}];`.
  **L1058 CN**: 添加一条独立语句或声明：`}];`。
- **L1059 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1059 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1060 EN**: Closes the current lexical scope or compound statement.
  **L1060 CN**: 结束当前词法作用域或复合语句块。
- **L1061 EN**: Blank line separating nearby declarations or logic blocks.
  **L1061 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1062 EN**: Comment explains nearby logic, constraints, or intent: `Blocks execution of all threads in a group until all group shared accesses`.
  **L1062 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Blocks execution of all threads in a group until all group shared accesses`。
- **L1063 EN**: Comment explains nearby logic, constraints, or intent: `have been completed and all threads in the group have reached this call.`.
  **L1063 CN**: 注释解释附近代码的逻辑、约束或设计意图：`have been completed and all threads in the group have reached this call.`。
- **L1064 EN**: Declares TableGen def record `hlsl_group_memory_barrier_with_group_sync`.
  **L1064 CN**: 声明 TableGen def 记录 `hlsl_group_memory_barrier_with_group_sync`。
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"GroupMemoryBarrierWithGroupSync",`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"GroupMemoryBarrierWithGroupSync",`。
- **L1066 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_group_memory_barrier_with_group_sync"> {`.
  **L1066 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_group_memory_barrier_with_group_sync"> {`。
- **L1067 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1067 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1068 EN**: Continues logic associated with callable symbol `GroupMemoryBarrierWithGroupSync`.
  **L1068 CN**: 继续与可调用符号 `GroupMemoryBarrierWithGroupSync` 相关的逻辑。
- **L1069 EN**: Continues the surrounding expression or declaration: `\brief Blocks execution of all threads in a group until all group shared`.
  **L1069 CN**: 继续构造周围的表达式或声明：`\brief Blocks execution of all threads in a group until all group shared`。
- **L1070 EN**: Continues the surrounding expression or declaration: `accesses have been completed and all threads in the group have reached this`.
  **L1070 CN**: 继续构造周围的表达式或声明：`accesses have been completed and all threads in the group have reached this`。
- **L1071 EN**: Continues the surrounding expression or declaration: `call.`.
  **L1071 CN**: 继续构造周围的表达式或声明：`call.`。
- **L1072 EN**: Adds a standalone statement or declaration: `}];`.
  **L1072 CN**: 添加一条独立语句或声明：`}];`。
- **L1073 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1073 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1074 EN**: Closes the current lexical scope or compound statement.
  **L1074 CN**: 结束当前词法作用域或复合语句块。
- **L1075 EN**: Blank line separating nearby declarations or logic blocks.
  **L1075 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1076 EN**: Comment explains nearby logic, constraints, or intent: `Determines if the specified value x is infinite.`.
  **L1076 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Determines if the specified value x is infinite.`。
- **L1077 EN**: Declares TableGen def record `hlsl_isinf`.
  **L1077 CN**: 声明 TableGen def 记录 `hlsl_isinf`。
- **L1078 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1078 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1079 EN**: Continues logic associated with callable symbol `isinf`.
  **L1079 CN**: 继续与可调用符号 `isinf` 相关的逻辑。
- **L1080 EN**: Continues the surrounding expression or declaration: `\brief Determines if the specified value \a x  is infinite.`.
  **L1080 CN**: 继续构造周围的表达式或声明：`\brief Determines if the specified value \a x  is infinite.`。

### Lines 1081-1104

````tablegen
\param x The specified input value.

Returns a value of the same size as the input, with a value set
to True if the x parameter is +INF or -INF. Otherwise, False.
}];
  let ReturnType = VaryingShape<BoolTy>;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Determines if the specified value x is Not a Number.
def hlsl_isnan : HLSLOneArgBuiltin<"isnan", "__builtin_hlsl_elementwise_isnan"> {
  let Doc = [{
\fn T isnan(T x)
\brief Determines if the specified value \a x  is Not a Number.
\param x The specified input value.

Returns a value of the same size as the input, with a value set
to True if the x parameter is NaN or QNaN. Otherwise, False.
}];
  let ReturnType = VaryingShape<BoolTy>;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}
````
- **L1081 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L1081 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L1082 EN**: Blank line separating nearby declarations or logic blocks.
  **L1082 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1083 EN**: Continues the surrounding expression or declaration: `Returns a value of the same size as the input, with a value set`.
  **L1083 CN**: 继续构造周围的表达式或声明：`Returns a value of the same size as the input, with a value set`。
- **L1084 EN**: Continues the surrounding expression or declaration: `to True if the x parameter is +INF or -INF. Otherwise, False.`.
  **L1084 CN**: 继续构造周围的表达式或声明：`to True if the x parameter is +INF or -INF. Otherwise, False.`。
- **L1085 EN**: Adds a standalone statement or declaration: `}];`.
  **L1085 CN**: 添加一条独立语句或声明：`}];`。
- **L1086 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<BoolTy>;`.
  **L1086 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<BoolTy>;`。
- **L1087 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1087 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1088 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1088 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1089 EN**: Closes the current lexical scope or compound statement.
  **L1089 CN**: 结束当前词法作用域或复合语句块。
- **L1090 EN**: Blank line separating nearby declarations or logic blocks.
  **L1090 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1091 EN**: Comment explains nearby logic, constraints, or intent: `Determines if the specified value x is Not a Number.`.
  **L1091 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Determines if the specified value x is Not a Number.`。
- **L1092 EN**: Declares TableGen def record `hlsl_isnan`.
  **L1092 CN**: 声明 TableGen def 记录 `hlsl_isnan`。
- **L1093 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1093 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1094 EN**: Continues logic associated with callable symbol `isnan`.
  **L1094 CN**: 继续与可调用符号 `isnan` 相关的逻辑。
- **L1095 EN**: Continues the surrounding expression or declaration: `\brief Determines if the specified value \a x  is Not a Number.`.
  **L1095 CN**: 继续构造周围的表达式或声明：`\brief Determines if the specified value \a x  is Not a Number.`。
- **L1096 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L1096 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1098 EN**: Continues the surrounding expression or declaration: `Returns a value of the same size as the input, with a value set`.
  **L1098 CN**: 继续构造周围的表达式或声明：`Returns a value of the same size as the input, with a value set`。
- **L1099 EN**: Continues the surrounding expression or declaration: `to True if the x parameter is NaN or QNaN. Otherwise, False.`.
  **L1099 CN**: 继续构造周围的表达式或声明：`to True if the x parameter is NaN or QNaN. Otherwise, False.`。
- **L1100 EN**: Adds a standalone statement or declaration: `}];`.
  **L1100 CN**: 添加一条独立语句或声明：`}];`。
- **L1101 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<BoolTy>;`.
  **L1101 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<BoolTy>;`。
- **L1102 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1102 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1103 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1103 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1104 EN**: Closes the current lexical scope or compound statement.
  **L1104 CN**: 结束当前词法作用域或复合语句块。

### Lines 1105-1128

````tablegen

// Returns the result of multiplying the specified value by two raised
// to the power of the specified exponent.
def hlsl_ldexp : HLSLBuiltin<"ldexp"> {
  let Doc = [{
\fn T ldexp(T X, T Exp)
\brief Returns the result of multiplying the specified value by two raised
to the power of the specified exponent.
\param X [in] The specified value.
\param Exp [in] The specified exponent.

This function uses the following formula: X * 2^Exp
}];
  // ldexp(X, Exp) = X * 2^Exp, implemented as exp2(Exp) * X.
  let ParamNames = ["X", "Exp"];
  let Body = "return exp2(Exp) * X;";
  let Args = [Varying, Varying];
  let ReturnType = Varying;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = [];
}

````
- **L1105 EN**: Blank line separating nearby declarations or logic blocks.
  **L1105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1106 EN**: Comment explains nearby logic, constraints, or intent: `Returns the result of multiplying the specified value by two raised`.
  **L1106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the result of multiplying the specified value by two raised`。
- **L1107 EN**: Comment explains nearby logic, constraints, or intent: `to the power of the specified exponent.`.
  **L1107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the power of the specified exponent.`。
- **L1108 EN**: Declares TableGen def record `hlsl_ldexp`.
  **L1108 CN**: 声明 TableGen def 记录 `hlsl_ldexp`。
- **L1109 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1109 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1110 EN**: Continues logic associated with callable symbol `ldexp`.
  **L1110 CN**: 继续与可调用符号 `ldexp` 相关的逻辑。
- **L1111 EN**: Continues the surrounding expression or declaration: `\brief Returns the result of multiplying the specified value by two raised`.
  **L1111 CN**: 继续构造周围的表达式或声明：`\brief Returns the result of multiplying the specified value by two raised`。
- **L1112 EN**: Continues the surrounding expression or declaration: `to the power of the specified exponent.`.
  **L1112 CN**: 继续构造周围的表达式或声明：`to the power of the specified exponent.`。
- **L1113 EN**: Continues the surrounding expression or declaration: `\param X [in] The specified value.`.
  **L1113 CN**: 继续构造周围的表达式或声明：`\param X [in] The specified value.`。
- **L1114 EN**: Continues the surrounding expression or declaration: `\param Exp [in] The specified exponent.`.
  **L1114 CN**: 继续构造周围的表达式或声明：`\param Exp [in] The specified exponent.`。
- **L1115 EN**: Blank line separating nearby declarations or logic blocks.
  **L1115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1116 EN**: Continues the surrounding expression or declaration: `This function uses the following formula: X * 2^Exp`.
  **L1116 CN**: 继续构造周围的表达式或声明：`This function uses the following formula: X * 2^Exp`。
- **L1117 EN**: Adds a standalone statement or declaration: `}];`.
  **L1117 CN**: 添加一条独立语句或声明：`}];`。
- **L1118 EN**: Comment explains nearby logic, constraints, or intent: `ldexp(X, Exp) X * 2^Exp, implemented as exp2(Exp) * X.`.
  **L1118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ldexp(X, Exp) X * 2^Exp, implemented as exp2(Exp) * X.`。
- **L1119 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["X", "Exp"];`.
  **L1119 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["X", "Exp"];`。
- **L1120 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Body = "return exp2(Exp) * X;";`.
  **L1120 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Body = "return exp2(Exp) * X;";`。
- **L1121 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying];`.
  **L1121 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying];`。
- **L1122 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L1122 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L1123 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1123 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1124 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L1124 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L1125 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L1125 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L1126 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1126 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1127 EN**: Closes the current lexical scope or compound statement.
  **L1127 CN**: 结束当前词法作用域或复合语句块。
- **L1128 EN**: Blank line separating nearby declarations or logic blocks.
  **L1128 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1129-1152

````tablegen
// Returns the length of the specified floating-point vector.
// Scalar length is equivalent to abs(X).
// Vector length is equivalent to sqrt(dot(X, X)).
def hlsl_length : HLSLBuiltin<"length"> {
  let Doc = [{
\fn T length(T x)
\brief Returns the length of the specified floating-point vector.
\param x [in] The vector of floats, or a scalar float.

Length is based on the following formula: sqrt(x[0]^2 + x[1]^2 + ...).
}];
  let DetailFunc = "length_impl";
  let ParamNames = ["X"];
  let Args = [Varying];
  let ReturnType = VaryingElemType;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = [];
}

// Returns the linear interpolation of x to y by s.
def hlsl_lerp : HLSLThreeArgBuiltin<"lerp", "__builtin_hlsl_lerp"> {
  let Doc = [{
````
- **L1129 EN**: Comment explains nearby logic, constraints, or intent: `Returns the length of the specified floating-point vector.`.
  **L1129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the length of the specified floating-point vector.`。
- **L1130 EN**: Comment explains nearby logic, constraints, or intent: `Scalar length is equivalent to abs(X).`.
  **L1130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar length is equivalent to abs(X).`。
- **L1131 EN**: Comment explains nearby logic, constraints, or intent: `Vector length is equivalent to sqrt(dot(X, X)).`.
  **L1131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector length is equivalent to sqrt(dot(X, X)).`。
- **L1132 EN**: Declares TableGen def record `hlsl_length`.
  **L1132 CN**: 声明 TableGen def 记录 `hlsl_length`。
- **L1133 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1133 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1134 EN**: Continues logic associated with callable symbol `length`.
  **L1134 CN**: 继续与可调用符号 `length` 相关的逻辑。
- **L1135 EN**: Continues the surrounding expression or declaration: `\brief Returns the length of the specified floating-point vector.`.
  **L1135 CN**: 继续构造周围的表达式或声明：`\brief Returns the length of the specified floating-point vector.`。
- **L1136 EN**: Continues the surrounding expression or declaration: `\param x [in] The vector of floats, or a scalar float.`.
  **L1136 CN**: 继续构造周围的表达式或声明：`\param x [in] The vector of floats, or a scalar float.`。
- **L1137 EN**: Blank line separating nearby declarations or logic blocks.
  **L1137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1138 EN**: Continues logic associated with callable symbol `sqrt`.
  **L1138 CN**: 继续与可调用符号 `sqrt` 相关的逻辑。
- **L1139 EN**: Adds a standalone statement or declaration: `}];`.
  **L1139 CN**: 添加一条独立语句或声明：`}];`。
- **L1140 EN**: Assigns a TableGen property that affects following records or inherited fields: `let DetailFunc = "length_impl";`.
  **L1140 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let DetailFunc = "length_impl";`。
- **L1141 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["X"];`.
  **L1141 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["X"];`。
- **L1142 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L1142 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L1143 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingElemType;`.
  **L1143 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingElemType;`。
- **L1144 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1144 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1145 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L1145 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L1146 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L1146 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L1147 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1147 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1148 EN**: Closes the current lexical scope or compound statement.
  **L1148 CN**: 结束当前词法作用域或复合语句块。
- **L1149 EN**: Blank line separating nearby declarations or logic blocks.
  **L1149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1150 EN**: Comment explains nearby logic, constraints, or intent: `Returns the linear interpolation of x to y by s.`.
  **L1150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the linear interpolation of x to y by s.`。
- **L1151 EN**: Declares TableGen def record `hlsl_lerp`.
  **L1151 CN**: 声明 TableGen def 记录 `hlsl_lerp`。
- **L1152 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1152 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。

### Lines 1153-1176

````tablegen
\fn T lerp(T x, T y, T s)
\brief Returns the linear interpolation of x to y by s.
\param x [in] The first-floating point value.
\param y [in] The second-floating point value.
\param s [in] A value that linearly interpolates between the x parameter and
the y parameter.

Linear interpolation is based on the following formula: x*(1-s) + y*s which
can equivalently be written as x + s(y-x).
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

def hlsl_lit_float : HLSLBuiltin<"lit"> {
  let DetailFunc = "lit_impl";
  let ParamNames = ["NDotL", "NDotH", "M"];
  let Args = [FloatTy, FloatTy, FloatTy];
  let ReturnType = VectorType<FloatTy, 4>;
}

// Returns a lighting coefficient vector.
def hlsl_lit_half : HLSLBuiltin<"lit"> {
  let Doc = [{
````
- **L1153 EN**: Continues logic associated with callable symbol `lerp`.
  **L1153 CN**: 继续与可调用符号 `lerp` 相关的逻辑。
- **L1154 EN**: Continues the surrounding expression or declaration: `\brief Returns the linear interpolation of x to y by s.`.
  **L1154 CN**: 继续构造周围的表达式或声明：`\brief Returns the linear interpolation of x to y by s.`。
- **L1155 EN**: Continues the surrounding expression or declaration: `\param x [in] The first-floating point value.`.
  **L1155 CN**: 继续构造周围的表达式或声明：`\param x [in] The first-floating point value.`。
- **L1156 EN**: Continues the surrounding expression or declaration: `\param y [in] The second-floating point value.`.
  **L1156 CN**: 继续构造周围的表达式或声明：`\param y [in] The second-floating point value.`。
- **L1157 EN**: Continues the surrounding expression or declaration: `\param s [in] A value that linearly interpolates between the x parameter and`.
  **L1157 CN**: 继续构造周围的表达式或声明：`\param s [in] A value that linearly interpolates between the x parameter and`。
- **L1158 EN**: Continues the surrounding expression or declaration: `the y parameter.`.
  **L1158 CN**: 继续构造周围的表达式或声明：`the y parameter.`。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1160 EN**: Continues the surrounding expression or declaration: `Linear interpolation is based on the following formula: x*(1-s) + y*s which`.
  **L1160 CN**: 继续构造周围的表达式或声明：`Linear interpolation is based on the following formula: x*(1-s) + y*s which`。
- **L1161 EN**: Continues logic associated with callable symbol `s`.
  **L1161 CN**: 继续与可调用符号 `s` 相关的逻辑。
- **L1162 EN**: Adds a standalone statement or declaration: `}];`.
  **L1162 CN**: 添加一条独立语句或声明：`}];`。
- **L1163 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1163 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1164 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1164 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1167 EN**: Declares TableGen def record `hlsl_lit_float`.
  **L1167 CN**: 声明 TableGen def 记录 `hlsl_lit_float`。
- **L1168 EN**: Assigns a TableGen property that affects following records or inherited fields: `let DetailFunc = "lit_impl";`.
  **L1168 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let DetailFunc = "lit_impl";`。
- **L1169 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["NDotL", "NDotH", "M"];`.
  **L1169 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["NDotL", "NDotH", "M"];`。
- **L1170 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [FloatTy, FloatTy, FloatTy];`.
  **L1170 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [FloatTy, FloatTy, FloatTy];`。
- **L1171 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VectorType<FloatTy, 4>;`.
  **L1171 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VectorType<FloatTy, 4>;`。
- **L1172 EN**: Closes the current lexical scope or compound statement.
  **L1172 CN**: 结束当前词法作用域或复合语句块。
- **L1173 EN**: Blank line separating nearby declarations or logic blocks.
  **L1173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1174 EN**: Comment explains nearby logic, constraints, or intent: `Returns a lighting coefficient vector.`.
  **L1174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a lighting coefficient vector.`。
- **L1175 EN**: Declares TableGen def record `hlsl_lit_half`.
  **L1175 CN**: 声明 TableGen def 记录 `hlsl_lit_half`。
- **L1176 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1176 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。

### Lines 1177-1200

````tablegen
\fn vector<T, 4> lit(T NDotL, T NDotH, T M)
\brief Returns a lighting coefficient vector.
\param NDotL The dot product of the normalized surface normal and the
light vector.
\param NDotH The dot product of the half-angle vector and the surface
normal.
\param M A specular exponent.

This function returns a lighting coefficient vector (ambient, diffuse,
specular, 1).
}];
  let DetailFunc = "lit_impl";
  let ParamNames = ["NDotL", "NDotH", "M"];
  let Args = [HalfTy, HalfTy, HalfTy];
  let ReturnType = VectorType<HalfTy, 4>;
}

// Returns the base-e logarithm of the input value, Val.
def hlsl_log : HLSLOneArgBuiltin<"log", "__builtin_elementwise_log"> {
  let Doc = [{
\fn T log(T Val)
\brief The base-e logarithm of the input value, \a Val parameter.
\param Val The input value.

````
- **L1177 EN**: Continues logic associated with callable symbol `lit`.
  **L1177 CN**: 继续与可调用符号 `lit` 相关的逻辑。
- **L1178 EN**: Continues the surrounding expression or declaration: `\brief Returns a lighting coefficient vector.`.
  **L1178 CN**: 继续构造周围的表达式或声明：`\brief Returns a lighting coefficient vector.`。
- **L1179 EN**: Continues the surrounding expression or declaration: `\param NDotL The dot product of the normalized surface normal and the`.
  **L1179 CN**: 继续构造周围的表达式或声明：`\param NDotL The dot product of the normalized surface normal and the`。
- **L1180 EN**: Continues the surrounding expression or declaration: `light vector.`.
  **L1180 CN**: 继续构造周围的表达式或声明：`light vector.`。
- **L1181 EN**: Continues the surrounding expression or declaration: `\param NDotH The dot product of the half-angle vector and the surface`.
  **L1181 CN**: 继续构造周围的表达式或声明：`\param NDotH The dot product of the half-angle vector and the surface`。
- **L1182 EN**: Continues the surrounding expression or declaration: `normal.`.
  **L1182 CN**: 继续构造周围的表达式或声明：`normal.`。
- **L1183 EN**: Continues the surrounding expression or declaration: `\param M A specular exponent.`.
  **L1183 CN**: 继续构造周围的表达式或声明：`\param M A specular exponent.`。
- **L1184 EN**: Blank line separating nearby declarations or logic blocks.
  **L1184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `This function returns a lighting coefficient vector (ambient, diffuse,`.
  **L1185 CN**: 继续一个多行参数列表、初始化器或聚合项：`This function returns a lighting coefficient vector (ambient, diffuse,`。
- **L1186 EN**: Continues the surrounding expression or declaration: `specular, 1).`.
  **L1186 CN**: 继续构造周围的表达式或声明：`specular, 1).`。
- **L1187 EN**: Adds a standalone statement or declaration: `}];`.
  **L1187 CN**: 添加一条独立语句或声明：`}];`。
- **L1188 EN**: Assigns a TableGen property that affects following records or inherited fields: `let DetailFunc = "lit_impl";`.
  **L1188 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let DetailFunc = "lit_impl";`。
- **L1189 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["NDotL", "NDotH", "M"];`.
  **L1189 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["NDotL", "NDotH", "M"];`。
- **L1190 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [HalfTy, HalfTy, HalfTy];`.
  **L1190 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [HalfTy, HalfTy, HalfTy];`。
- **L1191 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VectorType<HalfTy, 4>;`.
  **L1191 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VectorType<HalfTy, 4>;`。
- **L1192 EN**: Closes the current lexical scope or compound statement.
  **L1192 CN**: 结束当前词法作用域或复合语句块。
- **L1193 EN**: Blank line separating nearby declarations or logic blocks.
  **L1193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1194 EN**: Comment explains nearby logic, constraints, or intent: `Returns the base-e logarithm of the input value, Val.`.
  **L1194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the base-e logarithm of the input value, Val.`。
- **L1195 EN**: Declares TableGen def record `hlsl_log`.
  **L1195 CN**: 声明 TableGen def 记录 `hlsl_log`。
- **L1196 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1196 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1197 EN**: Continues logic associated with callable symbol `log`.
  **L1197 CN**: 继续与可调用符号 `log` 相关的逻辑。
- **L1198 EN**: Continues the surrounding expression or declaration: `\brief The base-e logarithm of the input value, \a Val parameter.`.
  **L1198 CN**: 继续构造周围的表达式或声明：`\brief The base-e logarithm of the input value, \a Val parameter.`。
- **L1199 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1199 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1200 EN**: Blank line separating nearby declarations or logic blocks.
  **L1200 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1201-1224

````tablegen
If \a Val is negative, this result is undefined. If \a Val is 0, this
function returns negative infinity.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the base-10 logarithm of the input value, Val.
def hlsl_log10 : HLSLOneArgBuiltin<"log10", "__builtin_elementwise_log10"> {
  let Doc = [{
\fn T log10(T Val)
\brief The base-10 logarithm of the input value, \a Val parameter.
\param Val The input value.

If \a Val is negative, this result is undefined. If \a Val is 0, this
function returns negative infinity.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the base-2 logarithm of the input value, Val.
def hlsl_log2 : HLSLOneArgBuiltin<"log2", "__builtin_elementwise_log2"> {
  let Doc = [{
````
- **L1201 EN**: Continues the surrounding expression or declaration: `If \a Val is negative, this result is undefined. If \a Val is 0, this`.
  **L1201 CN**: 继续构造周围的表达式或声明：`If \a Val is negative, this result is undefined. If \a Val is 0, this`。
- **L1202 EN**: Continues the surrounding expression or declaration: `function returns negative infinity.`.
  **L1202 CN**: 继续构造周围的表达式或声明：`function returns negative infinity.`。
- **L1203 EN**: Adds a standalone statement or declaration: `}];`.
  **L1203 CN**: 添加一条独立语句或声明：`}];`。
- **L1204 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1204 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1205 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1205 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1206 EN**: Closes the current lexical scope or compound statement.
  **L1206 CN**: 结束当前词法作用域或复合语句块。
- **L1207 EN**: Blank line separating nearby declarations or logic blocks.
  **L1207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1208 EN**: Comment explains nearby logic, constraints, or intent: `Returns the base-10 logarithm of the input value, Val.`.
  **L1208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the base-10 logarithm of the input value, Val.`。
- **L1209 EN**: Declares TableGen def record `hlsl_log10`.
  **L1209 CN**: 声明 TableGen def 记录 `hlsl_log10`。
- **L1210 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1210 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1211 EN**: Continues logic associated with callable symbol `log10`.
  **L1211 CN**: 继续与可调用符号 `log10` 相关的逻辑。
- **L1212 EN**: Continues the surrounding expression or declaration: `\brief The base-10 logarithm of the input value, \a Val parameter.`.
  **L1212 CN**: 继续构造周围的表达式或声明：`\brief The base-10 logarithm of the input value, \a Val parameter.`。
- **L1213 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1213 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1214 EN**: Blank line separating nearby declarations or logic blocks.
  **L1214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1215 EN**: Continues the surrounding expression or declaration: `If \a Val is negative, this result is undefined. If \a Val is 0, this`.
  **L1215 CN**: 继续构造周围的表达式或声明：`If \a Val is negative, this result is undefined. If \a Val is 0, this`。
- **L1216 EN**: Continues the surrounding expression or declaration: `function returns negative infinity.`.
  **L1216 CN**: 继续构造周围的表达式或声明：`function returns negative infinity.`。
- **L1217 EN**: Adds a standalone statement or declaration: `}];`.
  **L1217 CN**: 添加一条独立语句或声明：`}];`。
- **L1218 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1218 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1219 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1219 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1220 EN**: Closes the current lexical scope or compound statement.
  **L1220 CN**: 结束当前词法作用域或复合语句块。
- **L1221 EN**: Blank line separating nearby declarations or logic blocks.
  **L1221 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1222 EN**: Comment explains nearby logic, constraints, or intent: `Returns the base-2 logarithm of the input value, Val.`.
  **L1222 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the base-2 logarithm of the input value, Val.`。
- **L1223 EN**: Declares TableGen def record `hlsl_log2`.
  **L1223 CN**: 声明 TableGen def 记录 `hlsl_log2`。
- **L1224 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1224 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。

### Lines 1225-1248

````tablegen
\fn T log2(T Val)
\brief The base-2 logarithm of the input value, \a Val parameter.
\param Val The input value.

If \a Val is negative, this result is undefined. If \a Val is 0, this
function returns negative infinity.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the result of M * A + B (multiply-add).
def hlsl_mad : HLSLThreeArgBuiltin<"mad", "__builtin_hlsl_mad"> {
  let Doc = [{
\fn T mad(T M, T A, T B)
\brief The result of \a M * \a A + \a B.
\param M The multiplication value.
\param A The first addition value.
\param B The second addition value.
}];
  let VaryingTypes = AllNumericTypes;
  let VaryingMatDims = [];
}

````
- **L1225 EN**: Continues logic associated with callable symbol `log2`.
  **L1225 CN**: 继续与可调用符号 `log2` 相关的逻辑。
- **L1226 EN**: Continues the surrounding expression or declaration: `\brief The base-2 logarithm of the input value, \a Val parameter.`.
  **L1226 CN**: 继续构造周围的表达式或声明：`\brief The base-2 logarithm of the input value, \a Val parameter.`。
- **L1227 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1227 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1228 EN**: Blank line separating nearby declarations or logic blocks.
  **L1228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1229 EN**: Continues the surrounding expression or declaration: `If \a Val is negative, this result is undefined. If \a Val is 0, this`.
  **L1229 CN**: 继续构造周围的表达式或声明：`If \a Val is negative, this result is undefined. If \a Val is 0, this`。
- **L1230 EN**: Continues the surrounding expression or declaration: `function returns negative infinity.`.
  **L1230 CN**: 继续构造周围的表达式或声明：`function returns negative infinity.`。
- **L1231 EN**: Adds a standalone statement or declaration: `}];`.
  **L1231 CN**: 添加一条独立语句或声明：`}];`。
- **L1232 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1232 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1233 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1233 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1234 EN**: Closes the current lexical scope or compound statement.
  **L1234 CN**: 结束当前词法作用域或复合语句块。
- **L1235 EN**: Blank line separating nearby declarations or logic blocks.
  **L1235 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1236 EN**: Comment explains nearby logic, constraints, or intent: `Returns the result of M * A + B (multiply-add).`.
  **L1236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the result of M * A + B (multiply-add).`。
- **L1237 EN**: Declares TableGen def record `hlsl_mad`.
  **L1237 CN**: 声明 TableGen def 记录 `hlsl_mad`。
- **L1238 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1238 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1239 EN**: Continues logic associated with callable symbol `mad`.
  **L1239 CN**: 继续与可调用符号 `mad` 相关的逻辑。
- **L1240 EN**: Continues the surrounding expression or declaration: `\brief The result of \a M * \a A + \a B.`.
  **L1240 CN**: 继续构造周围的表达式或声明：`\brief The result of \a M * \a A + \a B.`。
- **L1241 EN**: Continues the surrounding expression or declaration: `\param M The multiplication value.`.
  **L1241 CN**: 继续构造周围的表达式或声明：`\param M The multiplication value.`。
- **L1242 EN**: Continues the surrounding expression or declaration: `\param A The first addition value.`.
  **L1242 CN**: 继续构造周围的表达式或声明：`\param A The first addition value.`。
- **L1243 EN**: Continues the surrounding expression or declaration: `\param B The second addition value.`.
  **L1243 CN**: 继续构造周围的表达式或声明：`\param B The second addition value.`。
- **L1244 EN**: Adds a standalone statement or declaration: `}];`.
  **L1244 CN**: 添加一条独立语句或声明：`}];`。
- **L1245 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1245 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1246 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1246 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1247 EN**: Closes the current lexical scope or compound statement.
  **L1247 CN**: 结束当前词法作用域或复合语句块。
- **L1248 EN**: Blank line separating nearby declarations or logic blocks.
  **L1248 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1249-1272

````tablegen
// Returns the greater of X and Y.
def hlsl_max : HLSLTwoArgBuiltin<"max", "__builtin_elementwise_max"> {
  let Doc = [{
\fn T max(T X, T Y)
\brief Return the greater of \a X and \a Y.
\param X The X input value.
\param Y The Y input value.
}];
  let VaryingTypes = AllNumericTypes;
  let VaryingMatDims = [];
}

// Returns the lesser of X and Y.
def hlsl_min : HLSLTwoArgBuiltin<"min", "__builtin_elementwise_min"> {
  let Doc = [{
\fn T min(T X, T Y)
\brief Return the lesser of \a X and \a Y.
\param X The X input value.
\param Y The Y input value.
}];
  let VaryingTypes = AllNumericTypes;
  let VaryingMatDims = [];
}

````
- **L1249 EN**: Comment explains nearby logic, constraints, or intent: `Returns the greater of X and Y.`.
  **L1249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the greater of X and Y.`。
- **L1250 EN**: Declares TableGen def record `hlsl_max`.
  **L1250 CN**: 声明 TableGen def 记录 `hlsl_max`。
- **L1251 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1251 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1252 EN**: Continues logic associated with callable symbol `max`.
  **L1252 CN**: 继续与可调用符号 `max` 相关的逻辑。
- **L1253 EN**: Continues the surrounding expression or declaration: `\brief Return the greater of \a X and \a Y.`.
  **L1253 CN**: 继续构造周围的表达式或声明：`\brief Return the greater of \a X and \a Y.`。
- **L1254 EN**: Continues the surrounding expression or declaration: `\param X The X input value.`.
  **L1254 CN**: 继续构造周围的表达式或声明：`\param X The X input value.`。
- **L1255 EN**: Continues the surrounding expression or declaration: `\param Y The Y input value.`.
  **L1255 CN**: 继续构造周围的表达式或声明：`\param Y The Y input value.`。
- **L1256 EN**: Adds a standalone statement or declaration: `}];`.
  **L1256 CN**: 添加一条独立语句或声明：`}];`。
- **L1257 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1257 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1258 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1258 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1259 EN**: Closes the current lexical scope or compound statement.
  **L1259 CN**: 结束当前词法作用域或复合语句块。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1261 EN**: Comment explains nearby logic, constraints, or intent: `Returns the lesser of X and Y.`.
  **L1261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the lesser of X and Y.`。
- **L1262 EN**: Declares TableGen def record `hlsl_min`.
  **L1262 CN**: 声明 TableGen def 记录 `hlsl_min`。
- **L1263 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1263 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1264 EN**: Continues logic associated with callable symbol `min`.
  **L1264 CN**: 继续与可调用符号 `min` 相关的逻辑。
- **L1265 EN**: Continues the surrounding expression or declaration: `\brief Return the lesser of \a X and \a Y.`.
  **L1265 CN**: 继续构造周围的表达式或声明：`\brief Return the lesser of \a X and \a Y.`。
- **L1266 EN**: Continues the surrounding expression or declaration: `\param X The X input value.`.
  **L1266 CN**: 继续构造周围的表达式或声明：`\param X The X input value.`。
- **L1267 EN**: Continues the surrounding expression or declaration: `\param Y The Y input value.`.
  **L1267 CN**: 继续构造周围的表达式或声明：`\param Y The Y input value.`。
- **L1268 EN**: Adds a standalone statement or declaration: `}];`.
  **L1268 CN**: 添加一条独立语句或声明：`}];`。
- **L1269 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1269 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1270 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1270 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1271 EN**: Closes the current lexical scope or compound statement.
  **L1271 CN**: 结束当前词法作用域或复合语句块。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1273-1296

````tablegen
// Marks a resource index as non-uniform (varying across wave threads).
def hlsl_non_uniform_resource_index :
    HLSLBuiltin<"NonUniformResourceIndex",
                "__builtin_hlsl_resource_nonuniformindex"> {
  let Doc = [{
\fn uint NonUniformResourceIndex(uint I)
\brief A compiler hint to indicate that a resource index varies across
threads within a wave (i.e., it is non-uniform).
\param I [in] Resource array index

The return value is the \a I parameter.
}];
  let Args = [UIntTy];
  let ReturnType = UIntTy;
}

// Returns the normalized unit vector of the specified floating-point vector.
def hlsl_normalize : HLSLOneArgBuiltin<"normalize", "__builtin_hlsl_normalize"> {
  let Doc = [{
\fn T normalize(T x)
\brief Returns the normalized unit vector of the specified floating-point
vector.
\param x [in] The vector of floats.

````
- **L1273 EN**: Comment explains nearby logic, constraints, or intent: `Marks a resource index as non-uniform (varying across wave threads).`.
  **L1273 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Marks a resource index as non-uniform (varying across wave threads).`。
- **L1274 EN**: Declares TableGen def record `hlsl_non_uniform_resource_index`.
  **L1274 CN**: 声明 TableGen def 记录 `hlsl_non_uniform_resource_index`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"NonUniformResourceIndex",`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"NonUniformResourceIndex",`。
- **L1276 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_resource_nonuniformindex"> {`.
  **L1276 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_resource_nonuniformindex"> {`。
- **L1277 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1277 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1278 EN**: Continues logic associated with callable symbol `NonUniformResourceIndex`.
  **L1278 CN**: 继续与可调用符号 `NonUniformResourceIndex` 相关的逻辑。
- **L1279 EN**: Continues the surrounding expression or declaration: `\brief A compiler hint to indicate that a resource index varies across`.
  **L1279 CN**: 继续构造周围的表达式或声明：`\brief A compiler hint to indicate that a resource index varies across`。
- **L1280 EN**: Continues logic associated with callable symbol `wave`.
  **L1280 CN**: 继续与可调用符号 `wave` 相关的逻辑。
- **L1281 EN**: Continues the surrounding expression or declaration: `\param I [in] Resource array index`.
  **L1281 CN**: 继续构造周围的表达式或声明：`\param I [in] Resource array index`。
- **L1282 EN**: Blank line separating nearby declarations or logic blocks.
  **L1282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1283 EN**: Continues the surrounding expression or declaration: `The return value is the \a I parameter.`.
  **L1283 CN**: 继续构造周围的表达式或声明：`The return value is the \a I parameter.`。
- **L1284 EN**: Adds a standalone statement or declaration: `}];`.
  **L1284 CN**: 添加一条独立语句或声明：`}];`。
- **L1285 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [UIntTy];`.
  **L1285 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [UIntTy];`。
- **L1286 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = UIntTy;`.
  **L1286 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = UIntTy;`。
- **L1287 EN**: Closes the current lexical scope or compound statement.
  **L1287 CN**: 结束当前词法作用域或复合语句块。
- **L1288 EN**: Blank line separating nearby declarations or logic blocks.
  **L1288 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1289 EN**: Comment explains nearby logic, constraints, or intent: `Returns the normalized unit vector of the specified floating-point vector.`.
  **L1289 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the normalized unit vector of the specified floating-point vector.`。
- **L1290 EN**: Declares TableGen def record `hlsl_normalize`.
  **L1290 CN**: 声明 TableGen def 记录 `hlsl_normalize`。
- **L1291 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1291 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1292 EN**: Continues logic associated with callable symbol `normalize`.
  **L1292 CN**: 继续与可调用符号 `normalize` 相关的逻辑。
- **L1293 EN**: Continues the surrounding expression or declaration: `\brief Returns the normalized unit vector of the specified floating-point`.
  **L1293 CN**: 继续构造周围的表达式或声明：`\brief Returns the normalized unit vector of the specified floating-point`。
- **L1294 EN**: Continues the surrounding expression or declaration: `vector.`.
  **L1294 CN**: 继续构造周围的表达式或声明：`vector.`。
- **L1295 EN**: Continues the surrounding expression or declaration: `\param x [in] The vector of floats.`.
  **L1295 CN**: 继续构造周围的表达式或声明：`\param x [in] The vector of floats.`。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1297-1320

````tablegen
Normalize is based on the following formula: x / length(x).
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the boolean OR of two boolean scalars, vectors, or matrices.
def hlsl_or : HLSLTwoArgBuiltin<"or", "__builtin_hlsl_or"> {
  let Doc = [{
\fn bool or(bool x, bool y)
\brief Logically ors two boolean vectors or matrices elementwise and
 produces a bool vector or matrix output.
}];
  let VaryingTypes = [BoolTy];
}

// Returns the value Val, raised to the power Pow.
def hlsl_pow : HLSLTwoArgBuiltin<"pow", "__builtin_elementwise_pow"> {
  let Doc = [{
\fn T pow(T Val, T Pow)
\brief Return the value \a Val, raised to the power \a Pow.
\param Val The input value.
\param Pow The specified power.
}];
````
- **L1297 EN**: Continues logic associated with callable symbol `length`.
  **L1297 CN**: 继续与可调用符号 `length` 相关的逻辑。
- **L1298 EN**: Adds a standalone statement or declaration: `}];`.
  **L1298 CN**: 添加一条独立语句或声明：`}];`。
- **L1299 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1299 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1300 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1300 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1301 EN**: Closes the current lexical scope or compound statement.
  **L1301 CN**: 结束当前词法作用域或复合语句块。
- **L1302 EN**: Blank line separating nearby declarations or logic blocks.
  **L1302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1303 EN**: Comment explains nearby logic, constraints, or intent: `Returns the boolean OR of two boolean scalars, vectors, or matrices.`.
  **L1303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the boolean OR of two boolean scalars, vectors, or matrices.`。
- **L1304 EN**: Declares TableGen def record `hlsl_or`.
  **L1304 CN**: 声明 TableGen def 记录 `hlsl_or`。
- **L1305 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1305 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1306 EN**: Continues logic associated with callable symbol `or`.
  **L1306 CN**: 继续与可调用符号 `or` 相关的逻辑。
- **L1307 EN**: Continues the surrounding expression or declaration: `\brief Logically ors two boolean vectors or matrices elementwise and`.
  **L1307 CN**: 继续构造周围的表达式或声明：`\brief Logically ors two boolean vectors or matrices elementwise and`。
- **L1308 EN**: Continues the surrounding expression or declaration: `produces a bool vector or matrix output.`.
  **L1308 CN**: 继续构造周围的表达式或声明：`produces a bool vector or matrix output.`。
- **L1309 EN**: Adds a standalone statement or declaration: `}];`.
  **L1309 CN**: 添加一条独立语句或声明：`}];`。
- **L1310 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [BoolTy];`.
  **L1310 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [BoolTy];`。
- **L1311 EN**: Closes the current lexical scope or compound statement.
  **L1311 CN**: 结束当前词法作用域或复合语句块。
- **L1312 EN**: Blank line separating nearby declarations or logic blocks.
  **L1312 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1313 EN**: Comment explains nearby logic, constraints, or intent: `Returns the value Val, raised to the power Pow.`.
  **L1313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the value Val, raised to the power Pow.`。
- **L1314 EN**: Declares TableGen def record `hlsl_pow`.
  **L1314 CN**: 声明 TableGen def 记录 `hlsl_pow`。
- **L1315 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1315 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1316 EN**: Continues logic associated with callable symbol `pow`.
  **L1316 CN**: 继续与可调用符号 `pow` 相关的逻辑。
- **L1317 EN**: Continues the surrounding expression or declaration: `\brief Return the value \a Val, raised to the power \a Pow.`.
  **L1317 CN**: 继续构造周围的表达式或声明：`\brief Return the value \a Val, raised to the power \a Pow.`。
- **L1318 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1318 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1319 EN**: Continues the surrounding expression or declaration: `\param Pow The specified power.`.
  **L1319 CN**: 继续构造周围的表达式或声明：`\param Pow The specified power.`。
- **L1320 EN**: Adds a standalone statement or declaration: `}];`.
  **L1320 CN**: 添加一条独立语句或声明：`}];`。

### Lines 1321-1344

````tablegen
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Reads the value from the lane across the X axis of the quad.
def hlsl_quad_read_across_x :
    HLSLOneArgBuiltin<"QuadReadAcrossX",
                      "__builtin_hlsl_quad_read_across_x"> {
  let Doc = [{
\brief Reads the value from the lane across the X axis of the quad.
\param Val The input value.
}];
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let VaryingMatDims = [];
}

// Reads the value from the lane across the X axis of the quad.
def hlsl_quad_read_across_y :
    HLSLOneArgBuiltin<"QuadReadAcrossY",
                      "__builtin_hlsl_quad_read_across_y"> {
  let Doc = [{
\brief Reads the value from the lane across the Y axis of the quad.
\param Val The input value.
````
- **L1321 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1321 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1322 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1322 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1323 EN**: Closes the current lexical scope or compound statement.
  **L1323 CN**: 结束当前词法作用域或复合语句块。
- **L1324 EN**: Blank line separating nearby declarations or logic blocks.
  **L1324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1325 EN**: Comment explains nearby logic, constraints, or intent: `Reads the value from the lane across the X axis of the quad.`.
  **L1325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reads the value from the lane across the X axis of the quad.`。
- **L1326 EN**: Declares TableGen def record `hlsl_quad_read_across_x`.
  **L1326 CN**: 声明 TableGen def 记录 `hlsl_quad_read_across_x`。
- **L1327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLOneArgBuiltin<"QuadReadAcrossX",`.
  **L1327 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLOneArgBuiltin<"QuadReadAcrossX",`。
- **L1328 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_quad_read_across_x"> {`.
  **L1328 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_quad_read_across_x"> {`。
- **L1329 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1329 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1330 EN**: Continues the surrounding expression or declaration: `\brief Reads the value from the lane across the X axis of the quad.`.
  **L1330 CN**: 继续构造周围的表达式或声明：`\brief Reads the value from the lane across the X axis of the quad.`。
- **L1331 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1331 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1332 EN**: Adds a standalone statement or declaration: `}];`.
  **L1332 CN**: 添加一条独立语句或声明：`}];`。
- **L1333 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1333 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1334 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1334 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1335 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1335 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1336 EN**: Closes the current lexical scope or compound statement.
  **L1336 CN**: 结束当前词法作用域或复合语句块。
- **L1337 EN**: Blank line separating nearby declarations or logic blocks.
  **L1337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1338 EN**: Comment explains nearby logic, constraints, or intent: `Reads the value from the lane across the X axis of the quad.`.
  **L1338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reads the value from the lane across the X axis of the quad.`。
- **L1339 EN**: Declares TableGen def record `hlsl_quad_read_across_y`.
  **L1339 CN**: 声明 TableGen def 记录 `hlsl_quad_read_across_y`。
- **L1340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLOneArgBuiltin<"QuadReadAcrossY",`.
  **L1340 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLOneArgBuiltin<"QuadReadAcrossY",`。
- **L1341 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_quad_read_across_y"> {`.
  **L1341 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_quad_read_across_y"> {`。
- **L1342 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1342 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1343 EN**: Continues the surrounding expression or declaration: `\brief Reads the value from the lane across the Y axis of the quad.`.
  **L1343 CN**: 继续构造周围的表达式或声明：`\brief Reads the value from the lane across the Y axis of the quad.`。
- **L1344 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1344 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。

### Lines 1345-1368

````tablegen
}];
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let VaryingMatDims = [];
}

// Converts the specified value from degrees to radians.
def hlsl_radians : HLSLOneArgBuiltin<"radians", "__builtin_hlsl_elementwise_radians"> {
  let Doc = [{
\fn T radians(T Val)
\brief Converts the specified value from degrees to radians.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Calculates a fast, approximate, per-component reciprocal ie 1 / x.
def hlsl_rcp : HLSLOneArgBuiltin<"rcp", "__builtin_hlsl_elementwise_rcp"> {
  let Doc = [{
\fn T rcp(T x)
\brief Calculates a fast, approximate, per-component reciprocal ie 1 / \a x.
\param x The specified input value.

The return value is the reciprocal of the \a x parameter.
````
- **L1345 EN**: Adds a standalone statement or declaration: `}];`.
  **L1345 CN**: 添加一条独立语句或声明：`}];`。
- **L1346 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1346 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1347 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1347 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1348 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1348 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1349 EN**: Closes the current lexical scope or compound statement.
  **L1349 CN**: 结束当前词法作用域或复合语句块。
- **L1350 EN**: Blank line separating nearby declarations or logic blocks.
  **L1350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1351 EN**: Comment explains nearby logic, constraints, or intent: `Converts the specified value from degrees to radians.`.
  **L1351 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converts the specified value from degrees to radians.`。
- **L1352 EN**: Declares TableGen def record `hlsl_radians`.
  **L1352 CN**: 声明 TableGen def 记录 `hlsl_radians`。
- **L1353 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1353 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1354 EN**: Continues logic associated with callable symbol `radians`.
  **L1354 CN**: 继续与可调用符号 `radians` 相关的逻辑。
- **L1355 EN**: Continues the surrounding expression or declaration: `\brief Converts the specified value from degrees to radians.`.
  **L1355 CN**: 继续构造周围的表达式或声明：`\brief Converts the specified value from degrees to radians.`。
- **L1356 EN**: Adds a standalone statement or declaration: `}];`.
  **L1356 CN**: 添加一条独立语句或声明：`}];`。
- **L1357 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1357 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1358 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1358 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1359 EN**: Closes the current lexical scope or compound statement.
  **L1359 CN**: 结束当前词法作用域或复合语句块。
- **L1360 EN**: Blank line separating nearby declarations or logic blocks.
  **L1360 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1361 EN**: Comment explains nearby logic, constraints, or intent: `Calculates a fast, approximate, per-component reciprocal ie 1 / x.`.
  **L1361 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Calculates a fast, approximate, per-component reciprocal ie 1 / x.`。
- **L1362 EN**: Declares TableGen def record `hlsl_rcp`.
  **L1362 CN**: 声明 TableGen def 记录 `hlsl_rcp`。
- **L1363 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1363 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1364 EN**: Continues logic associated with callable symbol `rcp`.
  **L1364 CN**: 继续与可调用符号 `rcp` 相关的逻辑。
- **L1365 EN**: Continues the surrounding expression or declaration: `\brief Calculates a fast, approximate, per-component reciprocal ie 1 / \a x.`.
  **L1365 CN**: 继续构造周围的表达式或声明：`\brief Calculates a fast, approximate, per-component reciprocal ie 1 / \a x.`。
- **L1366 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L1366 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L1367 EN**: Blank line separating nearby declarations or logic blocks.
  **L1367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1368 EN**: Continues the surrounding expression or declaration: `The return value is the reciprocal of the \a x parameter.`.
  **L1368 CN**: 继续构造周围的表达式或声明：`The return value is the reciprocal of the \a x parameter.`。

### Lines 1369-1392

````tablegen
}];
  let VaryingTypes = AllFloatTypes;
  let VaryingMatDims = [];
}

// Returns a reflection vector.
// Scalar reflect: V = I - 2 * N * I * N (dot product is just multiplication
// for scalars).
def hlsl_reflect : HLSLTwoArgDetail<"reflect", "reflect_impl"> {
  let Doc = [{
\fn T reflect(T I, T N)
\brief Returns a reflection using an incident ray, \a I, and a surface
normal, \a N.
\param I The incident ray.
\param N The surface normal.

The return value is a floating-point vector that represents the reflection
of the incident ray, \a I, off a surface with the normal \a N.

This function calculates the reflection vector using the following formula:
V = I - 2 * N * dot(I N) .

N must already be normalized in order to achieve the desired result.
}];
````
- **L1369 EN**: Adds a standalone statement or declaration: `}];`.
  **L1369 CN**: 添加一条独立语句或声明：`}];`。
- **L1370 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllFloatTypes;`.
  **L1370 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllFloatTypes;`。
- **L1371 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1371 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1372 EN**: Closes the current lexical scope or compound statement.
  **L1372 CN**: 结束当前词法作用域或复合语句块。
- **L1373 EN**: Blank line separating nearby declarations or logic blocks.
  **L1373 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1374 EN**: Comment explains nearby logic, constraints, or intent: `Returns a reflection vector.`.
  **L1374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a reflection vector.`。
- **L1375 EN**: Comment explains nearby logic, constraints, or intent: `Scalar reflect: V I - 2 * N * I * N (dot product is just multiplication`.
  **L1375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar reflect: V I - 2 * N * I * N (dot product is just multiplication`。
- **L1376 EN**: Comment explains nearby logic, constraints, or intent: `for scalars).`.
  **L1376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`for scalars).`。
- **L1377 EN**: Declares TableGen def record `hlsl_reflect`.
  **L1377 CN**: 声明 TableGen def 记录 `hlsl_reflect`。
- **L1378 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1378 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1379 EN**: Continues logic associated with callable symbol `reflect`.
  **L1379 CN**: 继续与可调用符号 `reflect` 相关的逻辑。
- **L1380 EN**: Continues the surrounding expression or declaration: `\brief Returns a reflection using an incident ray, \a I, and a surface`.
  **L1380 CN**: 继续构造周围的表达式或声明：`\brief Returns a reflection using an incident ray, \a I, and a surface`。
- **L1381 EN**: Continues the surrounding expression or declaration: `normal, \a N.`.
  **L1381 CN**: 继续构造周围的表达式或声明：`normal, \a N.`。
- **L1382 EN**: Continues the surrounding expression or declaration: `\param I The incident ray.`.
  **L1382 CN**: 继续构造周围的表达式或声明：`\param I The incident ray.`。
- **L1383 EN**: Continues the surrounding expression or declaration: `\param N The surface normal.`.
  **L1383 CN**: 继续构造周围的表达式或声明：`\param N The surface normal.`。
- **L1384 EN**: Blank line separating nearby declarations or logic blocks.
  **L1384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1385 EN**: Continues the surrounding expression or declaration: `The return value is a floating-point vector that represents the reflection`.
  **L1385 CN**: 继续构造周围的表达式或声明：`The return value is a floating-point vector that represents the reflection`。
- **L1386 EN**: Continues the surrounding expression or declaration: `of the incident ray, \a I, off a surface with the normal \a N.`.
  **L1386 CN**: 继续构造周围的表达式或声明：`of the incident ray, \a I, off a surface with the normal \a N.`。
- **L1387 EN**: Blank line separating nearby declarations or logic blocks.
  **L1387 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1388 EN**: Continues the surrounding expression or declaration: `This function calculates the reflection vector using the following formula:`.
  **L1388 CN**: 继续构造周围的表达式或声明：`This function calculates the reflection vector using the following formula:`。
- **L1389 EN**: Continues logic associated with callable symbol `dot`.
  **L1389 CN**: 继续与可调用符号 `dot` 相关的逻辑。
- **L1390 EN**: Blank line separating nearby declarations or logic blocks.
  **L1390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1391 EN**: Continues the surrounding expression or declaration: `N must already be normalized in order to achieve the desired result.`.
  **L1391 CN**: 继续构造周围的表达式或声明：`N must already be normalized in order to achieve the desired result.`。
- **L1392 EN**: Adds a standalone statement or declaration: `}];`.
  **L1392 CN**: 添加一条独立语句或声明：`}];`。

### Lines 1393-1416

````tablegen
  let ParamNames = ["I", "N"];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns a refraction vector using an entering ray, a surface normal, and
// a refraction index.
def hlsl_refract : HLSLBuiltin<"refract"> {
  let Doc = [{
\fn T refract(T I, T N, T eta)
\brief Returns a refraction using an entering ray, \a I, a surface
normal, \a N and refraction index \a eta
\param I The entering ray.
\param N The surface normal.
\param eta The refraction index.

The return value is a floating-point vector that represents the refraction
using the refraction index, \a eta, for the direction of the entering ray,
\a I, off a surface with the normal \a N.

This function calculates the refraction vector using the following formulas:
k = 1.0 - eta * eta * (1.0 - dot(N, I) * dot(N, I))
if k < 0.0 the result is 0.0
otherwise, the result is eta * I - (eta * dot(N, I) + sqrt(k)) * N
````
- **L1393 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["I", "N"];`.
  **L1393 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["I", "N"];`。
- **L1394 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1394 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1395 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1395 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1396 EN**: Closes the current lexical scope or compound statement.
  **L1396 CN**: 结束当前词法作用域或复合语句块。
- **L1397 EN**: Blank line separating nearby declarations or logic blocks.
  **L1397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1398 EN**: Comment explains nearby logic, constraints, or intent: `Returns a refraction vector using an entering ray, a surface normal, and`.
  **L1398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a refraction vector using an entering ray, a surface normal, and`。
- **L1399 EN**: Comment explains nearby logic, constraints, or intent: `a refraction index.`.
  **L1399 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a refraction index.`。
- **L1400 EN**: Declares TableGen def record `hlsl_refract`.
  **L1400 CN**: 声明 TableGen def 记录 `hlsl_refract`。
- **L1401 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1401 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1402 EN**: Continues logic associated with callable symbol `refract`.
  **L1402 CN**: 继续与可调用符号 `refract` 相关的逻辑。
- **L1403 EN**: Continues the surrounding expression or declaration: `\brief Returns a refraction using an entering ray, \a I, a surface`.
  **L1403 CN**: 继续构造周围的表达式或声明：`\brief Returns a refraction using an entering ray, \a I, a surface`。
- **L1404 EN**: Continues the surrounding expression or declaration: `normal, \a N and refraction index \a eta`.
  **L1404 CN**: 继续构造周围的表达式或声明：`normal, \a N and refraction index \a eta`。
- **L1405 EN**: Continues the surrounding expression or declaration: `\param I The entering ray.`.
  **L1405 CN**: 继续构造周围的表达式或声明：`\param I The entering ray.`。
- **L1406 EN**: Continues the surrounding expression or declaration: `\param N The surface normal.`.
  **L1406 CN**: 继续构造周围的表达式或声明：`\param N The surface normal.`。
- **L1407 EN**: Continues the surrounding expression or declaration: `\param eta The refraction index.`.
  **L1407 CN**: 继续构造周围的表达式或声明：`\param eta The refraction index.`。
- **L1408 EN**: Blank line separating nearby declarations or logic blocks.
  **L1408 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1409 EN**: Continues the surrounding expression or declaration: `The return value is a floating-point vector that represents the refraction`.
  **L1409 CN**: 继续构造周围的表达式或声明：`The return value is a floating-point vector that represents the refraction`。
- **L1410 EN**: Introduces an alias or helper declaration: `using the refraction index, \a eta, for the direction of the entering ray,`.
  **L1410 CN**: 引入一条别名或辅助声明：`using the refraction index, \a eta, for the direction of the entering ray,`。
- **L1411 EN**: Continues the surrounding expression or declaration: `\a I, off a surface with the normal \a N.`.
  **L1411 CN**: 继续构造周围的表达式或声明：`\a I, off a surface with the normal \a N.`。
- **L1412 EN**: Blank line separating nearby declarations or logic blocks.
  **L1412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1413 EN**: Continues the surrounding expression or declaration: `This function calculates the refraction vector using the following formulas:`.
  **L1413 CN**: 继续构造周围的表达式或声明：`This function calculates the refraction vector using the following formulas:`。
- **L1414 EN**: Continues logic associated with callable symbol `dot`.
  **L1414 CN**: 继续与可调用符号 `dot` 相关的逻辑。
- **L1415 EN**: Continues the surrounding expression or declaration: `if k < 0.0 the result is 0.0`.
  **L1415 CN**: 继续构造周围的表达式或声明：`if k < 0.0 the result is 0.0`。
- **L1416 EN**: Continues logic associated with callable symbol `dot`.
  **L1416 CN**: 继续与可调用符号 `dot` 相关的逻辑。

### Lines 1417-1440

````tablegen

I and N must already be normalized in order to achieve the desired result.

I and N must be a scalar or vector whose component type is
floating-point.

eta must be a 16-bit or 32-bit floating-point scalar.

Result type, the type of I, and the type of N must all be the same type.
}];
  let DetailFunc = "refract_impl";
  let ParamNames = ["I", "N", "eta"];
  let Args = [Varying, Varying, VaryingElemType];
  let ReturnType = Varying;
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingScalar = 1;
  let VaryingVecSizes = [2, 3, 4];
  let VaryingMatDims = [];
}

// Returns the value Val with the bit order reversed.
def hlsl_reversebits : HLSLOneArgBuiltin<"reversebits", "__builtin_elementwise_bitreverse"> {
  let Doc = [{
\fn T reversebits(T Val)
````
- **L1417 EN**: Blank line separating nearby declarations or logic blocks.
  **L1417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1418 EN**: Continues the surrounding expression or declaration: `I and N must already be normalized in order to achieve the desired result.`.
  **L1418 CN**: 继续构造周围的表达式或声明：`I and N must already be normalized in order to achieve the desired result.`。
- **L1419 EN**: Blank line separating nearby declarations or logic blocks.
  **L1419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1420 EN**: Continues the surrounding expression or declaration: `I and N must be a scalar or vector whose component type is`.
  **L1420 CN**: 继续构造周围的表达式或声明：`I and N must be a scalar or vector whose component type is`。
- **L1421 EN**: Continues the surrounding expression or declaration: `floating-point.`.
  **L1421 CN**: 继续构造周围的表达式或声明：`floating-point.`。
- **L1422 EN**: Blank line separating nearby declarations or logic blocks.
  **L1422 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1423 EN**: Continues the surrounding expression or declaration: `eta must be a 16-bit or 32-bit floating-point scalar.`.
  **L1423 CN**: 继续构造周围的表达式或声明：`eta must be a 16-bit or 32-bit floating-point scalar.`。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1425 EN**: Continues the surrounding expression or declaration: `Result type, the type of I, and the type of N must all be the same type.`.
  **L1425 CN**: 继续构造周围的表达式或声明：`Result type, the type of I, and the type of N must all be the same type.`。
- **L1426 EN**: Adds a standalone statement or declaration: `}];`.
  **L1426 CN**: 添加一条独立语句或声明：`}];`。
- **L1427 EN**: Assigns a TableGen property that affects following records or inherited fields: `let DetailFunc = "refract_impl";`.
  **L1427 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let DetailFunc = "refract_impl";`。
- **L1428 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["I", "N", "eta"];`.
  **L1428 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["I", "N", "eta"];`。
- **L1429 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, Varying, VaryingElemType];`.
  **L1429 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, Varying, VaryingElemType];`。
- **L1430 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L1430 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L1431 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1431 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1432 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L1432 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L1433 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingVecSizes = [2, 3, 4];`.
  **L1433 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingVecSizes = [2, 3, 4];`。
- **L1434 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1434 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Blank line separating nearby declarations or logic blocks.
  **L1436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1437 EN**: Comment explains nearby logic, constraints, or intent: `Returns the value Val with the bit order reversed.`.
  **L1437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the value Val with the bit order reversed.`。
- **L1438 EN**: Declares TableGen def record `hlsl_reversebits`.
  **L1438 CN**: 声明 TableGen def 记录 `hlsl_reversebits`。
- **L1439 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1439 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1440 EN**: Continues logic associated with callable symbol `reversebits`.
  **L1440 CN**: 继续与可调用符号 `reversebits` 相关的逻辑。

### Lines 1441-1464

````tablegen
\brief Return the value \a Val with the bit order reversed.
\param Val The input value.
}];
  let VaryingTypes = AllIntTypes;
  let VaryingMatDims = [];
}

// Rounds the specified value x to the nearest integer.
def hlsl_round : HLSLOneArgBuiltin<"round", "__builtin_elementwise_roundeven"> {
  let Doc = [{
\fn T round(T x)
\brief Rounds the specified value \a x to the nearest integer.
\param x The specified input value.

The return value is the \a x parameter, rounded to the nearest integer
within a floating-point type. Halfway cases are
rounded to the nearest even value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the reciprocal of the square root of the specified value,
// ie 1 / sqrt(x).
````
- **L1441 EN**: Continues the surrounding expression or declaration: `\brief Return the value \a Val with the bit order reversed.`.
  **L1441 CN**: 继续构造周围的表达式或声明：`\brief Return the value \a Val with the bit order reversed.`。
- **L1442 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1442 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1443 EN**: Adds a standalone statement or declaration: `}];`.
  **L1443 CN**: 添加一条独立语句或声明：`}];`。
- **L1444 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllIntTypes;`.
  **L1444 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllIntTypes;`。
- **L1445 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1445 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1446 EN**: Closes the current lexical scope or compound statement.
  **L1446 CN**: 结束当前词法作用域或复合语句块。
- **L1447 EN**: Blank line separating nearby declarations or logic blocks.
  **L1447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1448 EN**: Comment explains nearby logic, constraints, or intent: `Rounds the specified value x to the nearest integer.`.
  **L1448 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds the specified value x to the nearest integer.`。
- **L1449 EN**: Declares TableGen def record `hlsl_round`.
  **L1449 CN**: 声明 TableGen def 记录 `hlsl_round`。
- **L1450 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1450 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1451 EN**: Continues logic associated with callable symbol `round`.
  **L1451 CN**: 继续与可调用符号 `round` 相关的逻辑。
- **L1452 EN**: Continues the surrounding expression or declaration: `\brief Rounds the specified value \a x to the nearest integer.`.
  **L1452 CN**: 继续构造周围的表达式或声明：`\brief Rounds the specified value \a x to the nearest integer.`。
- **L1453 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L1453 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L1454 EN**: Blank line separating nearby declarations or logic blocks.
  **L1454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1455 EN**: Continues the surrounding expression or declaration: `The return value is the \a x parameter, rounded to the nearest integer`.
  **L1455 CN**: 继续构造周围的表达式或声明：`The return value is the \a x parameter, rounded to the nearest integer`。
- **L1456 EN**: Continues the surrounding expression or declaration: `within a floating-point type. Halfway cases are`.
  **L1456 CN**: 继续构造周围的表达式或声明：`within a floating-point type. Halfway cases are`。
- **L1457 EN**: Continues the surrounding expression or declaration: `rounded to the nearest even value.`.
  **L1457 CN**: 继续构造周围的表达式或声明：`rounded to the nearest even value.`。
- **L1458 EN**: Adds a standalone statement or declaration: `}];`.
  **L1458 CN**: 添加一条独立语句或声明：`}];`。
- **L1459 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1459 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1460 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1460 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1461 EN**: Closes the current lexical scope or compound statement.
  **L1461 CN**: 结束当前词法作用域或复合语句块。
- **L1462 EN**: Blank line separating nearby declarations or logic blocks.
  **L1462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1463 EN**: Comment explains nearby logic, constraints, or intent: `Returns the reciprocal of the square root of the specified value,`.
  **L1463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the reciprocal of the square root of the specified value,`。
- **L1464 EN**: Comment explains nearby logic, constraints, or intent: `ie 1 / sqrt(x).`.
  **L1464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ie 1 / sqrt(x).`。

### Lines 1465-1488

````tablegen
def hlsl_rsqrt : HLSLOneArgBuiltin<"rsqrt", "__builtin_hlsl_elementwise_rsqrt"> {
  let Doc = [{
\fn T rsqrt(T x)
\brief Returns the reciprocal of the square root of the specified value.
ie 1 / sqrt( \a x).
\param x The specified input value.

This function uses the following formula: 1 / sqrt(x).
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns input value, Val, clamped within the range of 0.0f to 1.0f.
def hlsl_saturate : HLSLOneArgBuiltin<"saturate", "__builtin_hlsl_elementwise_saturate"> {
  let Doc = [{
\fn T saturate(T Val)
\brief Returns input value, \a Val, clamped within the range of 0.0f
to 1.0f.
\param Val The input value.
}];
  let VaryingTypes = AllFloatTypes;
  let VaryingMatDims = [];
}
````
- **L1465 EN**: Declares TableGen def record `hlsl_rsqrt`.
  **L1465 CN**: 声明 TableGen def 记录 `hlsl_rsqrt`。
- **L1466 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1466 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1467 EN**: Continues logic associated with callable symbol `rsqrt`.
  **L1467 CN**: 继续与可调用符号 `rsqrt` 相关的逻辑。
- **L1468 EN**: Continues the surrounding expression or declaration: `\brief Returns the reciprocal of the square root of the specified value.`.
  **L1468 CN**: 继续构造周围的表达式或声明：`\brief Returns the reciprocal of the square root of the specified value.`。
- **L1469 EN**: Continues logic associated with callable symbol `sqrt`.
  **L1469 CN**: 继续与可调用符号 `sqrt` 相关的逻辑。
- **L1470 EN**: Continues the surrounding expression or declaration: `\param x The specified input value.`.
  **L1470 CN**: 继续构造周围的表达式或声明：`\param x The specified input value.`。
- **L1471 EN**: Blank line separating nearby declarations or logic blocks.
  **L1471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1472 EN**: Continues logic associated with callable symbol `sqrt`.
  **L1472 CN**: 继续与可调用符号 `sqrt` 相关的逻辑。
- **L1473 EN**: Adds a standalone statement or declaration: `}];`.
  **L1473 CN**: 添加一条独立语句或声明：`}];`。
- **L1474 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1474 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1475 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1475 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1476 EN**: Closes the current lexical scope or compound statement.
  **L1476 CN**: 结束当前词法作用域或复合语句块。
- **L1477 EN**: Blank line separating nearby declarations or logic blocks.
  **L1477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1478 EN**: Comment explains nearby logic, constraints, or intent: `Returns input value, Val, clamped within the range of 0.0f to 1.0f.`.
  **L1478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns input value, Val, clamped within the range of 0.0f to 1.0f.`。
- **L1479 EN**: Declares TableGen def record `hlsl_saturate`.
  **L1479 CN**: 声明 TableGen def 记录 `hlsl_saturate`。
- **L1480 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1480 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1481 EN**: Continues logic associated with callable symbol `saturate`.
  **L1481 CN**: 继续与可调用符号 `saturate` 相关的逻辑。
- **L1482 EN**: Continues the surrounding expression or declaration: `\brief Returns input value, \a Val, clamped within the range of 0.0f`.
  **L1482 CN**: 继续构造周围的表达式或声明：`\brief Returns input value, \a Val, clamped within the range of 0.0f`。
- **L1483 EN**: Continues the surrounding expression or declaration: `to 1.0f.`.
  **L1483 CN**: 继续构造周围的表达式或声明：`to 1.0f.`。
- **L1484 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1484 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1485 EN**: Adds a standalone statement or declaration: `}];`.
  **L1485 CN**: 添加一条独立语句或声明：`}];`。
- **L1486 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllFloatTypes;`.
  **L1486 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllFloatTypes;`。
- **L1487 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1487 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1488 EN**: Closes the current lexical scope or compound statement.
  **L1488 CN**: 结束当前词法作用域或复合语句块。

### Lines 1489-1512

````tablegen

// Returns -1 if Val is less than zero; 0 if Val equals zero; and 1 if Val is
// greater than zero.
def hlsl_sign : HLSLOneArgBuiltin<"sign", "__builtin_hlsl_elementwise_sign"> {
  let Doc = [{
\fn T sign(T Val)
\brief Returns -1 if \a Val is less than zero; 0 if \a Val equals zero; and
1 if \a Val is greater than zero.
\param Val The input value.
}];
  let ReturnType = VaryingShape<IntTy>;
  let VaryingTypes = AllNumericTypes;
  let VaryingMatDims = [];
}

// Returns the sine of the input value, Val.
def hlsl_sin : HLSLOneArgBuiltin<"sin", "__builtin_elementwise_sin"> {
  let Doc = [{
\fn T sin(T Val)
\brief Returns the sine of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
````
- **L1489 EN**: Blank line separating nearby declarations or logic blocks.
  **L1489 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1490 EN**: Comment explains nearby logic, constraints, or intent: `Returns -1 if Val is less than zero; 0 if Val equals zero; and 1 if Val is`.
  **L1490 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns -1 if Val is less than zero; 0 if Val equals zero; and 1 if Val is`。
- **L1491 EN**: Comment explains nearby logic, constraints, or intent: `greater than zero.`.
  **L1491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`greater than zero.`。
- **L1492 EN**: Declares TableGen def record `hlsl_sign`.
  **L1492 CN**: 声明 TableGen def 记录 `hlsl_sign`。
- **L1493 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1493 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1494 EN**: Continues logic associated with callable symbol `sign`.
  **L1494 CN**: 继续与可调用符号 `sign` 相关的逻辑。
- **L1495 EN**: Continues the surrounding expression or declaration: `\brief Returns -1 if \a Val is less than zero; 0 if \a Val equals zero; and`.
  **L1495 CN**: 继续构造周围的表达式或声明：`\brief Returns -1 if \a Val is less than zero; 0 if \a Val equals zero; and`。
- **L1496 EN**: Continues the surrounding expression or declaration: `1 if \a Val is greater than zero.`.
  **L1496 CN**: 继续构造周围的表达式或声明：`1 if \a Val is greater than zero.`。
- **L1497 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1497 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1498 EN**: Adds a standalone statement or declaration: `}];`.
  **L1498 CN**: 添加一条独立语句或声明：`}];`。
- **L1499 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<IntTy>;`.
  **L1499 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<IntTy>;`。
- **L1500 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1500 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1501 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1501 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1502 EN**: Closes the current lexical scope or compound statement.
  **L1502 CN**: 结束当前词法作用域或复合语句块。
- **L1503 EN**: Blank line separating nearby declarations or logic blocks.
  **L1503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1504 EN**: Comment explains nearby logic, constraints, or intent: `Returns the sine of the input value, Val.`.
  **L1504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the sine of the input value, Val.`。
- **L1505 EN**: Declares TableGen def record `hlsl_sin`.
  **L1505 CN**: 声明 TableGen def 记录 `hlsl_sin`。
- **L1506 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1506 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1507 EN**: Continues logic associated with callable symbol `sin`.
  **L1507 CN**: 继续与可调用符号 `sin` 相关的逻辑。
- **L1508 EN**: Continues the surrounding expression or declaration: `\brief Returns the sine of the input value, \a Val.`.
  **L1508 CN**: 继续构造周围的表达式或声明：`\brief Returns the sine of the input value, \a Val.`。
- **L1509 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1509 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1510 EN**: Adds a standalone statement or declaration: `}];`.
  **L1510 CN**: 添加一条独立语句或声明：`}];`。
- **L1511 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1511 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1512 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1512 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。

### Lines 1513-1536

````tablegen
}

// Returns the hyperbolic sine of the input value, Val.
def hlsl_sinh : HLSLOneArgBuiltin<"sinh", "__builtin_elementwise_sinh"> {
  let Doc = [{
\fn T sinh(T Val)
\brief Returns the hyperbolic sine of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns a smooth Hermite interpolation between 0 and 1.
def hlsl_smoothstep : HLSLThreeArgDetail<"smoothstep", "smoothstep_impl"> {
  let Doc = [{
\fn T smoothstep(T Min, T Max, T X)
\brief Returns a smooth Hermite interpolation between 0 and 1, if \a X is in
the range [\a Min, \a Max].
\param Min The minimum range of the x parameter.
\param Max The maximum range of the x parameter.
\param X The specified value to be interpolated.

The return value is 0.0 if \a X <= \a Min and 1.0 if \a X >= \a Max. When \a
````
- **L1513 EN**: Closes the current lexical scope or compound statement.
  **L1513 CN**: 结束当前词法作用域或复合语句块。
- **L1514 EN**: Blank line separating nearby declarations or logic blocks.
  **L1514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1515 EN**: Comment explains nearby logic, constraints, or intent: `Returns the hyperbolic sine of the input value, Val.`.
  **L1515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the hyperbolic sine of the input value, Val.`。
- **L1516 EN**: Declares TableGen def record `hlsl_sinh`.
  **L1516 CN**: 声明 TableGen def 记录 `hlsl_sinh`。
- **L1517 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1517 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1518 EN**: Continues logic associated with callable symbol `sinh`.
  **L1518 CN**: 继续与可调用符号 `sinh` 相关的逻辑。
- **L1519 EN**: Continues the surrounding expression or declaration: `\brief Returns the hyperbolic sine of the input value, \a Val.`.
  **L1519 CN**: 继续构造周围的表达式或声明：`\brief Returns the hyperbolic sine of the input value, \a Val.`。
- **L1520 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1520 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1521 EN**: Adds a standalone statement or declaration: `}];`.
  **L1521 CN**: 添加一条独立语句或声明：`}];`。
- **L1522 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1522 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1523 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1523 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1524 EN**: Closes the current lexical scope or compound statement.
  **L1524 CN**: 结束当前词法作用域或复合语句块。
- **L1525 EN**: Blank line separating nearby declarations or logic blocks.
  **L1525 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1526 EN**: Comment explains nearby logic, constraints, or intent: `Returns a smooth Hermite interpolation between 0 and 1.`.
  **L1526 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a smooth Hermite interpolation between 0 and 1.`。
- **L1527 EN**: Declares TableGen def record `hlsl_smoothstep`.
  **L1527 CN**: 声明 TableGen def 记录 `hlsl_smoothstep`。
- **L1528 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1528 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1529 EN**: Continues logic associated with callable symbol `smoothstep`.
  **L1529 CN**: 继续与可调用符号 `smoothstep` 相关的逻辑。
- **L1530 EN**: Continues the surrounding expression or declaration: `\brief Returns a smooth Hermite interpolation between 0 and 1, if \a X is in`.
  **L1530 CN**: 继续构造周围的表达式或声明：`\brief Returns a smooth Hermite interpolation between 0 and 1, if \a X is in`。
- **L1531 EN**: Continues the surrounding expression or declaration: `the range [\a Min, \a Max].`.
  **L1531 CN**: 继续构造周围的表达式或声明：`the range [\a Min, \a Max].`。
- **L1532 EN**: Continues the surrounding expression or declaration: `\param Min The minimum range of the x parameter.`.
  **L1532 CN**: 继续构造周围的表达式或声明：`\param Min The minimum range of the x parameter.`。
- **L1533 EN**: Continues the surrounding expression or declaration: `\param Max The maximum range of the x parameter.`.
  **L1533 CN**: 继续构造周围的表达式或声明：`\param Max The maximum range of the x parameter.`。
- **L1534 EN**: Continues the surrounding expression or declaration: `\param X The specified value to be interpolated.`.
  **L1534 CN**: 继续构造周围的表达式或声明：`\param X The specified value to be interpolated.`。
- **L1535 EN**: Blank line separating nearby declarations or logic blocks.
  **L1535 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1536 EN**: Continues the surrounding expression or declaration: `The return value is 0.0 if \a X <= \a Min and 1.0 if \a X >= \a Max. When \a`.
  **L1536 CN**: 继续构造周围的表达式或声明：`The return value is 0.0 if \a X <= \a Min and 1.0 if \a X >= \a Max. When \a`。

### Lines 1537-1560

````tablegen
Min < \a X < \a Max, the function performs smooth Hermite interpolation
between 0 and 1.
}];
  let ParamNames = ["Min", "Max", "X"];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the square root of the input value, Val.
def hlsl_sqrt : HLSLOneArgBuiltin<"sqrt", "__builtin_elementwise_sqrt"> {
  let Doc = [{
\fn T sqrt(T Val)
\brief Returns the square root of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns 1 if the x parameter is greater than or equal to the y parameter;
// otherwise, 0.
def hlsl_step : HLSLTwoArgBuiltin<"step", "__builtin_hlsl_step"> {
  let Doc = [{
\fn T step(T y, T x)
````
- **L1537 EN**: Continues the surrounding expression or declaration: `Min < \a X < \a Max, the function performs smooth Hermite interpolation`.
  **L1537 CN**: 继续构造周围的表达式或声明：`Min < \a X < \a Max, the function performs smooth Hermite interpolation`。
- **L1538 EN**: Continues the surrounding expression or declaration: `between 0 and 1.`.
  **L1538 CN**: 继续构造周围的表达式或声明：`between 0 and 1.`。
- **L1539 EN**: Adds a standalone statement or declaration: `}];`.
  **L1539 CN**: 添加一条独立语句或声明：`}];`。
- **L1540 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ParamNames = ["Min", "Max", "X"];`.
  **L1540 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ParamNames = ["Min", "Max", "X"];`。
- **L1541 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1541 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1542 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1542 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1543 EN**: Closes the current lexical scope or compound statement.
  **L1543 CN**: 结束当前词法作用域或复合语句块。
- **L1544 EN**: Blank line separating nearby declarations or logic blocks.
  **L1544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1545 EN**: Comment explains nearby logic, constraints, or intent: `Returns the square root of the input value, Val.`.
  **L1545 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the square root of the input value, Val.`。
- **L1546 EN**: Declares TableGen def record `hlsl_sqrt`.
  **L1546 CN**: 声明 TableGen def 记录 `hlsl_sqrt`。
- **L1547 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1547 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1548 EN**: Continues logic associated with callable symbol `sqrt`.
  **L1548 CN**: 继续与可调用符号 `sqrt` 相关的逻辑。
- **L1549 EN**: Continues the surrounding expression or declaration: `\brief Returns the square root of the input value, \a Val.`.
  **L1549 CN**: 继续构造周围的表达式或声明：`\brief Returns the square root of the input value, \a Val.`。
- **L1550 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1550 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1551 EN**: Adds a standalone statement or declaration: `}];`.
  **L1551 CN**: 添加一条独立语句或声明：`}];`。
- **L1552 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1552 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1553 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1553 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1554 EN**: Closes the current lexical scope or compound statement.
  **L1554 CN**: 结束当前词法作用域或复合语句块。
- **L1555 EN**: Blank line separating nearby declarations or logic blocks.
  **L1555 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1556 EN**: Comment explains nearby logic, constraints, or intent: `Returns 1 if the x parameter is greater than or equal to the y parameter;`.
  **L1556 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns 1 if the x parameter is greater than or equal to the y parameter;`。
- **L1557 EN**: Comment explains nearby logic, constraints, or intent: `otherwise, 0.`.
  **L1557 CN**: 注释解释附近代码的逻辑、约束或设计意图：`otherwise, 0.`。
- **L1558 EN**: Declares TableGen def record `hlsl_step`.
  **L1558 CN**: 声明 TableGen def 记录 `hlsl_step`。
- **L1559 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1559 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1560 EN**: Continues logic associated with callable symbol `step`.
  **L1560 CN**: 继续与可调用符号 `step` 相关的逻辑。

### Lines 1561-1584

````tablegen
\brief Returns 1 if the x parameter is greater than or equal to the y
parameter; otherwise, 0.
\param y [in] The first floating-point value to compare.
\param x [in] The second floating-point value to compare.

Step is based on the following formula: (x >= y) ? 1 : 0
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the tangent of the input value, Val.
def hlsl_tan : HLSLOneArgBuiltin<"tan", "__builtin_elementwise_tan"> {
  let Doc = [{
\fn T tan(T Val)
\brief Returns the tangent of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the hyperbolic tangent of the input value, Val.
def hlsl_tanh : HLSLOneArgBuiltin<"tanh", "__builtin_elementwise_tanh"> {
````
- **L1561 EN**: Continues the surrounding expression or declaration: `\brief Returns 1 if the x parameter is greater than or equal to the y`.
  **L1561 CN**: 继续构造周围的表达式或声明：`\brief Returns 1 if the x parameter is greater than or equal to the y`。
- **L1562 EN**: Continues the surrounding expression or declaration: `parameter; otherwise, 0.`.
  **L1562 CN**: 继续构造周围的表达式或声明：`parameter; otherwise, 0.`。
- **L1563 EN**: Continues the surrounding expression or declaration: `\param y [in] The first floating-point value to compare.`.
  **L1563 CN**: 继续构造周围的表达式或声明：`\param y [in] The first floating-point value to compare.`。
- **L1564 EN**: Continues the surrounding expression or declaration: `\param x [in] The second floating-point value to compare.`.
  **L1564 CN**: 继续构造周围的表达式或声明：`\param x [in] The second floating-point value to compare.`。
- **L1565 EN**: Blank line separating nearby declarations or logic blocks.
  **L1565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1566 EN**: Continues logic associated with callable symbol `formula:`.
  **L1566 CN**: 继续与可调用符号 `formula:` 相关的逻辑。
- **L1567 EN**: Adds a standalone statement or declaration: `}];`.
  **L1567 CN**: 添加一条独立语句或声明：`}];`。
- **L1568 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1568 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1569 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1569 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1570 EN**: Closes the current lexical scope or compound statement.
  **L1570 CN**: 结束当前词法作用域或复合语句块。
- **L1571 EN**: Blank line separating nearby declarations or logic blocks.
  **L1571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1572 EN**: Comment explains nearby logic, constraints, or intent: `Returns the tangent of the input value, Val.`.
  **L1572 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the tangent of the input value, Val.`。
- **L1573 EN**: Declares TableGen def record `hlsl_tan`.
  **L1573 CN**: 声明 TableGen def 记录 `hlsl_tan`。
- **L1574 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1574 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1575 EN**: Continues logic associated with callable symbol `tan`.
  **L1575 CN**: 继续与可调用符号 `tan` 相关的逻辑。
- **L1576 EN**: Continues the surrounding expression or declaration: `\brief Returns the tangent of the input value, \a Val.`.
  **L1576 CN**: 继续构造周围的表达式或声明：`\brief Returns the tangent of the input value, \a Val.`。
- **L1577 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1577 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1578 EN**: Adds a standalone statement or declaration: `}];`.
  **L1578 CN**: 添加一条独立语句或声明：`}];`。
- **L1579 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1579 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1580 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1580 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1581 EN**: Closes the current lexical scope or compound statement.
  **L1581 CN**: 结束当前词法作用域或复合语句块。
- **L1582 EN**: Blank line separating nearby declarations or logic blocks.
  **L1582 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1583 EN**: Comment explains nearby logic, constraints, or intent: `Returns the hyperbolic tangent of the input value, Val.`.
  **L1583 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the hyperbolic tangent of the input value, Val.`。
- **L1584 EN**: Declares TableGen def record `hlsl_tanh`.
  **L1584 CN**: 声明 TableGen def 记录 `hlsl_tanh`。

### Lines 1585-1608

````tablegen
  let Doc = [{
\fn T tanh(T Val)
\brief Returns the hyperbolic tangent of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Returns the truncated integer value of the input value, Val.
def hlsl_trunc : HLSLOneArgBuiltin<"trunc", "__builtin_elementwise_trunc"> {
  let Doc = [{
\fn T trunc(T Val)
\brief Returns the truncated integer value of the input value, \a Val.
\param Val The input value.
}];
  let VaryingTypes = [HalfTy, FloatTy];
  let VaryingMatDims = [];
}

// Evaluates a value for all active invocations in the group. The result is true
// if Value is equal for all active invocations in the group.
def hlsl_wave_active_all_equal :
    HLSLOneArgBuiltin<"WaveActiveAllEqual",
````
- **L1585 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1585 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1586 EN**: Continues logic associated with callable symbol `tanh`.
  **L1586 CN**: 继续与可调用符号 `tanh` 相关的逻辑。
- **L1587 EN**: Continues the surrounding expression or declaration: `\brief Returns the hyperbolic tangent of the input value, \a Val.`.
  **L1587 CN**: 继续构造周围的表达式或声明：`\brief Returns the hyperbolic tangent of the input value, \a Val.`。
- **L1588 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1588 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1589 EN**: Adds a standalone statement or declaration: `}];`.
  **L1589 CN**: 添加一条独立语句或声明：`}];`。
- **L1590 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1590 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1591 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1591 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1592 EN**: Closes the current lexical scope or compound statement.
  **L1592 CN**: 结束当前词法作用域或复合语句块。
- **L1593 EN**: Blank line separating nearby declarations or logic blocks.
  **L1593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1594 EN**: Comment explains nearby logic, constraints, or intent: `Returns the truncated integer value of the input value, Val.`.
  **L1594 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the truncated integer value of the input value, Val.`。
- **L1595 EN**: Declares TableGen def record `hlsl_trunc`.
  **L1595 CN**: 声明 TableGen def 记录 `hlsl_trunc`。
- **L1596 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1596 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1597 EN**: Continues logic associated with callable symbol `trunc`.
  **L1597 CN**: 继续与可调用符号 `trunc` 相关的逻辑。
- **L1598 EN**: Continues the surrounding expression or declaration: `\brief Returns the truncated integer value of the input value, \a Val.`.
  **L1598 CN**: 继续构造周围的表达式或声明：`\brief Returns the truncated integer value of the input value, \a Val.`。
- **L1599 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1599 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1600 EN**: Adds a standalone statement or declaration: `}];`.
  **L1600 CN**: 添加一条独立语句或声明：`}];`。
- **L1601 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [HalfTy, FloatTy];`.
  **L1601 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [HalfTy, FloatTy];`。
- **L1602 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1602 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1603 EN**: Closes the current lexical scope or compound statement.
  **L1603 CN**: 结束当前词法作用域或复合语句块。
- **L1604 EN**: Blank line separating nearby declarations or logic blocks.
  **L1604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1605 EN**: Comment explains nearby logic, constraints, or intent: `Evaluates a value for all active invocations in the group. The result is true`.
  **L1605 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Evaluates a value for all active invocations in the group. The result is true`。
- **L1606 EN**: Comment explains nearby logic, constraints, or intent: `if Value is equal for all active invocations in the group.`.
  **L1606 CN**: 注释解释附近代码的逻辑、约束或设计意图：`if Value is equal for all active invocations in the group.`。
- **L1607 EN**: Declares TableGen def record `hlsl_wave_active_all_equal`.
  **L1607 CN**: 声明 TableGen def 记录 `hlsl_wave_active_all_equal`。
- **L1608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLOneArgBuiltin<"WaveActiveAllEqual",`.
  **L1608 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLOneArgBuiltin<"WaveActiveAllEqual",`。

### Lines 1609-1632

````tablegen
                "__builtin_hlsl_wave_active_all_equal"> {
  let Doc = [{
\brief Evaluates a value for all active invocations in the group. The
result is true if Value is equal for all active invocations in the
group. Otherwise, the result is false.
\param Value The value to compare with
\return True if all values across all lanes are equal, false otherwise
}];
  let ReturnType = VaryingShape<BoolTy>;
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}

// Returns true if the expression is true in all active lanes in the current
// wave.
def hlsl_wave_active_all_true :
    HLSLBuiltin<"WaveActiveAllTrue", "__builtin_hlsl_wave_active_all_true"> {
  let Doc = [{
\brief Returns true if the expression is true in all active lanes in the
current wave.
\param Val The boolean expression to evaluate.
\return True if the expression is true in all lanes.
````
- **L1609 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_wave_active_all_equal"> {`.
  **L1609 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_wave_active_all_equal"> {`。
- **L1610 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1610 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1611 EN**: Continues the surrounding expression or declaration: `\brief Evaluates a value for all active invocations in the group. The`.
  **L1611 CN**: 继续构造周围的表达式或声明：`\brief Evaluates a value for all active invocations in the group. The`。
- **L1612 EN**: Continues the surrounding expression or declaration: `result is true if Value is equal for all active invocations in the`.
  **L1612 CN**: 继续构造周围的表达式或声明：`result is true if Value is equal for all active invocations in the`。
- **L1613 EN**: Continues the surrounding expression or declaration: `group. Otherwise, the result is false.`.
  **L1613 CN**: 继续构造周围的表达式或声明：`group. Otherwise, the result is false.`。
- **L1614 EN**: Continues the surrounding expression or declaration: `\param Value The value to compare with`.
  **L1614 CN**: 继续构造周围的表达式或声明：`\param Value The value to compare with`。
- **L1615 EN**: Continues the surrounding expression or declaration: `\return True if all values across all lanes are equal, false otherwise`.
  **L1615 CN**: 继续构造周围的表达式或声明：`\return True if all values across all lanes are equal, false otherwise`。
- **L1616 EN**: Adds a standalone statement or declaration: `}];`.
  **L1616 CN**: 添加一条独立语句或声明：`}];`。
- **L1617 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<BoolTy>;`.
  **L1617 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<BoolTy>;`。
- **L1618 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1618 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1619 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1619 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1620 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1620 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1621 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1621 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1622 EN**: Closes the current lexical scope or compound statement.
  **L1622 CN**: 结束当前词法作用域或复合语句块。
- **L1623 EN**: Blank line separating nearby declarations or logic blocks.
  **L1623 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1624 EN**: Comment explains nearby logic, constraints, or intent: `Returns true if the expression is true in all active lanes in the current`.
  **L1624 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns true if the expression is true in all active lanes in the current`。
- **L1625 EN**: Comment explains nearby logic, constraints, or intent: `wave.`.
  **L1625 CN**: 注释解释附近代码的逻辑、约束或设计意图：`wave.`。
- **L1626 EN**: Declares TableGen def record `hlsl_wave_active_all_true`.
  **L1626 CN**: 声明 TableGen def 记录 `hlsl_wave_active_all_true`。
- **L1627 EN**: Continues the surrounding expression or declaration: `HLSLBuiltin<"WaveActiveAllTrue", "__builtin_hlsl_wave_active_all_true"> {`.
  **L1627 CN**: 继续构造周围的表达式或声明：`HLSLBuiltin<"WaveActiveAllTrue", "__builtin_hlsl_wave_active_all_true"> {`。
- **L1628 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1628 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1629 EN**: Continues the surrounding expression or declaration: `\brief Returns true if the expression is true in all active lanes in the`.
  **L1629 CN**: 继续构造周围的表达式或声明：`\brief Returns true if the expression is true in all active lanes in the`。
- **L1630 EN**: Continues the surrounding expression or declaration: `current wave.`.
  **L1630 CN**: 继续构造周围的表达式或声明：`current wave.`。
- **L1631 EN**: Continues the surrounding expression or declaration: `\param Val The boolean expression to evaluate.`.
  **L1631 CN**: 继续构造周围的表达式或声明：`\param Val The boolean expression to evaluate.`。
- **L1632 EN**: Continues the surrounding expression or declaration: `\return True if the expression is true in all lanes.`.
  **L1632 CN**: 继续构造周围的表达式或声明：`\return True if the expression is true in all lanes.`。

### Lines 1633-1656

````tablegen
}];
  let Args = [Varying];
  let ReturnType = Varying;
  let VaryingTypes = [BoolTy];
  let VaryingScalar = 1;
  let IsConvergent = 1;
  let Availability = SM6_0;
}

// Returns true if the expression is true in any active lane in the current
// wave.
def hlsl_wave_active_any_true :
    HLSLBuiltin<"WaveActiveAnyTrue", "__builtin_hlsl_wave_active_any_true"> {
  let Doc = [{
\brief Returns true if the expression is true in any active lane in the
current wave.
\param Val The boolean expression to evaluate.
\return True if the expression is true in any lane.
}];
  let Args = [Varying];
  let ReturnType = Varying;
  let VaryingTypes = [BoolTy];
  let VaryingScalar = 1;
  let IsConvergent = 1;
````
- **L1633 EN**: Adds a standalone statement or declaration: `}];`.
  **L1633 CN**: 添加一条独立语句或声明：`}];`。
- **L1634 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L1634 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L1635 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L1635 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L1636 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [BoolTy];`.
  **L1636 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [BoolTy];`。
- **L1637 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L1637 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L1638 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1638 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1639 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1639 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1640 EN**: Closes the current lexical scope or compound statement.
  **L1640 CN**: 结束当前词法作用域或复合语句块。
- **L1641 EN**: Blank line separating nearby declarations or logic blocks.
  **L1641 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1642 EN**: Comment explains nearby logic, constraints, or intent: `Returns true if the expression is true in any active lane in the current`.
  **L1642 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns true if the expression is true in any active lane in the current`。
- **L1643 EN**: Comment explains nearby logic, constraints, or intent: `wave.`.
  **L1643 CN**: 注释解释附近代码的逻辑、约束或设计意图：`wave.`。
- **L1644 EN**: Declares TableGen def record `hlsl_wave_active_any_true`.
  **L1644 CN**: 声明 TableGen def 记录 `hlsl_wave_active_any_true`。
- **L1645 EN**: Continues the surrounding expression or declaration: `HLSLBuiltin<"WaveActiveAnyTrue", "__builtin_hlsl_wave_active_any_true"> {`.
  **L1645 CN**: 继续构造周围的表达式或声明：`HLSLBuiltin<"WaveActiveAnyTrue", "__builtin_hlsl_wave_active_any_true"> {`。
- **L1646 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1646 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1647 EN**: Continues the surrounding expression or declaration: `\brief Returns true if the expression is true in any active lane in the`.
  **L1647 CN**: 继续构造周围的表达式或声明：`\brief Returns true if the expression is true in any active lane in the`。
- **L1648 EN**: Continues the surrounding expression or declaration: `current wave.`.
  **L1648 CN**: 继续构造周围的表达式或声明：`current wave.`。
- **L1649 EN**: Continues the surrounding expression or declaration: `\param Val The boolean expression to evaluate.`.
  **L1649 CN**: 继续构造周围的表达式或声明：`\param Val The boolean expression to evaluate.`。
- **L1650 EN**: Continues the surrounding expression or declaration: `\return True if the expression is true in any lane.`.
  **L1650 CN**: 继续构造周围的表达式或声明：`\return True if the expression is true in any lane.`。
- **L1651 EN**: Adds a standalone statement or declaration: `}];`.
  **L1651 CN**: 添加一条独立语句或声明：`}];`。
- **L1652 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L1652 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L1653 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = Varying;`.
  **L1653 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = Varying;`。
- **L1654 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [BoolTy];`.
  **L1654 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [BoolTy];`。
- **L1655 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L1655 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L1656 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1656 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。

### Lines 1657-1680

````tablegen
  let Availability = SM6_0;
}

// Returns a uint4 containing a bitmask of the evaluation of the boolean
// expression for all active lanes in the current wave.
def hlsl_wave_active_ballot :
    HLSLBuiltin<"WaveActiveBallot", "__builtin_hlsl_wave_active_ballot"> {
  let Doc = [{
\brief Returns a uint4 containing a bitmask of the evaluation of the
boolean expression for all active lanes in the current wave.
The least-significant bit corresponds to the lane with index zero.
The bits corresponding to inactive lanes will be zero. The bits that
are greater than or equal to WaveGetLaneCount will be zero.
\param Val The boolean expression to evaluate.
\return uint4 bitmask
}];
  let Args = [Varying];
  let ReturnType = VectorType<UIntTy, 4>;
  let VaryingTypes = [BoolTy];
  let VaryingScalar = 1;
  let IsConvergent = 1;
  let Availability = SM6_0;
}

````
- **L1657 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1657 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1658 EN**: Closes the current lexical scope or compound statement.
  **L1658 CN**: 结束当前词法作用域或复合语句块。
- **L1659 EN**: Blank line separating nearby declarations or logic blocks.
  **L1659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1660 EN**: Comment explains nearby logic, constraints, or intent: `Returns a uint4 containing a bitmask of the evaluation of the boolean`.
  **L1660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a uint4 containing a bitmask of the evaluation of the boolean`。
- **L1661 EN**: Comment explains nearby logic, constraints, or intent: `expression for all active lanes in the current wave.`.
  **L1661 CN**: 注释解释附近代码的逻辑、约束或设计意图：`expression for all active lanes in the current wave.`。
- **L1662 EN**: Declares TableGen def record `hlsl_wave_active_ballot`.
  **L1662 CN**: 声明 TableGen def 记录 `hlsl_wave_active_ballot`。
- **L1663 EN**: Continues the surrounding expression or declaration: `HLSLBuiltin<"WaveActiveBallot", "__builtin_hlsl_wave_active_ballot"> {`.
  **L1663 CN**: 继续构造周围的表达式或声明：`HLSLBuiltin<"WaveActiveBallot", "__builtin_hlsl_wave_active_ballot"> {`。
- **L1664 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1664 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1665 EN**: Continues the surrounding expression or declaration: `\brief Returns a uint4 containing a bitmask of the evaluation of the`.
  **L1665 CN**: 继续构造周围的表达式或声明：`\brief Returns a uint4 containing a bitmask of the evaluation of the`。
- **L1666 EN**: Continues the surrounding expression or declaration: `boolean expression for all active lanes in the current wave.`.
  **L1666 CN**: 继续构造周围的表达式或声明：`boolean expression for all active lanes in the current wave.`。
- **L1667 EN**: Continues the surrounding expression or declaration: `The least-significant bit corresponds to the lane with index zero.`.
  **L1667 CN**: 继续构造周围的表达式或声明：`The least-significant bit corresponds to the lane with index zero.`。
- **L1668 EN**: Continues the surrounding expression or declaration: `The bits corresponding to inactive lanes will be zero. The bits that`.
  **L1668 CN**: 继续构造周围的表达式或声明：`The bits corresponding to inactive lanes will be zero. The bits that`。
- **L1669 EN**: Continues the surrounding expression or declaration: `are greater than or equal to WaveGetLaneCount will be zero.`.
  **L1669 CN**: 继续构造周围的表达式或声明：`are greater than or equal to WaveGetLaneCount will be zero.`。
- **L1670 EN**: Continues the surrounding expression or declaration: `\param Val The boolean expression to evaluate.`.
  **L1670 CN**: 继续构造周围的表达式或声明：`\param Val The boolean expression to evaluate.`。
- **L1671 EN**: Continues the surrounding expression or declaration: `\return uint4 bitmask`.
  **L1671 CN**: 继续构造周围的表达式或声明：`\return uint4 bitmask`。
- **L1672 EN**: Adds a standalone statement or declaration: `}];`.
  **L1672 CN**: 添加一条独立语句或声明：`}];`。
- **L1673 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L1673 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L1674 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VectorType<UIntTy, 4>;`.
  **L1674 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VectorType<UIntTy, 4>;`。
- **L1675 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [BoolTy];`.
  **L1675 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [BoolTy];`。
- **L1676 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L1676 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L1677 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1677 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1678 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1678 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1679 EN**: Closes the current lexical scope or compound statement.
  **L1679 CN**: 结束当前词法作用域或复合语句块。
- **L1680 EN**: Blank line separating nearby declarations or logic blocks.
  **L1680 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1681-1704

````tablegen
// Returns the bitwise AND of all values across all active non-helper lanes.
def hlsl_wave_active_bit_and :
    HLSLOneArgBuiltin<"WaveActiveBitAnd",
                      "__builtin_hlsl_wave_active_bit_and"> {
  let VaryingTypes = [UIntTy, UInt64Ty];
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}

// Returns the bitwise OR of all values across all active non-helper lanes.
def hlsl_wave_active_bit_or :
    HLSLOneArgBuiltin<"WaveActiveBitOr",
                      "__builtin_hlsl_wave_active_bit_or"> {
  let Doc = [{
\brief Returns the bitwise OR of all the values of <expr> across all active
non-helper lanes in the current wave, and replicates it back to
all active non-helper lanes.
}];
  let VaryingTypes = [UIntTy, UInt64Ty];
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}
````
- **L1681 EN**: Comment explains nearby logic, constraints, or intent: `Returns the bitwise AND of all values across all active non-helper lanes.`.
  **L1681 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the bitwise AND of all values across all active non-helper lanes.`。
- **L1682 EN**: Declares TableGen def record `hlsl_wave_active_bit_and`.
  **L1682 CN**: 声明 TableGen def 记录 `hlsl_wave_active_bit_and`。
- **L1683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLOneArgBuiltin<"WaveActiveBitAnd",`.
  **L1683 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLOneArgBuiltin<"WaveActiveBitAnd",`。
- **L1684 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_wave_active_bit_and"> {`.
  **L1684 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_wave_active_bit_and"> {`。
- **L1685 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [UIntTy, UInt64Ty];`.
  **L1685 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [UIntTy, UInt64Ty];`。
- **L1686 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1686 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1687 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1687 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1688 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1688 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1689 EN**: Closes the current lexical scope or compound statement.
  **L1689 CN**: 结束当前词法作用域或复合语句块。
- **L1690 EN**: Blank line separating nearby declarations or logic blocks.
  **L1690 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1691 EN**: Comment explains nearby logic, constraints, or intent: `Returns the bitwise OR of all values across all active non-helper lanes.`.
  **L1691 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the bitwise OR of all values across all active non-helper lanes.`。
- **L1692 EN**: Declares TableGen def record `hlsl_wave_active_bit_or`.
  **L1692 CN**: 声明 TableGen def 记录 `hlsl_wave_active_bit_or`。
- **L1693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLOneArgBuiltin<"WaveActiveBitOr",`.
  **L1693 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLOneArgBuiltin<"WaveActiveBitOr",`。
- **L1694 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_wave_active_bit_or"> {`.
  **L1694 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_wave_active_bit_or"> {`。
- **L1695 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1695 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1696 EN**: Continues the surrounding expression or declaration: `\brief Returns the bitwise OR of all the values of <expr> across all active`.
  **L1696 CN**: 继续构造周围的表达式或声明：`\brief Returns the bitwise OR of all the values of <expr> across all active`。
- **L1697 EN**: Continues the surrounding expression or declaration: `non-helper lanes in the current wave, and replicates it back to`.
  **L1697 CN**: 继续构造周围的表达式或声明：`non-helper lanes in the current wave, and replicates it back to`。
- **L1698 EN**: Continues the surrounding expression or declaration: `all active non-helper lanes.`.
  **L1698 CN**: 继续构造周围的表达式或声明：`all active non-helper lanes.`。
- **L1699 EN**: Adds a standalone statement or declaration: `}];`.
  **L1699 CN**: 添加一条独立语句或声明：`}];`。
- **L1700 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [UIntTy, UInt64Ty];`.
  **L1700 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [UIntTy, UInt64Ty];`。
- **L1701 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1701 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1702 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1702 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1703 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1703 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1704 EN**: Closes the current lexical scope or compound statement.
  **L1704 CN**: 结束当前词法作用域或复合语句块。

### Lines 1705-1728

````tablegen

// Returns the bitwise XOR of all values across all active non-helper lanes.
def hlsl_wave_active_bit_xor :
    HLSLOneArgBuiltin<"WaveActiveBitXor",
                      "__builtin_hlsl_wave_active_bit_xor"> {
  let VaryingTypes = [UIntTy, UInt64Ty];
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}

// Counts the number of boolean variables which evaluate to true across all
// active lanes in the current wave.
def hlsl_wave_active_count_bits :
    HLSLBuiltin<"WaveActiveCountBits",
                "__builtin_hlsl_wave_active_count_bits"> {
  let Doc = [{
\brief Counts the number of boolean variables which evaluate to true across
all active lanes in the current wave.
\param Val The input boolean value.
\return The number of lanes for which the boolean variable evaluates to
true, across all active lanes in the current wave.
}];
  let Args = [Varying];
````
- **L1705 EN**: Blank line separating nearby declarations or logic blocks.
  **L1705 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1706 EN**: Comment explains nearby logic, constraints, or intent: `Returns the bitwise XOR of all values across all active non-helper lanes.`.
  **L1706 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the bitwise XOR of all values across all active non-helper lanes.`。
- **L1707 EN**: Declares TableGen def record `hlsl_wave_active_bit_xor`.
  **L1707 CN**: 声明 TableGen def 记录 `hlsl_wave_active_bit_xor`。
- **L1708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLOneArgBuiltin<"WaveActiveBitXor",`.
  **L1708 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLOneArgBuiltin<"WaveActiveBitXor",`。
- **L1709 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_wave_active_bit_xor"> {`.
  **L1709 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_wave_active_bit_xor"> {`。
- **L1710 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [UIntTy, UInt64Ty];`.
  **L1710 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [UIntTy, UInt64Ty];`。
- **L1711 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1711 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1712 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1712 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1713 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1713 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1714 EN**: Closes the current lexical scope or compound statement.
  **L1714 CN**: 结束当前词法作用域或复合语句块。
- **L1715 EN**: Blank line separating nearby declarations or logic blocks.
  **L1715 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1716 EN**: Comment explains nearby logic, constraints, or intent: `Counts the number of boolean variables which evaluate to true across all`.
  **L1716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Counts the number of boolean variables which evaluate to true across all`。
- **L1717 EN**: Comment explains nearby logic, constraints, or intent: `active lanes in the current wave.`.
  **L1717 CN**: 注释解释附近代码的逻辑、约束或设计意图：`active lanes in the current wave.`。
- **L1718 EN**: Declares TableGen def record `hlsl_wave_active_count_bits`.
  **L1718 CN**: 声明 TableGen def 记录 `hlsl_wave_active_count_bits`。
- **L1719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"WaveActiveCountBits",`.
  **L1719 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"WaveActiveCountBits",`。
- **L1720 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_wave_active_count_bits"> {`.
  **L1720 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_wave_active_count_bits"> {`。
- **L1721 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1721 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1722 EN**: Continues the surrounding expression or declaration: `\brief Counts the number of boolean variables which evaluate to true across`.
  **L1722 CN**: 继续构造周围的表达式或声明：`\brief Counts the number of boolean variables which evaluate to true across`。
- **L1723 EN**: Continues the surrounding expression or declaration: `all active lanes in the current wave.`.
  **L1723 CN**: 继续构造周围的表达式或声明：`all active lanes in the current wave.`。
- **L1724 EN**: Continues the surrounding expression or declaration: `\param Val The input boolean value.`.
  **L1724 CN**: 继续构造周围的表达式或声明：`\param Val The input boolean value.`。
- **L1725 EN**: Continues the surrounding expression or declaration: `\return The number of lanes for which the boolean variable evaluates to`.
  **L1725 CN**: 继续构造周围的表达式或声明：`\return The number of lanes for which the boolean variable evaluates to`。
- **L1726 EN**: Continues the surrounding expression or declaration: `true, across all active lanes in the current wave.`.
  **L1726 CN**: 继续构造周围的表达式或声明：`true, across all active lanes in the current wave.`。
- **L1727 EN**: Adds a standalone statement or declaration: `}];`.
  **L1727 CN**: 添加一条独立语句或声明：`}];`。
- **L1728 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L1728 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。

### Lines 1729-1752

````tablegen
  let ReturnType = VaryingShape<UIntTy>;
  let VaryingTypes = [BoolTy];
  let VaryingScalar = 1;
  let IsConvergent = 1;
  let Availability = SM6_0;
}

// Returns the maximum value across all active lanes in the wave.
def hlsl_wave_active_max :
    HLSLOneArgBuiltin<"WaveActiveMax", "__builtin_hlsl_wave_active_max"> {
  let Doc = [{
\brief Returns the maximum value across all active lanes in the wave.
\param Val The input value.
}];
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}

// Returns the minimum value across all active lanes in the wave.
def hlsl_wave_active_min :
    HLSLOneArgBuiltin<"WaveActiveMin", "__builtin_hlsl_wave_active_min"> {
  let Doc = [{
````
- **L1729 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<UIntTy>;`.
  **L1729 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<UIntTy>;`。
- **L1730 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [BoolTy];`.
  **L1730 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [BoolTy];`。
- **L1731 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L1731 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L1732 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1732 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1733 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1733 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1734 EN**: Closes the current lexical scope or compound statement.
  **L1734 CN**: 结束当前词法作用域或复合语句块。
- **L1735 EN**: Blank line separating nearby declarations or logic blocks.
  **L1735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1736 EN**: Comment explains nearby logic, constraints, or intent: `Returns the maximum value across all active lanes in the wave.`.
  **L1736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the maximum value across all active lanes in the wave.`。
- **L1737 EN**: Declares TableGen def record `hlsl_wave_active_max`.
  **L1737 CN**: 声明 TableGen def 记录 `hlsl_wave_active_max`。
- **L1738 EN**: Continues the surrounding expression or declaration: `HLSLOneArgBuiltin<"WaveActiveMax", "__builtin_hlsl_wave_active_max"> {`.
  **L1738 CN**: 继续构造周围的表达式或声明：`HLSLOneArgBuiltin<"WaveActiveMax", "__builtin_hlsl_wave_active_max"> {`。
- **L1739 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1739 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1740 EN**: Continues the surrounding expression or declaration: `\brief Returns the maximum value across all active lanes in the wave.`.
  **L1740 CN**: 继续构造周围的表达式或声明：`\brief Returns the maximum value across all active lanes in the wave.`。
- **L1741 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1741 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1742 EN**: Adds a standalone statement or declaration: `}];`.
  **L1742 CN**: 添加一条独立语句或声明：`}];`。
- **L1743 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1743 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1744 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1744 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1745 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1745 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1746 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1746 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1747 EN**: Closes the current lexical scope or compound statement.
  **L1747 CN**: 结束当前词法作用域或复合语句块。
- **L1748 EN**: Blank line separating nearby declarations or logic blocks.
  **L1748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1749 EN**: Comment explains nearby logic, constraints, or intent: `Returns the minimum value across all active lanes in the wave.`.
  **L1749 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the minimum value across all active lanes in the wave.`。
- **L1750 EN**: Declares TableGen def record `hlsl_wave_active_min`.
  **L1750 CN**: 声明 TableGen def 记录 `hlsl_wave_active_min`。
- **L1751 EN**: Continues the surrounding expression or declaration: `HLSLOneArgBuiltin<"WaveActiveMin", "__builtin_hlsl_wave_active_min"> {`.
  **L1751 CN**: 继续构造周围的表达式或声明：`HLSLOneArgBuiltin<"WaveActiveMin", "__builtin_hlsl_wave_active_min"> {`。
- **L1752 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1752 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。

### Lines 1753-1776

````tablegen
\brief Returns the minimum value across all active lanes in the wave.
\param Val The input value.
}];
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}

// Returns the product of values across all active lanes in the wave.
def hlsl_wave_active_product :
    HLSLOneArgBuiltin<"WaveActiveProduct",
                      "__builtin_hlsl_wave_active_product"> {
  let Doc = [{
\brief Returns the product of values across all active lanes in the wave.
\param Val The input value.
}];
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}

// Returns the sum of values across all active lanes in the wave.
````
- **L1753 EN**: Continues the surrounding expression or declaration: `\brief Returns the minimum value across all active lanes in the wave.`.
  **L1753 CN**: 继续构造周围的表达式或声明：`\brief Returns the minimum value across all active lanes in the wave.`。
- **L1754 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1754 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1755 EN**: Adds a standalone statement or declaration: `}];`.
  **L1755 CN**: 添加一条独立语句或声明：`}];`。
- **L1756 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1756 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1757 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1757 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1758 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1758 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1759 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1759 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1760 EN**: Closes the current lexical scope or compound statement.
  **L1760 CN**: 结束当前词法作用域或复合语句块。
- **L1761 EN**: Blank line separating nearby declarations or logic blocks.
  **L1761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1762 EN**: Comment explains nearby logic, constraints, or intent: `Returns the product of values across all active lanes in the wave.`.
  **L1762 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the product of values across all active lanes in the wave.`。
- **L1763 EN**: Declares TableGen def record `hlsl_wave_active_product`.
  **L1763 CN**: 声明 TableGen def 记录 `hlsl_wave_active_product`。
- **L1764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLOneArgBuiltin<"WaveActiveProduct",`.
  **L1764 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLOneArgBuiltin<"WaveActiveProduct",`。
- **L1765 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_wave_active_product"> {`.
  **L1765 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_wave_active_product"> {`。
- **L1766 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1766 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1767 EN**: Continues the surrounding expression or declaration: `\brief Returns the product of values across all active lanes in the wave.`.
  **L1767 CN**: 继续构造周围的表达式或声明：`\brief Returns the product of values across all active lanes in the wave.`。
- **L1768 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1768 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1769 EN**: Adds a standalone statement or declaration: `}];`.
  **L1769 CN**: 添加一条独立语句或声明：`}];`。
- **L1770 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1770 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1771 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1771 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1772 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1772 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1773 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1773 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1774 EN**: Closes the current lexical scope or compound statement.
  **L1774 CN**: 结束当前词法作用域或复合语句块。
- **L1775 EN**: Blank line separating nearby declarations or logic blocks.
  **L1775 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1776 EN**: Comment explains nearby logic, constraints, or intent: `Returns the sum of values across all active lanes in the wave.`.
  **L1776 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the sum of values across all active lanes in the wave.`。

### Lines 1777-1800

````tablegen
def hlsl_wave_active_sum :
    HLSLOneArgBuiltin<"WaveActiveSum", "__builtin_hlsl_wave_active_sum"> {
  let Doc = [{
\brief Returns the sum of values across all active lanes in the wave.
\param Val The input value.
}];
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}

// Returns the number of lanes in the current wave.
def hlsl_wave_get_lane_count :
    HLSLBuiltin<"WaveGetLaneCount", "__builtin_hlsl_wave_get_lane_count"> {
  let Doc = [{
\brief Returns the number of lanes in the current wave.
}];
  let ReturnType = UIntTy;
  let IsConvergent = 1;
  let Availability = SM6_0;
}

// Returns the index of the current lane within the current wave.
````
- **L1777 EN**: Declares TableGen def record `hlsl_wave_active_sum`.
  **L1777 CN**: 声明 TableGen def 记录 `hlsl_wave_active_sum`。
- **L1778 EN**: Continues the surrounding expression or declaration: `HLSLOneArgBuiltin<"WaveActiveSum", "__builtin_hlsl_wave_active_sum"> {`.
  **L1778 CN**: 继续构造周围的表达式或声明：`HLSLOneArgBuiltin<"WaveActiveSum", "__builtin_hlsl_wave_active_sum"> {`。
- **L1779 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1779 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1780 EN**: Continues the surrounding expression or declaration: `\brief Returns the sum of values across all active lanes in the wave.`.
  **L1780 CN**: 继续构造周围的表达式或声明：`\brief Returns the sum of values across all active lanes in the wave.`。
- **L1781 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1781 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1782 EN**: Adds a standalone statement or declaration: `}];`.
  **L1782 CN**: 添加一条独立语句或声明：`}];`。
- **L1783 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1783 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1784 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1784 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1785 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1785 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1786 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1786 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1787 EN**: Closes the current lexical scope or compound statement.
  **L1787 CN**: 结束当前词法作用域或复合语句块。
- **L1788 EN**: Blank line separating nearby declarations or logic blocks.
  **L1788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1789 EN**: Comment explains nearby logic, constraints, or intent: `Returns the number of lanes in the current wave.`.
  **L1789 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the number of lanes in the current wave.`。
- **L1790 EN**: Declares TableGen def record `hlsl_wave_get_lane_count`.
  **L1790 CN**: 声明 TableGen def 记录 `hlsl_wave_get_lane_count`。
- **L1791 EN**: Continues the surrounding expression or declaration: `HLSLBuiltin<"WaveGetLaneCount", "__builtin_hlsl_wave_get_lane_count"> {`.
  **L1791 CN**: 继续构造周围的表达式或声明：`HLSLBuiltin<"WaveGetLaneCount", "__builtin_hlsl_wave_get_lane_count"> {`。
- **L1792 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1792 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1793 EN**: Continues the surrounding expression or declaration: `\brief Returns the number of lanes in the current wave.`.
  **L1793 CN**: 继续构造周围的表达式或声明：`\brief Returns the number of lanes in the current wave.`。
- **L1794 EN**: Adds a standalone statement or declaration: `}];`.
  **L1794 CN**: 添加一条独立语句或声明：`}];`。
- **L1795 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = UIntTy;`.
  **L1795 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = UIntTy;`。
- **L1796 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1796 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1797 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1797 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1798 EN**: Closes the current lexical scope or compound statement.
  **L1798 CN**: 结束当前词法作用域或复合语句块。
- **L1799 EN**: Blank line separating nearby declarations or logic blocks.
  **L1799 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1800 EN**: Comment explains nearby logic, constraints, or intent: `Returns the index of the current lane within the current wave.`.
  **L1800 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the index of the current lane within the current wave.`。

### Lines 1801-1824

````tablegen
def hlsl_wave_get_lane_index :
    HLSLBuiltin<"WaveGetLaneIndex", "__builtin_hlsl_wave_get_lane_index"> {
  let Doc = [{
\brief Returns the index of the current lane within the current wave.
}];
  let ReturnType = UIntTy;
  let IsConvergent = 1;
  let Availability = SM6_0;
}

// Returns true if the current lane is the first active lane in the wave.
def hlsl_wave_is_first_lane :
    HLSLBuiltin<"WaveIsFirstLane", "__builtin_hlsl_wave_is_first_lane"> {
  let Doc = [{
\brief Returns true if the current lane is the first active lane in the wave.
}];
  let ReturnType = BoolTy;
  let IsConvergent = 1;
  let Availability = SM6_0;
}

// Returns the count of bits set to 1 on prior lanes.
def hlsl_wave_prefix_count_bits :
    HLSLBuiltin<"WavePrefixCountBits",
````
- **L1801 EN**: Declares TableGen def record `hlsl_wave_get_lane_index`.
  **L1801 CN**: 声明 TableGen def 记录 `hlsl_wave_get_lane_index`。
- **L1802 EN**: Continues the surrounding expression or declaration: `HLSLBuiltin<"WaveGetLaneIndex", "__builtin_hlsl_wave_get_lane_index"> {`.
  **L1802 CN**: 继续构造周围的表达式或声明：`HLSLBuiltin<"WaveGetLaneIndex", "__builtin_hlsl_wave_get_lane_index"> {`。
- **L1803 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1803 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1804 EN**: Continues the surrounding expression or declaration: `\brief Returns the index of the current lane within the current wave.`.
  **L1804 CN**: 继续构造周围的表达式或声明：`\brief Returns the index of the current lane within the current wave.`。
- **L1805 EN**: Adds a standalone statement or declaration: `}];`.
  **L1805 CN**: 添加一条独立语句或声明：`}];`。
- **L1806 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = UIntTy;`.
  **L1806 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = UIntTy;`。
- **L1807 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1807 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1808 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1808 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1809 EN**: Closes the current lexical scope or compound statement.
  **L1809 CN**: 结束当前词法作用域或复合语句块。
- **L1810 EN**: Blank line separating nearby declarations or logic blocks.
  **L1810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1811 EN**: Comment explains nearby logic, constraints, or intent: `Returns true if the current lane is the first active lane in the wave.`.
  **L1811 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns true if the current lane is the first active lane in the wave.`。
- **L1812 EN**: Declares TableGen def record `hlsl_wave_is_first_lane`.
  **L1812 CN**: 声明 TableGen def 记录 `hlsl_wave_is_first_lane`。
- **L1813 EN**: Continues the surrounding expression or declaration: `HLSLBuiltin<"WaveIsFirstLane", "__builtin_hlsl_wave_is_first_lane"> {`.
  **L1813 CN**: 继续构造周围的表达式或声明：`HLSLBuiltin<"WaveIsFirstLane", "__builtin_hlsl_wave_is_first_lane"> {`。
- **L1814 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1814 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1815 EN**: Continues the surrounding expression or declaration: `\brief Returns true if the current lane is the first active lane in the wave.`.
  **L1815 CN**: 继续构造周围的表达式或声明：`\brief Returns true if the current lane is the first active lane in the wave.`。
- **L1816 EN**: Adds a standalone statement or declaration: `}];`.
  **L1816 CN**: 添加一条独立语句或声明：`}];`。
- **L1817 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = BoolTy;`.
  **L1817 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = BoolTy;`。
- **L1818 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1818 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1819 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1819 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1820 EN**: Closes the current lexical scope or compound statement.
  **L1820 CN**: 结束当前词法作用域或复合语句块。
- **L1821 EN**: Blank line separating nearby declarations or logic blocks.
  **L1821 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1822 EN**: Comment explains nearby logic, constraints, or intent: `Returns the count of bits set to 1 on prior lanes.`.
  **L1822 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the count of bits set to 1 on prior lanes.`。
- **L1823 EN**: Declares TableGen def record `hlsl_wave_prefix_count_bits`.
  **L1823 CN**: 声明 TableGen def 记录 `hlsl_wave_prefix_count_bits`。
- **L1824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSLBuiltin<"WavePrefixCountBits",`.
  **L1824 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSLBuiltin<"WavePrefixCountBits",`。

### Lines 1825-1848

````tablegen
                "__builtin_hlsl_wave_prefix_count_bits"> {
  let Doc = [{
\brief Returns the count of bits of Expr set to 1 on prior lanes.
\param Expr The boolean expression to evaluate.
\return the count of bits set to 1 on prior lanes.
}];
  let Args = [Varying];
  let ReturnType = VaryingShape<UIntTy>;
  let VaryingTypes = [BoolTy];
  let VaryingScalar = 1;
  let IsConvergent = 1;
  let Availability = SM6_0;
}

// Returns the product of values across lanes with smaller indices in the wave.
def hlsl_wave_prefix_product :
    HLSLOneArgBuiltin<"WavePrefixProduct", "__builtin_hlsl_wave_prefix_product"> {
  let Doc = [{
\brief Returns the product of values across lanes with smaller indices in the wave.
\param Val The input value.
}];
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let Availability = SM6_0;
````
- **L1825 EN**: Continues the surrounding expression or declaration: `"__builtin_hlsl_wave_prefix_count_bits"> {`.
  **L1825 CN**: 继续构造周围的表达式或声明：`"__builtin_hlsl_wave_prefix_count_bits"> {`。
- **L1826 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1826 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1827 EN**: Continues the surrounding expression or declaration: `\brief Returns the count of bits of Expr set to 1 on prior lanes.`.
  **L1827 CN**: 继续构造周围的表达式或声明：`\brief Returns the count of bits of Expr set to 1 on prior lanes.`。
- **L1828 EN**: Continues the surrounding expression or declaration: `\param Expr The boolean expression to evaluate.`.
  **L1828 CN**: 继续构造周围的表达式或声明：`\param Expr The boolean expression to evaluate.`。
- **L1829 EN**: Continues the surrounding expression or declaration: `\return the count of bits set to 1 on prior lanes.`.
  **L1829 CN**: 继续构造周围的表达式或声明：`\return the count of bits set to 1 on prior lanes.`。
- **L1830 EN**: Adds a standalone statement or declaration: `}];`.
  **L1830 CN**: 添加一条独立语句或声明：`}];`。
- **L1831 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying];`.
  **L1831 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying];`。
- **L1832 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ReturnType = VaryingShape<UIntTy>;`.
  **L1832 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ReturnType = VaryingShape<UIntTy>;`。
- **L1833 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = [BoolTy];`.
  **L1833 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = [BoolTy];`。
- **L1834 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingScalar = 1;`.
  **L1834 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingScalar = 1;`。
- **L1835 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1835 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1836 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1836 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1837 EN**: Closes the current lexical scope or compound statement.
  **L1837 CN**: 结束当前词法作用域或复合语句块。
- **L1838 EN**: Blank line separating nearby declarations or logic blocks.
  **L1838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1839 EN**: Comment explains nearby logic, constraints, or intent: `Returns the product of values across lanes with smaller indices in the wave.`.
  **L1839 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the product of values across lanes with smaller indices in the wave.`。
- **L1840 EN**: Declares TableGen def record `hlsl_wave_prefix_product`.
  **L1840 CN**: 声明 TableGen def 记录 `hlsl_wave_prefix_product`。
- **L1841 EN**: Continues the surrounding expression or declaration: `HLSLOneArgBuiltin<"WavePrefixProduct", "__builtin_hlsl_wave_prefix_product"> {`.
  **L1841 CN**: 继续构造周围的表达式或声明：`HLSLOneArgBuiltin<"WavePrefixProduct", "__builtin_hlsl_wave_prefix_product"> {`。
- **L1842 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1842 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1843 EN**: Continues the surrounding expression or declaration: `\brief Returns the product of values across lanes with smaller indices in the wave.`.
  **L1843 CN**: 继续构造周围的表达式或声明：`\brief Returns the product of values across lanes with smaller indices in the wave.`。
- **L1844 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1844 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1845 EN**: Adds a standalone statement or declaration: `}];`.
  **L1845 CN**: 添加一条独立语句或声明：`}];`。
- **L1846 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1846 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1847 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1847 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1848 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1848 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。

### Lines 1849-1872

````tablegen
  let VaryingMatDims = [];
}

// Returns the sum of values across lanes with smaller indices in the wave.
def hlsl_wave_prefix_sum :
    HLSLOneArgBuiltin<"WavePrefixSum", "__builtin_hlsl_wave_prefix_sum"> {
  let Doc = [{
\brief Returns the sum of values across lanes with smaller indices in the wave.
\param Val The input value.
}];
  let VaryingTypes = AllNumericTypes;
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}

// Reads the value from the specified lane index in the wave.
def hlsl_wave_read_lane_at :
    HLSLTwoArgBuiltin<"WaveReadLaneAt", "__builtin_hlsl_wave_read_lane_at"> {
  let Doc = [{
\brief Returns the value of the expression for the given lane index within
the specified wave.
\param Val The value to read.
\param Index The lane index.
````
- **L1849 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1849 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1850 EN**: Closes the current lexical scope or compound statement.
  **L1850 CN**: 结束当前词法作用域或复合语句块。
- **L1851 EN**: Blank line separating nearby declarations or logic blocks.
  **L1851 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1852 EN**: Comment explains nearby logic, constraints, or intent: `Returns the sum of values across lanes with smaller indices in the wave.`.
  **L1852 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the sum of values across lanes with smaller indices in the wave.`。
- **L1853 EN**: Declares TableGen def record `hlsl_wave_prefix_sum`.
  **L1853 CN**: 声明 TableGen def 记录 `hlsl_wave_prefix_sum`。
- **L1854 EN**: Continues the surrounding expression or declaration: `HLSLOneArgBuiltin<"WavePrefixSum", "__builtin_hlsl_wave_prefix_sum"> {`.
  **L1854 CN**: 继续构造周围的表达式或声明：`HLSLOneArgBuiltin<"WavePrefixSum", "__builtin_hlsl_wave_prefix_sum"> {`。
- **L1855 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1855 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1856 EN**: Continues the surrounding expression or declaration: `\brief Returns the sum of values across lanes with smaller indices in the wave.`.
  **L1856 CN**: 继续构造周围的表达式或声明：`\brief Returns the sum of values across lanes with smaller indices in the wave.`。
- **L1857 EN**: Continues the surrounding expression or declaration: `\param Val The input value.`.
  **L1857 CN**: 继续构造周围的表达式或声明：`\param Val The input value.`。
- **L1858 EN**: Adds a standalone statement or declaration: `}];`.
  **L1858 CN**: 添加一条独立语句或声明：`}];`。
- **L1859 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllNumericTypes;`.
  **L1859 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllNumericTypes;`。
- **L1860 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1860 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1861 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1861 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1862 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1862 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1863 EN**: Closes the current lexical scope or compound statement.
  **L1863 CN**: 结束当前词法作用域或复合语句块。
- **L1864 EN**: Blank line separating nearby declarations or logic blocks.
  **L1864 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1865 EN**: Comment explains nearby logic, constraints, or intent: `Reads the value from the specified lane index in the wave.`.
  **L1865 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reads the value from the specified lane index in the wave.`。
- **L1866 EN**: Declares TableGen def record `hlsl_wave_read_lane_at`.
  **L1866 CN**: 声明 TableGen def 记录 `hlsl_wave_read_lane_at`。
- **L1867 EN**: Continues the surrounding expression or declaration: `HLSLTwoArgBuiltin<"WaveReadLaneAt", "__builtin_hlsl_wave_read_lane_at"> {`.
  **L1867 CN**: 继续构造周围的表达式或声明：`HLSLTwoArgBuiltin<"WaveReadLaneAt", "__builtin_hlsl_wave_read_lane_at"> {`。
- **L1868 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Doc = [{`.
  **L1868 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Doc = [{`。
- **L1869 EN**: Continues the surrounding expression or declaration: `\brief Returns the value of the expression for the given lane index within`.
  **L1869 CN**: 继续构造周围的表达式或声明：`\brief Returns the value of the expression for the given lane index within`。
- **L1870 EN**: Continues the surrounding expression or declaration: `the specified wave.`.
  **L1870 CN**: 继续构造周围的表达式或声明：`the specified wave.`。
- **L1871 EN**: Continues the surrounding expression or declaration: `\param Val The value to read.`.
  **L1871 CN**: 继续构造周围的表达式或声明：`\param Val The value to read.`。
- **L1872 EN**: Continues the surrounding expression or declaration: `\param Index The lane index.`.
  **L1872 CN**: 继续构造周围的表达式或声明：`\param Index The lane index.`。

### Lines 1873-1879

````tablegen
}];
  let Args = [Varying, UIntTy];
  let VaryingTypes = AllTypesWithBool;
  let IsConvergent = 1;
  let Availability = SM6_0;
  let VaryingMatDims = [];
}
````
- **L1873 EN**: Adds a standalone statement or declaration: `}];`.
  **L1873 CN**: 添加一条独立语句或声明：`}];`。
- **L1874 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Args = [Varying, UIntTy];`.
  **L1874 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Args = [Varying, UIntTy];`。
- **L1875 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingTypes = AllTypesWithBool;`.
  **L1875 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingTypes = AllTypesWithBool;`。
- **L1876 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IsConvergent = 1;`.
  **L1876 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IsConvergent = 1;`。
- **L1877 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Availability = SM6_0;`.
  **L1877 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Availability = SM6_0;`。
- **L1878 EN**: Assigns a TableGen property that affects following records or inherited fields: `let VaryingMatDims = [];`.
  **L1878 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let VaryingMatDims = [];`。
- **L1879 EN**: Closes the current lexical scope or compound statement.
  **L1879 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Compiler configuration / 编译器配置**
  - **EN**: Holds low-level enums, constants, feature switches, and descriptive metadata used across Clang.
  - **CN**: 保存 Clang 各处复用的底层枚举、常量、特性开关与描述性元数据。
- **Target and language metadata / 目标与语言元数据**
  - **EN**: Models ABI, address-space, builtin, and target-feature information shared by many subsystems.
  - **CN**: 建模多个子系统共享的 ABI、地址空间、builtin 与目标特性信息。
- **Generated definition tables / 生成式定义表**
  - **EN**: Many Basic headers use `.td` or `.def` files to describe families of diagnostics, attributes, or builtins declaratively.
  - **CN**: 许多 Basic 头文件使用 `.td` 或 `.def` 文件以声明式方式描述诊断、属性或 builtin 家族。
- **TableGen declarative modeling / TableGen 声明式建模**
  - **EN**: Uses TableGen records, inheritance, and generators to describe families of compiler entities compactly.
  - **CN**: 使用 TableGen 记录、继承与生成器紧凑地描述一组编译器实体。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。
- **Attribute metadata / 属性元数据**
  - **EN**: Captures attribute spellings, subjects, semantic flags, and generated helper data.
  - **CN**: 刻画属性的拼写、适用对象、语义标志与生成的辅助数据。
- **ABI contracts / ABI 契约**
  - **EN**: Records ABI-relevant enums and conventions used to represent constructors, destructors, or calling details.
  - **CN**: 记录用于表示构造函数、析构函数或调用细节的 ABI 相关枚举与约定。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Indexing and tooling / 索引与工具支持**
  - **EN**: Exposes APIs used by source indexing, diagnostics retrieval, and IDE tooling.
  - **CN**: 暴露源码索引、诊断提取与 IDE 工具使用的 API。
- **Source-location handling / 源码位置处理**
  - **EN**: Tracks files, offsets, and source ranges needed by diagnostics and tooling.
  - **CN**: 跟踪诊断与工具所需的文件、偏移与源码范围。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `for`, `HLSLArgType`, `HLSLReturnType`, `HLSLType`, `VaryingShape`, `VectorType`, `ShaderModel`, `MatDim`, `HLSLBuiltin`, `HLSLOneArgBuiltin`, `HLSLTwoArgBuiltin`, `HLSLThreeArgBuiltin`
- **Functions or callables / 函数或可调用对象**: `_HLSL_AVAILABILITY`, `_HLSL_16BIT_AVAILABILITY`, `defined`, `type`, `dimensions`, `_HLSL_BUILTIN_ALIAS`, `DetailFunc`, `VaryingTypes`, `position`, `func`, `functions`, `arguments`
- **TableGen records / TableGen 记录**: `HLSLArgType;`, `HLSLReturnType;`, `VoidTy`, `HLSLType`, `BoolTy`, `HalfTy`, `FloatTy`, `DoubleTy`, `Int16Ty`, `UInt16Ty`, `IntTy`, `UIntTy`, `Int64Ty`, `UInt64Ty`, `AllFloatTypes`, `SignedIntTypes`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
