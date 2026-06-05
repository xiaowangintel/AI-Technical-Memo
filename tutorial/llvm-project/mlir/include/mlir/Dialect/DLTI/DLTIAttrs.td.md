# DLTIAttrs.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/DLTI/DLTIAttrs.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the DLTI dialect, focused on public header declarations and `DLTIAttrs`.
  - **CN**: 为 DLTI 方言定义聚焦 `DLTIAttrs` 的声明式 TableGen 规格，覆盖公共头文件声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===- DLTIAttrs.td - DLTI dialect attributes definition --*- tablegen -*-===//
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

#ifndef MLIR_DIALECT_DLTI_DLTIATTRS_TD
#define MLIR_DIALECT_DLTI_DLTIATTRS_TD

include "mlir/Dialect/DLTI/DLTI.td"
include "mlir/Interfaces/DataLayoutInterfaces.td"
include "mlir/IR/AttrTypeBase.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 16-22
```tablegen
class DLTIAttr<string name, list<Trait> traits = [],
                string baseCppClass = "::mlir::Attribute">
    : AttrDef<DLTI_Dialect, name, traits, baseCppClass> { }

//===----------------------------------------------------------------------===//
// DataLayoutEntryAttr
//===----------------------------------------------------------------------===//
```
- **EN**: Introduces declarations for `DLTIAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DLTIAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 23-34
```tablegen

def DLTI_DataLayoutEntryAttr :
    DLTIAttr<"DataLayoutEntry", [DataLayoutEntryInterface]> {
  let summary = "An attribute to represent an entry of a data layout specification.";
  let description = [{
    A data layout entry attribute is a key-value pair where the key is a type or
    an identifier and the value is another attribute. These entries form a data
    layout specification.
  }];
  let parameters = (ins
    "DataLayoutEntryKey":$key, "Attribute":$value
  );
```
- **EN**: Introduces declarations for `DLTI_DataLayoutEntryAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DLTI_DataLayoutEntryAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 35-42
```tablegen
  // TODO: We do not generate storage class because llvm::PointerUnion
  // does not work with hash_key method.
  let genStorageClass = 0;
  let mnemonic = "dl_entry";
  let genVerifyDecl = 0;
  let hasCustomAssemblyFormat = 1;
  let extraClassDeclaration = [{
    /// Returns the entry with the given key and value.
```
- **EN**: Introduces declarations for `because`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `because` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 43-48
```tablegen
    static DataLayoutEntryAttr get(StringAttr key, Attribute value);
    static DataLayoutEntryAttr get(MLIRContext *context, Type key, Attribute value);
    static DataLayoutEntryAttr get(Type key, Attribute value);
  }];
}

```
- **EN**: Declares APIs or declarative rules around `get`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `get` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 49-60
```tablegen
//===----------------------------------------------------------------------===//
// DataLayoutSpecAttr
//===----------------------------------------------------------------------===//

def DLTI_DataLayoutSpecAttr :
    DLTIAttr<"DataLayoutSpec", [DataLayoutSpecInterface]> {
  let summary = "An attribute to represent a data layout specification.";
  let description = [{
    A data layout specification is a list of entries that specify (partial) data
    layout information. It is expected to be attached to operations that serve
    as scopes for data layout requests.
  }];
```
- **EN**: Introduces declarations for `DLTI_DataLayoutSpecAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DLTI_DataLayoutSpecAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 61-68
```tablegen
  let parameters = (ins
    ArrayRefParameter<"DataLayoutEntryInterface", "">:$entries
  );
  let mnemonic = "dl_spec";
  let genVerifyDecl = 1;
  let hasCustomAssemblyFormat = 1;
  let extraClassDeclaration = [{
    /// Combines this specification with `specs`, enclosing specifications listed
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 69-74
```tablegen
    /// from outermost to innermost. This overwrites the older entries with the
    /// same key as the newer entries if the entries are compatible. Returns null
    /// if the specifications are not compatible.
    DataLayoutSpecAttr combineWith(ArrayRef<DataLayoutSpecInterface> specs) const;

    /// Returns the endiannes identifier.
```
- **EN**: Declares APIs or declarative rules around `combineWith`.
- **CN**: 声明与 `combineWith` 相关的 API 或声明式规则。

### Lines 75-80
```tablegen
    StringAttr getEndiannessIdentifier(MLIRContext *context) const;

    /// Returns the default memory space identifier.
    StringAttr getDefaultMemorySpaceIdentifier(MLIRContext *context) const;

    /// Returns the alloca memory space identifier.
```
- **EN**: Declares APIs or declarative rules around `getEndiannessIdentifier`, `getDefaultMemorySpaceIdentifier`.
- **CN**: 声明与 `getEndiannessIdentifier`, `getDefaultMemorySpaceIdentifier` 相关的 API 或声明式规则。

### Lines 81-86
```tablegen
    StringAttr getAllocaMemorySpaceIdentifier(MLIRContext *context) const;

    /// Returns the mangling mode identifier.
    StringAttr getManglingModeIdentifier(MLIRContext *context) const;

    /// Returns the program memory space identifier.
```
- **EN**: Declares APIs or declarative rules around `getAllocaMemorySpaceIdentifier`, `getManglingModeIdentifier`.
- **CN**: 声明与 `getAllocaMemorySpaceIdentifier`, `getManglingModeIdentifier` 相关的 API 或声明式规则。

### Lines 87-92
```tablegen
    StringAttr getProgramMemorySpaceIdentifier(MLIRContext *context) const;

    /// Returns the global memory space identifier.
    StringAttr getGlobalMemorySpaceIdentifier(MLIRContext *context) const;

    /// Returns the stack alignment identifier.
```
- **EN**: Declares APIs or declarative rules around `getProgramMemorySpaceIdentifier`, `getGlobalMemorySpaceIdentifier`.
- **CN**: 声明与 `getProgramMemorySpaceIdentifier`, `getGlobalMemorySpaceIdentifier` 相关的 API 或声明式规则。

### Lines 93-98
```tablegen
    StringAttr getStackAlignmentIdentifier(MLIRContext *context) const;

    /// Returns the function pointer alignment identifier.
    StringAttr getFunctionPointerAlignmentIdentifier(MLIRContext *context) const;

    /// Returns the legal int widths identifier.
```
- **EN**: Declares APIs or declarative rules around `getStackAlignmentIdentifier`, `getFunctionPointerAlignmentIdentifier`.
- **CN**: 声明与 `getStackAlignmentIdentifier`, `getFunctionPointerAlignmentIdentifier` 相关的 API 或声明式规则。

### Lines 99-107
```tablegen
    StringAttr getLegalIntWidthsIdentifier(MLIRContext *context) const;

    /// Returns the attribute associated with the key.
    FailureOr<Attribute> query(DataLayoutEntryKey key) {
      return ::llvm::cast<mlir::DataLayoutSpecInterface>(*this).queryHelper(key);
    }
  }];
}

```
- **EN**: Implements logic around `getLegalIntWidthsIdentifier`, `query`, `DataLayoutSpecInterface>`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getLegalIntWidthsIdentifier`, `query`, `DataLayoutSpecInterface>` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 108-117
```tablegen
//===----------------------------------------------------------------------===//
// MapAttr
//===----------------------------------------------------------------------===//

def DLTI_MapAttr : DLTIAttr<"Map", [DLTIQueryInterface]> {
  let summary = "A mapping of DLTI-information by way of key-value pairs";
  let description = [{
    A Data Layout and Target Information map is a list of entries effectively
    encoding a dictionary, mapping DLTI-related keys to DLTI-related values.

```
- **EN**: Introduces declarations for `DLTI_MapAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DLTI_MapAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 118-129
```tablegen
    This attribute's main purpose is to facilate querying IR for arbitrary
    key-value associations that encode DLTI. Facility functions exist to perform
    recursive lookups on nested DLTI-map/query interface-implementing
    attributes.

    Consider the following flat encoding of a single-key dictionary
    ```
    #dlti.map<"CPU::cache::L1::size_in_bytes" = 65536 : i32>>
    ```
    versus nested maps, which make it possible to obtain sub-dictionaries of
    related information (with the following example making use of other
    attributes that also implement the `DLTIQueryInterface`):
```
- **EN**: Implements logic around `information`; this block expresses reusable interface-based behavior; works with dialect IR entities such as ops, types, or attributes; uses declarative TableGen records to describe reusable IR contracts.
- **CN**: 围绕 `information` 实现具体逻辑；该代码块表达基于接口的可复用行为，并处理方言 IR 实体，如操作、类型或属性，并使用声明式 TableGen 记录描述可复用 IR 契约。

### Lines 130-136
```tablegen
    ```
    #dlti.target_system_spec<"CPU" =
      #dlti.target_device_spec<"cache" =
        #dlti.map<"L1" = #dlti.map<"size_in_bytes" = 65536 : i32>,
                  "L1d" = #dlti.map<"size_in_bytes" = 32768 : i32> >>>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 137-142
```tablegen
    With the flat encoding, the implied structure of the key is ignored, that is
    the only successful query (as expressed in the Transform Dialect) is:
    `transform.dlti.query ["CPU::cache::L1::size_in_bytes"] at %op`,
    where `%op` is a handle to an operation which associates the flat-encoding
    `#dlti.map` attribute.

```
- **EN**: Implements logic around `query`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `query` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 143-154
```tablegen
    For querying nested dictionaries, the relevant keys need to be separately
    provided. That is, if `%op` is an handle to an op which has the nesting
    `#dlti.target_system_spec`-attribute from above attached, then
    `transform.dlti.query ["CPU","cache","L1","size_in_bytes"] at %op` gives
    back the first leaf value contained. To access the other leaf, we need to do
    `transform.dlti.query ["CPU","cache","L1d","size_in_bytes"] at %op`.
   }];
  let parameters = (ins
    ArrayRefParameter<"DataLayoutEntryInterface", "">:$entries
  );
  let mnemonic = "map";
  let genVerifyDecl = 1;
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 155-166
```tablegen
  let hasCustomAssemblyFormat = 1;
  let extraClassDeclaration = [{
    /// Returns the attribute associated with the key.
    FailureOr<Attribute> query(DataLayoutEntryKey key) {
      for (DataLayoutEntryInterface entry : getEntries())
        if (entry.getKey() == key)
            return entry.getValue();
      return ::mlir::failure();
    }
  }];
}

```
- **EN**: Implements logic around `query`, `getEntries`, `getKey`, `getValue`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `query`, `getEntries`, `getKey`, `getValue`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 167-178
```tablegen
//===----------------------------------------------------------------------===//
// TargetSystemSpecAttr
//===----------------------------------------------------------------------===//

def DLTI_TargetSystemSpecAttr :
    DLTIAttr<"TargetSystemSpec", [TargetSystemSpecInterface]> {
  let summary = "An attribute to represent target system specification.";
  let description = [{
    A system specification describes the overall system containing
    multiple devices, with each device having a unique ID (string)
    and its corresponding TargetDeviceSpec object.

```
- **EN**: Introduces declarations for `DLTI_TargetSystemSpecAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DLTI_TargetSystemSpecAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 179-190
```tablegen
    Example:
    ```
    dlti.target_system_spec =
     #dlti.target_system_spec<
      "CPU" = #dlti.target_device_spec<
        "L1_cache_size_in_bytes" = 4096: ui32>,
      "GPU" = #dlti.target_device_spec<
        "max_vector_op_width" = 64 : ui32>,
      "XPU" = #dlti.target_device_spec<
        "max_vector_op_width" = 4096 : ui32>>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 191-201
```tablegen
    The verifier checks that keys are strings and pointed to values implement
    DLTI's TargetDeviceSpecInterface.
  }];
  let parameters = (ins
    ArrayRefParameter<"DataLayoutEntryInterface">:$entries
  );
  let mnemonic = "target_system_spec";
  let genVerifyDecl = 1;
  let hasCustomAssemblyFormat = 1;
  let extraClassDeclaration = [{
    /// Return the device specification that matches the given device ID
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 202-213
```tablegen
    std::optional<TargetDeviceSpecInterface>
    getDeviceSpecForDeviceID(
      TargetSystemSpecInterface::DeviceID deviceID);

    /// Returns the attribute associated with the key.
    FailureOr<Attribute> query(DataLayoutEntryKey key) const {
      return ::llvm::cast<mlir::TargetSystemSpecInterface>(*this).queryHelper(key);
    }
  }];
  let extraClassDefinition = [{
    std::optional<TargetDeviceSpecInterface>
    $cppClass::getDeviceSpecForDeviceID(
```
- **EN**: Implements logic around `getDeviceSpecForDeviceID`, `query`, `TargetSystemSpecInterface>`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getDeviceSpecForDeviceID`, `query`, `TargetSystemSpecInterface>` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 214-225
```tablegen
        TargetSystemSpecInterface::DeviceID deviceID) {
      for (const auto& entry : getEntries()) {
        if (entry.getKey() == DataLayoutEntryKey(deviceID))
          if (auto deviceSpec =
              ::llvm::dyn_cast<TargetDeviceSpecInterface>(entry.getValue()))
            return deviceSpec;
      }
      return std::nullopt;
    }
  }];
}

```
- **EN**: Implements logic around `getEntries`, `getKey`, `getValue`.
- **CN**: 围绕 `getEntries`, `getKey`, `getValue` 实现具体逻辑。

### Lines 226-237
```tablegen
//===----------------------------------------------------------------------===//
// TargetDeviceSpecAttr
//===----------------------------------------------------------------------===//

def DLTI_TargetDeviceSpecAttr :
    DLTIAttr<"TargetDeviceSpec", [TargetDeviceSpecInterface]> {
  let summary = "An attribute to represent target device specification.";
  let description = [{
    Each device specification describes a single device and its
    hardware properties. Each device specification can contain any number
    of optional hardware properties (e.g., max_vector_op_width below).

```
- **EN**: Introduces declarations for `DLTI_TargetDeviceSpecAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DLTI_TargetDeviceSpecAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 238-249
```tablegen
    Example:
    ```
    #dlti.target_device_spec<"max_vector_op_width" = 64 : ui32>
    ```
  }];
  let parameters = (ins
    ArrayRefParameter<"DataLayoutEntryInterface">:$entries
  );
  let mnemonic = "target_device_spec";
  let genVerifyDecl = 1;
  let hasCustomAssemblyFormat = 1;

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 250-257
```tablegen
  let extraClassDeclaration = [{
    /// Returns the attribute associated with the key.
    FailureOr<Attribute> query(DataLayoutEntryKey key) const {
      return ::llvm::cast<mlir::TargetDeviceSpecInterface>(*this).queryHelper(key);
    }
  }];
}

```
- **EN**: Implements logic around `query`, `TargetDeviceSpecInterface>`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `query`, `TargetDeviceSpecInterface>` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 258-269
```tablegen
//===----------------------------------------------------------------------===//
// FunctionPointerAlignmentAttr
//===----------------------------------------------------------------------===//

def DLTI_FunctionPointerAlignmentAttr :
    DLTIAttr<"FunctionPointerAlignment", []> {
  let summary = "An attribute to represent function pointer alignment.";
  let description = [{
    Function pointer alignment specifies the minimum alignment of function
    pointers, it's a multiple of `alignment`. This alignment can also depend
    on the target function, indicated by `function_dependent`.
    Example:
```
- **EN**: Introduces declarations for `DLTI_FunctionPointerAlignmentAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DLTI_FunctionPointerAlignmentAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 270-281
```tablegen
    ```
    #dlti.dl_entry<"dlti.function_pointer_alignment",
               #dlti.function_pointer_alignment<64, function_dependent = false>>
    ```
  }];
  let parameters = (ins
    "uint64_t":$alignment,
    "bool":$function_dependent
  );
  let assemblyFormat = [{
    `<`
    $alignment `,`
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 282-287
```tablegen
    `function_dependent` `=` $function_dependent
    `>`
  }];
  let mnemonic = "function_pointer_alignment";
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 288-289
```tablegen

#endif  // MLIR_DIALECT_DLTI_DLTIATTRS_TD
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
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

- **TableGen includes / TableGen 包含**: `mlir/Dialect/DLTI/DLTI.td`, `mlir/Interfaces/DataLayoutInterfaces.td`, `mlir/IR/AttrTypeBase.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (1), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
