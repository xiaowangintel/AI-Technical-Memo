# TritonTypes.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/Triton/IR/TritonTypes.td`
- **EN:** Declares or defines type records and helper utilities for this subsystem.
- **CN:** 声明或定义该子系统的类型记录与辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
   1: #ifndef TRITON_TYPES
   2: #define TRITON_TYPES
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-7
```tablegen
   4: include "mlir/IR/AttrTypeBase.td"
   5: include "mlir/IR/BuiltinTypeInterfaces.td"
   6: include "triton/Dialect/Triton/IR/TritonDialect.td"
   7: include "triton/Dialect/Triton/IR/TritonTypeInterfaces.td"
```
**EN:** This block imports the direct dependencies needed here, including mlir/IR/AttrTypeBase.td, mlir/IR/BuiltinTypeInterfaces.td, triton/Dialect/Triton/IR/TritonDialect.td, and triton/Dialect/Triton/IR/TritonTypeInterfaces.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/IR/AttrTypeBase.td, mlir/IR/BuiltinTypeInterfaces.td, triton/Dialect/Triton/IR/TritonDialect.td, and triton/Dialect/Triton/IR/TritonTypeInterfaces.td。

### Lines 9-16
```tablegen
   9: //
  10: // Types
  11: //
  12: class TritonTypeDef<string name, string _mnemonic, list<Trait> traits = []>
  13:     : TypeDef<Triton_Dialect, name, traits> {
  14:     // Used by printer/parser
  15:     let mnemonic = _mnemonic;
  16: }
```
**EN:** This block introduces `TritonTypeDef`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `TritonTypeDef`。

### Lines 18-21
```tablegen
  18: // Floating-point Type
  19: def TT_Float : AnyTypeOf<[F8E4M3FN, F8E4M3FNUZ, F8E5M2, F8E5M2FNUZ, F16, BF16, F32, F64], "floating-point">;
  20: def TT_FloatTensor : RankedTensorOf<[TT_Float]>;
  21: def TT_FloatLike : AnyTypeOf<[TT_Float, TT_FloatTensor]>;
```
**EN:** This TableGen def record defines `TT_Float`. It is specialized from `AnyTypeOf<[F8E4M3FN, F8E4M3FNUZ, F8E5M2, F8E5M2FNUZ, F16, BF16, F32, F64], "floating-point">;`.
**CN:** 该 TableGen def 记录定义了 `TT_Float`。 它基于 `AnyTypeOf<[F8E4M3FN, F8E4M3FNUZ, F8E5M2, F8E5M2FNUZ, F16, BF16, F32, F64], "floating-point">;` 进一步特化。

### Lines 23-26
```tablegen
  23: // Boolean Type
  24: // TT_Bool -> I1
  25: def TT_BoolTensor : RankedTensorOf<[I1]>;
  26: def TT_BoolLike : AnyTypeOf<[I1, TT_BoolTensor]>;
```
**EN:** This TableGen def record defines `TT_BoolTensor`. It is specialized from `RankedTensorOf<[I1]>;`.
**CN:** 该 TableGen def 记录定义了 `TT_BoolTensor`。 它基于 `RankedTensorOf<[I1]>;` 进一步特化。

### Lines 28-32
```tablegen
  28: // Integer Type
  29: def I4 : I<4>;
  30: def TT_Int : AnyTypeOf<[I1, I4, I8, I16, I32, I64], "integer">;
  31: def TT_IntTensor : RankedTensorOf<[TT_Int]>;
  32: def TT_IntLike : AnyTypeOf<[TT_Int, TT_IntTensor]>;
```
**EN:** This TableGen def record defines `I4`. It is specialized from `I<4>;`.
**CN:** 该 TableGen def 记录定义了 `I4`。 它基于 `I<4>;` 进一步特化。

### Lines 34-37
```tablegen
  34: // I32 Type
  35: // TT_I32 -> I32
  36: // TT_I32Tensor -> I32Tensor
  37: def TT_I32Like : AnyTypeOf<[I32, I32Tensor]>;
```
**EN:** This TableGen def record defines `TT_I32Like`. It is specialized from `AnyTypeOf<[I32, I32Tensor]>;`.
**CN:** 该 TableGen def 记录定义了 `TT_I32Like`。 它基于 `AnyTypeOf<[I32, I32Tensor]>;` 进一步特化。

### Lines 39-42
```tablegen
  39: // I64 Type
  40: // TT_I64 -> I64
  41: // TT_I64Tensor -> I64Tensor
  42: def TT_I64Like : AnyTypeOf<[I64, I64Tensor]>;
```
**EN:** This TableGen def record defines `TT_I64Like`. It is specialized from `AnyTypeOf<[I64, I64Tensor]>;`.
**CN:** 该 TableGen def 记录定义了 `TT_I64Like`。 它基于 `AnyTypeOf<[I64, I64Tensor]>;` 进一步特化。

### Lines 44-51
```tablegen
  44: // Pointer Type in TableGen
  45: class TT_PtrOf<list<Type> pointeeTypes> :
  46:     DialectType<Triton_Dialect,
  47:                 And<[CPred<"::mlir::isa<::mlir::triton::PointerType>($_self)">,
  48:                      Concat<"[](::mlir::Type pointeeType) { return ",
  49:                             SubstLeaves<"$_self", "pointeeType", AnyTypeOf<pointeeTypes>.predicate>,
  50:                                         "; }(::mlir::cast<::mlir::triton::PointerType>($_self).getPointeeType())">]>,
  51:                 "ptr", "::mlir::triton::PointerType">;
```
**EN:** This block introduces `TT_PtrOf`, the main class/struct defined here. Within the declaration, methods such as getPointeeType expose its core API. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `TT_PtrOf`。 其中 getPointeeType 等方法构成了它的主要接口。 它还通过继承复用基类能力。

### Lines 53-55
```tablegen
  53: // Pointer Type in C++ (corresponding to `TT_PtrOf`)
  54: def TT_PtrType : TritonTypeDef<"Pointer", "ptr"> {
  55:     let summary = "Pointer type (`::mlir::triton::PointerType`) in Triton IR type system";
```
**EN:** This TableGen def record defines `TT_PtrType` with the summary “Pointer type (`::mlir::triton::PointerType`) in Triton IR type system”. It is specialized from `TritonTypeDef<"Pointer", "ptr">`.
**CN:** 该 TableGen def 记录定义了 `TT_PtrType`，其摘要为“Pointer type (`::mlir::triton::PointerType`) in Triton IR type system”。 它基于 `TritonTypeDef<"Pointer", "ptr">` 进一步特化。

### Lines 57-59
```tablegen
  57:     let description = [{
  58:         Pointer type in Triton IR type system. Pointer types may only point to scalar element types.
  59:     }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 61-61
```tablegen
  61:     let parameters = (ins "Type":$pointeeType, "int":$addressSpace);
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 63-70
```tablegen
  63:     let builders = [
  64:         TypeBuilderWithInferredContext<(ins
  65:             "Type":$pointeeType,
  66:             "int":$addressSpace
  67:         ), [{
  68:             return $_get(pointeeType.getContext(), pointeeType, addressSpace);
  69:         }]>
  70:     ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 72-73
```tablegen
  72:     let hasCustomAssemblyFormat = 1;
  73:     let genVerifyDecl = 1;
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 75-76
```tablegen
  75:     let skipDefaultBuilders = 1;
  76: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 78-79
```tablegen
  78: // Scalar Pointer Type: `ptr<>`
  79: def TT_Ptr : TT_PtrOf<[AnyType]>;
```
**EN:** This TableGen def record defines `TT_Ptr`. It is specialized from `TT_PtrOf<[AnyType]>;`.
**CN:** 该 TableGen def 记录定义了 `TT_Ptr`。 它基于 `TT_PtrOf<[AnyType]>;` 进一步特化。

### Lines 81-82
```tablegen
  81: // Tensor of Pointer Type: `tensor<ptr<>>`
  82: def TT_PtrTensor : RankedTensorOf<[TT_Ptr]>;
```
**EN:** This TableGen def record defines `TT_PtrTensor`. It is specialized from `RankedTensorOf<[TT_Ptr]>;`.
**CN:** 该 TableGen def 记录定义了 `TT_PtrTensor`。 它基于 `RankedTensorOf<[TT_Ptr]>;` 进一步特化。

### Lines 84-85
```tablegen
  84: // Tensor of Pointer Type or Pointer type: `tensor<ptr<>>` or `ptr<>`
  85: def TT_PtrLike : AnyTypeOf<[TT_Ptr, TT_PtrTensor]>;
```
**EN:** This TableGen def record defines `TT_PtrLike`. It is specialized from `AnyTypeOf<[TT_Ptr, TT_PtrTensor]>;`.
**CN:** 该 TableGen def 记录定义了 `TT_PtrLike`。 它基于 `AnyTypeOf<[TT_Ptr, TT_PtrTensor]>;` 进一步特化。

### Lines 87-89
```tablegen
  87: // Tensor Type
  88: def TT_FpIntTensor : RankedTensorOf<[TT_Float, TT_Int]>;
  89: def TT_Tensor : RankedTensorOf<[TT_Float, TT_Int, TT_Ptr]>;
```
**EN:** This TableGen def record defines `TT_FpIntTensor`. It is specialized from `RankedTensorOf<[TT_Float, TT_Int]>;`.
**CN:** 该 TableGen def 记录定义了 `TT_FpIntTensor`。 它基于 `RankedTensorOf<[TT_Float, TT_Int]>;` 进一步特化。

### Lines 91-92
```tablegen
  91: // Any Type in Triton IR
  92: def TT_Type : AnyTypeOf<[TT_FloatLike, TT_IntLike, TT_PtrLike]>;
```
**EN:** This TableGen def record defines `TT_Type`. It is specialized from `AnyTypeOf<[TT_FloatLike, TT_IntLike, TT_PtrLike]>;`.
**CN:** 该 TableGen def 记录定义了 `TT_Type`。 它基于 `AnyTypeOf<[TT_FloatLike, TT_IntLike, TT_PtrLike]>;` 进一步特化。

### Lines 94-99
```tablegen
  94: // Type constraint for any type implementing TensorDescInterface
  95: def TT_AnyTensorDescType : Type<
  96:   CPred<"::mlir::isa<::mlir::triton::TensorDescInterface>($_self)">,
  97:   "tensor descriptor type",
  98:   "::mlir::triton::TensorDescInterface"
  99: >;
```
**EN:** This TableGen def record defines `TT_AnyTensorDescType`. It is specialized from `Type<`.
**CN:** 该 TableGen def 记录定义了 `TT_AnyTensorDescType`。 它基于 `Type<` 进一步特化。

### Lines 101-103
```tablegen
 101: // Result type of MakeTensorDescriptor
 102: def TT_TensorDescType : TritonTypeDef<"TensorDesc", "tensordesc", [TT_TensorDescInterface]> {
 103:   let summary = "Tensor descriptor type (`::mlir::triton::TensorDescType`) in Triton IR type system";
```
**EN:** This TableGen def record defines `TT_TensorDescType` with the summary “Tensor descriptor type (`::mlir::triton::TensorDescType`) in Triton IR type system”. It is specialized from `TritonTypeDef<"TensorDesc", "tensordesc", [TT_TensorDescInterface]>`.
**CN:** 该 TableGen def 记录定义了 `TT_TensorDescType`，其摘要为“Tensor descriptor type (`::mlir::triton::TensorDescType`) in Triton IR type system”。 它基于 `TritonTypeDef<"TensorDesc", "tensordesc", [TT_TensorDescInterface]>` 进一步特化。

### Lines 105-107
```tablegen
 105:   let description = [{
 106:       A portable abstraction for TMA descriptors.
 107:       This is the base tensor descriptor type for tiled tensor memory access.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 109-111
```tablegen
 109:       Shape and elementType describe the block dimensions and data type.
 110:       The optional sharedLayout attribute carries the shared memory encoding
 111:       (e.g. swizzle pattern) that is assigned during lowering.
```
**EN:** This block declares or defines callable APIs such as encoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 encoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 113-115
```tablegen
 113:       For specialized access patterns like im2col, see TensorDescIm2ColType
 114:       in the TritonNvidiaGPU dialect.
 115:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 117-121
```tablegen
 117:   let parameters = (ins
 118:     ArrayRefParameter<"int64_t">:$shape,
 119:     "Type":$elementType,
 120:     OptionalParameter<"Attribute">:$sharedLayout
 121:   );
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 123-157
```tablegen
 123:   let builders = [
 124:     // Builder from shape + elementType + sharedLayout
 125:     TypeBuilderWithInferredContext<(ins
 126:       "llvm::ArrayRef<int64_t>":$shape,
 127:       "Type":$elementType,
 128:       "Attribute":$sharedLayout
 129:     ), [{
 130:       return $_get(elementType.getContext(), shape, elementType, sharedLayout);
 131:     }]>,
 132:     // Builder with signedness
 133:     TypeBuilderWithInferredContext<(ins
 134:       "llvm::ArrayRef<int64_t>":$shape,
 135:       "Type":$elementType,
 136:       "bool":$isSigned
 137:     ), [{
 138:       if (auto intTy = llvm::dyn_cast<IntegerType>(elementType)) {
 139:         auto sem = isSigned ? IntegerType::Signed : IntegerType::Unsigned;
 140:         elementType = IntegerType::get(elementType.getContext(), intTy.getWidth(), sem);
 141:       }
 142:       return $_get(elementType.getContext(), shape, elementType, Attribute{});
 143:     }]>,
 144:     // Builder with signedness and shared layout
 145:     TypeBuilderWithInferredContext<(ins
 146:       "llvm::ArrayRef<int64_t>":$shape,
 147:       "Type":$elementType,
 148:       "Attribute":$sharedLayout,
 149:       "bool":$isSigned
 150:     ), [{
 151:       if (auto intTy = llvm::dyn_cast<IntegerType>(elementType)) {
 152:         auto sem = isSigned ? IntegerType::Signed : IntegerType::Unsigned;
 153:         elementType = IntegerType::get(elementType.getContext(), intTy.getWidth(), sem);
 154:       }
 155:       return $_get(elementType.getContext(), shape, elementType, sharedLayout);
 156:     }]>,
 157:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 159-162
```tablegen
 159:   let hasCustomAssemblyFormat = 1;
 160:   let skipDefaultBuilders = 1;
 161:   let genVerifyDecl = 1;
 162: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 164-164
```tablegen
 164: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** interfaces  
  **CN:** 接口
- **EN:** attributes  
  **CN:** 属性
- **EN:** types  
  **CN:** 类型
- **EN:** traits  
  **CN:** 特征约束

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/IR/AttrTypeBase.td`
  - `mlir/IR/BuiltinTypeInterfaces.td`
  - `triton/Dialect/Triton/IR/TritonDialect.td`
  - `triton/Dialect/Triton/IR/TritonTypeInterfaces.td`
