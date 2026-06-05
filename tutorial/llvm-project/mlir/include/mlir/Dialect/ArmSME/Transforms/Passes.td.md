# Passes.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/ArmSME/Transforms/Passes.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the ArmSME dialect, focused on rewrite patterns, passes, and canonicalization helpers and `Passes`.
  - **CN**: 为 ArmSME 方言定义聚焦 `Passes` 的声明式 TableGen 规格，覆盖重写模式、Pass 与规范化辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- Passes.td - ArmSME pass definition file ------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```tablegen

#ifndef MLIR_DIALECT_ARMSME_TRANSFORMS_PASSES_TD
#define MLIR_DIALECT_ARMSME_TRANSFORMS_PASSES_TD

include "mlir/Pass/PassBase.td"
include "mlir/IR/EnumAttr.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 15-26
```tablegen
def ArmStreamingMode : I32EnumAttr<"ArmStreamingMode", "Armv9 Streaming SVE mode",
    [
      I32EnumAttrCase<"Disabled", 0, "disabled">,
      // Streaming: Streaming-mode is part of the function interface (ABI).
      I32EnumAttrCase<"Streaming", 1, "arm_streaming">,
      // StreamingLocally: PSTATE.SM is kept internal and the callee manages it
      // on entry/exit.
      I32EnumAttrCase<"StreamingLocally", 2, "arm_locally_streaming">,
      // StreamingCompatible: the function may be entered in either
      // non-streaming mode (PSTATE.SM=0) or in streaming mode (PSTATE.SM=1)
      I32EnumAttrCase<"StreamingCompatible", 3, "arm_streaming_compatible">,
    ]>{
```
- **EN**: Introduces declarations for `ArmStreamingMode`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmStreamingMode` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 27-38
```tablegen
  let cppNamespace = "mlir::arm_sme";
  let genSpecializedAttr = 0;
}

// https://arm-software.github.io/acle/main/acle.html#sme-attributes-relating-to-za
// See also the LLVM definitions: https://llvm.org/docs/AArch64SME.html
//
// Various frontends (e.g. Flang) that build on top of this may restrict or
// enforce how these attributes are used, both individually and in terms of
// combinations that are allowed.
//
// The MLIR interface here does not make any attempt to perform any checking,
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 39-50
```tablegen
// it is up to the higher level to ensure that these attributes are used in a
// way that both makes sense and is legal according to the Arm architecture.
def ArmZaMode : I32EnumAttr<"ArmZaMode", "Armv9 ZA storage mode",
    [
      I32EnumAttrCase<"Disabled", 0, "disabled">,
      // A function's ZA state is created on entry and destroyed on exit.
      I32EnumAttrCase<"NewZA", 1, "arm_new_za">,
      // A function with a Shared-ZA interfaces that takes ZA as input.
      I32EnumAttrCase<"InZA", 2, "arm_in_za">,
      // A function with a Shared-ZA interfaces that returns ZA as output.
      I32EnumAttrCase<"OutZA", 3, "arm_out_za">,
      // A function with a Shared-ZA interfaces that takes ZA as input and
```
- **EN**: Introduces declarations for `ArmZaMode`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmZaMode` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 51-60
```tablegen
      // returns ZA as output.
      I32EnumAttrCase<"InOutZA", 4, "arm_inout_za">,
      // A function with a Shared-ZA interface that does not read ZA and
      // returns with ZA unchanged.
      I32EnumAttrCase<"PreservesZA", 5, "arm_preserves_za">,
    ]>{
  let cppNamespace = "mlir::arm_sme";
  let genSpecializedAttr = 0;
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 61-67
```tablegen
def EnableArmStreaming
    : Pass<"enable-arm-streaming", "mlir::func::FuncOp"> {
  let summary = "Enable Armv9 Streaming SVE mode";
  let description = [{
    Enables the Armv9 Streaming SVE mode [1] for func.func ops by annotating
    them with attributes. See options for more details.

```
- **EN**: Introduces declarations for `EnableArmStreaming`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `EnableArmStreaming` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 68-79
```tablegen
    [1] https://developer.arm.com/documentation/ddi0616/aa
  }];
  let constructor = "mlir::arm_sme::createEnableArmStreamingPass()";
  let options = [
    Option<"streamingMode", "streaming-mode", "mlir::arm_sme::ArmStreamingMode",
          /*default=*/"mlir::arm_sme::ArmStreamingMode::Streaming",
          "Select how streaming-mode is managed at the function-level.",
          [{::llvm::cl::values(
                clEnumValN(mlir::arm_sme::ArmStreamingMode::Disabled,
                           "disabled", "Streaming mode is disabled."),
                clEnumValN(mlir::arm_sme::ArmStreamingMode::Streaming,
                           "streaming",
```
- **EN**: Implements logic around `createEnableArmStreamingPass`, `values`, `clEnumValN`; this block packages logic as an MLIR pass or pass-related API.
- **CN**: 围绕 `createEnableArmStreamingPass`, `values`, `clEnumValN` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API。

### Lines 80-91
```tablegen
                           "Streaming mode is part of the function interface "
                           "(ABI), caller manages PSTATE.SM on entry/exit."),
                clEnumValN(mlir::arm_sme::ArmStreamingMode::StreamingLocally,
                           "streaming-locally",
                           "Streaming mode is internal to the function, callee "
                           "manages PSTATE.SM on entry/exit."),
                clEnumValN(mlir::arm_sme::ArmStreamingMode::StreamingCompatible,
                           "streaming-compatible",
                           "Function supports both streaming and non-streaming "
                           "modes.")
          )}]>,
    Option<"zaMode", "za-mode", "mlir::arm_sme::ArmZaMode",
```
- **EN**: Implements logic around `clEnumValN`; this block expresses reusable interface-based behavior; uses declarative TableGen records to describe reusable IR contracts.
- **CN**: 围绕 `clEnumValN` 实现具体逻辑；该代码块表达基于接口的可复用行为，并使用声明式 TableGen 记录描述可复用 IR 契约。

### Lines 92-103
```tablegen
           /*default=*/"mlir::arm_sme::ArmZaMode::Disabled",
           "Select how ZA-storage is managed at the function-level.",
           [{::llvm::cl::values(
                 clEnumValN(mlir::arm_sme::ArmZaMode::Disabled,
                            "disabled", "ZA storage is disabled."),
                 clEnumValN(mlir::arm_sme::ArmZaMode::NewZA,
                            "new-za",
                            "The function has ZA state. The ZA state is "
                            "created on entry and destroyed on exit."),
                 clEnumValN(mlir::arm_sme::ArmZaMode::InZA,
                            "in-za",
                            "The function uses ZA state. The ZA state may "
```
- **EN**: Implements logic around `values`, `clEnumValN`.
- **CN**: 围绕 `values`, `clEnumValN` 实现具体逻辑。

### Lines 104-115
```tablegen
                            "be used for input."),
                 clEnumValN(mlir::arm_sme::ArmZaMode::OutZA,
                            "out-za",
                            "The function uses ZA state. The ZA state may "
                            "be used for output."),
                 clEnumValN(mlir::arm_sme::ArmZaMode::InOutZA,
                            "inout-za",
                            "The function uses ZA state. The ZA state may "
                            "be used for input and/or output."),
                 clEnumValN(mlir::arm_sme::ArmZaMode::PreservesZA,
                            "preserves-za",
                            "The function shares ZA state. The ZA state may "
```
- **EN**: Implements logic around `clEnumValN`.
- **CN**: 围绕 `clEnumValN` 实现具体逻辑。

### Lines 116-127
```tablegen
                            "not be used for input and/or output and the "
                            "function must return with ZA unchanged")
           )}]>,
    Option<"ifRequiredByOps", "if-required-by-ops", "bool",
           /*default=*/"false",
           "Only apply the selected streaming/ZA modes if the function contains"
           " ops that implement the ArmSMETileOpInterface.">,
    Option<"ifScalableAndSupported", "if-scalable-and-supported",
           "bool", /*default=*/"false",
           "Only apply the selected streaming/ZA modes if the function contains"
           " supported scalable vector operations.">
  ];
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 128-139
```tablegen
  let dependentDialects = ["func::FuncDialect"];
}

def TestTileAllocation
    : Pass<"test-arm-sme-tile-allocation", "mlir::func::FuncOp"> {
  let summary = "Tests SME 'virtual tile' allocation";
  let description = [{
    This pass does tile allocation for SME "virtual tiles". It is run at the
    'func.func' op level, and assigns tile IDs (via an attribute) to all ops
    that implement the `ArmSMETileOpInterface`. Note: This pass is only intended
    to be used for testing, tile allocation is done as part of the ArmSME to
    LLVM conversion (`convert-arm-sme-to-llvm`).
```
- **EN**: Introduces declarations for `TestTileAllocation`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `TestTileAllocation` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 140-151
```tablegen
  }];
  let options = [
    Option<"dumpTileLiveRanges", "dump-tile-live-ranges",
           "bool", /*default=*/"false",
           "Dump the live ranges of SME tiles (for debugging)">,
    Option<"preprocessOnly", "preprocess-only", "bool", /*default=*/"false",
           "Only preprocess IR so it is ready for tile allocation "
           "(but do not allocate any tiles)">
  ];
  let dependentDialects = ["func::FuncDialect", "arm_sme::ArmSMEDialect"];
}

```
- **EN**: Declares APIs or declarative rules around `tiles`.
- **CN**: 声明与 `tiles` 相关的 API 或声明式规则。

### Lines 152-158
```tablegen
def OuterProductFusion
    : Pass<"arm-sme-outer-product-fusion", "mlir::func::FuncOp"> {
  let summary = "Fuse 'arm_sme.outerproduct' operations into 2-way or 4-way widening variants";
  let description = [{
    This pass fuses 'arm_sme.outerproduct' operations that are chained via the
    accumulator into 2-way or 4-way ArmSME outer product operations.

```
- **EN**: Introduces declarations for `OuterProductFusion`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `OuterProductFusion` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 159-165
```tablegen
    For example:
    ```mlir
    %a0_ext = arith.extf %a0 : vector<[4]xf16> to vector<[4]xf32>
    %b0_ext = arith.extf %b0 : vector<[4]xf16> to vector<[4]xf32>
    %a1_ext = arith.extf %a1 : vector<[4]xf16> to vector<[4]xf32>
    %b1_ext = arith.extf %b1 : vector<[4]xf16> to vector<[4]xf32>

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 166-171
```tablegen
    %0 = arm_sme.outerproduct %a0_ext, %b0_ext : vector<[4]xf32>, vector<[4]xf32>
    %1 = arm_sme.outerproduct %a1_ext, %b1_ext acc(%0) : vector<[4]xf32>, vector<[4]xf32>
    ```

    Becomes:

```
- **EN**: Implements logic around `acc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `acc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 172-177
```tablegen
    ```mlir
    %a_packed = vector.interleave %a0, %a1 : vector<[4]xf16> -> vector<[8]xf16>
    %b_packed = vector.interleave %b0, %b1 : vector<[4]xf16> -> vector<[8]xf16>
    %0 = arm_sme.fmopa_2way %a_packed, %b_packed : vector<[8]xf16>, vector<[8]xf16> into vector<[4]x[4]xf32>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 178-185
```tablegen
    For further information on the 2-way or 4-way widening ops see:
    https://mlir.llvm.org/docs/Dialects/ArmSME/#arm_smefmopa_2way-arm_smefmopa_2wayop
    https://mlir.llvm.org/docs/Dialects/ArmSME/#arm_smesmopa_4way-arm_smesmopa_4wayop
  }];
  let constructor = "mlir::arm_sme::createOuterProductFusionPass()";
  let dependentDialects = ["func::FuncDialect", "arm_sme::ArmSMEDialect"];
}

```
- **EN**: Declares APIs or declarative rules around `createOuterProductFusionPass`; this block packages logic as an MLIR pass or pass-related API.
- **CN**: 声明与 `createOuterProductFusionPass` 相关的 API 或声明式规则；该代码块将逻辑组织为 MLIR Pass 或相关 API。

### Lines 186-195
```tablegen
def VectorLegalization
  : Pass<"arm-sme-vector-legalization", "mlir::ModuleOp"> {
  let summary = "Legalize vectors for ArmSME";
  let description = [{
    This pass legalizes vector operations so that they can be lowered to ArmSME.
    This includes decomposing operations that operate on vector types larger
    than a single SME tile (e.g. `vector<[8]x[8]xf32>`) into multiple SME
    tile-sized operations, as well as rewrites needed to get operations into
    forms compatible with SME lowerings.

```
- **EN**: Introduces declarations for `VectorLegalization`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `VectorLegalization` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 196-207
```tablegen
    Note: Decomposition is currently limited to vector types that are an exact
    multiple of SME tiles. That is scalable in two dimensions, with both the
    rows and columns divisible by the SVE vector length for the element type.
  }];
  let constructor = "mlir::arm_sme::createVectorLegalizationPass()";
  let dependentDialects = [
    "func::FuncDialect",
    "arm_sme::ArmSMEDialect",
    "vector::VectorDialect",
    "arith::ArithDialect",
    "index::IndexDialect"
  ];
```
- **EN**: Declares APIs or declarative rules around `createVectorLegalizationPass`; this block packages logic as an MLIR pass or pass-related API; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `createVectorLegalizationPass` 相关的 API 或声明式规则；该代码块将逻辑组织为 MLIR Pass 或相关 API，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 208-210
```tablegen
}

#endif // MLIR_DIALECT_ARMSME_TRANSFORMS_PASSES_TD
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Rewrite and lowering support / 重写与 lowering 支持**:
  - **EN**: Collects patterns or pass declarations used to canonicalize, legalize, or lower IR.
  - **CN**: 汇集用于规范化、合法化或 lowering IR 的模式或 Pass 声明。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
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

- **TableGen includes / TableGen 包含**: `mlir/Pass/PassBase.td`, `mlir/IR/EnumAttr.td`
- **Subsystem categories / 子系统类别**: pass infrastructure and registration support / Pass 基础设施与注册支持 (1), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
