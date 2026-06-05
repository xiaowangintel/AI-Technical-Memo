# ArmSMEOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/ArmSME/IR/ArmSMEOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines the ArmSME dialect ops. It also defines custom attributes and types that are used to define the Ops.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/ArmSME/IR`，围绕 ArmSME 方言公开 `ArmSMEOps` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```tablegen
//===-- ArmSMEOps.td - ArmSME dialect operation definitions *- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the ArmSME dialect ops. It also defines custom attributes
// and types that are used to define the Ops.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 13-23
```tablegen

#ifndef ARMSME_OPS
#define ARMSME_OPS

include "ArmSME.td"
include "mlir/IR/EnumAttr.td"
include "mlir/IR/OpBase.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/Dialect/LLVMIR/LLVMOpBase.td"
include "mlir/Interfaces/InferTypeOpInterface.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 24-39
```tablegen
//===----------------------------------------------------------------------===//
// ArmSME op interfaces
//===----------------------------------------------------------------------===//

def ArmSMETileType : I32EnumAttr<"ArmSMETileType", "Arm SME tile type",
    [
      I32EnumAttrCase<"ZAB", 0, "za.b">,
      I32EnumAttrCase<"ZAH", 1, "za.h">,
      I32EnumAttrCase<"ZAS", 2, "za.s">,
      I32EnumAttrCase<"ZAD", 3, "za.d">,
      I32EnumAttrCase<"ZAQ", 4, "za.q">,
    ]>{
  let cppNamespace = "mlir::arm_sme";
  let genSpecializedAttr = 0;
}

```
- **EN**: Introduces declarations for `ArmSMETileType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSMETileType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 40-59
```tablegen
def ArmSMETileOpInterface : OpInterface<"ArmSMETileOpInterface"> {
  let description = [{
    An interface for operations that use Arm SME tiles. These operations need to
    be assigned a tile ID, an i32 attribute, which specifies which virtual tile
    within the ZA storage to use. The number of tiles available depends on the
    type of the tile. This is summarized below:

    | Tile Vector Types                                                       | Possible Tile IDs   |
    |-------------------------------------------------------------------------|---------------------|
    | `vector<[16]x[16]xi8>`                                                  | 0                   |
    | `vector<[8]x[8]xi16>`, `vector<[8]x[8]xf16>`, or `vector<[8]x[8]xbf16>` | 0 and 1             |
    | `vector<[4]x[4]xi32>` or `vector<[4]x[4]xf32>`                          | 0 to 3 (inclusive)  |
    | `vector<[2]x[2]xi64>` or `vector<[2]x[2]xf64>`                          | 0 to 7 (inclusive)  |
    | `vector<[1]x[1]xi128>`                                                  | 0 to 15 (inclusive) |
  }];
  let methods = [
    InterfaceMethod<
      "Sets the tile ID for this operation.",
      /*returnType=*/"void",
      /*methodName=*/"setTileId",
```
- **EN**: Introduces declarations for `ArmSMETileOpInterface`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSMETileOpInterface` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 60-79
```tablegen
      /*arguments=*/(ins "mlir::IntegerAttr":$tileId),
      /*methodBody=*/[{}],
      /*defaultImpl=*/ [{
        if (!tileId)
          return;
        ::mlir::Operation* op = this->getOperation();
        op->setAttr("tile_id", tileId);
      }]
    >,
    InterfaceMethod<
      [{
        Returns the tile ID assigned to this operation. This will be null before
        tile allocation.
      }],
      /*returnType=*/"mlir::IntegerAttr",
      /*methodName=*/"getTileId",
      /*arguments=*/(ins),
      /*methodBody=*/[{}],
      /*defaultImpl=*/ [{
        ::mlir::Operation* op = this->getOperation();
```
- **EN**: Implements logic around `getOperation`, `setAttr`.
- **CN**: 围绕 `getOperation`, `setAttr` 实现具体逻辑。

### Lines 80-89
```tablegen
        return op->getAttrOfType<mlir::IntegerAttr>("tile_id");
      }]
    >,
    InterfaceMethod<
      "Returns the VectorType of the tile used by this operation.",
      /*returnType=*/"VectorType",
      /*methodName=*/"getTileType"
    >
  ];

```
- **EN**: Declares APIs or declarative rules around `IntegerAttr>`.
- **CN**: 声明与 `IntegerAttr>` 相关的 API 或声明式规则。

### Lines 90-99
```tablegen
  let extraSharedClassDeclaration = [{
    bool isInMemoryTile() {
      auto tileId = getTileId();
      return tileId && tileId.getInt() >= kInMemoryTileIdBase;
    }
  }];

  let verify = [{ return detail::verifyArmSMETileOpInterface($_op); }];
}

```
- **EN**: Implements logic around `isInMemoryTile`, `getTileId`, `getInt`, `verifyArmSMETileOpInterface`; this block checks structural or semantic invariants.
- **CN**: 围绕 `isInMemoryTile`, `getTileId`, `getInt`, `verifyArmSMETileOpInterface` 实现具体逻辑；该代码块检查结构或语义不变式。

### Lines 100-109
```tablegen
//===----------------------------------------------------------------------===//
// ArmSME type definitions
//===----------------------------------------------------------------------===//

class SMETileType<Type datatype, list<int> dims, string description>
  : ShapedContainerType<[datatype],
      And<[IsVectorOfRankPred<[2]>, IsVectorTypeWithAllDimsScalablePred,
           IsVectorOfShape<dims>]>,
  description>;

```
- **EN**: Introduces declarations for `SMETileType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMETileType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 110-119
```tablegen
def nxnxv16i8  : SMETileType<I8,   [16, 16], "vector<[16]x[16]xi8>">;
def nxnxv8i16  : SMETileType<I16,  [8,  8 ], "vector<[8]x[8]xi16>">;
def nxnxv4i32  : SMETileType<I32,  [4,  4 ], "vector<[4]x[4]xi32>">;
def nxnxv2i64  : SMETileType<I64,  [2,  2 ], "vector<[2]x[2]xi64>">;
def nxnxv1i128 : SMETileType<I128, [1,  1 ], "vector<[1]x[1]xi128>">;

def nxnxv8f16  : SMETileType<F16,  [8,  8 ], "vector<[8]x[8]xf16>">;
def nxnxv8bf16 : SMETileType<BF16, [8,  8 ], "vector<[8]x[8]xbf16>">;
def nxnxv4f32  : SMETileType<F32,  [4,  4 ], "vector<[4]x[4]xf32>">;
def nxnxv2f64  : SMETileType<F64,  [2,  2 ], "vector<[2]x[2]xf64>">;
```
- **EN**: Introduces declarations for `nxnxv16i8`, `nxnxv8i16`, `nxnxv4i32`, `nxnxv2i64`, and 5 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `nxnxv16i8`, `nxnxv8i16`, `nxnxv4i32`, `nxnxv2i64`, and 5 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 120-130
```tablegen

def SMETile : AnyTypeOf<[nxnxv16i8, nxnxv8i16, nxnxv4i32, nxnxv2i64, nxnxv1i128,
                         nxnxv8f16, nxnxv8bf16, nxnxv4f32, nxnxv2f64],
                        "a vector type that fits into a SME tile",
                        "VectorType">
{
  let description = [{
    Possible vector types:

    Integer elements:

```
- **EN**: Introduces declarations for `SMETile`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMETile` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 131-145
```tablegen
    * `vector<[16]x[16]xi8>`
    * `vector<[8]x[8]xi16>`
    * `vector<[4]x[4]xi32>`
    * `vector<[2]x[2]xi64>`
    * `vector<[1]x[1]xi128>`

    Floating point elements:

    * `vector<[8]x[8]xf16>`
    * `vector<[8]x[8]xbf16>`
    * `vector<[4]x[4]xf32>`
    * `vector<[2]x[2]xf64>`
  }];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 146-160
```tablegen
class HasMatchingMaskTypeConstraint<string vector, string mask> :
  OptionalTypesMatchWith<
    mask # " has i1 element type and same shape as " # vector,
    vector, mask,
    "::llvm::cast<mlir::VectorType>($_self).cloneWith({}, IntegerType::get($_ctxt, 1))">;

class TileSliceMaskConstraint<string tile, string mask> :
  TypesMatchWith<
    "`" # mask # "` has i1 element type and the shape is a slice of `" # tile # "`",
    tile, mask,
    "VectorType("
      "VectorType::Builder("
        "::llvm::cast<mlir::VectorType>($_self)"
      ").dropDim(0).setElementType(IntegerType::get($_self.getContext(), 1)))">;

```
- **EN**: Introduces declarations for `HasMatchingMaskTypeConstraint`, `TileSliceMaskConstraint`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `HasMatchingMaskTypeConstraint`, `TileSliceMaskConstraint` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 161-172
```tablegen
//===----------------------------------------------------------------------===//
// ArmSME attr definitions
//===----------------------------------------------------------------------===//

def TileSliceLayout : I32EnumAttr<"TileSliceLayout", "Layout of a tile slice", [
  I32EnumAttrCase<"Horizontal", 0, "horizontal">,
  I32EnumAttrCase<"Vertical", 1, "vertical">,
]> {
  let cppNamespace = "::mlir::arm_sme";
  let genSpecializedAttr = 0;
}

```
- **EN**: Introduces declarations for `TileSliceLayout`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `TileSliceLayout` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 173-187
```tablegen
/// An attribute that specifies the layout of a tile slice in a tile.
def ArmSME_TileSliceLayoutAttr : EnumAttr<ArmSME_Dialect, TileSliceLayout,
                                          "layout"> {
  let assemblyFormat = "`<` $value `>`";
  let defaultValue = "TileSliceLayout::Horizontal";
}

def CombiningKind : I32EnumAttr<"CombiningKind", "Kind of combining function", [
  I32EnumAttrCase<"Add", 0, "add">,
  I32EnumAttrCase<"Sub", 1, "sub">,
]> {
  let cppNamespace = "::mlir::arm_sme";
  let genSpecializedAttr = 0;
}

```
- **EN**: Introduces declarations for `ArmSME_TileSliceLayoutAttr`, `CombiningKind`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSME_TileSliceLayoutAttr`, `CombiningKind` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 188-200
```tablegen
/// An attribute that specifies how to combine a newly produced value with the
/// accumulator. This is similar to vector::CombiningKindAttr, but limited to
/// the functions that are valid for SME outer products. Add corresponds to a
/// MOPA and sub to a MOPS.
/// E.g. For f32:
/// FMOPA: https://developer.arm.com/documentation/ddi0602/2022-03/SME-Instructions/FMOPA--non-widening---Floating-point-outer-product-and-accumulate-
/// FMOPS: https://developer.arm.com/documentation/ddi0602/2022-03/SME-Instructions/FMOPS--non-widening---Floating-point-outer-product-and-subtract-
def ArmSME_CombiningKindAttr : EnumAttr<ArmSME_Dialect, CombiningKind,
                                          "kind"> {
  let assemblyFormat = "`<` $value `>`";
  let defaultValue = "CombiningKind::Add";
}

```
- **EN**: Introduces declarations for `ArmSME_CombiningKindAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSME_CombiningKindAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 201-210
```tablegen
def TypeSize : I32EnumAttr<"TypeSize", "Size of a vector element type", [
  I32EnumAttrCase<"Byte"  , 0, "byte">,
  I32EnumAttrCase<"Half"  , 1, "half">,
  I32EnumAttrCase<"Word"  , 2, "word">,
  I32EnumAttrCase<"Double", 3, "double">,
]> {
  let cppNamespace = "::mlir::arm_sme";
  let genSpecializedAttr = 0;
}

```
- **EN**: Introduces declarations for `TypeSize`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `TypeSize` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 211-220
```tablegen
def ArmSME_TypeSizeAttr : EnumAttr<ArmSME_Dialect, TypeSize,
                                   "type_size"> {
  let assemblyFormat = "`<` $value `>`";
}

//===----------------------------------------------------------------------===//
// ArmSME op definitions
//===----------------------------------------------------------------------===//

class ArmSME_Op<string mnemonic, list<Trait> traits = []> :
```
- **EN**: Introduces declarations for `ArmSME_TypeSizeAttr`, `ArmSME_Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSME_TypeSizeAttr`, `ArmSME_Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 221-230
```tablegen
  Op<ArmSME_Dialect, mnemonic, traits> {}

def GetTileOp : ArmSME_Op<"get_tile", [ArmSMETileOpInterface, Pure]> {
  let summary = "Creates an undefined value of SME virtual tile type";
  let description = [{
    Creates a new SME "virtual tile" value within a function. The contents of
    the tile returned from this operation are undefined.

    Example 1:

```
- **EN**: Introduces declarations for `GetTileOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GetTileOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 231-243
```tablegen
    ```mlir
    // Create an 8-bit element "virtual tile" value:
    %za0_b = arm_sme.get_tile: vector<[16]x[16]xi8>
    ```

    Example 2:

    ```mlir
    // Create two 16-bit element "virtual tiles" values:
    %za0_h = arm_sme.get_tile : vector<[8]x[8]xi16>
    %za1_h = arm_sme.get_tile : vector<[8]x[8]xi16>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 244-253
```tablegen
    Example 3:
    ```mlir
    // Create an 128-bit element "virtual tile" value:
    %za0_q = arm_sme.get_tile : vector<[1]x[1]xi128>
    ```
  }];

  let results = (outs SMETile:$tile);
  let assemblyFormat = "attr-dict `:` type($tile)";

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 254-267
```tablegen
  let extraClassDeclaration = [{
    VectorType getTileType() {
      return ::llvm::cast<VectorType>(getTile().getType());
    }
  }];
}

def ZeroOp : ArmSME_Op<"zero", [ArmSMETileOpInterface, Pure]> {
  let summary = "Creates a zero-initialized value of SME virtual tile type";
  let results = (outs SMETile:$res);
  let description = [{
    Creates a new SME "virtual tile" value within a function. The contents of
    the tile returned from this operation are zero-initialized.

```
- **EN**: Introduces declarations for `ZeroOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ZeroOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 268-287
```tablegen
    Example 1: Zero an 8-bit element ZA tile.

    ```mlir
    %0 = arm_sme.zero : vector<[16]x[16]xi8>
    ```

    Example 2: Zero a 64-bit element ZA tile.

    ```mlir
    %0 = arm_sme.zero : vector<[2]x[2]xi64>
    ```
  }];
  let extraClassDeclaration = [{
    VectorType getVectorType() {
      return ::llvm::cast<VectorType>(getRes().getType());
    }
    VectorType getTileType() {
      return getVectorType();
    }
  }];
```
- **EN**: Implements logic around `getVectorType`, `getRes`, `getTileType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getVectorType`, `getRes`, `getTileType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 288-302
```tablegen
  let assemblyFormat = "attr-dict `:` type($res)";
}

def CopyTileOp : ArmSME_Op<"copy_tile", [
  Pure,
  ArmSMETileOpInterface,
  AllTypesMatch<["tile", "result"]>
]> {
  let summary = "Copies an SME tile value";
  let arguments = (ins SMETile:$tile);
  let results = (outs SMETile:$result);
  let description = [{
    Copies an SME "virtual tile" value to a new SSA value. This operation is
    primarily intended to be used to normalize the IR prior to tile allocation.

```
- **EN**: Introduces declarations for `CopyTileOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CopyTileOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 303-316
```tablegen
    Example:

    ```mlir
    %copy = arm_sme.copy_tile %tile : vector<[4]x[4]xf32>
    ```
  }];
  let extraClassDeclaration = [{
    VectorType getTileType() {
      return ::llvm::cast<VectorType>(getResult().getType());
    }
  }];
  let assemblyFormat = "$tile attr-dict `:` type($result)";
}

```
- **EN**: Implements logic around `getTileType`, `getResult`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTileType`, `getResult`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 317-336
```tablegen
def TileLoadOp : ArmSME_Op<"tile_load", [
  ArmSMETileOpInterface,
  AttrSizedOperandSegments,
  AllElementTypesMatch<["result", "base"]>,
  OptionalTypesMatchWith<
    "padding type matches element type of result",
    "result", "padding",
    "::llvm::cast<VectorType>($_self).getElementType()"
  >,
  HasMatchingMaskTypeConstraint<"result", "mask">,
  PredOpTrait<
    "both `padding` and `mask` should be provided or neither",
    CPred<"bool(getPadding()) == bool(getMask())">
  >,
]> {
  let summary = "Tile load operation";
  let description = [{
    Loads a 2D SME "virtual tile" from memory defined by a base and indices,
    with the shape defined by the 2D scalable vector type of the result tile.
    An optional tile slice layout attribute specifies whether the slices of the
```
- **EN**: Introduces declarations for `TileLoadOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `TileLoadOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 337-349
```tablegen
    tile being loaded are horizontal (default) or vertical. The slice of memory
    must be contiguous. The memref must be either rank 1 or rank 2 with dynamic
    dimensions, since the operation is scalable, and the element type must be a
    scalar that matches the element type of the result.

    An optional SSA value `padding` of the same elemental type as the MemRef is
    provided to specify a fallback value in the case of masking.

    An optional SSA value `mask` may be specified to mask out elements read
    from the MemRef. The `mask` type is an `i1` vector with a shape that
    matches how elements are read from the MemRef. Elements whose corresponding
    mask element is `0` are masked out and replaced with `padding`.

```
- **EN**: Implements logic around `horizontal`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `horizontal` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 350-361
```tablegen
    If either `padding` or `mask` are specified, both must be specified.

    Example 1: Load an 8-bit element ZA tile with horizontal layout (default) from memory (ZA0.B).
    ```mlir
    %tile = arm_sme.tile_load %base[%c0, %c0] : memref<?x?xi8>, vector<[16]x[16]xi8>
    ```

    Example 2: Load a FP 32-bit element ZA tile with vertical layout from memory.
    ```mlir
    %tile = arm_sme.tile_load %base[%c0, %c0] layout<vertical> : memref<?x?xf32>, vector<[4]x[4]xf32>
    ```

```
- **EN**: Implements logic around `layout`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `layout` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 362-379
```tablegen
    Example 3: Load a 128-bit element ZA tile with horizontal layout (default) from memory.
    ```mlir
    %tile = arm_sme.tile_load %base[%c0, %c0] layout<horizontal> : memref<?x?xi128>, vector<[1]x[1]xi128>
    ```

    Example 4: Masked load of int 32-bit element ZA tile with horizontal layout (default) from memory.
    ```mlir
    %tile = arm_sme.tile_load %base[%c0, %c0], %pad, %mask : memref<?x?xf32>, vector<[4]x[4]xf32>
    ```
  }];
  let arguments = (ins
    Arg<MemRefRankOf<[AnyType], [2]>, "the reference to load from", [MemRead]>:$base,
    Variadic<Index>:$indices,
    Optional<AnyType>:$padding, Optional<AnyVectorOfNonZeroRank>:$mask,
    ArmSME_TileSliceLayoutAttr:$layout
  );
  let results = (outs SMETile:$result);

```
- **EN**: Declares APIs or declarative rules around `layout`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `layout` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 380-391
```tablegen
  let extraClassDeclaration = [{
    MemRefType getMemRefType() {
      return ::llvm::cast<MemRefType>(getBase().getType());
    }
    VectorType getVectorType() {
      return ::llvm::cast<VectorType>(getResult().getType());
    }
    VectorType getTileType() {
      return getVectorType();
    }
  }];

```
- **EN**: Implements logic around `getMemRefType`, `getBase`, `getVectorType`, `getResult`, and 1 more symbols.
- **CN**: 围绕 `getMemRefType`, `getBase`, `getVectorType`, `getResult`, and 1 more symbols 实现具体逻辑。

### Lines 392-402
```tablegen
  let builders = [
    OpBuilder<(ins "VectorType":$resultType, "Value":$base,
                   "ValueRange":$indices, "TileSliceLayout":$layout), [{
      build($_builder, $_state, resultType, base, indices, {}, {}, layout);
    }]>,
    OpBuilder<(ins "VectorType":$resultType, "Value":$base,
                   "ValueRange":$indices), [{
      build($_builder, $_state, resultType, base, indices, {}, {}, {});
    }]>,
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑。

### Lines 403-422
```tablegen
  let assemblyFormat =
    "$base `[` $indices `]` (`,` $padding `,` $mask^)? (`layout` `` $layout^)?"
      "attr-dict `:` type($base) `,` type($result)";
}

def TileStoreOp : ArmSME_Op<"tile_store", [
  ArmSMETileOpInterface,
  AttrSizedOperandSegments,
  AllElementTypesMatch<["valueToStore", "base"]>,
  HasMatchingMaskTypeConstraint<"valueToStore", "mask">,
]> {
  let summary = "Tile store operation";
  let description = [{
    Stores a 2D SME "virtual tile" to memory defined by a base and indices,
    with the shape defined by the 2D scalable vector type of the tile being
    stored. An optional tile slice layout attribute specifies whether the
    slices of the tile being stored are horizontal (default) or vertical. The
    slice of memory must be contiguous. The memref must be either rank 1 or
    rank 2 with dynamic dimensions, since the operation is scalable, and the
    element type must be a scalar that matches the element type of the result.
```
- **EN**: Introduces declarations for `TileStoreOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `TileStoreOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 423-436
```tablegen

    An optional `mask` may be provided, the shape of which corresponds to the
    `tile`, and selects which elements of the tile will be stored.

    Example 1: Store an 8-bit element ZA tile with horizontal (default) layout to memory (ZA0.B).
    ```mlir
    arm_sme.tile_store %tile, %base[%c0, %c0] : vector<[16]x[16]xi8>, memref<?x?xi8>
    ```

    Example 2: Store a FP 32-bit element ZA tile with vertical layout to memory.
    ```mlir
    arm_sme.tile_store %tile, %base[%c0, %c0] layout<vertical> : vector<[4]x[4]xf32>, memref<?x?xf32>
    ```

```
- **EN**: Implements logic around `horizontal`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `horizontal` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 437-456
```tablegen
    Example 3: Store a 128-bit element ZA tile with horizontal (default) layout to memory.
    ```mlir
    arm_sme.tile_store %tile, %base[%c0, %c0] layout<horizontal> : vector<[1]x[1]xi128>, memref<?x?xi128>
    ```

    Example 4: Masked store a int 32-bit element ZA tile with vertical layout to memory.
    ```mlir
    arm_sme.tile_store %tile, %base[%c0, %c0], %mask layout<vertical> : vector<[4]x[4]xf32>, memref<?x?xf32>
    ```
  }];
  let arguments = (ins SMETile:$valueToStore,
    Arg<MemRefRankOf<[AnyType], [2]>, "the reference to store to", [MemWrite]>:$base,
    Variadic<Index>:$indices, Optional<AnyVectorOfNonZeroRank>:$mask,
    ArmSME_TileSliceLayoutAttr:$layout
  );
  let extraClassDeclaration = [{
    MemRefType getMemRefType() {
      return ::llvm::cast<MemRefType>(getBase().getType());
    }
    VectorType getVectorType() {
```
- **EN**: Implements logic around `horizontal`, `getMemRefType`, `getBase`, `getVectorType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `horizontal`, `getMemRefType`, `getBase`, `getVectorType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 457-470
```tablegen
      return ::llvm::cast<VectorType>(getValueToStore().getType());
    }
    VectorType getTileType() {
      return getVectorType();
    }
  }];

  let builders = [
    OpBuilder<(ins "Value":$valueToStore, "Value":$base,
                   "ValueRange":$indices), [{
      build($_builder, $_state, valueToStore, base, indices, {});
    }]>,
  ];

```
- **EN**: Implements logic around `getValueToStore`, `getTileType`, `getVectorType`, `OpBuilder`, and 1 more symbols.
- **CN**: 围绕 `getValueToStore`, `getTileType`, `getVectorType`, `OpBuilder`, and 1 more symbols 实现具体逻辑。

### Lines 471-489
```tablegen
  let assemblyFormat =
    "$valueToStore `,` $base `[` $indices `]` (`,` $mask^)? (`layout` `` $layout^)?"
      "attr-dict `:` type($base) `,` type($valueToStore)";
}

def LoadTileSliceOp : ArmSME_Op<"load_tile_slice", [
  ArmSMETileOpInterface,
  AllElementTypesMatch<["tile", "base"]>,
  AllTypesMatch<["tile", "result"]>, TileSliceMaskConstraint<"result", "mask">
]> {
  let summary = "Tile slice load and update operation";
  let description = [{
    Loads a 1D tile slice from memory into a 2D SME "virtual tile". The tile
    slice is defined by the dimension of the 2D scalable vector type pointed by
    the index. A tile slice index describes where in the input tile the tile
    slice is loaded to. An optional tile slice layout attribute specifies
    whether the tile slice being loaded at the given index is horizontal
    (default) or vertical. The updated tile is returned as the result.

```
- **EN**: Introduces declarations for `LoadTileSliceOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LoadTileSliceOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 490-502
```tablegen
    The slice of memory read is defined by a base and indices and must be
    contiguous. The memref must be either rank 1 or rank 2, have dynamic
    dimensions since the operation is scalable, and the element type must be a
    scalar that matches the element type of the result.

    The provided `mask` is used to specify which elements of the tile slice
    will be loaded.

    Example 1: Load a vector<[16]xi8> tile slice from memory into tile horizontally (default) at given index.
    ```mlir
    %tile_update = arm_sme.load_tile_slice %base[%c0], %mask, %tile, %tile_slice_index : memref<?x?xi8>, vector<[16]xi1>, vector<[16]x[16]xi8>
    ```

```
- **EN**: Implements logic around `horizontally`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `horizontally` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 503-519
```tablegen
    Example 2: Load a vector<[4]xf32> tile slice from memory into tile vertically at given index.
    ```mlir
    %tile_update = arm_sme.load_tile_slice %base[%c0], %mask, %tile, %tile_slice_index layout<vertical> : memref<?x?xf32>, vector<[4]xi1>, vector<[4]x[4]xf32>
    ```

    Example 3: Load a vector<[1]xi128> tile slice from memory into tile vertically at given index.
    ```mlir
    %tile_update = arm_sme.load_tile_slice %base[%c0], %mask, %tile, %tile_slice_index layout<vertical> : memref<?x?xi128>, vector<[1]xi1>, vector<[1]x[1]xi128>
    ```
  }];
  let arguments = (ins
    Arg<AnyMemRef, "the reference to load from", [MemRead]>:$base, SVEPredicate:$mask,
    SMETile:$tile, Variadic<Index>:$indices, Index:$tile_slice_index,
    ArmSME_TileSliceLayoutAttr:$layout
  );
  let results = (outs SMETile:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 520-531
```tablegen
  let extraClassDeclaration = [{
    MemRefType getMemRefType() {
      return ::llvm::cast<MemRefType>(getBase().getType());
    }
    VectorType getVectorType() {
      return ::llvm::cast<VectorType>(getResult().getType());
    }
    VectorType getTileType() {
      return getVectorType();
    }
  }];

```
- **EN**: Implements logic around `getMemRefType`, `getBase`, `getVectorType`, `getResult`, and 1 more symbols.
- **CN**: 围绕 `getMemRefType`, `getBase`, `getVectorType`, `getResult`, and 1 more symbols 实现具体逻辑。

### Lines 532-551
```tablegen
  let assemblyFormat = [{
    $base `[` $indices `]` `,` $mask `,` $tile `,` $tile_slice_index
      (`layout` `` $layout^)? attr-dict `:` type($base) `,` type($mask) `,`
                                            type($result)
  }];
}

def StoreTileSliceOp : ArmSME_Op<"store_tile_slice", [
  ArmSMETileOpInterface,
  AllElementTypesMatch<["tile", "base"]>,
  TileSliceMaskConstraint<"tile", "mask">
]> {
  let summary = "Tile slice store operation";
  let description = [{
    Stores a 1D tile slice from a 2D SME "virtual tile" into memory. The tile
    slice is defined by the dimension of the 2D scalable vector type pointed by
    the index. A tile slice index describes where in the input tile the tile
    slice is stored from. An optional tile slice layout attribute specifies
    whether the tile slice being stored from the given index is horizontal
    (default) or vertical.
```
- **EN**: Introduces declarations for `StoreTileSliceOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `StoreTileSliceOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 552-565
```tablegen

    The slice of memory written is defined by a base and indices and must be
    contiguous. The memref must be either rank 1 or rank 2, have dynamic
    dimensions since the operation is scalable, and the element type must be a
    scalar that matches the element type of the input tile.

    The provided `mask` is used to specify which elements of the tile slice
    will be stored.

    Example 1: Store vector<[16]xi8> horizontal (default) tile slice from tile at given index to memory.
    ```mlir
    arm_sme.store_tile_slice %tile, %tile_slice_index, %mask, %base[%c0] : vector<[16]x[16]xi8>, vector<[16]xi1>, memref<?x?xi8>
    ```

```
- **EN**: Implements logic around `horizontal`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `horizontal` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 566-585
```tablegen
    Example 2: Store vector<[4]xf32> vertical tile slice from tile at given index to memory.
    ```mlir
    arm_sme.store_tile_slice %tile, %tile_slice_index, %mask, %base[%c0] layout<vertical> : vector<[4]x[4]xf32>, vector<[4]xi1>, memref<?x?xf32>
    ```

    Example 3: Store a vector<[1]xi128> vertical tile slice from tile at given index to memory.
    ```mlir
    arm_sme.store_tile_slice %tile, %tile_slice_index, %mask, %base[%c0] layout<vertical> : vector<[1]x[1]xi128>, vector<[1]xi1>, memref<?x?xi128>
    ```
  }];
  let arguments = (ins
    SMETile:$tile, Index:$tile_slice_index, SVEPredicate:$mask,
    Arg<AnyMemRef, "the reference to store to", [MemWrite]>:$base,
    Variadic<Index>:$indices, ArmSME_TileSliceLayoutAttr:$layout
  );
  let extraClassDeclaration = [{
    MemRefType getMemRefType() {
      return ::llvm::cast<MemRefType>(getBase().getType());
    }
    VectorType getVectorType() {
```
- **EN**: Implements logic around `getMemRefType`, `getBase`, `getVectorType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMemRefType`, `getBase`, `getVectorType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 586-598
```tablegen
      return ::llvm::cast<VectorType>(getTile().getType());
    }
    VectorType getTileType() {
      return getVectorType();
    }
  }];

  let assemblyFormat = [{
    $tile `,` $tile_slice_index `,` $mask `,` $base `[` $indices `]` (`layout` `` $layout^)?
      attr-dict `:` type($base) `,` type($mask) `,` type($tile)
  }];
}

```
- **EN**: Implements logic around `getTile`, `getTileType`, `getVectorType`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getTile`, `getTileType`, `getVectorType`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 599-617
```tablegen
def InsertTileSliceOp : ArmSME_Op<"insert_tile_slice", [
    ArmSMETileOpInterface, Pure,
    AllTypesMatch<["tile", "result"]>,
    TypesMatchWith<
      "type of 'vector' matches type of 'tile' slice",
      "tile", "vector",
      "VectorType::get("
        "::llvm::cast<mlir::VectorType>($_self).getShape().drop_front(),"
        "::llvm::cast<mlir::VectorType>($_self).getElementType(),"
        "/*scalableDims=*/{true})">,
]> {
  let summary = "Insert 1-D scalable vector into slice of 2-D tile";
  let description = [{
    Inserts a 1-D scalable vector into a slice of a 2-D scalable vector tile at
    the given index. The type of the 1-D scalable vector to be inserted must
    match the type of the tile slice. A tile slice is a 1-D vector of
    horizontally or vertically contiguous elements within a ZA tile. The updated
    tile is returned as the result.

```
- **EN**: Introduces declarations for `InsertTileSliceOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `InsertTileSliceOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 618-635
```tablegen
    An optional tile slice layout attribute specifies whether the tile slice is
    horizontal (default) or vertical.

    Example 1: Insert `vector<[16]xi8>` into tile horizontally at the given index.
    ```mlir
    %tile_update = arm_sme.insert_tile_slice %vector, %tile[%tile_slice_index] : vector<[16]xi8> into vector<[16]x[16]xi8>
    ```

    Example 2: Insert `vector<[2]xf64>` into tile vertically at the given index.
    ```mlir
    %tile_update = arm_sme.insert_tile_slice %vector, %tile[%tile_slice_index] layout<vertical> : vector<[2]xf64> into vector<[2]x[2]xf64>
    ```
  }];
  let arguments = (ins
      SVEVector:$vector, SMETile:$tile, Index:$tile_slice_index,
      ArmSME_TileSliceLayoutAttr:$layout);
  let results = (outs SMETile:$result);

```
- **EN**: Declares APIs or declarative rules around `horizontal`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `horizontal` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 636-647
```tablegen
  let extraClassDeclaration = [{
    VectorType getTileType() {
      return ::llvm::cast<VectorType>(getTile().getType());
    }
  }];

  let assemblyFormat = [{
    $vector `,` $tile `[` $tile_slice_index `]` (`layout` `` $layout^)?
      attr-dict `:` type($vector) `into` type($result)
  }];
}

```
- **EN**: Implements logic around `getTileType`, `getTile`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTileType`, `getTile`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 648-660
```tablegen
def ExtractTileSliceOp : ArmSME_Op<"extract_tile_slice", [
    ArmSMETileOpInterface, Pure,
    TypesMatchWith<
      "type of 'result' matches type of 'tile' slice",
      "tile", "result",
      "VectorType(VectorType::Builder(::llvm::cast<mlir::VectorType>($_self)).dropDim(0))">,
]> {
  let summary = "Extract 1-D scalable vector from slice of 2-D tile";
  let description = [{
    Extracts a 1-D scalable slice from a 2-D scalable tile at the given index.
    A tile slice is a 1-D vector of horizontally or vertically contiguous
    elements within a ZA tile.

```
- **EN**: Introduces declarations for `ExtractTileSliceOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ExtractTileSliceOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 661-674
```tablegen
    An optional tile slice layout attribute specifies whether the tile slice is
    horizontal (default) or vertical.

    Example 1: Extract `vector<[16]xi8>` from tile horizontally at the given index.
    ```mlir
    %slice = arm_sme.extract_tile_slice %tile[%tile_slice_index] : vector<[16]xi8> from vector<[16]x[16]xi8>
    ```

    Example 2: Extract `vector<[2]xf64>` from tile vertically at the given index.
    ```mlir
    %slice = arm_sme.extract_tile_slice %tile[%tile_slice_index] layout<vertical> : vector<[2]xf64> from vector<[2]x[2]xf64>
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `horizontal`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `horizontal` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 675-687
```tablegen
  let arguments = (ins
    SMETile:$tile, Index:$tile_slice_index,
    ArmSME_TileSliceLayoutAttr:$layout
  );
  let results = (outs SVEVector:$result);

  let extraClassDeclaration = [{
    VectorType getSliceType() { return getResult().getType(); }
    VectorType getTileType() {
      return ::llvm::cast<VectorType>(getTile().getType());
    }
  }];

```
- **EN**: Implements logic around `getSliceType`, `getTileType`, `getTile`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getSliceType`, `getTileType`, `getTile` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 688-703
```tablegen
  let assemblyFormat = [{
      $tile `[` $tile_slice_index `]` (`layout` `` $layout^)? attr-dict
      `:` type($result) `from` type($tile)
  }];
}

class OuterProductResultTileTypeConstraint<string operand> :
  OptionalTypesMatchWith<operand # "type is derived from `lhs` and `rhs`",
    "lhs", operand,
    "[&]{"
    "  auto vectorType = ::llvm::cast<mlir::VectorType>($_self);"
    "  int64_t size = vectorType.getDimSize(0);"
    "  return VectorType::get("
    "    { size, size }, vectorType.getElementType(), { true, true });"
    "}()">;

```
- **EN**: Introduces declarations for `OuterProductResultTileTypeConstraint`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `OuterProductResultTileTypeConstraint` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 704-720
```tablegen
def OuterProductOp :
  ArmSME_Op<"outerproduct", [
    Pure,
    ArmSMETileOpInterface,
    AttrSizedOperandSegments,
    AllTypesMatch<["lhs", "rhs"]>,
    HasMatchingMaskTypeConstraint<"lhs", "lhsMask">,
    HasMatchingMaskTypeConstraint<"rhs", "rhsMask">,
    PredOpTrait<
      "both `lhsMask` and `rhsMask` should be provided or neither",
      CPred<"bool(getLhsMask()) == bool(getRhsMask())">>,
    OuterProductResultTileTypeConstraint<"result">,
    OuterProductResultTileTypeConstraint<"acc">
  ]>
{
  let summary = "Outer product with optional fused add/sub";

```
- **EN**: Introduces declarations for `OuterProductOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `OuterProductOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 721-732
```tablegen
  let description = [{
    This operation represents an outer product that fits within an SME tile.
    All operands must be SVE vectors and the result a SME tile. Unlike
    `vector.outerproduct` masking is on the operands (rather than the result),
    which mirrors the SME instructions.

    Example 1: Unmasked outerproduct (without accumulator)
    ```mlir
    // Not specifying an accumulator implicitly zeros the destination tile.
    %result = arm_sme.outerproduct $lhs, $rhs : vector<[4]xf32>, vector<[4]xf32>
    ```

```
- **EN**: Implements logic around `operands`, `outerproduct`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `operands`, `outerproduct` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 733-744
```tablegen
    Example 2: Unmasked outerproduct (with accumulator)
    ```mlir
    %result = arm_sme.outerproduct $lhs, $rhs acc($accumulator)
                : vector<[4]xf32>, vector<[4]xf32>
    ```

    Example 3: Masked outerproduct
    ```mlir
    %result = arm_sme.outerproduct $lhs, $rhs masks($lhsMask, $rhsMask)
                : vector<[4]xf32>, vector<[4]xf32>
    ```

```
- **EN**: Implements logic around `outerproduct`, `acc`, `masks`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `outerproduct`, `acc`, `masks` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 745-759
```tablegen
    Example 4: Masked outerproduct (with accumulator)
    ```mlir
    %result = arm_sme.outerproduct $lhs, $rhs acc($accumulator) masks($lhsMask, $rhsMask)
                : vector<[4]xf32>, vector<[4]xf32>
    ```
  }];

let arguments = (ins
    SVEVector:$lhs, SVEVector:$rhs,
    Optional<SVEPredicate>:$lhsMask,
    Optional<SVEPredicate>:$rhsMask,
    Optional<SMETile>: $acc,
    ArmSME_CombiningKindAttr:$kind);
  let results = (outs SMETile:$result);

```
- **EN**: Declares APIs or declarative rules around `outerproduct`, `acc`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `outerproduct`, `acc` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 760-778
```tablegen
  let assemblyFormat = [{
    $lhs `,` $rhs
    oilist(
        `kind` `` $kind
      | `acc` `` `(` $acc `)`
      | `masks` `` `(` $lhsMask `,` $rhsMask `)`
    ) attr-dict `:` type($lhs) `,` type($rhs)
  }];

  let extraClassDeclaration = [{
    VectorType getLhsType() { return llvm::cast<VectorType>(getLhs().getType()); }
    VectorType getRhsType() { return llvm::cast<VectorType>(getRhs().getType()); }
    VectorType getResultType() { return llvm::cast<VectorType>(getResult().getType()); }
    VectorType getTileType() {
      return getResultType();
    }
  }];
}

```
- **EN**: Implements logic around `oilist`, `type`, `getLhsType`, `getRhsType`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `oilist`, `type`, `getLhsType`, `getRhsType`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 779-798
```tablegen
class OuterProductWideningBase<string mnemonic,
                               list<Type> allowedInputVectorTypes,
                               list<Type> allowedResultVectorTypes,
                               int numOuterProducts> :
  ArmSME_Op<mnemonic, [
    Pure,
    ArmSMETileOpInterface,
    AttrSizedOperandSegments,
    AllTypesMatch<["lhs", "rhs"]>,
    HasMatchingMaskTypeConstraint<"lhs", "lhsMask">,
    HasMatchingMaskTypeConstraint<"rhs", "rhsMask">,
    PredOpTrait<
      "both `lhsMask` and `rhsMask` should be provided or neither",
      CPred<"bool(getLhsMask()) == bool(getRhsMask())">
    >,
    OptionalTypesMatchWith<"`result` and `acc` have the same type",
                           "result", "acc", "::llvm::cast<Type>($_self)">,
    // This trait ensures the input types match the correct output type for ops
    // that takes multiple inputs and outputs (i.e., 4-way).
    PredOpTrait<
```
- **EN**: Introduces declarations for `OuterProductWideningBase`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `OuterProductWideningBase` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 799-810
```tablegen
      "tile element size equals input element size * " # numOuterProducts,
      CPred<"getTileType().getElementTypeBitWidth() == "
            "(getLhsType().getElementTypeBitWidth() * " # numOuterProducts # ")">
    >,
  ]> {

  let arguments = (ins
    AnyTypeOf<allowedInputVectorTypes>:$lhs, AnyVectorOfNonZeroRank:$rhs,
    Optional<AnyVectorOfNonZeroRank>:$lhsMask, Optional<AnyVectorOfNonZeroRank>:$rhsMask,
    Optional<AnyVectorOfNonZeroRank>:$acc);
  let results = (outs AnyTypeOf<allowedResultVectorTypes>:$result);

```
- **EN**: Implements logic around `getTileType`, `getLhsType`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getTileType`, `getLhsType` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 811-828
```tablegen
  let assemblyFormat = [{
    $lhs `,` $rhs
    oilist(
        `acc` `` `(` $acc `)`
      | `masks` `` `(` $lhsMask `,` $rhsMask `)`
    ) attr-dict `:` type($lhs) `,` type($rhs) `into` type($result)
  }];

  let extraClassDeclaration = [{
    VectorType getLhsType() { return llvm::cast<VectorType>(getLhs().getType()); }
    VectorType getRhsType() { return llvm::cast<VectorType>(getRhs().getType()); }
    VectorType getResultType() { return llvm::cast<VectorType>(getResult().getType()); }
    VectorType getTileType() {
      return getResultType();
    }
  }];
}

```
- **EN**: Implements logic around `oilist`, `type`, `getLhsType`, `getRhsType`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `oilist`, `type`, `getLhsType`, `getRhsType`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 829-840
```tablegen
class OuterProduct2Way<string mnemonic,
                       list<Type> allowedInputVectorTypes,
                       list<Type> allowedResultVectorTypes>
  : OuterProductWideningBase<mnemonic, allowedInputVectorTypes,
                             allowedResultVectorTypes, /*numOuterProducts=*/2>;

def FMopa2WayOp
  : OuterProduct2Way<"fmopa_2way",
      [ScalableVectorOfRankAndLengthAndType<[1], [8], [F16, BF16]>],
      [nxnxv4f32]> {
  let summary = "Floating-point sum of 2 outer products and accumulate";

```
- **EN**: Introduces declarations for `OuterProduct2Way`, `FMopa2WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `OuterProduct2Way`, `FMopa2WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 841-851
```tablegen
  let description = [{
    This operation represents a sum of 2 widened outer products. It takes 2 1-D
    scalable vectors as input and a 2-D scalable vector (ZA tile) as output.

    For example (fp16 to fp32):

    ```mlir
    %result = arm_sme.fmopa_2way %lhs, %rhs :
      vector<[8]xf16>, vector<[8]xf16> into vector<[4]x[4]xf32>
    ```

```
- **EN**: Implements logic around `vector`, `example`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `vector`, `example` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 852-862
```tablegen
    The `lhs` encodes a matrix of shape SVLSx2 and the `rhs` a matrix of
    2xSVLS, where SVLS (spec [1], section B2.1) is the number of 32-bit
    elements in a vector of SVL bits. To illustrate, below is a breakdown of
    this operation for fp16 to fp32, SVL=128 (i.e., vscale=1):

    ```
                          LHS                          RHS
               [A0 A1 A2 A3 A4 A5 A6 A7]    [B0 B1 B2 B3 B4 B5 B6 B7]

    ----------------------------------------------------------------------------

```
- **EN**: Implements logic around `SVLS`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `SVLS` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 863-873
```tablegen
                                  implicit layout

                              [A0 A1]    |
                              [A2 A3]    |    [B0 B2 B4 B6]
                              [A4 A5]    |    [B1 B3 B5 B7]
                              [A6 A7]    |

    ----------------------------------------------------------------------------

                                  2 outer products

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 874-884
```tablegen
                      Acol0 ⊗ Brow0      |           Acol1 ⊗ Brow1
                      -------------      |           -------------
                                         |
                  [B0 B2 B4 B6]          |       [B1 B3 B5 B7]
                                         |
             [A0  [A0B0 A0B2 A0B4 A0B6]  |  [A1  [A1B1 A1B3 A1B5 A1B7]
              A2  [A2B0 A2B2 A2B4 A2B6]  |   A3  [A3B1 A3B3 A3B5 A3B7]
              A4  [A4B0 A4B2 A4B4 A4B6]  |   A5  [A5B1 A5B3 A5B5 A5B7]
              A6] [A6B0 A6B2 A6B4 A6B6]  |   A7] [A7B1 A7B3 A7B5 A7B7]
                                         |

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 885-895
```tablegen
    ----------------------------------------------------------------------------

                              sum of 2 outer products

                           Acol0 ⊗ Brow0 + Acol1 ⊗ Brow1

                 [A0B0 + A1B1 A0B2 + A1B3 A0B4 + A1B5 A0B6 + A1B7]
                 [A2B0 + A3B1 A2B2 + A3B3 A2B4 + A3B5 A2B6 + A3B7]
                 [A4B0 + A5B1 A4B2 + A5B3 A4B4 + A5B5 A4B6 + A5B7]
                 [A6B0 + A7B1 A6B2 + A7B3 A6B4 + A7B5 A6B6 + A7B7]

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 896-909
```tablegen
    ----------------------------------------------------------------------------
    ```

    This operation enables the folding of 2 outer products chained via the
    accumulator into a single outer product.

    For example:

    ```mlir
    %a0_ext = arith.extf %a0 : vector<[4]xf16> to vector<[4]xf32>
    %b0_ext = arith.extf %b0 : vector<[4]xf16> to vector<[4]xf32>
    %a1_ext = arith.extf %a1 : vector<[4]xf16> to vector<[4]xf32>
    %b1_ext = arith.extf %b1 : vector<[4]xf16> to vector<[4]xf32>

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 910-922
```tablegen
    %0 = arm_sme.outerproduct %a0_ext, %b0_ext : vector<[4]xf32>, vector<[4]xf32>
    %1 = arm_sme.outerproduct %a1_ext, %b1_ext acc(%0) : vector<[4]xf32>, vector<[4]xf32>
    ```

    The 2 outer products in the example above can be fused into a single outer
    product as follows:

    ```mlir
    %a_packed = vector.interleave %a0, %a1 : vector<[4]xf16> -> vector<[8]xf16>
    %b_packed = vector.interleave %b0, %b1 : vector<[4]xf16> -> vector<[8]xf16>
    %0 = arm_sme.fmopa_2way %a_packed, %b_packed : vector<[8]xf16>, vector<[8]xf16> into vector<[4]x[4]xf32>
    ```

```
- **EN**: Implements logic around `acc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `acc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 923-934
```tablegen
    This is implemented in the `-arm-sme-outer-product-fusion` pass.

    Example: FP16 to FP32
    ```mlir
    %result = arm_sme.fmopa_2way $lhs, $rhs : vector<[8]xf16>, vector<[8]xf16> into vector<[4]x[4]xf32>
    ```

    Example: BF16 to FP32
    ```mlir
    %result = arm_sme.fmopa_2way $lhs, $rhs : vector<[8]xbf16>, vector<[8]xbf16> into vector<[4]x[4]xf32>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 935-948
```tablegen
    | Spec | Features |
    | ---- | -------- |
    | [FMOPA (widening, 2-way, FP16 to FP32)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/FMOPA--widening--2-way--FP16-to-FP32---Half-precision-floating-point-sum-of-outer-products-and-accumulate-) | +sme |
    | [BFMOPA (widening, 2-way, BF16 to FP32)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/BFMOPA--widening---BFloat16-sum-of-outer-products-and-accumulate-) | +sme |

    [1] https://developer.arm.com/documentation/ddi0616
  }];
}

// TODO: support:
// - FMOPA 2-way FP8 to FP16
// - FMOPA 4-way FP16 to FP32
// once intrinsic support lands in the backend.

```
- **EN**: Declares APIs or declarative rules around `FMOPA`, `BFMOPA`.
- **CN**: 声明与 `FMOPA`, `BFMOPA` 相关的 API 或声明式规则。

### Lines 949-962
```tablegen
def FMops2WayOp
  : OuterProduct2Way<"fmops_2way",
      [ScalableVectorOfRankAndLengthAndType<[1], [8], [F16, BF16]>],
      [nxnxv4f32]> {
  let summary = "Floating-point sum of 2 outer products and subtract";
  let description = [{
    Equivalent to `fmopa_2way` but outer products are subtracted from
    destination `result`.

    Example: FP16 to FP32
    ```mlir
    %result = arm_sme.fmops_2way $lhs, $rhs : vector<[8]xf16>, vector<[8]xf16> into vector<[4]x[4]xf32>
    ```

```
- **EN**: Introduces declarations for `FMops2WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `FMops2WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 963-978
```tablegen
    Example: BF16 to FP32
    ```mlir
    %result = arm_sme.fmops_2way $lhs, $rhs : vector<[8]xbf16>, vector<[8]xbf16> into vector<[4]x[4]xf32>
    ```

    Refer to
    [fmopa_2way](#arm_smefmopa_2way-arm_smefmopa2wayop) for a detailed
    description of 2-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [FMOPS (widening, 2-way, FP16 to FP32)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/FMOPS--widening---Half-precision-floating-point-sum-of-outer-products-and-subtract-) | +sme |
    | [BFMOPS (widening, 2-way, BF16 to FP32)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/BMOPS--Bitwise-exclusive-NOR-population-count-outer-product-and-subtract-) | +sme |
  }];
}

```
- **EN**: Declares APIs or declarative rules around `FMOPS`, `BFMOPS`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `FMOPS`, `BFMOPS` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 979-989
```tablegen
def SMopa2WayOp
  : OuterProduct2Way<"smopa_2way",
      [ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32]> {
  let summary = "Signed integer sum of 2 outer products and accumulate";
  let description = [{
    Example:
    ```mlir
    %result = arm_sme.smopa_2way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `SMopa2WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMopa2WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 990-999
```tablegen
    Refer to
    [fmopa_2way](#arm_smefmopa_2way-arm_smefmopa2wayop) for a detailed
    description of 2-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [SMOPA (2-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/SMOPA--2-way---Signed-integer-sum-of-outer-products-and-accumulate-) | +sme2 |
  }];
}

```
- **EN**: Declares APIs or declarative rules around `SMOPA`.
- **CN**: 声明与 `SMOPA` 相关的 API 或声明式规则。

### Lines 1000-1010
```tablegen
def SMops2WayOp
  : OuterProduct2Way<"smops_2way",
      [ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32]> {
  let summary = "Signed integer sum of 2 outer products and subtract";
  let description = [{
    Example:
    ```mlir
    %result = arm_sme.smops_2way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `SMops2WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMops2WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1011-1020
```tablegen
    Refer to
    [fmopa_2way](#arm_smefmopa_2way-arm_smefmopa2wayop) for a detailed
    description of 2-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [SMOPS (2-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/SMOPS--2-way---Signed-integer-sum-of-outer-products-and-subtract-) | +sme2 |
  }];
}

```
- **EN**: Declares APIs or declarative rules around `SMOPS`.
- **CN**: 声明与 `SMOPS` 相关的 API 或声明式规则。

### Lines 1021-1031
```tablegen
def UMopa2WayOp
  : OuterProduct2Way<"umopa_2way",
      [ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32]> {
  let summary = "Unsiged integer sum of 2 outer products and accumulate";
  let description = [{
    Example:
    ```mlir
    %result = arm_sme.umopa_2way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `UMopa2WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UMopa2WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1032-1041
```tablegen
    Refer to
    [fmopa_2way](#arm_smefmopa_2way-arm_smefmopa2wayop) for a detailed
    description of 2-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [UMOPA (2-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/UMOPA--2-way---Unsigned-integer-sum-of-outer-products-and-accumulate-) | +sme2 |
  }];
}

```
- **EN**: Declares APIs or declarative rules around `UMOPA`.
- **CN**: 声明与 `UMOPA` 相关的 API 或声明式规则。

### Lines 1042-1052
```tablegen
def UMops2WayOp
  : OuterProduct2Way<"umops_2way",
      [ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32]> {
  let summary = "Unsiged integer sum of 2 outer products and subtract";
  let description = [{
    Example:
    ```mlir
    %result = arm_sme.umops_2way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `UMops2WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UMops2WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1053-1062
```tablegen
    Refer to
    [fmopa_2way](#arm_smefmopa_2way-arm_smefmopa2wayop) for a detailed
    description of 2-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [UMOPS (2-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/UMOPS--2-way---Unsigned-integer-sum-of-outer-products-and-subtract-) | +sme2 |
  }];
}

```
- **EN**: Declares APIs or declarative rules around `UMOPS`.
- **CN**: 声明与 `UMOPS` 相关的 API 或声明式规则。

### Lines 1063-1078
```tablegen
class OuterProduct4Way<string mnemonic,
                       list<Type> allowedInputVectorTypes,
                       list<Type> allowedResultVectorTypes>
  : OuterProductWideningBase<mnemonic, allowedInputVectorTypes,
                             allowedResultVectorTypes, /*numOuterProducts=*/4>;

def SMopa4WayOp
  : OuterProduct4Way<"smopa_4way",
      [ScalableVectorOfRankAndLengthAndType<[1], [16], [I8]>,
       ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32, nxnxv2i64]> {
  let summary = "Signed integer sum of 4 outer products and accumulate";
  let description = [{
    This operation represents a sum of 4 widened outer products. It takes 2 1-D
    scalable vectors as input and a 2-D scalable vector (ZA tile) as output.

```
- **EN**: Introduces declarations for `OuterProduct4Way`, `SMopa4WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `OuterProduct4Way`, `SMopa4WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1079-1090
```tablegen
    For example (i8 to i32):

    ```mlir
    %result = arm_sme.smopa_4way $lhs, $rhs :
      vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

    The `lhs` encodes a matrix of shape SVLSx4 and the `rhs` a matrix of
    4xSVLS, where SVLS (spec [1], section B2.1) is the number of 32-bit
    elements in a vector of SVL bits. To illustrate, below is a breakdown of
    this operation for i8 to i32, SVL=128 (i.e., vscale=1):

```
- **EN**: Implements logic around `example`, `SVLS`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `example`, `SVLS` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1091-1101
```tablegen
    ```
                                        LHS
              [A0 A1 A2 A3 A4 A5 A6 A7 A8 A9 A10 A11 A12 A15 A14 A15]

                                        RHS
              [B0 B1 B2 B3 B4 B5 B6 B7 B8 B9 B10 B11 B12 B13 B14 B15]

    ----------------------------------------------------------------------------

                                  implicit layout

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1102-1121
```tablegen
                    [A0   A1  A2  A3]    |    [B0 B4  B8 B12]
                    [A4   A5  A6  A7]    |    [B1 B5  B9 B13]
                    [A8   A9 A10 A11]    |    [B2 B6 B10 B14]
                    [A12 A13 A14 A15]    |    [B3 B7 B11 B15]

    ----------------------------------------------------------------------------

                                  4 outer products

                 Acol0 ⊗ Brow0           |            Acol1 ⊗ Brow1
                 -------------           |            -------------
                                         |
             [B0 B4 B8 B12]              |        [B1 B5 B9 B13]
                                         |
       [A0   [ A0B0  A0B4  A0B8  A0B12]  |  [A1   [ A1B1  A1B5  A1B9  A1B13]
        A4   [ A4B0  A4B4  A4B8  A4B12]  |   A5   [ A5B1  A5B5  A5B9  A5B13]
        A8   [ A8B0  A8B4  A8B8  A8B12]  |   A9   [ A9B1  A9B5  A9B9  A9B13]
        A12] [A12B0 A12B4 A12B8 A12B12]  |   A13] [A13B1 A13B5 A13B9 A13B13]
                                         |
                 Acol2 ⊗ Brow2           |            Acol3 ⊗ Brow3
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1122-1131
```tablegen
                 -------------           |            -------------
                                         |
             [B2, B6, B10, B14]          |        [B3 B7 B11 B15]
                                         |
       [A2   [ A2B2  A2B6  A2B10  A2B14] |  [A3   [ A3B3  A3B7  A3B11  A3B15]
        A6   [ A6B2  A6B6  A6B10  A6B14] |   A7   [ A7B3  A7B7  A7B11  A7B15]
        A10  [A10B2 A10B6 A10B10 A10B14] |   A11  [A11B3 A11B7 A11B11 A11B15]
        A14] [A14B2 A14B6 A14B10 A14B14] |   A15] [A15B3 A15B7 A15B11 A15B15]
                                         |

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1132-1142
```tablegen
    ----------------------------------------------------------------------------

                              sum of 4 outer products

           Acol0 ⊗ Brow0 + Acol1 ⊗ Brow1 + Acol2 ⊗ Brow2 + Acol3 ⊗ Brow3

     [ A0B0 +  A1B1 +  A2B2 +  A3B3 ... ...  A0B12 +  A1B13 +  A2B14 +  A3B15]
     [ A4B0 +  A5B1 +  A6B2 +  A7B3 ... ...  A4B12 +  A5B13 +  A6B14 +  A7B15]
     [ A8B0 +  A9B1 + A10B2 + A11B3 ... ...  A8B12 +  A9B13 + A10B14 + A11B15]
     [A12B0 + A13B1 + A14B2 + A15B3 ... ... A12B12 + A13B13 + A14B14 + A15B15]

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1143-1154
```tablegen
    ----------------------------------------------------------------------------
    ```

    This operation enables the folding of 4 outer products chained via the
    accumulator into a single outer product.

    For example:

    ```mlir
    %a0_ext = arith.extsi %a0 : vector<[4]xi8> to vector<[4]xi32>
    %b0_ext = arith.extsi %b0 : vector<[4]xi8> to vector<[4]xi32>

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1155-1169
```tablegen
    %a1_ext = arith.extsi %a1 : vector<[4]xi8> to vector<[4]xi32>
    %b1_ext = arith.extsi %b1 : vector<[4]xi8> to vector<[4]xi32>

    %a2_ext = arith.extsi %a2 : vector<[4]xi8> to vector<[4]xi32>
    %b2_ext = arith.extsi %b2 : vector<[4]xi8> to vector<[4]xi32>

    %a3_ext = arith.extsi %a3 : vector<[4]xi8> to vector<[4]xi32>
    %b3_ext = arith.extsi %b3 : vector<[4]xi8> to vector<[4]xi32>

    %0 = arm_sme.outerproduct %a0_ext, %b0_ext : vector<[4]xi32>, vector<[4]xi32>
    %1 = arm_sme.outerproduct %a1_ext, %b1_ext acc(%0) : vector<[4]xi32>, vector<[4]xi32>
    %2 = arm_sme.outerproduct %a2_ext, %b2_ext acc(%1) : vector<[4]xi32>, vector<[4]xi32>
    %3 = arm_sme.outerproduct %a3_ext, %b3_ext acc(%2) : vector<[4]xi32>, vector<[4]xi32>
    ```

```
- **EN**: Implements logic around `acc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `acc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1170-1181
```tablegen
    The 4 outer products in the example above can be fused into a single outer
    product as follows:

    ```mlir
    %lhs0 = vector.interleave %a0, %a2 : vector<[4]xi8> -> vector<[8]xi8>
    %lhs1 = vector.interleave %a1, %a3 : vector<[4]xi8> -> vector<[8]xi8>
    %lhs = vector.interleave %lhs0, %lhs1 : vector<[8]xi8> -> vector<[16]xi8>

    %rhs0 = vector.interleave %b0, %b2 : vector<[4]xi8> -> vector<[8]xi8>
    %rhs1 = vector.interleave %b1, %b3 : vector<[4]xi8> -> vector<[8]xi8>
    %rhs = vector.interleave %rhs0, %rhs1 : vector<[8]xi8> -> vector<[16]xi8>

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1182-1191
```tablegen
    %0 = arm_sme.smopa_4way %lhs, %rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

    This is implemented in the `-arm-sme-outer-product-fusion` pass.

    Example: I8 to I32
    ```mlir
    %result = arm_sme.smopa_4way $lhs, $rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1192-1202
```tablegen
    Example: I16 to I64
    ```mlir
    %result = arm_sme.smopa_4way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[2]x[2]xi64>
    ```

    | Spec | Features |
    | ---- | -------- |
    | [SMOPA (4-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/SMOPA--4-way---Signed-integer-sum-of-outer-products-and-accumulate-) | +sme (32-bit), +sme-i16i64 (64-bit)|
  }];
}

```
- **EN**: Declares APIs or declarative rules around `SMOPA`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `SMOPA` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1203-1212
```tablegen
def SMops4WayOp
  : OuterProduct4Way<"smops_4way",
      [ScalableVectorOfRankAndLengthAndType<[1], [16], [I8]>,
       ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32, nxnxv2i64]> {
  let summary = "Signed integer sum of 4 outer products and subtract";
  let description = [{
    Equivalent to `smopa_4way` but outer products are subtracted from
    destination `result`.

```
- **EN**: Introduces declarations for `SMops4WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMops4WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1213-1222
```tablegen
    Example: I8 to I32
    ```mlir
    %result = arm_sme.smops_4way $lhs, $rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

    Example: I16 to I64
    ```mlir
    %result = arm_sme.smops_4way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[2]x[2]xi64>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1223-1232
```tablegen
    Refer to [smopa_4way](#arm_smesmopa_4way-arm_smesmopa4wayop) for a
    detailed description of 4-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [SMOPS (4-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/SMOPS--4-way---Signed-integer-sum-of-outer-products-and-subtract-) | +sme (32-bit), +sme-i16i64 (64-bit)|
  }];
}

def UMopa4WayOp
```
- **EN**: Introduces declarations for `UMopa4WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UMopa4WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1233-1243
```tablegen
  : OuterProduct4Way<"umopa_4way",
      [ScalableVectorOfRankAndLengthAndType<[1], [16], [I8]>,
       ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32, nxnxv2i64]> {
  let summary = "Unsigned integer sum of 4 outer products and accumulate";
  let description = [{
    Example: I8 to I32
    ```mlir
    %result = arm_sme.umopa_4way $lhs, $rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1244-1257
```tablegen
    Example: I16 to I64
    ```mlir
    %result = arm_sme.umopa_4way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[2]x[2]xi64>
    ```

    Refer to [smopa_4way](#arm_smesmopa_4way-arm_smesmopa4wayop) for a
    detailed description of 4-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [UMOPA (4-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/UMOPA--4-way---Unsigned-integer-sum-of-outer-products-and-accumulate-) | +sme (32-bit), +sme-i16i64 (64-bit)|
  }];
}

```
- **EN**: Declares APIs or declarative rules around `UMOPA`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `UMOPA` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1258-1269
```tablegen
def UMops4WayOp
  : OuterProduct4Way<"umops_4way",
      [ScalableVectorOfRankAndLengthAndType<[1], [16], [I8]>,
       ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32, nxnxv2i64]> {
  let summary = "Unsigned integer sum of 4 outer products and subtract";
  let description = [{
    Example: I8 to I32
    ```mlir
    %result = arm_sme.umops_4way $lhs, $rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `UMops4WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UMops4WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1270-1283
```tablegen
    Example: I16 to I64
    ```mlir
    %result = arm_sme.umops_4way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[2]x[2]xi64>
    ```

    Refer to [smopa_4way](#arm_smesmopa_4way-arm_smesmopa4wayop) for a
    detailed description of 4-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [UMOPS (4-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/UMOPS--4-way---Unsigned-integer-sum-of-outer-products-and-subtract-) | +sme (32-bit), +sme-i16i64 (64-bit)|
  }];
}

```
- **EN**: Declares APIs or declarative rules around `UMOPS`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `UMOPS` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1284-1295
```tablegen
def SuMopa4WayOp
  : OuterProduct4Way<"sumopa_4way",
      [ScalableVectorOfRankAndLengthAndType<[1], [16], [I8]>,
       ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32, nxnxv2i64]> {
  let summary = "Signed by unsigned integer sum of 4 outer products and accumulate";
  let description = [{
    Example: I8 to I32
    ```mlir
    %result = arm_sme.sumopa_4way $lhs, $rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `SuMopa4WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SuMopa4WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1296-1309
```tablegen
    Example: I16 to I64
    ```mlir
    %result = arm_sme.sumopa_4way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[2]x[2]xi64>
    ```

    Refer to [smopa_4way](#arm_smesmopa_4way-arm_smesmopa4wayop) for a
    detailed description of 4-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [SUMOPA (4-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/SUMOPA--Signed-by-unsigned-integer-sum-of-outer-products-and-accumulate-) | +sme (32-bit), +sme-i16i64 (64-bit)|
  }];
}

```
- **EN**: Declares APIs or declarative rules around `SUMOPA`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `SUMOPA` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1310-1321
```tablegen
def SuMops4WayOp
  : OuterProduct4Way<"sumops_4way",
      [ScalableVectorOfRankAndLengthAndType<[1], [16], [I8]>,
       ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32, nxnxv2i64]> {
  let summary = "Signed by unsigned integer sum of 4 outer products and subtract";
  let description = [{
    Example: I8 to I32
    ```mlir
    %result = arm_sme.sumops_4way $lhs, $rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `SuMops4WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SuMops4WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1322-1335
```tablegen
    Example: I16 to I64
    ```mlir
    %result = arm_sme.sumops_4way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[2]x[2]xi64>
    ```

    Refer to [smopa_4way](#arm_smesmopa_4way-arm_smesmopa4wayop) for a
    detailed description of 4-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [SUMOPS (4-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/SUMOPS--Signed-by-unsigned-integer-sum-of-outer-products-and-subtract-) | +sme (32-bit), +sme-i16i64 (64-bit)|
  }];
}

```
- **EN**: Declares APIs or declarative rules around `SUMOPS`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `SUMOPS` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1336-1347
```tablegen
def UsMopa4WayOp
  : OuterProduct4Way<"usmopa_4way",
      [ScalableVectorOfRankAndLengthAndType<[1], [16], [I8]>,
       ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32, nxnxv2i64]> {
  let summary = "Unsigned by signed integer sum of 4 outer products and accumulate";
  let description = [{
    Example: I8 to I32
    ```mlir
    %result = arm_sme.usmopa_4way $lhs, $rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `UsMopa4WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UsMopa4WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1348-1361
```tablegen
    Example: I16 to I64
    ```mlir
    %result = arm_sme.usmopa_4way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[2]x[2]xi64>
    ```

    Refer to [smopa_4way](#arm_smesmopa_4way-arm_smesmopa4wayop) for a
    detailed description of 4-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [USMOPA (4-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/USMOPA--Unsigned-by-signed-integer-sum-of-outer-products-and-accumulate-) | +sme (32-bit), +sme-i16i64 (64-bit)|
  }];
}

```
- **EN**: Declares APIs or declarative rules around `USMOPA`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `USMOPA` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1362-1373
```tablegen
def UsMops4WayOp
  : OuterProduct4Way<"usmops_4way",
      [ScalableVectorOfRankAndLengthAndType<[1], [16], [I8]>,
       ScalableVectorOfRankAndLengthAndType<[1], [8], [I16]>],
      [nxnxv4i32, nxnxv2i64]> {
  let summary = "Unsigned by signed integer sum of 4 outer products and subtract";
  let description = [{
    Example: I8 to I32
    ```mlir
    %result = arm_sme.usmops_4way $lhs, $rhs : vector<[16]xi8>, vector<[16]xi8> into vector<[4]x[4]xi32>
    ```

```
- **EN**: Introduces declarations for `UsMops4WayOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UsMops4WayOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1374-1387
```tablegen
    Example: I16 to I64
    ```mlir
    %result = arm_sme.usmops_4way $lhs, $rhs : vector<[8]xi16>, vector<[8]xi16> into vector<[2]x[2]xi64>
    ```

    Refer to [smopa_4way](#arm_smesmopa_4way-arm_smesmopa4wayop) for a
    detailed description of 4-way outer products.

    | Spec | Features |
    | ---- | -------- |
    | [USMOPS (4-way)](https://developer.arm.com/documentation/ddi0602/2023-09/SME-Instructions/USMOPS--Unsigned-by-signed-integer-sum-of-outer-products-and-subtract-) | +sme (32-bit), +sme-i16i64 (64-bit)|
  }];
}

```
- **EN**: Declares APIs or declarative rules around `USMOPS`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `USMOPS` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1388-1407
```tablegen
def StreamingVLOp : ArmSME_Op<"streaming_vl", [Pure]>
{
  let summary = "Query the streaming vector length";

  let description = [{
    This operation returns the streaming vector length (SVL) for a given type
    size. Unlike `vector.vscale` the value returned is invariant to the
    streaming mode.

    Example:
    ```mlir
    // Streaming vector length in:
    // - bytes (8-bit, SVL.B)
    %svl_b = arm_sme.streaming_vl <byte>
    // - half words (16-bit, SVL.H)
    %svl_h = arm_sme.streaming_vl <half>
    // - words (32-bit, SVL.W)
    %svl_w = arm_sme.streaming_vl <word>
    // - double words (64-bit, SVL.D)
    %svl_d = arm_sme.streaming_vl <double>
```
- **EN**: Introduces declarations for `StreamingVLOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `StreamingVLOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1408-1417
```tablegen
    ```
  }];

  let arguments = (ins ArmSME_TypeSizeAttr: $type_size);
  let results = (outs Index);

  let assemblyFormat = "$type_size attr-dict";
}

#endif // ARMSME_OPS
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

- **TableGen includes / TableGen 包含**: `ArmSME.td`, `mlir/IR/EnumAttr.td`, `mlir/IR/OpBase.td`, `mlir/Interfaces/SideEffectInterfaces.td`, `mlir/Dialect/LLVMIR/LLVMOpBase.td`, `mlir/Interfaces/InferTypeOpInterface.td`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (2), dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (1)
