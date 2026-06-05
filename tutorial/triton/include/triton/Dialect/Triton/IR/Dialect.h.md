# Dialect.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/Triton/IR/Dialect.h`
- **EN:** Declares the MLIR dialect interface and imports generated declarations for this subsystem.
- **CN:** 声明该子系统的 MLIR 方言接口，并引入生成的声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_DIALECT_TRITON_IR_DIALECT_H_
   2: #define TRITON_DIALECT_TRITON_IR_DIALECT_H_
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-19
```cpp
   4: #include "mlir/Dialect/Arith/IR/Arith.h"
   5: #include "mlir/Dialect/ControlFlow/IR/ControlFlow.h"
   6: #include "mlir/Dialect/Func/IR/FuncOps.h"
   7: #include "mlir/Dialect/Math/IR/Math.h"
   8: #include "mlir/Dialect/SCF/IR/SCF.h"
   9: #include "mlir/Dialect/Tensor/IR/Tensor.h"
  10: #include "mlir/IR/BuiltinOps.h"
  11: #include "mlir/IR/Dialect.h"
  12: #include "mlir/Interfaces/ControlFlowInterfaces.h"
  13: #include "mlir/Interfaces/FunctionInterfaces.h"
  14: #include "mlir/Interfaces/SideEffectInterfaces.h"
  15: #include "triton/Dialect/Triton/IR/Dialect.h.inc"
  16: #include "triton/Dialect/Triton/IR/OpInterfaces.h"
  17: #include "triton/Dialect/Triton/IR/OpsEnums.h.inc"
  18: #include "triton/Dialect/Triton/IR/Traits.h"
  19: #include "triton/Dialect/Triton/IR/Types.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Dialect/Arith/IR/Arith.h, mlir/Dialect/ControlFlow/IR/ControlFlow.h, mlir/Dialect/Func/IR/FuncOps.h, mlir/Dialect/Math/IR/Math.h, mlir/Dialect/SCF/IR/SCF.h, and mlir/Dialect/Tensor/IR/Tensor.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Dialect/Arith/IR/Arith.h, mlir/Dialect/ControlFlow/IR/ControlFlow.h, mlir/Dialect/Func/IR/FuncOps.h, mlir/Dialect/Math/IR/Math.h, mlir/Dialect/SCF/IR/SCF.h, and mlir/Dialect/Tensor/IR/Tensor.h。

### Lines 21-22
```cpp
  21: #define GET_OP_CLASSES
  22: #include "triton/Dialect/Triton/IR/Ops.h.inc"
```
**EN:** This block enables MLIR/TableGen-generated declarations and then includes the generated `.inc` fragments that materialize them.
**CN:** 该代码块先打开 MLIR/TableGen 生成代码所需的宏开关，再引入真正展开声明的 `.inc` 片段。

### Lines 24-25
```cpp
  24: namespace mlir {
  25: namespace triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir and triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir and triton 下。

### Lines 27-30
```cpp
  27: struct GlobalMemory : public SideEffects::Resource::Base<GlobalMemory> {
  28:   StringRef getName() const final { return "<GlobalMemory>"; }
  29:   SideEffects::Resource *getParent() const override { return nullptr; }
  30: };
```
**EN:** This block introduces `GlobalMemory`, the main class/struct defined here. Within the declaration, methods such as getName and getParent expose its core API. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `GlobalMemory`。 其中 getName and getParent 等方法构成了它的主要接口。 它还通过继承复用基类能力。

### Lines 32-35
```cpp
  32: class DialectInferLayoutInterface
  33:     : public DialectInterface::Base<DialectInferLayoutInterface> {
  34: public:
  35:   DialectInferLayoutInterface(Dialect *dialect) : Base(dialect) {}
```
**EN:** This block introduces `DialectInferLayoutInterface`, the main class/struct defined here. Within the declaration, methods such as Base expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `DialectInferLayoutInterface`。 其中 Base 等方法构成了它的主要接口。

### Lines 37-40
```cpp
  37:   virtual LogicalResult
  38:   inferTransOpEncoding(Attribute operandEncoding, ArrayRef<int64_t> shape,
  39:                        ArrayRef<int32_t> order, Attribute &resultEncoding,
  40:                        std::optional<Location> loc) const = 0;
```
**EN:** This block declares or defines callable APIs such as inferTransOpEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferTransOpEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 42-45
```cpp
  42:   virtual LogicalResult
  43:   inferReduceOpEncoding(Attribute operandEncoding, unsigned axis,
  44:                         Attribute &resultEncoding,
  45:                         std::optional<Location> loc) const = 0;
```
**EN:** This block declares or defines callable APIs such as inferReduceOpEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferReduceOpEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 47-50
```cpp
  47:   virtual LogicalResult
  48:   inferExpandDimsOpEncoding(Attribute operandEncoding, unsigned axis,
  49:                             Attribute &resultEncoding,
  50:                             std::optional<Location> loc) const = 0;
```
**EN:** This block declares or defines callable APIs such as inferExpandDimsOpEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferExpandDimsOpEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 52-57
```cpp
  52:   // Note: This function only verifies the operand encoding.  It doesn't infer
  53:   // the result encoding.
  54:   virtual LogicalResult
  55:   inferDotOpEncoding(Attribute operandEncoding, unsigned opIdx,
  56:                      Attribute retEncoding,
  57:                      std::optional<Location> loc) const = 0;
```
**EN:** This block declares or defines callable APIs such as inferDotOpEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferDotOpEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 59-70
```cpp
  59:   // Tries to compute the encoding for the result of a reshape operation that
  60:   // makes the reshape a "nop", i.e. the same GPU threads contain the same
  61:   // elements as before the reshape using legacy layouts.  This is not always
  62:   // possible (in which case we fallback to using LinearLayouts)
  63:   // If allowReorder is set, an existing value in dstEnc is preferred when it
  64:   // still yields a non-expensive view.
  65:   // In the future we'll always use LinearLayouts
  66:   virtual LogicalResult
  67:   inferReshapeOpEncoding(ArrayRef<int64_t> srcShape, Attribute srcEnc,
  68:                          ArrayRef<int64_t> dstShape, Attribute &dstEnc,
  69:                          bool allowReorder,
  70:                          std::optional<Location> loc) const = 0;
```
**EN:** This block declares or defines callable APIs such as inferReshapeOpEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferReshapeOpEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 72-76
```cpp
  72:   // Check if two layouts are structurally the same, even if their names are
  73:   // different
  74:   virtual LogicalResult
  75:   verifyLayoutsAreEqual(ArrayRef<int64_t> shape, Attribute expected,
  76:                         Attribute got, std::optional<Location> loc) const = 0;
```
**EN:** This block declares or defines callable APIs such as verifyLayoutsAreEqual, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 verifyLayoutsAreEqual 等可调用 API，用来封装这里提供的核心行为。

### Lines 78-81
```cpp
  78:   virtual LogicalResult
  79:   inferDefaultJoinOpEncoding(Attribute srcEnc, Attribute &dstEnc,
  80:                              ArrayRef<int64_t> shape,
  81:                              std::optional<Location> loc) const = 0;
```
**EN:** This block declares or defines callable APIs such as inferDefaultJoinOpEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferDefaultJoinOpEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 83-86
```cpp
  83:   virtual LogicalResult
  84:   inferSplitOpEncoding(Attribute srcEnc, Attribute &dstEnc,
  85:                        ArrayRef<int64_t> shape,
  86:                        std::optional<Location> loc) const = 0;
```
**EN:** This block declares or defines callable APIs such as inferSplitOpEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferSplitOpEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 88-92
```cpp
  88:   // Verify that the encoding are compatible to be used together in a dot
  89:   // operation
  90:   virtual LogicalResult
  91:   verifyDotOpEncodingCompatibility(Operation *op, Attribute operandEncodingA,
  92:                                    Attribute operandEncodingB) const = 0;
```
**EN:** This block declares or defines callable APIs such as verifyDotOpEncodingCompatibility, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 verifyDotOpEncodingCompatibility 等可调用 API，用来封装这里提供的核心行为。

### Lines 94-97
```cpp
  94:   // Verify that the encodings are compatible to be used together in a cat
  95:   // operation.
  96:   virtual LogicalResult
  97:   verifyCatOpEncodingCompatibility(Operation *op) const = 0;
```
**EN:** This block declares or defines callable APIs such as verifyCatOpEncodingCompatibility, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 verifyCatOpEncodingCompatibility 等可调用 API，用来封装这里提供的核心行为。

### Lines 99-103
```cpp
  99:   virtual LogicalResult
 100:   inferFp4ToFpOpEncoding(ArrayRef<int64_t> shape, int axis, Attribute inEnc,
 101:                          Attribute &outEnc, bool fwdInference,
 102:                          std::optional<Location> loc) const = 0;
 103: };
```
**EN:** This block declares or defines callable APIs such as inferFp4ToFpOpEncoding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inferFp4ToFpOpEncoding 等可调用 API，用来封装这里提供的核心行为。

### Lines 105-108
```cpp
 105: class DialectVerifyTensorLayoutInterface
 106:     : public DialectInterface::Base<DialectVerifyTensorLayoutInterface> {
 107: public:
 108:   DialectVerifyTensorLayoutInterface(Dialect *dialect) : Base(dialect) {}
```
**EN:** This block introduces `DialectVerifyTensorLayoutInterface`, the main class/struct defined here. Within the declaration, methods such as Base expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `DialectVerifyTensorLayoutInterface`。 其中 Base 等方法构成了它的主要接口。

### Lines 110-112
```cpp
 110:   virtual LogicalResult
 111:   verifyTensorLayout(Attribute layout, RankedTensorType type, Operation *op,
 112:                      function_ref<InFlightDiagnostic()> emitError) const = 0;
```
**EN:** This block declares or defines callable APIs such as verifyTensorLayout and InFlightDiagnostic, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 verifyTensorLayout and InFlightDiagnostic 等可调用 API，用来封装这里提供的核心行为。

### Lines 114-117
```cpp
 114:   virtual LogicalResult
 115:   verifyMemDescLayout(Attribute layout, Type type, Operation *op,
 116:                       function_ref<InFlightDiagnostic()> emitError) const = 0;
 117: };
```
**EN:** This block declares or defines callable APIs such as verifyMemDescLayout and InFlightDiagnostic, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 verifyMemDescLayout and InFlightDiagnostic 等可调用 API，用来封装这里提供的核心行为。

### Lines 119-128
```cpp
 119: // Descriptor gather and scatter have restrictions on the tile sizes.
 120: LogicalResult verifyGatherScatterResultType(Operation *op,
 121:                                             ShapedType resultType,
 122:                                             ShapedType indicesType);
 123: LogicalResult verifyGatherScatterOp(Operation *op, ShapedType blockType,
 124:                                     ShapedType resultType,
 125:                                     ShapedType indicesType);
 126: LogicalResult verifyDescriptorLoadStoreOp(Operation *op,
 127:                                           TensorDescInterface desc,
 128:                                           ShapedType tensor);
```
**EN:** This block declares or defines callable APIs such as verifyGatherScatterResultType, verifyGatherScatterOp, and verifyDescriptorLoadStoreOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 verifyGatherScatterResultType, verifyGatherScatterOp, and verifyDescriptorLoadStoreOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 130-136
```cpp
 130: LogicalResult deduceScaleFactor(ArrayRef<int64_t> lhsShape,
 131:                                 std::optional<ArrayRef<int64_t>> lhsScaleShape,
 132:                                 ScaleDotElemType lhsFormat, bool lhsKPack,
 133:                                 ArrayRef<int64_t> rhsShape,
 134:                                 std::optional<ArrayRef<int64_t>> rhsScaleShape,
 135:                                 ScaleDotElemType rhsFormat, bool rhsKPack,
 136:                                 int32_t &scaleFactor, std::string &errMsg);
```
**EN:** This block declares or defines callable APIs such as deduceScaleFactor, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 deduceScaleFactor 等可调用 API，用来封装这里提供的核心行为。

### Lines 138-139
```cpp
 138: } // namespace triton
 139: } // namespace mlir
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 141-141
```cpp
 141: #endif // TRITON_IR_DIALECT_H_
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** axis reasoning  
  **CN:** 轴信息推理
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** dot-product lowering  
  **CN:** 点积降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** interfaces  
  **CN:** 接口
- **EN:** attributes  
  **CN:** 属性

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Dialect/Arith/IR/Arith.h`
  - `mlir/Dialect/ControlFlow/IR/ControlFlow.h`
  - `mlir/Dialect/Func/IR/FuncOps.h`
  - `mlir/Dialect/Math/IR/Math.h`
  - `mlir/Dialect/SCF/IR/SCF.h`
  - `mlir/Dialect/Tensor/IR/Tensor.h`
  - `mlir/IR/BuiltinOps.h`
  - `mlir/IR/Dialect.h`
  - `mlir/Interfaces/ControlFlowInterfaces.h`
  - `mlir/Interfaces/FunctionInterfaces.h`
  - `mlir/Interfaces/SideEffectInterfaces.h`
  - `triton/Dialect/Triton/IR/OpInterfaces.h`
  - `triton/Dialect/Triton/IR/Traits.h`
  - `triton/Dialect/Triton/IR/Types.h`
- **Generated includes / 生成代码依赖:**
  - `triton/Dialect/Triton/IR/Dialect.h.inc`
  - `triton/Dialect/Triton/IR/OpsEnums.h.inc`
  - `triton/Dialect/Triton/IR/Ops.h.inc`
