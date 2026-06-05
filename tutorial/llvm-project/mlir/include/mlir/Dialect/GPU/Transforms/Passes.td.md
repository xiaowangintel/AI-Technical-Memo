# Passes.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/GPU/Transforms/Passes.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the GPU dialect, focused on rewrite patterns, passes, and canonicalization helpers and `Passes`.
  - **CN**: 为 GPU 方言定义聚焦 `Passes` 的声明式 TableGen 规格，覆盖重写模式、Pass 与规范化辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- Passes.td - GPU pass definition file ---------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-13
```tablegen

#ifndef MLIR_DIALECT_GPU_PASSES
#define MLIR_DIALECT_GPU_PASSES

include "mlir/Pass/PassBase.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 14-19
```tablegen
def GpuLaunchSinkIndexComputationsPass
    : Pass<"gpu-launch-sink-index-computations"> {
  let summary = "Sink index computations into gpu.launch body";
  let dependentDialects = ["mlir::gpu::GPUDialect"];
}

```
- **EN**: Introduces declarations for `GpuLaunchSinkIndexComputationsPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuLaunchSinkIndexComputationsPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 20-27
```tablegen
def GpuKernelOutliningPass : Pass<"gpu-kernel-outlining", "ModuleOp"> {
  let summary = "Outline gpu.launch bodies to kernel functions";
  let dependentDialects = ["mlir::DLTIDialect", "cf::ControlFlowDialect"];
  let options = [Option<"dataLayoutStr", "data-layout-str", "std::string",
                        /*default=*/"",
                        "String description of the data layout">];
}

```
- **EN**: Introduces declarations for `GpuKernelOutliningPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuKernelOutliningPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 28-33
```tablegen
def GpuAsyncRegionPass : Pass<"gpu-async-region", "func::FuncOp"> {
  let summary = "Make GPU ops async";
  let dependentDialects = ["async::AsyncDialect"];
}

def GpuMapParallelLoopsPass
```
- **EN**: Introduces declarations for `GpuAsyncRegionPass`, `GpuMapParallelLoopsPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuAsyncRegionPass`, `GpuMapParallelLoopsPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 34-42
```tablegen
    : Pass<"gpu-map-parallel-loops", "mlir::func::FuncOp"> {
  let summary = "Greedily maps loops to GPU hardware dimensions.";
  let description = [{
    Maps the parallel loops found in the given function to workgroups. The first
    loop encountered will be mapped to the global workgroup and the second loop
    encountered to the local workgroup. Within each mapping, the first three
    dimensions are mapped to x/y/z hardware ids and all following dimensions are
    mapped to sequential loops.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 43-54
```tablegen
    Ordering of the loop mapping against the different dimensions is controlled
    by the `mapping-policy` option.
    Two policies are supported:
       1. `outermost-first` (default): the outermost loop maps to X, then Y
          and finally Z.
       2. `innermost-first`: the innermost loop maps to X, then Y and finally Z.
  }];
  let dependentDialects = ["mlir::gpu::GPUDialect"];
  let options = [Option<"mappingPolicyStr", "mapping-policy", "std::string",
                        /*default=*/"\"outermost-first\"",
                        "Policy outlining how to assign loops to GPU dimensions."
                        "Supported values are `outermost-first` and `innermost-first`.">];
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 55-66
```tablegen
}

def GpuEliminateBarriers
    : Pass<"gpu-eliminate-barriers", "mlir::func::FuncOp"> {
  let summary = "Erase unnecessary barriers";
  let description = [{
    Barrier elimination pass. If a barrier does not enforce any conflicting
    pair of memory effects, including a pair that is enforced by another
    barrier, it is unnecessary and can be removed. Adapted from
    "High-Performance GPU-to-CPU Transpilation and Optimization via High-Level
    Parallel Constructs" by Moses, Ivanov, Domke, Endo, Doerfert, and Zinenko in
    PPoPP 2023 and implementation in Polygeist.
```
- **EN**: Introduces declarations for `GpuEliminateBarriers`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuEliminateBarriers` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 67-74
```tablegen
  }];
  let dependentDialects = [
    "mlir::gpu::GPUDialect",
    "mlir::memref::MemRefDialect",
    "mlir::scf::SCFDialect"
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 75-86
```tablegen
def GpuDecomposeMemrefsPass : Pass<"gpu-decompose-memrefs"> {
  let summary = "Decomposes memref index computation into explicit ops.";
  let description = [{
    This pass decomposes memref index computation into explicit computations on
    sizes/strides, obtained from `memref.extract_memref_metadata` which it tries
    to place outside of `gpu.launch` body. Memrefs are then reconstructed using
    `memref.reinterpret_cast`.
    This is needed for as some targets (SPIR-V) lower memrefs to bare pointers
    and sizes/strides for dynamically-sized memrefs are not available inside
    `gpu.launch`.
  }];
  let dependentDialects = [
```
- **EN**: Introduces declarations for `GpuDecomposeMemrefsPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuDecomposeMemrefsPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 87-92
```tablegen
    "mlir::gpu::GPUDialect", "mlir::memref::MemRefDialect",
    "mlir::affine::AffineDialect"
  ];
}

def GpuModuleToBinaryPass
```
- **EN**: Introduces declarations for `GpuModuleToBinaryPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuModuleToBinaryPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 93-99
```tablegen
    : Pass<"gpu-module-to-binary", ""> {
  let summary = "Transforms a GPU module into a GPU binary.";
  let description = [{
    This pass searches for all nested GPU modules and serializes the module
    using the target attributes attached to the module, producing a GPU binary
    with an object for every target.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 100-111
```tablegen
    The `format` argument can have the following values:
    1. `offloading`, `llvm`: produces an offloading representation.
    2. `assembly`, `isa`: produces assembly code.
    3. `binary`, `bin`: produces binaries.
    4. `fatbinary`, `fatbin`: produces fatbinaries.
  }];
  let options = [
    Option<"toolkitPath", "toolkit", "std::string", [{""}],
           "Toolkit path.">,
    ListOption<"linkFiles", "l", "std::string",
           "Extra files to link to.">,
    Option<"cmdOptions", "opts", "std::string", [{""}],
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 112-119
```tablegen
           "Command line options to pass to the tools.">,
    Option<"compilationTarget", "format", "std::string", [{"fatbin"}],
           "The target representation of the compilation process.">,
    Option<"elfSection", "section", "std::string", [{""}],
           "ELF section where binary is to be located.">
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 120-125
```tablegen
def GpuNVVMAttachTarget: Pass<"nvvm-attach-target", ""> {
  let summary = "Attaches an NVVM target attribute to a GPU Module.";
  let description = [{
    This pass searches for all GPU Modules in the immediate regions and attaches
    an NVVM target if the module matches the name specified by the `module` argument.

```
- **EN**: Introduces declarations for `GpuNVVMAttachTarget`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuNVVMAttachTarget` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 126-137
```tablegen
    Example:
    ```
    // File: in.mlir:
    gpu.module @nvvm_module_1 {...}
    gpu.module @nvvm_module_2 {...}
    gpu.module @rocdl_module_1 {...}
    // mlir-opt --nvvm-attach-target="module=nvvm.* chip=sm_90" in.mlir
    gpu.module @nvvm_module_1 [#nvvm.target<chip = "sm_90">] {...}
    gpu.module @nvvm_module_2 [#nvvm.target<chip = "sm_90">] {...}
    gpu.module @rocdl_module_1 {...}
    ```
  }];
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 138-149
```tablegen
  let options = [
    Option<"moduleMatcher", "module", "std::string",
           /*default=*/ [{""}],
           "Regex used to identify the modules to attach the target to.">,
    Option<"triple", "triple", "std::string",
           /*default=*/ "\"nvptx64-nvidia-cuda\"",
           "Target triple.">,
    Option<"chip", "chip", "std::string",
           /*default=*/"\"sm_75\"",
           "Target chip.">,
    Option<"features", "features", "std::string",
           /*default=*/"\"\"",
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 150-161
```tablegen
           "Target features.">,
    Option<"optLevel", "O", "unsigned",
           /*default=*/"2",
           "Optimization level.">,
    Option<"fastFlag", "fast", "bool",
           /*default=*/"false",
           "Enable fast math mode.">,
    Option<"ftzFlag", "ftz", "bool",
           /*default=*/"false",
           "Enable flush to zero for denormals.">,
    Option<"compilerDiagnosticsFlag", "collect-compiler-diagnostics", "bool",
           /*default=*/"false",
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 162-173
```tablegen
           "Enable collection of compiler diagnostics.">,
    ListOption<"linkLibs", "l", "std::string",
           "Extra bitcode libraries paths to link to.">,
    Option<"cmdOptions", "ptxas-cmd-options", "std::string",
           /*default=*/ [{""}],
           "Command line options passed to downstream compiler">,
    Option<"verifyTarget", "verify-target-arch", "bool",
           /*default=*/"true",
           "Enable verification of the target architecture">,
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 174-179
```tablegen
def GpuROCDLAttachTarget: Pass<"rocdl-attach-target", ""> {
  let summary = "Attaches a ROCDL target attribute to a GPU Module.";
  let description = [{
    This pass searches for all GPU Modules in the immediate regions and attaches
    a ROCDL target if the module matches the name specified by the `module` argument.

```
- **EN**: Introduces declarations for `GpuROCDLAttachTarget`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuROCDLAttachTarget` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 180-191
```tablegen
    Example:
    ```
    // File: in.mlir:
    gpu.module @nvvm_module_1 {...}
    gpu.module @nvvm_module_2 {...}
    gpu.module @rocdl_module_1 {...}
    // mlir-opt --nvvm-attach-target="module=rocdl.* chip=gfx90a" in.mlir
    gpu.module @nvvm_module_1 {...}
    gpu.module @nvvm_module_2 {...}
    gpu.module @rocdl_module_1 [#rocdl.target<chip = "gfx90a">] {...}
    ```
  }];
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 192-203
```tablegen
  let options = [
    Option<"moduleMatcher", "module", "std::string",
           /*default=*/ [{""}],
           "Regex used to identify the modules to attach the target to.">,
    Option<"triple", "triple", "std::string",
           /*default=*/ "\"amdgcn-amd-amdhsa\"",
           "Target triple.">,
    Option<"chip", "chip", "std::string",
           /*default=*/"\"gfx900\"",
           "Target chip.">,
    Option<"features", "features", "std::string",
           /*default=*/"\"\"",
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 204-215
```tablegen
           "Target features.">,
    Option<"abiVersion", "abi", "std::string",
           /*default=*/"\"600\"",
           "ABI version.">,
    Option<"optLevel", "O", "unsigned",
           /*default=*/"2",
           "Optimization level.">,
    Option<"wave64Flag", "wave64", "bool",
           /*default=*/"true",
           "Use Wave64 mode.">,
    Option<"fastFlag", "fast", "bool",
           /*default=*/"false",
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 216-227
```tablegen
           "Enable fast relaxed math opt.">,
    Option<"dazFlag", "daz", "bool",
           /*default=*/"false",
           "Enable denormals are zero opt.">,
    Option<"finiteOnlyFlag", "finite-only", "bool",
           /*default=*/"false",
           "Enable finite only opt.">,
    Option<"unsafeMathFlag", "unsafe-math", "bool",
           /*default=*/"false",
           "Enable unsafe math opt.">,
    Option<"correctSqrtFlag", "correct-sqrt", "bool",
           /*default=*/"true",
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 228-233
```tablegen
           "Enable correct rounded sqrt.">,
    ListOption<"linkLibs", "l", "std::string",
           "Extra bitcode libraries paths to link to.">,
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 234-239
```tablegen
def GpuSPIRVAttachTarget: Pass<"spirv-attach-target", ""> {
  let summary = "Attaches an SPIR-V target attribute to a GPU Module.";
  let description = [{
    This pass searches for all GPU Modules in the immediate regions and attaches
    an SPIR-V target if the module matches the name specified by the `module` argument.

```
- **EN**: Introduces declarations for `GpuSPIRVAttachTarget`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuSPIRVAttachTarget` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 240-251
```tablegen
    Example:
    ```
    // Given the following file: in1.mlir:
    gpu.module @nvvm_module_1 {...}
    gpu.module @spirv_module_1 {...}
    // With
    // mlir-opt --spirv-attach-target="module=spirv.* ver=v1.0 caps=Kernel" in1.mlir
    // it will generate,
    gpu.module @nvvm_module_1 {...}
    gpu.module @spirv_module_1 [#spirv.target<#spirv.vce<v1.0, [Kernel], []>, #spirv.resource_limits<>>] {...}
    ```
  }];
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 252-263
```tablegen
  let options = [
    Option<"moduleMatcher", "module", "std::string",
           /*default=*/ [{""}],
           "Regex used to identify the modules to attach the target to.">,
    Option<"spirvVersion", "ver", "std::string",
           /*default=*/ "\"v1.0\"",
           "SPIR-V Version.">,
    ListOption<"spirvCapabilities", "caps", "std::string",
           "List of supported SPIR-V Capabilities">,
    ListOption<"spirvExtensions", "exts", "std::string",
           "List of supported SPIR-V Extensions">,
    Option<"clientApi", "client_api", "std::string",
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 264-275
```tablegen
           /*default=*/ "\"Unknown\"",
           "Client API">,
    Option<"deviceVendor", "vendor", "std::string",
           /*default=*/ "\"Unknown\"",
           "Device Vendor">,
    Option<"deviceType", "device_type", "std::string",
           /*default=*/ "\"Unknown\"",
           "Device Type">,
    Option<"deviceId", "device_id", "uint32_t",
           /*default=*/ "",
           "Device ID">,
  ];
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 276-283
```tablegen
}

def GpuXeVMAttachTarget : Pass<"xevm-attach-target", ""> {
  let summary = "Attaches a XeVM target attribute to a GPU Module.";
  let description = [{
    This pass searches for all GPU Modules in the immediate regions and attaches
    a XeVM target if the module matches the name specified by the `module` argument.

```
- **EN**: Introduces declarations for `GpuXeVMAttachTarget`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GpuXeVMAttachTarget` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 284-295
```tablegen
    Example:
    ```
    // File: in.mlir:
    gpu.module @nvvm_module_1 {...}
    gpu.module @rocdl_module_2 {...}
    gpu.module @xevm_module_3 {...}
    // mlir-opt --xevm-attach-target="module=xevm.* chip=pvc" in.mlir
    gpu.module @nvvm_module_1 {...}
    gpu.module @rocdl_module_2 {...}
    gpu.module @xevm_module_3 [#xevm.target<chip = "pvc">] {...}
    ```
  }];
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 296-307
```tablegen
  let options =
      [Option<"moduleMatcher", "module", "std::string",
              /*default=*/[{""}],
              "Regex used to identify the modules to attach the target to.">,
       Option<"triple", "triple", "std::string",
              /*default=*/"\"spirv64-unknown-unknown\"", "Target triple.">,
       Option<"chip", "chip", "std::string",
              /*default=*/"\"bmg\"", "Target chip.">,
       Option<"optLevel", "O", "unsigned",
              /*default=*/"2", "Optimization level.">,
       ListOption<"linkLibs", "l", "std::string",
                  "Extra bitcode libraries paths to link to.">,
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 308-312
```tablegen
       Option<"cmdOptions", "cmd-options", "std::string",
              /*default=*/[{""}],
              "Command line options passed to downstream compiler">];
}
#endif // MLIR_DIALECT_GPU_PASSES
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

- **TableGen includes / TableGen 包含**: `mlir/Pass/PassBase.td`
- **Subsystem categories / 子系统类别**: pass infrastructure and registration support / Pass 基础设施与注册支持 (1)
