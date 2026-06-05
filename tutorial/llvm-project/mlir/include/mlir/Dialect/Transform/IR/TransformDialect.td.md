# TransformDialect.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/IR/TransformDialect.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR TransformDialect component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: For description, see docs/Dialects/Transform.md.
- **用途（CN）**: 为 MLIR 的 TransformDialect 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
````tablegen
//===- TransformDialect.td - Transform dialect definition --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORM_IR_TRANSFORMDIALECT
#define MLIR_DIALECT_TRANSFORM_IR_TRANSFORMDIALECT

include "mlir/IR/OpBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 14-86
````tablegen
def Transform_Dialect : Dialect {
  let summary = "Fine-grain transformation control dialect";
  // For description, see docs/Dialects/Transform.md.

  let name = "transform";
  let cppNamespace = "::mlir::transform";

  let hasOperationAttrVerify = 1;
  let extraClassDeclaration = [{
    /// Symbol name for the default entry point "named sequence".
    constexpr const static ::llvm::StringLiteral
        kTransformEntryPointSymbolName = "__transform_main";

    /// Name of the attribute attachable to the symbol table operation
    /// containing named sequences. This is used to trigger verification.
    constexpr const static ::llvm::StringLiteral
        kWithNamedSequenceAttrName = "transform.with_named_sequence";

    /// Name of the attribute attachable to an operation so it can be
    /// identified as root by the default interpreter pass.
    constexpr const static ::llvm::StringLiteral kTargetTagAttrName =
        "transform.target_tag";

    /// Names of the attributes indicating whether an argument of an external
    /// transform dialect symbol is consumed or only read.
    constexpr const static ::llvm::StringLiteral kArgConsumedAttrName =
        "transform.consumed";
    constexpr const static ::llvm::StringLiteral kArgReadOnlyAttrName =
        "transform.readonly";

    /// Names of the attributes indicating whether an argument of an external
    /// transform dialect symbol is consumed or only read.
    StringAttr getConsumedAttrName() const {
      return StringAttr::get(getContext(), kArgConsumedAttrName);
    }
    StringAttr getReadOnlyAttrName() const {
      return StringAttr::get(getContext(), kArgReadOnlyAttrName);
    }

    template <typename DataTy>
    const DataTy &getExtraData() const {
      return *static_cast<const DataTy *>(
          extraData.at(::mlir::TypeID::get<DataTy>()).get());
    }

    /// Parses an attribute registered by this dialect or one of its extensions.
    ::mlir::Attribute parseAttribute(::mlir::DialectAsmParser &parser,
        ::mlir::Type type) const override;

    /// Prints an attribute registered by this dialect or one of its extensions.
    void printAttribute(::mlir::Attribute attribute,
        ::mlir::DialectAsmPrinter &printer) const override;

    /// Parser callback for an individual attribute registered by this dialect or
    /// its extensions.
    using ExtensionAttributeParsingHook =
        ::mlir::Attribute (*)(::mlir::AsmParser &, ::mlir::Type);

    /// Printer callback for an individual attribute registered by this dialect or
    /// its extensions.
    using ExtensionAttributePrintingHook =
        std::function<void(::mlir::Attribute, ::mlir::AsmPrinter &)>;

    /// Parses a type registered by this dialect or one of its extensions.
    ::mlir::Type parseType(::mlir::DialectAsmParser &parser) const override;

    /// Prints a type registered by this dialect or one of its extensions.
    void printType(::mlir::Type type, ::mlir::DialectAsmPrinter &printer)
        const override;

    /// Parser callback for an individual type registered by this dialect or
    /// its extensions.
    using ExtensionTypeParsingHook = ::mlir::Type (*)(::mlir::AsmParser &);
````
- **EN**: This TableGen block defines `Transform_Dialect` as a `def` record for `TransformDialect`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `Transform_Dialect` 定义为 `def` 记录，用于描述 `TransformDialect` 相关的声明式信息。 其中涉及 语义文档。

### Lines 87-159
````tablegen
    /// Printer callback for an individual type registered by this dialect or
    /// its extensions.
    using ExtensionTypePrintingHook =
        std::function<void(::mlir::Type, ::mlir::AsmPrinter &)>;

    /// Loads the given module into the transform symbol library module.
    LogicalResult loadIntoLibraryModule(::mlir::OwningOpRef<::mlir::ModuleOp> &&
                                        library);

    /// Returns the transform symbol library module available to all dialect
    /// users.
    ModuleOp getLibraryModule() const {
      if (libraryModule)
        return libraryModule.get();
      return ModuleOp();
    }

  private:
    /// Initializes the transform symbol library module. Must be called from
    /// `TransformDialect::initialize` for the library module to work.
    void initializeLibraryModule();

    /// Registers operations specified as template parameters with this
    /// dialect. Checks that they implement the required interfaces.
    template <typename... OpTys>
    void addOperationsChecked() {
      (addOperationIfNotRegistered<OpTys>(), ...);
    }
    template <typename OpTy>
    void addOperationIfNotRegistered();

    /// Reports a repeated registration error of an op with the given name.
    [[noreturn]] void reportDuplicateOpRegistration(StringRef opName);

    /// Registers attributes specified as template parameters with the Transform
    /// dialect. Checks that they meet the requirements for Transform IR attributes.
    template <typename... AttrTys>
    void addAttributesChecked() {
      (addAttributeIfNotRegistered<AttrTys>(), ...);
    }
    template <typename AttrTy>
    void addAttributeIfNotRegistered();

    /// Reports a repeated registration error of an attribute with the given name.
    [[noreturn]] void reportDuplicateAttributeRegistration(StringRef attrName);

    /// Registers dialect attributes with the context.
    void initializeAttributes();

    /// Registers types specified as template parameters with the Transform
    /// dialect. Checks that they meet the requirements for Transform IR types.
    template <typename... TypeTys>
    void addTypesChecked() {
      (addTypeIfNotRegistered<TypeTys>(), ...);
    }
    template <typename Type>
    void addTypeIfNotRegistered();

    /// Reports a repeated registration error of a type with the given
    /// mnemonic.
    [[noreturn]] void reportDuplicateTypeRegistration(StringRef mnemonic);

    /// Registers dialect types with the context.
    void initializeTypes();

    // Give extensions access to injection functions.
    template <typename, typename...>
    friend class TransformDialectExtension;

    /// Gets a mutable reference to extra data of the kind specified as
    /// template argument. Allocates the data on the first call.
    template <typename DataTy>
    DataTy &getOrCreateExtraData();
````
- **EN**: This block groups callable interfaces such as `void`, `loadIntoLibraryModule`, `getLibraryModule`, `get`, indicating how `TransformDialect` is queried or updated.
- **CN**: 该代码块聚合了 `void`, `loadIntoLibraryModule`, `getLibraryModule`, `get` 等可调用接口，展示了如何查询或更新 `TransformDialect`。

### Lines 161-197
````tablegen
    //===----------------------------------------------------------------===//
    // Data fields
    //===----------------------------------------------------------------===//

    /// Additional data associated with and owned by the dialect. Accessible
    /// to extensions.
    ::llvm::DenseMap<
        ::mlir::TypeID,
        std::unique_ptr<::mlir::transform::detail::TransformDialectDataBase>>
        extraData;

    /// A map from type mnemonic to its parsing function for the remainder of
    /// the syntax. The parser has access to the mnemonic, so it is used for
    /// further dispatch.
    ::llvm::StringMap<ExtensionTypeParsingHook> typeParsingHooks;

    /// A map from type TypeID to its printing function. No need to do string
    /// lookups when the type is fully constructed.
    ::llvm::DenseMap<::mlir::TypeID, ExtensionTypePrintingHook>
        typePrintingHooks;

    /// A map from attribute TypeID to its parsing function for the remainder of
    /// the syntax. The parser has access to the TypeID, so it is used for
    /// further dispatch.
    ::llvm::StringMap<ExtensionAttributeParsingHook>
        attributeParsingHooks;

    /// A map from attribute TypeID to its printing function. No need to do string
    /// lookups when the attribute is fully constructed.
    ::llvm::DenseMap<::mlir::TypeID, ExtensionAttributePrintingHook>
        attributePrintingHooks;

    /// Module containing symbols, e.g. named sequences, that will be resolved
    /// by the interpreter when used.
    ::mlir::OwningOpRef<::mlir::ModuleOp> libraryModule;
  }];
}
````
- **EN**: This section focuses on data fields, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“Data fields”这一主题，把相关声明与辅助接口组织在一起。

### Lines 199-200
````tablegen
// Base class for ops that belong to the transform dialect. Ops defined in
// extensions of this dialect may also use this.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 202-207
````tablegen
class TransformDialectOp<string mnemonic, list<Trait> traits = []>
    : Op<Transform_Dialect, mnemonic, traits>;

// Trait for operations that may be top-level operations in Transform IR.
// Operations must have one single-block region and must be usable without
// operands. See the C++ definition of the trait for more information.
````
- **EN**: This TableGen block defines `TransformDialectOp` as a `class` record for `TransformDialect`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TransformDialectOp` 定义为 `class` 记录，用于描述 `TransformDialect` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 208-211
````tablegen
def PossibleTopLevelTransformOpTrait
    : NativeOpTrait<"PossibleTopLevelTransformOpTrait"> {
  let cppNamespace = "::mlir::transform";
}
````
- **EN**: This TableGen block defines `PossibleTopLevelTransformOpTrait` as a `def` record for `TransformDialect`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `PossibleTopLevelTransformOpTrait` 定义为 `def` 记录，用于描述 `TransformDialect` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 212-212
````tablegen
#endif // MLIR_DIALECT_TRANSFORM_IR_TRANSFORMDIALECT
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/IR/OpBase.td
- Transform_Dialect builds on Dialect
- PossibleTopLevelTransformOpTrait builds on NativeOpTrait<"PossibleTopLevelTransformOpTrait">
