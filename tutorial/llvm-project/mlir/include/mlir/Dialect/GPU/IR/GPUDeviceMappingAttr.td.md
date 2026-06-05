# GPUDeviceMappingAttr.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/GPU/IR/GPUDeviceMappingAttr.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines the attribute used to map loops to gpu.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/GPU/IR`，围绕 GPU 方言公开 `GPUDeviceMappingAttr` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- GPUDeviceMappingAttr.td - Attribute definition -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-15
```tablegen
//
// Defines the attribute used to map loops to gpu.
//
//===----------------------------------------------------------------------===//

#ifndef GPU_DEVICE_MAPPING_ATTR
#define GPU_DEVICE_MAPPING_ATTR

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 16-21
```tablegen
include "mlir/Dialect/GPU/IR/GPUBase.td"
include "mlir/IR/EnumAttr.td"
include "mlir/Dialect/SCF/IR/DeviceMappingInterface.td"

def DimX : I64EnumAttrCase<"DimX", 0, "x">;
def DimY : I64EnumAttrCase<"DimY", 1, "y">;
```
- **EN**: Introduces declarations for `DimX`, `DimY`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DimX`, `DimY` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 22-27
```tablegen
def DimZ : I64EnumAttrCase<"DimZ", 2, "z">;
def LinearDim0 : I64EnumAttrCase<"LinearDim0", 3, "linear_dim_0">;
def LinearDim1 : I64EnumAttrCase<"LinearDim1", 4, "linear_dim_1">;
def LinearDim2 : I64EnumAttrCase<"LinearDim2", 5, "linear_dim_2">;
def LinearDim3 : I64EnumAttrCase<"LinearDim3", 6, "linear_dim_3">;
def LinearDim4 : I64EnumAttrCase<"LinearDim4", 7, "linear_dim_4">;
```
- **EN**: Introduces declarations for `DimZ`, `LinearDim0`, `LinearDim1`, `LinearDim2`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DimZ`, `LinearDim0`, `LinearDim1`, `LinearDim2`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 28-33
```tablegen
def LinearDim5 : I64EnumAttrCase<"LinearDim5", 8, "linear_dim_5">;
def LinearDim6 : I64EnumAttrCase<"LinearDim6", 9, "linear_dim_6">;
def LinearDim7 : I64EnumAttrCase<"LinearDim7", 10, "linear_dim_7">;
def LinearDim8 : I64EnumAttrCase<"LinearDim8", 11, "linear_dim_8">;
def LinearDim9 : I64EnumAttrCase<"LinearDim9", 12, "linear_dim_9">;

```
- **EN**: Introduces declarations for `LinearDim5`, `LinearDim6`, `LinearDim7`, `LinearDim8`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LinearDim5`, `LinearDim6`, `LinearDim7`, `LinearDim8`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 34-43
```tablegen
// TODO: This would be better represented with separate Grid and Linear Mapping
// ids. Unfortunately it is not yet possible to have an optional EnumParameter
// so we currently embed the 2 modes in the same enum.
def MappingIdEnum : I64EnumAttr<"MappingId", "Mapping ids for loop mapping", [
    DimX, DimY, DimZ,
    LinearDim0, LinearDim1, LinearDim2, LinearDim3, LinearDim4, 
    LinearDim5, LinearDim6, LinearDim7, LinearDim8, LinearDim9]> {
  let cppNamespace = "::mlir::gpu";
}

```
- **EN**: Introduces declarations for `MappingIdEnum`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MappingIdEnum` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 44-52
```tablegen
def GPUBlockMappingAttr : GPU_Attr<"GPUBlockMapping", "block", [
  DeclareAttrInterfaceMethods<DeviceMappingAttrInterface> ] >  {
  let parameters = (ins
    EnumParameter<MappingIdEnum>:$block
  );
  let assemblyFormat = "`<` params `>`";
  let description = [{
    An attribute that allows defining thread block parallelism for GPU devices.

```
- **EN**: Introduces declarations for `GPUBlockMappingAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUBlockMappingAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 53-58
```tablegen
    Thread blocks (aka workgroup) are grouped into a grid described by a 
    3-dimensional rectangle.
    This attribute indicates that thread block parallelism is desired.
    It can be consumed by lowering to generate GPU code.
    2 modes are supported: (1) 3D mapping mode and (2) linear mapping mode.

```
- **EN**: Implements logic around `blocks`, `supported:`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `blocks`, `supported:` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 59-64
```tablegen
    #### 3D mapping mode

    The 3D block id is simply the 3D index of the block `(bidx, bidy, bidz)`. 
    If required, predication occurs on a per-dimension basis. This allows 
    specifying predication on a 3D sub-rectangle of the grid.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 65-70
```tablegen
    #### Linear mapping mode

    The linear block id is obtained by linearizing the index of the block. 
    If required, predication occurs on the linear id. This allows specifying
    predication on a 1D subset of the (linearized) grid.

```
- **EN**: Implements logic around `the`.
- **CN**: 围绕 `the` 实现具体逻辑。

### Lines 71-78
```tablegen
    For instance, if the basis is denoted as (GX, GY, GZ) and the block id is
    denoted by (bx, by, bz), the block id is:
      `linear_id = bx + by * GX + bz * GX * GBY)`.
    The linear block id is fixed for the duration of a GPU kernel.
    
    This linear id mapping attribute indicates a different linearization relation
    is applied locally to a loop nest. 
    
```
- **EN**: Implements logic around `as`, `by`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `as`, `by` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 79-88
```tablegen
    For instance, if the new basis is denoted as (LBD0, LBD1, LBD2, LBD3) the 
    block id in the new basis is:
      ```(linear_id mod LBD0 , 
          (linear_id / LBD0) mod * LBD1, 
          (linear_id / (LBD0 * LBD1)) mod LBD2, 
          (linear_id / (LBD0 * LBD1 * LBD2)) mod LBD3)```.
    This reinterpretation is only fixed for the duration of a loop nest.
  }];
}

```
- **EN**: Declares APIs or declarative rules around `as`.
- **CN**: 声明与 `as` 相关的 API 或声明式规则。

### Lines 89-98
```tablegen
def GPUWarpgroupMappingAttr
    : GPU_Attr<"GPUWarpgroupMapping", "warpgroup", [
      DeclareAttrInterfaceMethods<DeviceMappingAttrInterface> ]> {
  let parameters = (ins
    EnumParameter<MappingIdEnum>:$warpgroup
  );
  let assemblyFormat = "`<` params `>`";
  let description = [{
    An attribute that allows defining warpgroup parallelism for GPU devices.

```
- **EN**: Introduces declarations for `GPUWarpgroupMappingAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUWarpgroupMappingAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 99-105
```tablegen
    Threads of proper granularity (e.g. multiple of 
    "kNumWarpsPerGroup * kWarpSize" on CUDA devices) can be grouped into
    warpgroups described by a 3-dimensional rectangle. 
    This attribute indicates that warpgroup parallelism is desired. 
    It can be consumed by lowering to generate GPU code.
    2 modes are supported: (1) 3D mapping mode and (2) linear mapping mode.

```
- **EN**: Implements logic around `granularity`, `supported:`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `granularity`, `supported:` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 106-112
```tablegen
    #### 3D mapping mode

    The 3D warpgroup id is simply the adjusted 3D index of the thread 
    `(tidx / (kNumWarpsPerGroup * kWarpSize), tidy, tidz)`.
    If required, predication occurs on a per-dimension basis. This allows 
    specifying predication on a 3D sub-rectangle of the warpgroups.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 113-119
```tablegen
    #### Linear mapping mode

    The linear warpgroup id is obtained by linearizing the index of the warpgroup.
    If required, predication occurs on the linear id. This allows specifying
    predication on a 1D "kNumWarpsPerGroup * kWarpSize"-aligned subset of the 
    (linearized) block.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 120-125
```tablegen
    For instance, if the basis is denoted as (BX, BY, BZ) and the thread id is
    id is denoted by (tx, ty, tz), the linear warpgroup id is:
      ```linear_id = (tx + ty * BX + tz * BX * BY) 
                 / (kNumWarpsPerGroup * kWarpSize)```.
    The linear warpgroup id is fixed for the duration of a GPU kernel.
    
```
- **EN**: Implements logic around `as`, `by`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `as`, `by` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 126-137
```tablegen
    This linear id mapping attribute indicates a different linearization relation
    is applied locally to a loop nest. 
    
    For instance, if the new basis is denoted as (LWGD0, LWGD1, LWGD2, LWGD3) the 
    warpgroup id in the new basis is:
      ```(linear_id mod LWGD0 , 
          (linear_id / LWGD0) mod * LWGD1, 
          (linear_id / (LWGD0 * LWGD1)) mod LWGD2, 
          (linear_id / (LWGD0 * LWGD1 * LWGD2)) mod LWGD3)```.
    This reinterpretation is only fixed for the duration of a loop nest.
  }];
}
```
- **EN**: Declares APIs or declarative rules around `as`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `as` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 138-148
```tablegen

def GPUWarpMappingAttr
    : GPU_Attr<"GPUWarpMapping", "warp", [
      DeclareAttrInterfaceMethods<DeviceMappingAttrInterface> ]> {
  let parameters = (ins
    EnumParameter<MappingIdEnum>:$warp
  );
  let assemblyFormat = "`<` params `>`";
  let description = [{
    An attribute that allows defining warp parallelism for GPU devices.

```
- **EN**: Introduces declarations for `GPUWarpMappingAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUWarpMappingAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 149-154
```tablegen
    Threads of proper granularity (e.g. multiple of "warp size" on CUDA devices) 
    can be grouped into warps described by a 3-dimensional rectangle. 
    This attribute indicates that warp parallelism is desired.
    It can be consumed by lowering to generate GPU code.
    2 modes are supported: (1) 3D mapping mode and (2) linear mapping mode.

```
- **EN**: Implements logic around `granularity`, `supported:`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `granularity`, `supported:` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 155-161
```tablegen
    #### 3D mapping mode

    The 3D warp id is simply the adjusted 3D index of the thread 
    `(tidx / kWarpSize, tidy, tidz)`.
    If required, predication occurs on a per-dimension basis. This allows 
    specifying predication on a 3D sub-rectangle of the warpgroups.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 162-167
```tablegen
    #### Linear mapping mode

    The linear warp id is obtained by linearizing the index of the warp.
    If required, predication occurs on the linear id. This allows specifying
    predication on a 1D "kWarpSize"-aligned subset of the (linearized) block.

```
- **EN**: Implements logic around `the`.
- **CN**: 围绕 `the` 实现具体逻辑。

### Lines 168-175
```tablegen
    For instance, if the basis is denoted as (BX, BY, BZ) and the thread id is
    id is denoted by (tx, ty, tz), the linear warp id is:
      `linear_id = (tx + ty * BX + tz * BX * BY) / kWarpSize`.
    The linear warp id is fixed for the duration of a GPU kernel.
    
    This linear id mapping attribute indicates a different linearization relation
    is applied locally to a loop nest. 
    
```
- **EN**: Implements logic around `as`, `by`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `as`, `by` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 176-185
```tablegen
    For instance, if the new basis is denoted as (LWD0, LWD1, LWD2, LWD3) the 
    warp id in the new basis is:
      ```(linear_id mod LWD0 , 
          (linear_id / LWD0) mod * LWD1, 
          (linear_id / (LWD0 * LWD1)) mod LWD2, 
          (linear_id / (LWD0 * LWD1 * LWD2)) mod LWD3)```.
    This reinterpretation is only fixed for the duration of a loop nest.
  }];
}

```
- **EN**: Declares APIs or declarative rules around `as`.
- **CN**: 声明与 `as` 相关的 API 或声明式规则。

### Lines 186-195
```tablegen
def GPUThreadMappingAttr
    : GPU_Attr<"GPUThreadMapping", "thread", [
      DeclareAttrInterfaceMethods<DeviceMappingAttrInterface> ]> {
  let parameters = (ins
    EnumParameter<MappingIdEnum>:$thread
  );
  let assemblyFormat = "`<` params `>`";
  let description = [{
    An attribute that allows defining thread parallelism for GPU devices.

```
- **EN**: Introduces declarations for `GPUThreadMappingAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUThreadMappingAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 196-202
```tablegen
    Thread (aka work item) are grouped into a thread blocks described by a 
    3-dimensional rectangle.
    This attribute indicates that thread parallelism is desired.
    It can be consumed by lowering to generate GPU.

    #### 3D mapping mode

```
- **EN**: Implements logic around `Thread`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `Thread` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 203-208
```tablegen
    The 3D thread id is simply the 3D index of the thread `(tidx, tidy, tidz)`. 
    If required, predication occurs on a per-dimension basis. This allows 
    specifying predication on a 3D sub-rectangle of the block.

    #### Linear mapping mode

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 209-217
```tablegen
    The linear thread id is obtained by linearizing the index of the thread. 
    If required, predication occurs on the linear id. This allows specifying
    predication on a 1D subset of the (linearized) block.

    For instance, if the basis is denoted as (BX, BY, BZ) and the thread id is
    id is denoted by (tx, ty, tz), the linear thread id is:
      ```linear_id = (tx + ty * BX + tz * BX * BY)```.
    The linear thread id is fixed for the duration of a GPU kernel.
    
```
- **EN**: Implements logic around `the`, `as`, `by`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `the`, `as`, `by` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 218-229
```tablegen
    This linear id mapping attribute indicates a different linearization relation
    is applied locally to a loop nest. 
    
    For instance, if the new basis is denoted as (LTD0, LTD1, LTD2, LTD3) the 
    thread id in the new basis is:
      ```(linear_id mod LTD0 , 
          (linear_id / LTD0) mod * LTD1, 
          (linear_id / (LTD0 * LTD1)) mod LTD2, 
          (linear_id / (LTD0 * LTD1 * LTD2)) mod LTD3)```.
    This reinterpretation is only fixed for the duration of a loop nest.
  }];
}
```
- **EN**: Declares APIs or declarative rules around `as`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `as` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 230-240
```tablegen

def GPULaneMappingAttr
    : GPU_Attr<"GPULaneMapping", "lane", [
      DeclareAttrInterfaceMethods<DeviceMappingAttrInterface> ]> {
  let parameters = (ins
    EnumParameter<MappingIdEnum>:$lane
  );
  let assemblyFormat = "`<` params `>`";
  let description = [{
    An attribute that allows defining lane parallelism for GPU devices.

```
- **EN**: Introduces declarations for `GPULaneMappingAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPULaneMappingAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 241-246
```tablegen
    It can be consumed by lowering to generate GPU.

    #### 3D mapping mode

    Unsupported

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 247-254
```tablegen
    #### Linear mapping mode

    The linear lane id is obtained by linearizing the index of the lane.
    If required, predication occurs on the linear id. This allows specifying
    predication on a 1D subset of the (linearized) lanes.
  }];
}

```
- **EN**: Declares APIs or declarative rules around `the`.
- **CN**: 声明与 `the` 相关的 API 或声明式规则。

### Lines 255-265
```tablegen
def GPUMappingMaskAttr : GPU_Attr<"GPUMappingMask", "mask", [
  DeclareAttrInterfaceMethods<DeviceMaskingAttrInterface> ] >  {
  let parameters = (ins "uint64_t":$mask);
  let assemblyFormat = "`<` params `>`";
  let description = [{
    Attribute describing how to filter the processing units that a region is
    mapped to. The masking is a bitfield that specifies for each processing
    unit whether it is active or not.
  }];
}

```
- **EN**: Introduces declarations for `GPUMappingMaskAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUMappingMaskAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 266-274
```tablegen
def GPUMemorySpaceMappingAttr : GPU_Attr<"GPUMemorySpaceMapping", "memory_space", [
  DeclareAttrInterfaceMethods<DeviceMappingAttrInterface> ] >  {
  let parameters = (ins
    EnumParameter<GPU_AddressSpaceEnum>:$address_space
  );
  let assemblyFormat = "`<` params `>`";
  let description = [{
    An attribute that allows defining memory hierarchy for GPU devices.

```
- **EN**: Introduces declarations for `GPUMemorySpaceMappingAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUMemorySpaceMappingAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 275-282
```tablegen
    GPU Memory has three memory space, global, workgroup, and private. The global memory
    is visible to all workitems and workgroups, the workgroup memory is only available for workitems
    within a workgroup, and private memory is only visible to a single workitem. This attribute indicates
    that using memory hiearchy is desired. It can be consumed by lowering to
    move data to a specific address space in GPU code.
  }];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 283-283
```tablegen
#endif // GPU_DEVICE_MAPPING_ATTR
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
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Dialect/GPU/IR/GPUBase.td`, `mlir/IR/EnumAttr.td`, `mlir/Dialect/SCF/IR/DeviceMappingInterface.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
