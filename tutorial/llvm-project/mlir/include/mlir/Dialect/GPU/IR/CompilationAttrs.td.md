# CompilationAttrs.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/GPU/IR/CompilationAttrs.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines GPU compilation related attributes.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/GPU/IR`，围绕 GPU 方言公开 `CompilationAttrs` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- CompilationAttrs.td - GPU compilation attributes ---*- tablegen -*-===//
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
// This file defines GPU compilation related attributes.
//
//===----------------------------------------------------------------------===//

#ifndef GPU_COMPILATION_ATTRS
#define GPU_COMPILATION_ATTRS

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 16-21
```tablegen
include "mlir/Dialect/GPU/IR/GPUBase.td"
include "mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td"

//===----------------------------------------------------------------------===//
// GPU kernel metadata attribute
//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 22-27
```tablegen

def GPU_KernelMetadataAttr : GPU_Attr<"KernelMetadata", "kernel_metadata"> {
  let description = [{
    GPU attribute for storing metadata related to a compiled kernel. The
    attribute contains the name and arguments type of the kernel.

```
- **EN**: Introduces declarations for `GPU_KernelMetadataAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_KernelMetadataAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 28-34
```tablegen
    The attribute also contains optional parameters for storing the arguments
    attributes as well as a dictionary for additional metadata, like occupancy
    information or other function attributes.

    Note: The `arg_attrs` parameter is expected to follow all the constraints
    imposed by the `mlir::FunctionOpInterface` interface.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 35-46
```tablegen
    Examples:
    ```mlir
      #gpu.kernel_metadata<@kernel1, (i32) -> (), arg_attrs = [...],  metadata = {reg_count = 255, ...}>
      #gpu.kernel_metadata<@kernel2, (i32, f64) -> ()>
    ```
  }];
  let parameters = (ins
    "StringAttr":$name,
    "Type":$function_type,
    OptionalParameter<"ArrayAttr", "arguments attributes">:$arg_attrs,
    OptionalParameter<"DictionaryAttr", "metadata dictionary">:$metadata
  );
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 47-58
```tablegen
  let assemblyFormat = [{
    `<` $name `,` $function_type (`,` struct($arg_attrs, $metadata)^)? `>`
  }];
  let builders = [
    AttrBuilderWithInferredContext<(ins "StringAttr":$name,
                                        "Type":$functionType,
                                        CArg<"ArrayAttr", "nullptr">:$argAttrs,
                                        CArg<"DictionaryAttr",
                                             "nullptr">:$metadata), [{
      assert(name && "invalid name");
      return $_get(name.getContext(), name, functionType, argAttrs, metadata);
    }]>,
```
- **EN**: Implements logic around `function_type`, `AttrBuilderWithInferredContext`, `assert`, `_get`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `function_type`, `AttrBuilderWithInferredContext`, `assert`, `_get` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 59-65
```tablegen
    AttrBuilderWithInferredContext<(ins "FunctionOpInterface":$kernel,
                                         CArg<"DictionaryAttr",
                                              "nullptr">:$metadata)>
  ];
  let genVerifyDecl = 1;
  let extraClassDeclaration = [{
    /// Compare two kernels based on the name.
```
- **EN**: Implements logic around `AttrBuilderWithInferredContext`.
- **CN**: 围绕 `AttrBuilderWithInferredContext` 实现具体逻辑。

### Lines 66-71
```tablegen
    bool operator<(const KernelMetadataAttr& other) const {
      return getName().getValue() < other.getName().getValue();
    }

    /// Returns the metadata attribute corresponding to `key` or `nullptr`
    /// if missing.
```
- **EN**: Implements logic around `operator`, `getName`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `operator`, `getName` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 72-83
```tablegen
    Attribute getAttr(StringRef key) const {
      DictionaryAttr attrs = getMetadata();
      return attrs ? attrs.get(key) : nullptr;
    }
    template <typename ConcreteAttr>
    ConcreteAttr getAttr(StringRef key) const {
      return llvm::dyn_cast_or_null<ConcreteAttr>(getAttr(key));
    }
    Attribute getAttr(StringAttr key) const {
      DictionaryAttr attrs = getMetadata();
      return attrs ? attrs.get(key) : nullptr;
    }
```
- **EN**: Implements logic around `getAttr`, `getMetadata`, `get`, `dyn_cast_or_null`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getAttr`, `getMetadata`, `get`, `dyn_cast_or_null` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 84-89
```tablegen
    template <typename ConcreteAttr>
    ConcreteAttr getAttr(StringAttr key) const {
      return llvm::dyn_cast_or_null<ConcreteAttr>(getAttr(key));
    }

    /// Returns the attribute dictionary at position `index`.
```
- **EN**: Implements logic around `getAttr`, `dyn_cast_or_null`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getAttr`, `dyn_cast_or_null` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 90-95
```tablegen
    DictionaryAttr getArgAttrDict(unsigned index) {
      ArrayAttr argArray = getArgAttrs();
      return argArray ? llvm::cast<DictionaryAttr>(argArray[index]) : nullptr;
    }

    /// Return the specified attribute, if present, for the argument at 'index',
```
- **EN**: Implements logic around `getArgAttrDict`, `getArgAttrs`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getArgAttrDict`, `getArgAttrs` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 96-105
```tablegen
    /// null otherwise.
    Attribute getArgAttr(unsigned index, StringAttr name) {
      DictionaryAttr argDict = getArgAttrDict(index);
      return argDict ? argDict.get(name) : nullptr;
    }
    Attribute getArgAttr(unsigned index, StringRef name) {
      DictionaryAttr argDict = getArgAttrDict(index);
      return argDict ? argDict.get(name) : nullptr;
    }

```
- **EN**: Implements logic around `getArgAttr`, `getArgAttrDict`, `get`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getArgAttr`, `getArgAttrDict`, `get` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 106-111
```tablegen
    /// Returns a new KernelMetadataAttr that contains `attrs` in the metadata dictionary.
    KernelMetadataAttr appendMetadata(ArrayRef<NamedAttribute> attrs) const;
  }];
}

//===----------------------------------------------------------------------===//
```
- **EN**: Declares APIs or declarative rules around `appendMetadata`.
- **CN**: 声明与 `appendMetadata` 相关的 API 或声明式规则。

### Lines 112-120
```tablegen
// GPU kernel table attribute
//===----------------------------------------------------------------------===//

def GPU_KernelTableAttr : GPU_Attr<"KernelTable", "kernel_table"> {
  let description = [{
    GPU attribute representing a list of `#gpu.kernel_metadata` attributes. This
    attribute supports searching kernels by name. All kernels in the table must
    have an unique name.

```
- **EN**: Introduces declarations for `GPU_KernelTableAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_KernelTableAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 121-128
```tablegen
    Examples:
    ```mlir
      // Empty table.
      #gpu.kernel_table<>

      // Table with a single kernel.
      #gpu.kernel_table<[#gpu.kernel_metadata<kernel0, () -> () >]>

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 129-140
```tablegen
      // Table with multiple kernels.
      #gpu.kernel_table<[
        #gpu.kernel_metadata<"kernel0", (i32, f32) -> (), metadata = {sgpr_count = 255}>,
        #gpu.kernel_metadata<"kernel1", (i32) -> ()>
      ]>
    ```
  }];
  let parameters = (ins
    OptionalArrayRefParameter<"KernelMetadataAttr", "array of kernels">:$kernel_table
  );
  let assemblyFormat = [{
    `<` (`[` qualified($kernel_table)^ `]`)? `>`
```
- **EN**: Implements logic around `qualified`; this block handles textual assembly parsing or printing concerns; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `qualified` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并涉及目标平台或加速器专用语义。

### Lines 141-152
```tablegen
  }];
  let builders = [
    AttrBuilder<(ins "ArrayRef<KernelMetadataAttr>":$kernels,
                     CArg<"bool", "false">:$isSorted)>
  ];
  let skipDefaultBuilders = 1;
  let genVerifyDecl = 1;
  let extraClassDeclaration = [{
    llvm::ArrayRef<KernelMetadataAttr>::iterator begin() const {
      return getKernelTable().begin();
    }
    llvm::ArrayRef<KernelMetadataAttr>::iterator end() const {
```
- **EN**: Implements logic around `AttrBuilder`, `begin`, `getKernelTable`, `end`.
- **CN**: 围绕 `AttrBuilder`, `begin`, `getKernelTable`, `end` 实现具体逻辑。

### Lines 153-161
```tablegen
      return getKernelTable().end();
    }
    size_t size() const {
      return getKernelTable().size();
    }
    bool empty() const {
      return getKernelTable().empty();
    }

```
- **EN**: Implements logic around `getKernelTable`, `size`, `empty`.
- **CN**: 围绕 `getKernelTable`, `size`, `empty` 实现具体逻辑。

### Lines 162-167
```tablegen
    /// Returns the kernel with name `key` or `nullptr` if not present.
    KernelMetadataAttr lookup(StringRef key) const;
    KernelMetadataAttr lookup(StringAttr key) const;
  }];
}

```
- **EN**: Declares APIs or declarative rules around `lookup`.
- **CN**: 声明与 `lookup` 相关的 API 或声明式规则。

### Lines 168-173
```tablegen
//===----------------------------------------------------------------------===//
// GPU object attribute.
//===----------------------------------------------------------------------===//

// For documentation on this enum cases, see the `GPU_ObjectAttr` docs.
def GPU_ObjectOffload : I32EnumAttrCase<"Offload", 1, "offload">;
```
- **EN**: Introduces declarations for `cases`, `GPU_ObjectOffload`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `cases`, `GPU_ObjectOffload` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 174-184
```tablegen
def GPU_ObjectISA : I32EnumAttrCase<"Assembly", 2, "assembly">;
def GPU_ObjectBinary : I32EnumAttrCase<"Binary", 3, "bin">;
def GPU_ObjectFatbin : I32EnumAttrCase<"Fatbin", 4, "fatbin">;
def GPU_CompilationTargetEnum : GPU_I32Enum<
  "CompilationTarget", "GPU compilation format", [
    GPU_ObjectOffload,
    GPU_ObjectISA,
    GPU_ObjectBinary,
    GPU_ObjectFatbin
  ]>;

```
- **EN**: Introduces declarations for `GPU_ObjectISA`, `GPU_ObjectBinary`, `GPU_ObjectFatbin`, `GPU_CompilationTargetEnum`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ObjectISA`, `GPU_ObjectBinary`, `GPU_ObjectFatbin`, `GPU_CompilationTargetEnum` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 185-191
```tablegen
def GPU_ObjectAttr : GPU_Attr<"Object", "object"> {
  let description = [{
    A GPU object attribute glues together a GPU target, the object kind, a
    binary string with the object, the object properties, and kernel metadata,
    encapsulating how the object was generated and its properties with the
    object itself.

```
- **EN**: Introduces declarations for `GPU_ObjectAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ObjectAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 192-203
```tablegen
    There are four object formats:
    1. `Offload`: represents generic objects not described by the other three
    formats, and its meaning is target-dependent. For example, on the NVPTX and
    AMDGPU targets, this format is associated with LLVM bitcode.
    2. `Assembly`: represents GPU assembly code. For example, in the NVPTX
    target, assembly is PTX code, which can be JITted at runtime.
    3. `Binary`: represents executable code for a GPU single architecture. For
    example, PTX code that was compiled for a specific compute capability. Note
    that this format is likely to throw an error if there is an architecture
    mismatch between the compiled and running architecture.
    4. `Fatbin`: represents a GPU fat binary with executable code for multiple
    architectures. This format is the default; thus, it gets elided inassembly
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 204-211
```tablegen
    code.

    Object properties are specified through the `properties` dictionary
    attribute and can be used to define additional information.

    Kernel metadata is specified through the `kernels` parameter, and can be
    used to specify additional information on a kernel by kernel basis.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 212-223
```tablegen
    The target attribute must implement or promise the `TargetAttrInterface`
    interface.

    ```
      #gpu.object<#rocdl.target, offload = "..."> // An offload object.
      #gpu.object<#nvvm.target, properties = {O = 3 : i32}, assembly = "..."> // An assembly object with additional properties.
      #gpu.object<#rocdl.target, bin = "..."> // A binary object.
      #gpu.object<#nvvm.target, "..."> // A fatbin object.
      #gpu.object<#nvvm.target, kernels = #gpu.kernel_table<...>, "..."> // An object with a kernel table.
    ```
  }];
  let parameters = (ins
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 224-235
```tablegen
    "Attribute":$target,
    DefaultValuedParameter<"CompilationTarget", "CompilationTarget::Fatbin">:$format,
    "StringAttr":$object,
    OptionalParameter<"DictionaryAttr">:$properties,
    OptionalParameter<"KernelTableAttr">:$kernels
  );
  let builders = [
    AttrBuilderWithInferredContext<(ins "Attribute":$target,
                                        "CompilationTarget":$format,
                                        "StringAttr":$object,
                                        CArg<"DictionaryAttr", "nullptr">:$properties,
                                        CArg<"KernelTableAttr", "nullptr">:$kernels), [{
```
- **EN**: Implements logic around `AttrBuilderWithInferredContext`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `AttrBuilderWithInferredContext` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 236-247
```tablegen
      assert(target && "invalid target");
      return $_get(target.getContext(), target, format, object, properties, kernels);
    }]>
  ];
  let assemblyFormat = [{ `<`
      $target `,`  (`properties` `=` $properties^ `,`)?
      (`kernels` `=` $kernels^ `,`)?
      custom<Object>($format, $object)
    `>`
  }];
  let genVerifyDecl = 1;
}
```
- **EN**: Implements logic around `assert`, `_get`, `custom`; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `assert`, `_get`, `custom` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 248-254
```tablegen

def GPUObjectArrayAttr :
  TypedArrayAttrBase<GPU_ObjectAttr, "an array of GPU object attributes">;

//===----------------------------------------------------------------------===//
// GPU offloading LLVM translation handler attributes.
//===----------------------------------------------------------------------===//
```
- **EN**: Introduces declarations for `GPUObjectArrayAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUObjectArrayAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 255-264
```tablegen

def GPU_SelectObjectAttr : GPU_Attr<"SelectObject", "select_object", [
      OffloadingTranslationAttrTrait
    ]> {
  let description = [{
    This GPU offloading handler selects a single GPU object for embedding. The
    object is selected based on the `target` parameter, this parameter can be
    either a number -i.e. selects the ith-target, or the target itself -i.e.
    searches for the specified target in the object array.

```
- **EN**: Introduces declarations for `GPU_SelectObjectAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SelectObjectAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 265-276
```tablegen
    The first object in a `gpu.binary` operation is selected if no target is
    specified.
  }];
  let parameters = (ins
    OptionalParameter<"Attribute", "Target to select for embedding.">:$target
  );
  let assemblyFormat = [{
    (`<` $target^ `>`)?
  }];
  let genVerifyDecl = 1;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 277-277
```tablegen
#endif // GPU_COMPILATION_ATTRS
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
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Dialect/GPU/IR/GPUBase.td`, `mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (2)
